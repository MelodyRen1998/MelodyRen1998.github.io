---
title: 简单数据爬取（以B站弹幕为例）
author: Yimeng
date: '2021-04-27'
permalink: /posts/2021/04/data-crawler-python/
categories:
  - Tools
tags:
  - data-collection
  - python
  - data-crawler
---

即不需要额外抓取ajax数据包，都在网页检查栏中能找到。

## 分析网页

点击弹幕列表，查看历史弹幕，并选择任意一天的历史弹幕，此时就能找到存储该日期弹幕的ajax数据包，所有弹幕数据放在一个i标签里。有的网上博客是说能找到弹幕数据，但我2021年初重新试了一下，弹幕内容已经加密了，不过按照**下面我用的请求地址**，还是能抓取到弹幕的内容，只不过在网页检查中找不到这个请求地址。

所以暂时用这个请求地址，后续如果发现如何解密再来更新。

![弹幕内容加密后的请求信息](https://tva1.sinaimg.cn/large/008eGmZEly1gord65csw8j31gy0ppai6.jpg)

![加密之后的弹幕列表](https://tva1.sinaimg.cn/large/008eGmZEly1gord5p4t7lj30uf06k3z6.jpg)

参考[这篇博客](https://blog.csdn.net/weixin_38753213/article/details/109554851)给出的请求地址，使用`https://api.bilibili.com/x/v1/dm/list.so?oid=%d`请求网址，可以得到不加密的弹幕内容：

![未加密弹幕内容](https://tva1.sinaimg.cn/large/008i3skNly1gpy263zpvkj31b70u0nlc.jpg)

可以发现Request URL关键就是 **oid** 参数，是一个类似于视频标识ID之类的东西，换个oid可以访问其他视频弹幕页面。B站的视频可以看到的ID是用**bvid**作为标识的，因此需要先对这个进行转换：

```python
def get_cid(bvid):
    '''
    通过视频的bvid获得视频的cid/oid
    输入：视频的bvid
    输出：视频的cid/oid
    '''
    url = 'https://api.bilibili.com/x/player/pagelist?bvid=%s&jsonp=jsonp'%bvid
    res = requests.get(url)
    data = res.json()
    return data['data'][0]['cid']
```

## 实际操作

以下分别给出需要爬取内容、解析内容使用的一些函数，使用的包有`requests`,`re`等。

```python
import re
import time
import requests
import datetime
import pandas as pd
```

```python
def del_repeat(data,key):
    '''
    字典列表去重
    输入：字典列表，去重关键字
    输出：去重后的列表
    '''
    new_data = [] # 用于存储去重后的list
    values = []   # 用于存储当前已有的值
    for d in data:
        if d[key] not in values:
            new_data.append(d)
            values.append(d[key])
    return new_data
```

```python
def get_info(bvid):
    '''
    获取视频信息
    输入：视频的bvid
    输出：视频的相关信息（视频标题、播放量、弹幕数、上传时间）
    '''
    url = 'https://api.bilibili.com/x/web-interface/view/detail?bvid=%s'%bvid
    res = requests.get(url)
    data = res.json()
    title = data['data']['View']['title'] # 视频标题
    view = data['data']['View']['stat']['view'] # 播放量
    dm = data['data']['View']['stat']['danmaku'] # 弹幕数
    upload = time.strftime('%Y-%m-%d',time.localtime(data['data']['View']['pubdate'])) # 上传日期
    info = (title,view,dm,upload)
    return info
```

```python
test_info = get_info('BV1wy4y1p7Ms')
test_info
# ('奔驰和宝马的质量到底怎么样？真实买车拆解告诉你真相！', 394097, 6367, '2021-01-21')
```

```python
def get_cid(bvid):
    '''
    通过视频的bvid获得视频的cid/oid
    输入：视频的bvid
    输出：视频的cid/oid
    '''
    url = 'https://api.bilibili.com/x/player/pagelist?bvid=%s&jsonp=jsonp'%bvid
    res = requests.get(url)
    data = res.json()
    return data['data'][0]['cid']
```

通过正则表达式处理，解析HTML网页的内容：

```python
def parse_dm(text):
    '''
    解析视频弹幕
    输入：视频弹幕的原数据
    输出：弹幕的解析结果
    '''
    result = [] # 用于存储解析结果
    data = re.findall('<d p="(.*?)">(.*?)</d>',text)
    for d in data:
        item = {} # 每条弹幕数据
        dm = d[0].split(',') # 弹幕的相关详细，如出现时间，用户等
        item['出现时间'] = float(dm[0])
        item['模式'] = int(dm[1])
        item['字号'] = int(dm[2])
        item['颜色'] = int(dm[3])
        item['评论时间'] = time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(int(dm[4])))
        item['弹幕池'] = int(dm[5])
        item['用户ID'] = dm[6] # 并非真实用户ID，而是CRC32检验后的十六进制结果
        item['rowID'] = dm[7] # 弹幕在数据库中的ID，用于“历史弹幕”功能
        item['弹幕内容'] = d[1]
        result.append(item)    
    return result
```

下面分两种爬取的情境。第一种，只爬取当前显示的弹幕内容，不指定全部日期。此方法获取的弹幕数量有限制，限制1000条。

```python
def get_dm(bvid):
    '''
    获取视频弹幕（此方法获取的弹幕数量有限制，限制1000条）
    输入：视频的bvid
    输出：视频的弹幕
    '''
    oid = get_cid(bvid) # 这里的cid和oid是一样的
    url = 'https://api.bilibili.com/x/v1/dm/list.so?oid=%d'%oid
    res = requests.get(url)
    res.encoding = 'utf-8'
    text = res.text
    dms = parse_dm(text) # 解析弹幕
    return dms
```

第二种，爬取当前日期开始计算，所有的历史弹幕内容。

```python
# 用上面那个函数，因为git_history暂时没有用
def get_dms(bvid):
    '''
    获取视频弹幕（此方法获取的弹幕数量更多）
    输入：视频的bvid
    输出：视频的弹幕
    '''
    print('视频解析中...')
    info = get_info(bvid)
    print('视频解析完成!')
    print('【视频标题】： %s\n【视频播放量】：%d\n【弹幕数量】：  %d\n【上传日期】：  %s'%(info[0],info[1],info[2],info[3]))
    dms = get_dm(bvid) # 存储弹幕
    if len(dms) >= info[2]: # 如果弹幕数量已抓满
        return dms
    else:
        dms = []
        date = time.strftime('%Y-%m-%d',time.localtime(time.time())) # 从今天开始
        while True:
            dm = get_history(bvid,date)
            dms.extend(dm)
            print('"%s"弹幕爬取完成!(%d条)'%(date,len(dm)))
            if len(dm) == 0: # 如果为空
                break
            end = dm[-1]['评论时间'].split()[0] # 取最后一条弹幕的日期
            if end == date: # 如果最后一条仍为当天，则往下推一天
                end = (datetime.datetime.strptime(end,'%Y-%m-%d')-datetime.timedelta(days=1)).strftime('%Y-%m-%d')
            if end == info[3]: # 如果已经到达上传那天
                break
            else:
                date = end
        dm = get_history(bvid,info[3]) # 避免忽略上传那天的部分数据
        dms.extend(dm)
        print('弹幕爬取完成!(共%d条)'%len(dms))
        print('数据去重中...')
        dms = del_repeat(dms,'rowID') # 按rowID给弹幕去重
        print('数据去重完成!(共%d条)'%len(dms))
        return dms
```

## 一些说明

弹幕数据中的用户ID经过加密，使用[这篇博客](https://www.52pojie.cn/thread-1235704-1-1.html)中给出的解码方法，对爬取的弹幕发送用户ID进行解码。p.s. 解码需要时间较长（大约2000个ID需要20-30分钟）

```python
# run.py
# utf-8
import sys
import time
import pandas as pd

CRCPOLYNOMIAL = 0xEDB88320
crctable = [0 for x in range(256)]

def create_table():
    for i in range(256):
        crcreg = i
        for _ in range(8):
            if (crcreg & 1) != 0:
                crcreg = CRCPOLYNOMIAL ^ (crcreg >> 1)
            else:
                crcreg = crcreg >> 1
        crctable[i] = crcreg

def crc32(string):
    crcstart = 0xFFFFFFFF
    for i in range(len(str(string))):
        index = (crcstart ^ ord(str(string)[i])) & 255
        crcstart = (crcstart >> 8) ^ crctable[index]
    return crcstart

def crc32_last_index(string):
    crcstart = 0xFFFFFFFF
    for i in range(len(str(string))):
        index = (crcstart ^ ord(str(string)[i])) & 255
        crcstart = (crcstart >> 8) ^ crctable[index]
    return index

def get_crc_index(t):
    for i in range(256):
        if crctable[i] >> 24 == t:
            return i
    return -1

def deep_check(i, index):
    string = ""
    tc=0x00
    hashcode = crc32(i)
    tc = hashcode & 0xff ^ index[2]
    if not (tc <= 57 and tc >= 48):
        return [0]
    string += str(tc - 48)
    hashcode = crctable[index[2]] ^ (hashcode >>8)
    tc = hashcode & 0xff ^ index[1]
    if not (tc <= 57 and tc >= 48):
        return [0]
    string += str(tc - 48)
    hashcode = crctable[index[1]] ^ (hashcode >> 8)
    tc = hashcode & 0xff ^ index[0]
    if not (tc <= 57 and tc >= 48):
        return [0]
    string += str(tc - 48)
    hashcode = crctable[index[0]] ^ (hashcode >> 8)
    return [1, string]

def main(string):
    index = [0 for x in range(4)]
    i = 0
    ht = int(f'0x{string}', 16) ^ 0xffffffff
    for i in range(3,-1,-1):
        index[3-i] = get_crc_index(ht >> (i*8))
        snum = crctable[index[3-i]]
        ht ^= snum >> ((3-i)*8)
    for i in range(100000000):
        lastindex = crc32_last_index(i)
        if lastindex == index[3]:
            deepCheckData = deep_check(i, index)
            if deepCheckData[0]:
                break
    if i == 100000000:
        return -1
    return f"{i}{deepCheckData[1]}"

if __name__ == "__main__":
    create_table()
    # start_time = time.time()
    df = pd.read_csv(sys.argv[1])
    user_hash_ID = list(df.用户ID)
    user_id = []
    for hash_id in user_hash_ID:
        # print(main(hash_id))
        user_id.append(main(hash_id))
    unique_user_id = list(set(user_id))
    print("完成{}个用户ID的转换，不重复的用户ID共{}个".format(len(user_id), len(unique_user_id)))
    # 包含重复的User ID (rowID unique)
    user_id_df_dup = pd.DataFrame(user_id)
    user_id_df_dup.to_csv(sys.argv[2], header = False, index = False)
    # 不重复的User ID
    user_id_df = pd.DataFrame(unique_user_id)
    user_id_df.to_csv(sys.argv[3], header = False, index = False)
    # print(main(sys.argv[1]))
    # end_time = time.time()
    # print(f"耗时: {round(end_time - start_time, 2)}秒")

```
