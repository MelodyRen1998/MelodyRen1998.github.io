---
title: Mac OS Ubuntu虚拟机安装SQL Server 2019
author: Yimeng
date: '2020-10-11'
permalink: /posts/2020/10/mac-os-ubuntu-sql-server-2019/
categories:
  - Tools
tags:
  - Ubuntu
  - vertue-machine
  - sql-server
---

**参考链接**

  - [Quickstart: Install SQL Server and create a database on Ubuntu](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-ubuntu?view=sql-server-ver15)
  - [Install tools on Ubuntu 16.04](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver15#ubuntu)
  - [Install Microsoft SQL Server 2019 on Ubuntu 20.04/18.04/16.04 LTS](https://computingforgeeks.com/how-to-install-ms-sql-on-ubuntu/)

- 版本说明
  - Ubuntu: 20.04
 
 
**TODO LIST**

- [ ] SQL Server 2019 on docker: 如何导入外部数据或通过外部连接这个服务器
- [ ] SQL Server 2019 on ubuntu via MAC terminal: 在配置SQL时遇到`system has notbeen booted with systemd as init system (PID 1)`的问题，暂时没有解决，导致无法继续连接数据库

![Screen Shot 2020-10-10 at 7.11.29 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlro6uw3aj30x2076ae8.jpg)


## 更换安装镜像源

在安装之前，首先确保Ubuntu的安装镜像源已经更换至任意一个国内镜像源，这里我使用清华的镜像。

#### 寻找更新镜像源

[清华大学开源软件镜像站：Ubuntu镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

选择对应的Ubuntu版本，复制内容。

![Screen Shot 2020-10-11 at 10.38.27 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlru7p7hbj31ju0r2wo2.jpg)

#### 备份 Ubuntu 默认的源地址

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

#### 更新源服务器列表

```bash
sudo vim /etc/apt/sources.list
```

![Screen Shot 2020-10-11 at 5.38.21 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlroeua9ij31a10u0nhd.jpg)

- 删除默认的安装镜像内容
- 把复制的更新源信息粘贴过来
- 键入`esc`，再键入`:wq`后保存且退出。



## 安装 SQL Server

在Ubuntu Terminal运行一下命令安装 mssql-server 包：

- 导入公共存储库 GPG 密钥：

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```

- 为 SQL Server 2019 注册 Microsoft SQL Server Ubuntu 存储库：

```bash
sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/18.04/mssql-server-2019.list)"
```

**注意**：这里不需要更换 `ubuntu/20.04/` ，仍然使用 `ubuntu/18.04/`即可。

- 运行以下命令以安装 SQL Server：

```bash
sudo apt-get update
sudo apt-get install -y mssql-server
```

- 包安装完成后，运行 `mssql-conf setup`，按照提示设置 SA 密码并选择版本：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo /opt/mssql/bin/mssql-conf setup
Choose an edition of SQL Server:
  1) Evaluation (free, no production use rights, 180-day limit)
  2) Developer (free, no production use rights)
  3) Express (free)
  4) Web (PAID)
  5) Standard (PAID)
  6) Enterprise (PAID)
  7) Enterprise Core (PAID)
  8) I bought a license through a retail sales channel and have a product key to enter.

Details about editions can be found at
https://go.microsoft.com/fwlink/?LinkId=852748&clcid=0x409

Use of PAID editions of this software requires separate licensing through a
Microsoft Volume Licensing program.
By choosing a PAID edition, you are verifying that you have the appropriate
number of licenses in place to install and run this software.

Enter your edition(1-8): 2
The license terms for this product can be found in
/usr/share/doc/mssql-server or downloaded from:
https://go.microsoft.com/fwlink/?LinkId=855862&clcid=0x409

The privacy statement can be viewed at:
https://go.microsoft.com/fwlink/?LinkId=853010&clcid=0x409

Do you accept the license terms? [Yes/No]:Yes

Enter the SQL Server system administrator password:  <SetStrongNewPassword>
Confirm the SQL Server system administrator password: <ConfirmStrongPassword>
Configuring SQL Server...

ForceFlush is enabled for this instance. 
ForceFlush feature is enabled for log durability.
Created symlink /etc/systemd/system/multi-user.target.wants/mssql-server.service → /lib/systemd/system/mssql-server.service.
Setup has completed successfully. SQL Server is now starting.
```

**注意**：SA密码设置规则最少 8 个字符，包括大写和小写字母、十进制数字和/或非字母数字符号。

- 完成配置后，验证服务是否正在运行：

```bash
systemctl status mssql-server --no-pager
```

如下图所示，则为正常运行。

![Screen Shot 2020-10-11 at 5.55.12 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrojnbcjj31a10u0qno.jpg)

此时，SQL Server 2019 已在 Ubuntu 虚拟机上运行。

## 安装 SQL Server 命令行工具

若要创建数据库，则需要使用可在 SQL Server 上运行 Transact-SQL 语句的工具进行连接。 以下步骤将安装 SQL Server 命令行工具：[sqlcmd](https://docs.microsoft.com/zh-cn/sql/tools/sqlcmd-utility?view=sql-server-ver15) 和 [bcp](https://docs.microsoft.com/zh-cn/sql/tools/bcp-utility?view=sql-server-ver15)。

通过以下步骤在 Ubuntu 上安装 **mssql-tools** 和 **unixODBC **插件：

**Ubuntu 20.04**

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
sudo apt update 
sudo ACCEPT_EULA=Y apt install mssql-tools unixodbc-dev
```

**注意**：请将`/ubuntu/20.04`调整为对应 Ubuntu 版本的序号。



## 配置 SQL 环境 (optional)

向 bash shell 中的 PATH 环境变量添加 `/opt/mssql-tools/bin/`

要使 sqlcmd/bcp 能从登陆会话的 bash shell 进行访问，请使用下列命令修改 `~/.bash_profile` 文件中的 PATH ：

```bash
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
```

要使 sqlcmd/bcp 能从交互式/非登录会话的 bash shell 进行访问，请使用下列命令修改 `~/.bashrc` 文件中的 PATH ：

```bash
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
```

## 连接 MS SQL 控制台

```bash
sqlcmd -S 127.0.0.1 -U SA 
```

根据提示输入密码后，出现`1>`则表示连接成功。

## 新建数据库

以下步骤创建一个名为 `TestDB` 的新数据库。

1. 在 sqlcmd 命令提示符中，使用以下命令以创建测试数据库：

```sql
1> create database testDB;
```

2. 在下一行中，编写一个查询以返回服务器上所有数据库的名称：

```sql
2> select name from sys.databases;
```

3. 前两个命令没有立即执行。 必须在新行中键入 `GO` 才能执行以前的命令：

```sql
3> GO
```

4. 使用命令`QUIT`退出SQL命令行：

```sql
1> QUIT
```

![Screen Shot 2020-10-11 at 6.16.02 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlronrr1hj31a10u0dxh.jpg)

## TPCH 数据库创建

### 创建 TPCH 数据库

![Screen Shot 2020-10-11 at 6.26.32 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrotu9r1j30j8088gm5.jpg)



### 在 tpch 数据库中创建相应的表

![Screen Shot 2020-10-11 at 6.29.58 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrox0h7oj30ie0g8tah.jpg)

创建好8张表后，使用命令：

```sql
SELECT
  *
FROM
  INFORMATION_SCHEMA.TABLES;
GO
```

查看当前所有表的名称：

![Screen Shot 2020-10-11 at 6.57.55 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrp0wbnnj31gh0u0neb.jpg)

### 加载服务器的数据到数据库中

首先，确保数据已经去除竖线：

```bash
head -10 new_region.tbl  # 查看文件前2行
```

![Screen Shot 2020-10-11 at 8.41.22 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrp6dmo6j31qy0b8adv.jpg)

使用命令将已经去除行末竖线的数据加载到数据库`TPCH`中，并创建相应的表：

```SQL
BULK INSERT region FROM '/home/renyimeng/renyimeng/tpch-dbgen/TPCH_data/new_region.tbl' WITH (FIELDTERMINATOR = '|',ROWTERMINATOR = '\n');
```



### 验证表中记录行数

使用如下命令查看`region`表的行数：

```sql
select count(*) as region_rows from region;
```

输出结果应当如下所示：

![Screen Shot 2020-10-11 at 9.03.41 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrpamci6j30m40sqjsv.jpg)

![Screen Shot 2020-10-11 at 9.03.52 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjlrpdft83j30kq0fg752.jpg)

