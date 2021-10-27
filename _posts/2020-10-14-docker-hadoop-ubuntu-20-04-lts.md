---
title: docker 环境下搭建 hadoop 集群（ubuntu 20.04 LTS系统）
author: Yimeng
date: '2020-10-14'
permalink: /posts/2020/10/docker-hadoop-ubuntu-20-04-lts/
categories:
  - Tools
tags:
  - Ubuntu
  - hadoop
---

主要参考资料：
- [docker环境下搭建hadoop集群(ubuntu16.04 LTS系统)](https://blog.csdn.net/weixin_42051109/article/details/82744993)
- [Spark分布式搭建（2）——ubuntu14.04下修改hostname和hosts](https://blog.csdn.net/xummgg/article/details/50634327)
- [hadoop 集群子节点不启动 spark-slave1: ssh: Could not resolve hostname spark-slave1: Name or service not known](https://www.cnblogs.com/yangchas/p/10469393.html)

**主要步骤**：

安装ubuntu系统 -> 下载docker -> 在docker里拉取hadoop镜像 -> 在此镜像里创建三个容器 (Master、Slave1、Slave2) -> 完成分布式部署

**TODO**

- [ ] [docker命令不需要敲sudo的方法](https://www.cnblogs.com/ksir16/p/6530587.html)



## 安装 ubuntu 系统

我使用 VMware虚拟机 安装 ubuntu 系统，具体原因与[安装 SQL Server](https://melodyren1998.github.io/2020/10/11/MacOS-Ubuntu%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AE%89%E8%A3%85SQL-Server-2019/)有关。

- 版本：ubuntu 20.04

## 下载 docker

参照链接：https://docs.docker.com/engine/install/ubuntu/

根据官方安装步骤来就可以。

- 版本：docker 19.03.13

![Screen Shot 2020-10-13 at 8.39.55 AM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjoxk5blmhj31d00d8q5b.jpg)

## 拉取 hadoop 镜像

这里我选择使用阿里云库拉取 hadoop 镜像 `registry.cn-beijing.aliyuncs.com/bitnp/docker-spark-hadoop`，这个镜像已经提前下载好了我们需要的工具：jdk, hadoop, spark等。

在虚拟机的终端（以下简称终端）里配置加速器：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://lqbkkmob.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 创建容器并配置

接下来创建容器 Master、Slave1、Slave2，并对容器进行配置及ssh的互联。

### step 1

在终端执行命令：

```bash
docker pull registry.cn-beijing.aliyuncs.com/bitnp/docker-spark-hadoop
```

![Screen Shot 2020-10-13 at 9.15.50 AM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjoxknsemvj319o0s843n.jpg)

### step 2

经过一段时间后，镜像已经下载到本地计算机，可使用指令`sudo docker images`查看是否下载成功：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker images
[sudo] password for renyimeng: 
REPOSITORY                                                   TAG                 IMAGE ID            CREATED             SIZE
hello-world                                                  latest              bf756fb1ae65        9 months ago        13.3kB
registry.cn-beijing.aliyuncs.com/bitnp/docker-spark-hadoop   latest              8b768e1604ad        2 years ago         2.11GB
```

### step 3

这时，我们要在这个hadoop镜像里创建三个容器 (Master、Slave1、Slave2)，敲上如下指令：(我们先把三个容器创建出来，再慢慢里面添加配置)

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker run -it --name Master -h Master registry.cn-beijing.aliyuncs.com/bitnp/docker-spark-hadoop /bin/bash
[root@Master local]#
```

### step 4

此时把Master空的容器创建出来了，当然里面什么也没配置，这时候敲上`Ctrl+P+Q`，会返回到初始目录，并且不会退出Master容器，假如你按下`Ctrl+C`，也会退出到初始目录，但是，这时候也把Master容器退出了，敲上`Ctrl+P+Q`后会出现下面情景，代码如下：

```bash
[root@Master local]# read escape sequence
renyimeng@renyimeng-virtual-machine:~$ 
```

### step 5

修改一下代码的容器名，依次创建出容器Slave1和容器Slave2：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker run -it --name Slave1 -h Slave1 registry.cn-beijing.aliyuncs.com/bitnp/docker-spark-hadoop /bin/bash
[root@Master local]#
renyimeng@renyimeng-virtual-machine:~$ sudo docker run -it --name Slave2 -h Slave2 registry.cn-beijing.aliyuncs.com/bitnp/docker-spark-hadoop /bin/bash
[root@Master local]#
```

### step 6

至此，三个空容器已经创建完成，接下来我们要使用`ssh`把三个容器连接起来。

我的 docker 中目前只有 yum，因此可以用 yum 下载 vim 来编辑文件，还可以用 yum 下载 openssh-clients、openssh-server，如果你在 docker 里面连 yum 都没有，那么你先使用`Ctrl+P+Q`退出，在初始目录用`apt-get`下载一个 yum (指令是 `sudo apt-get install yum` )，然后在 docker 里面就可以使用了。

先对Master容器进行配置，进入Master容器，敲上指令 `docker attach Master`：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker attach Master
[root@Master local]#
```

我们先下载vim，敲上指令`yum -y install vim`

```bash
[root@Master local]# yum -y install vim
.......过程省略
Complete!
[root@Master local]#
```

再把`openssh-clients`和`openssh-server`下载下来，注意按我说的顺序下载，先下`openssh-clients`：

```bash
[root@Master local]# yum -y install openssh-clients
.......
Complete!
[root@Master local]# yum -y install openssh-server
.......
Complete!
[root@Master local]
```

### step 7

这时我们配置Master容器的ssh密钥。先执行指令 `/usr/sbin/sshd`，会出现下列情景：

```bash
[root@Master local]# /usr/sbin/sshd
Could not load host key: /etc/ssh/ssh_host_rsa_key
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
Could not load host key: /etc/ssh/ssh_host_ed25519_key
sshd: no hostkeys available -- exiting.
```

再执行指令 `/usr/sbin/sshd-keygen -A` ，之后再输入一遍 `/usr/sbin/sshd` 即可：

```bash
[root@Master local]# /usr/sbin/sshd
[root@Master local]# 
```

**上述三步必不可少**，不然会出现  `ssh: connect to host localhost port 22: Cannot assign requested address`  等错误，错误[参考链接](https://blog.csdn.net/leon_wzm/article/details/78690439?utm_source=debugrun&utm_medium=referral)。

然后我们就开始制作密钥，输入指令 `ssh-keygen -t rsa` ，然后一路回车：

```bash
[root@Master local]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:BPC1QSB9LcIro8HoVtPSFoHMvDjMqgmdkhCoi+M167c root@Master
The key's randomart image is:
+---[RSA 2048]----+
|. + +=+o+.       |
|o  = ++ooo.      |
|++. + o+o.       |
|+=o=o+..         |
|=+o++o  S        |
|Oo+o             |
|=+. o            |
|o. . .           |
|  ...E.          |
+----[SHA256]-----+
[root@Master local]#
```

生成的密钥如图所示存在 `/root/.ssh/id_rsa.pub` 文件中了，我们需要把密钥存储在 `/root/.ssh/authorized_keys` 文件中，指令是： `cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys` 

```bash
[root@Master local]# cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```

此时，你可以使用指令  `cat /root/.ssh/authorized_keys`  查看`authorized_keys` 文件中是否有你刚才存入的密钥：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker attach Master
[sudo] password for renyimeng: 
[root@Master local]# cat /root/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0P4iS91BK9KuHoupZFTnzNi0iz9LxSvCU/JkAceI5Ypgjecf0OhLDkaZbu07A9vPUJ6zBYdypwvgT6sMnGpqCxsmePo8iObcUyxl7WzPYS0xlFJgJOkYFqtcNs4hOAdV8+uiU+58SJYVtIYuIoW50BNOkoELaVI+0QRWvul9M0UKEwAJFKGfAadYLRlgbcfZmMmzJbyxflBZK0ArfY3uF2zJWLN9ID38CziVFxB1QTTfrNuKvzMUuWR1UcjhDU5bNVUC14mbixYihWSkfBIxDMAtsPw1RRauYaieFSm5xVhf4yaQegIZ+4dnA9SxqSIdG0pUS+CJbeljDpLzlTM5n root@Master
```

到这里，Master 容器的 ssh 已经配好了，可以使用指令 `ssh localhost` 验证一下 ，敲上之后，需要再输入 `yes`，虽然有 warning，但是不用管它，带回配置了 `/etc/ssh/sshd_config` 文件就什么警告也没了，验证完敲上 `Ctrl+D` 即可断开与localhost 的 ssh 连接。

```bash
[root@Master local]# ssh localhost
[root@Master ~]# 
```

为了ssh链接时的美观简洁，我们配置其 `/etc/ssh/sshd_config` 文件，输入指令  `vim /etc/ssh/sshd_config` 进入编辑文件模式：

```bash
[root@Master local]# vim /etc/ssh/sshd_config
```

进去编辑文件之后，敲上 **i** ，此时进入编辑模式，按照我下面所列的代码，找到它，并改成这样：

```shell
Port 22

PermitRootLogin yes

PubkeyAuthentication yes

PasswordAuthentication yes

ChallengeResponseAuthentication no

UsePAM yes

PrintLastLog no
```

然后按下 Esc 键，进入命令模式， 再敲上指令 `:wq`  退出这个编辑文件界面，回到Master 容器界面。

再敲上指令 `vim /etc/ssh/ssh_config`  ，找到 StrictHostKeyChecking no (_大约在第35行_) ，将注释去掉，并把 ask 改成 no：

```bash
[root@Master local]# vim /etc/ssh/ssh_config 
```

```shell
StrictHostKeyChecking no
```

改完之后同样敲上 Esc 键，进入命令模式， 再敲上指令 `:wq`  退出这个编辑文件界面，回到 Master 容器界面。

这时候按下`Ctrl+P+Q`，返回到初始目录，查看此时三个容器的ip地址，输入命令： `sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(sudo docker ps -aq)` 如下：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(sudo docker ps -aq) 
/Slave2 - 172.17.0.4
/Slave1 - 172.17.0.3
/Master - 172.17.0.2
```

然后进入 Master 容器 (指令是 `sudo docker attach Master` )，打开 `/etc/hosts` 文件，把上述内容填上，目的是给每个节点的 ip 附上名字，ssh 连接的时候就可以直接 `ssh Slave1`，而不是 `ssh 172.17.0.3` 这么麻烦了，所以，我们再次进入Master 容器 ，编辑 /etc/hosts 文件，所以敲上指令 `vim /etc/hosts`  ，进入编辑模式，将容器名及其对应的 ip 填入 ，修改完之后回到Master 容器：

```bash
renyimeng@renyimeng-virtual-machine:~$ sudo docker attach Master
[root@Master ~]# vim /etc/hosts
```

```
127.0.0.1       localhost
172.17.0.2      Master
172.17.0.3      Slave1
172.17.0.4      Slave2

::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

至此，Master 容器已经配置完成，然后我们敲上 `Ctrl+P+Q` 退出当前的 Master 容器，然后敲上 `sudo docker attach Slave1` ，进入 Slave1 容器，用和 Master 容器相同的方法，把 Slave1 也配置一遍 (就是从下载 vim 开始到配置 `/etc/hosts` 文件这几个步骤)，再用相同的方式 把 Slave2 也配置完。

### step 8

三个容器Master、Slave1、Slave2 的配置终于接近尾声，仅差最后一步，我们需要把三个容器的每个密钥都要放在自己容器的 `/root/.ssh/authorized_keys` 文件中，只有这样才能把三个容器的互信建立起来，假如你不这样做，你在 Master 容器中用 ssh 连接其他容器 (比如连接 Slave1)，那么它会提示你输入 Slave1 的密码，[**而这个密码你输入什么也不对**](https://blog.csdn.net/kunlong0909/article/details/7284174)。

因为每个容器的 `/root/.ssh/authorized_keys` 文件都需要填入 所有容器的密钥，而此时我们刚配置完 Slave2 容器，那么直接输入 `vim /root/.ssh/authorized_keys` ，进去文件，然后按 i 进入编辑模式，把 Slave2 的密钥拷贝到一个文件中 (你在电脑桌面新建一个临时文件即可)，保存结束，退出文件，然后`Ctrl+P+Q`退出 Slave2 容器，然后 敲上 `sudo docker attach Slave1` ，进入 Slave1 容器，相同的方式把 Slave1 的密钥也拷贝出来，然后退出 Slave1，进入 Master 容器，把刚才拷贝的两个密钥追加到`/root/.ssh/authorized_keys` 文件中(就是进入这个文件切换为编辑模式，把那两个密钥复制过来就行)，然后把这个三个密钥拷贝出来，复制到 Slave1 的 `/root/.ssh/authorized_keys` 文件中，也同样复制到 Slave1 的 `root/.ssh/authorized_keys` 文件中。用到的详细指令如下：

- `vim /root/.ssh/authorized_keys`  进去之后 按下 **i** ，复制密钥到一个临时新建的文件 (在桌面临时建一个就行)
- 按下 Esc ， 敲上  `:wq`  保存并退出文件， 敲上 `Ctrl+P+Q` 退回初始目录 
- `sudo docker attach Slave1` 进入Slave1容器
- 重复第前两步
- `sudo docker attach Master`  进入Master容器
- `vim /root/.ssh/authorized_keys` 进去之后 按下 **i** ，把刚才复制的两个密钥放到到这个文件中，并把这个三个密钥临时存到一个文件中
- 按下 Esc ， 敲上  `:wq`  保存并退出文件， **敲上 `/usr/sbin/sshd`**，再敲上 `Ctrl+P+Q` 退回初始目录 ，
- `sudo docker attach Slave1` 进入Slave1容器
- `vim /root/.ssh/authorized_keys` 进去之后 按下 **i** ，删除所有，把刚才复制的三个密钥放到到这个文件中
- 按下 Esc ， 敲上  `:wq`  保存并退出文件，**敲上 `/usr/sbin/sshd`**，再敲上 `Ctrl+P+Q` 退回初始目录
- `sudo docker attach Slave2` 进入Slave2容器
- `vim /root/.ssh/authorized_keys`  进去之后 按下 **i** ，删除所有，把刚才复制的三个密钥放到到这个文件中
- 按下 Esc ， 敲上  `:wq`  保存并退出文件，**敲上 `/usr/sbin/sshd`**， 敲上 `Ctrl+P+Q` 退回初始目录 ，`sudo docker attach Master` 进入 Master容器，**敲上 `/usr/sbin/sshd`**

在所有密钥都放入各个容器之后，我们进行验证一下，在 Master 容器中，我们输入指令 `ssh Slave1` ，看到的情景有[下面几个情况](https://blog.csdn.net/leon_wzm/article/details/78690439?utm_source=debugrun&utm_medium=referral)：

> 如果你出现下面错误，那么你肯定没在**每个容器**中的 `/etc/hosts` 文件中输入**所有的容器**的名字及其对应的 ip 地址。
>
> **注意**：这个问题在重新启动虚拟机后存在自动回复一个容器的hosts文件对应一个ip地址的问题，需要手动再添加一次。
>
> ```bash
> [root@Master local]# ssh Slave1
> ssh: Could not resolve hostname slave1: Name or service not known
> [root@Master local]# ssh Slave2
> ssh: Could not resolve hostname slave2: Name or service not known
> ```



> 假如你出现下面错误，那么你肯定没在退出每个容器前**敲上指令 /usr/sbin/sshd** ，看上述**13**个步骤的**加粗部分**，很重要！
>
> ```bash
> [root@Master local]# ssh Slave1
> ssh: connect to host slave1 port 22: Connection refused
> [root@Master local]#
> [root@Master local]# ssh Slave2
> ssh: connect to host slave1 port 22: Connection refused
> [root@Master local]#
> ```

现在我们终于可以成功的验证了，现在 Master 容器中连接 Slave1 ，指令是 `ssh Slave1` 如下：

```bash
[root@Master local]# ssh Slave1
[root@Slave1 ~]#
```

由图可知，成功了，敲上 Ctrl +D 断开与 Slave1 的连接 ，再验证与 Slave2 的连接，指令是 `ssh Slave2` ，成功后，Ctrl+D 断开与 Slave2 的连接。

```bash
[root@Master local]# ssh Slave1
[root@Slave1 ~]# logout
Connection to slave1 closed.
[root@Master local]# ssh Slave2
[root@Slave2 ~]# 
[root@Slave2 ~]# logout
Connection to slave1 closed.
[root@Master local]#
```

> 假如你连接 Slave1 时，还提示了很多信息，不像我这样简洁，比如还有 LastLogin 之类的提示信息，那你肯定是**每个容器**的 `vim /etc/ssh/sshd_config` 和 `vim /etc/ssh/ssh_config` 文件的配置没有跟我的一样，在上述步骤中，会有这两个文件的配置。

- ssh互联成功之后，我们便开始使用hadoop进行实战，但在这之前还需配置**每个容器的环境变量**：

  - 首先我们查看 JAVA_HOME 的地址，在 hadoop-env.sh 文件中，我们并不知道这个文件的具体路径，没关系，我们只需知道它的名字，使用指令 `find / -name hadoop-env.sh`  搜索它即可，(注意这时候我们目前是在 Master 容器界面里)：

    ```bash
    [root@Master local]# find / -name hadoop-env.sh
    /usr/local/hadoop-2.7.5/etc/hadoop/hadoop-env.sh
    [root@Master local]#
    ```

    用指令 `vim /usr/local/hadoop-2.7.5/etc/hadoop/hadoop-env.sh` 进入这个文件，看到 JAVA_HOME 这一行，记录它的路径然后退出这个文件即可：

    ```bash
    [root@Master local]# vim /usr/local/hadoop-2.7.5/etc/hadoop/hadoop-env.sh
    ```

    ```
    export JAVA_HOME=/usr/local/jdk1.8.0_162
    ```

    在这个镜像中，JAVA_HOME 路径给我们设置好了，我们需要记住这个地址，会在其他配置中用到。

    接下来我们依次配置**每个容器**的 core-site.xml 和 yarn-site.xml 和 mapred-site.xml 及 hdfs-site.xml 文件。

  - 首先使用 `find / -name core-site.xml` 的具体路径，然后用指令 `vim + 文件路径` 进入这个文件：

    ```bash
    [root@Master local]# find / -name core-site.xml
    /usr/local/hadoop-2.7.5/etc/hadoop/core-site.xml
    [root@Master local]# vim /usr/local/hadoop-2.7.5/etc/hadoop/core-site.xml
    ```

    里面的配置改成如下图所示：然后保存退出

    ```xml
    <!-- Put site-specific property overrides in this file. -->
    <configuration>
          <property>
              <name>fs.defaultFS</name>
              <value>hdfs://Master:9000</value>
          </property>
          <property>
             <name>io.file.buffer.size</name>
             <value>131072</value>
         </property>
         <property>
              <name>hadoop.tmp.dir</name>
              <value>/usr/local/hadoop-2.7.5/tmp</value>
         </property>
    </configuration>
    ```

  - 进入 `yarn-site.xml` 进行配置，结束后保存退出

    ```bash
    [root@Master local]# vim /usr/local/hadoop-2.7.5/etc/hadoop/yarn-site.xml
    ```

    ```xml
    <configuration>
         <property>
             <name>yarn.nodemanager.aux-services</name>
             <value>mapreduce_shuffle</value>
         </property>
         <property>
             <name>yarn.resourcemanager.address</name>
             <value>Master:8032</value>
         </property>
         <property>
             <name>yarn.resourcemanager.scheduler.address</name>
             <value>Master:8030</value>
         </property>
         <property>
             <name>yarn.resourcemanager.resource-tracker.address</name>
             <value>Master:8031</value>
         </property>
         <property>
             <name>yarn.resourcemanager.admin.address</name>
             <value>Master:8033</value>
         </property>
         <property>
             <name>yarn.resourcemanager.webapp.address</name>
             <value>Master:8088</value>
         </property>
         <property>
             <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
             <value>org.apache.hadoop.mapred.ShuffleHandler</value>
         </property>
    </configuration>
    ```

  - 进入 `mapred-site.xml` 进行配置，结束后保存退出

    ```bash
    [root@Master local]# vim /usr/local/hadoop-2.7.5/etc/hadoop/mapred-site.xml
    ```

    ```xml
    <!-- Put site-specific property overrides in this file. -->
    <configuration>
     <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
    </configuration>
    ```

  - 进入 `hdfs-site.xml` 进行配置，结束后保存退出

    ```bash
    [root@Master local]# vim /usr/local/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
    ```

    ```xml
    <!-- Put site-specific property overrides in this file. -->
    <configuration>
        <property>
          <name>dfs.replication</name>
          <value>2</value>
        </property>
        <property>
          <name>dfs.namenode.name.dir</name>
          <value>file:/usr/local/hadoop-2.7.5/hdfs/name</value>
        </property>
    </configuration>
    ```

- 在上一步骤中是配置 Master 容器的环境变量，我们还需要进入Slave1容器，相同的代码把 Slave1的环境变量也配置完，当然容器slave2也是如此。***唯一不同的是在最后一步的 `hdfs-site.xml` 中，Master容器设置的是 namenode，而Slave1和Slave2设置的是 datanode，如下图***：

  ```bash
  [root@Slave1 local]# vim /usr/local/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
  ```

  ```xml
  <!-- Put site-specific property overrides in this file. -->
  <configuration>
      <property>
        <name>dfs.replication</name>
        <value>2</value>
      </property>
      <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop-2.7.5/hdfs/data</value>
      </property>
  </configuration>
  ```

- 格式化提前步骤

  - 现在我们在 Master 容器中通过 ssh 连接 Slave1 (或 Slave2)，删除其 hdfs 所有目录 (这个目录就在刚才的`hdfs-site.xml`文件中，忘了的话你再vim打开它，把 datanode 保存的路径记下来，我的是 `/usr/local/hadoop-2.7.5/hdfs/data`) ，并重新创建，代码如下。**因为下一步的格式化只能用一次，用两次的话就需要用到这一步，为了保险，我们在第一次就直接用这一步，以免不必要的麻烦**。

    ```bash
    [root@Master local]# ssh Slave1
    [root@Slave1 ~]# rm -rf /usr/local/hadoop-2.7.5/hdfs
    [root@Slave1 ~]# mkdir -p /usr/local/hadoop-2.7.5/hdfs/data
    ```

  -  在 Slave1 删除并创建之后，我们 Ctrl+D 断开与 Slave1 的 ssh 连接，然后 使用指令 ssh Slave2 与 Slave2 容器进行连接，与 Slave1 相同，我们需要把 hdfs目录删除，并重新创建，结束之后我们 Ctrl+D 断开与 Slave2 的连接，回到Master容器界面，代码如下：

    ```bash
    [root@Slave1 ~]# logout                                          
    Connection to slave1 closed.
    [root@Master local]# ssh Slave2
    [root@Slave2 ~]# 
    [root@Slave2 ~]# rm -rf /usr/local/hadoop-2.7.5/hdfs
    [root@Slave2 ~]# mkdir -p /usr/local/hadoop-2.7.5/hdfs/data
    [root@Slave2 ~]# logout                                          
    Connection to slave1 closed.
    [root@Master local]#
    ```

  -  Slave1 和 Slave2 都已经删除并重建 hdfs 目录了，现在我们把 Master 容器也这么做，注意 Master 容器创建的是 `name` 子文件，不再是 `data` 子文件里

    ```bash
    [root@Master local]# rm -rf /usr/local/hadoop-2.7.5/hdfs
    [root@Master local]# mkdir -p /usr/local/hadoop-2.7.5/hdfs/name
    [root@Master local]#
    ```

- 现在我们格式化 NameNode HDFS 目录， 在 Master 容器中，使用指令 `hdfs namenode -format`

```bash
[root@Master local]# hdfs namenode -format
..........一堆代码，此处省略
[root@Master local]#
```

- 启动 hadoop 集群

  - 我们需要进入 sbin 文件夹，来启动 hadoop 集群，我们不清楚 sbin 文件的具体路径，所以我们可以使用 指令 ``find / -name sbin`  找到路径，然后 cd +路径  进入这个路径。

    ```bash
    [root@Master local]# find / -name sbin
    /usr/local/sbin
    /usr/local/hadoop-2.7.5/sbin
    /usr/local/spark-2.3.0-bin-hadoop2.7/sbin
    /usr/lib/debug/usr/sbin
    /usr/lib/debug/sbin
    /usr/sbin
    /sbin
    [root@Master local]# cd /usr/local/hadoop-2.7.5/sbin
    [root@Master sbin]#
    ```

  - 查看当前路径下的文件，可以得到：

    ```bash
    [root@Master sbin]# cd /usr/local/hadoop-2.7.5/sbin
    [root@Master sbin]# ls
    distribute-exclude.sh  mr-jobhistory-daemon.sh  start-dfs.sh         stop-dfs.cmd
    hadoop-daemon.sh       refresh-namenodes.sh     start-secure-dns.sh  stop-dfs.sh
    hadoop-daemons.sh      slaves.sh                start-yarn.cmd       stop-secure-dns.sh
    hdfs-config.cmd        start-all.cmd            start-yarn.sh        stop-yarn.cmd
    hdfs-config.sh         start-all.sh             stop-all.cmd         stop-yarn.sh
    httpfs.sh              start-balancer.sh        stop-all.sh          yarn-daemon.sh
    kms.sh                 start-dfs.cmd            stop-balancer.sh     yarn-daemons.sh
    ```

    

  - 

  - 原作者的教程说使用指令 `./start-all.sh` 来启动集群，尝试后报错，网上检索后显示是 spark 中有类似的启动集群的方法，因此为了避免混淆，在最新的hadoop 集群启动时，使用 `start-dfs.sh `和 `start-yarn.sh` 两句命令即可：

    ```bash
    [root@Master sbin]# start-dfs.sh
    Starting namenodes on [Master]
    Master: starting namenode, logging to /usr/local/hadoop-2.7.5/logs/hadoop-root-namenode-Master.out
    Slave2: starting datanode, logging to /usr/local/hadoop-2.7.5/logs/hadoop-root-datanode-Slave2.out
    Slave1: starting datanode, logging to /usr/local/hadoop-2.7.5/logs/hadoop-root-datanode-Slave1.out
    Starting secondary namenodes [0.0.0.0]
    0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.7.5/logs/hadoop-root-secondarynamenode-Master.out
    [root@Master sbin]# start-yarn.sh
    starting yarn daemons
    starting resourcemanager, logging to /usr/local/hadoop-2.7.5/logs/yarn--resourcemanager-Master.out
    Slave1: starting nodemanager, logging to /usr/local/hadoop-2.7.5/logs/yarn-root-nodemanager-Slave1.out
    Slave2: starting nodemanager, logging to /usr/local/hadoop-2.7.5/logs/yarn-root-nodemanager-Slave2.out
    [root@Master sbin]#  
    ```

  -  使用 `jps` 查看 namenode 是否启动，此时看的是 Master 容器的 namenode是否启动。

    ```bash
    [root@Master sbin]# jps
    593 ResourceManager
    436 SecondaryNameNode
    852 Jps
    245 NameNode
    [root@Master sbin]#
    ```

    ![Screen Shot 2020-10-14 at 3.01.06 PM](https://tva1.sinaimg.cn/large/007S8ZIlly1gjp1kvqc3lj30ek06474t.jpg)

  - 这里我们可以使用 `ssh Slave1` (或`ssh Slave2`) 进入Slave1容器，然后使用指令`jps` 查看 datanode 是否启动，此时会出现 `-bash: jps: command not found` 错误：

    ```bash
    [root@Master sbin]# ssh Slave1
    [root@Slave1 ～]# jps
    -bash: jps: command not found
    ```

    这时我们需要配置 `/etc/profile` 文件，每个容器 (包括Master、Slave1、Slave2) 都需要配置这个文件，使用指令 `vim /etc/profile`，末尾添加代码 ：

    ```bash
    export JAVA_HOME=/usr/local/jdk1.8.0_162
    export HADOOP_HOME=/usr/local/hadoop-2.7.5
    export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    ```

    保存退出后，执行生效刚才的 /etc/profile 文件，即使用指令 `source /etc/profile`   ，如图所示：

    ```bash
    [root@Master local]# vim /etc/profile
    ....进入之后末尾添加下列代码(根据自己的实际配置填写，比如java版本可能每个人都不一样)
    export JAVA_HOME=/usr/local/jdk1.8.0_162
    export HADOOP_HOME=/usr/local/hadoop-2.7.5
    export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    .....保存退出后
     
    [root@Master local]# source /etc/profile
    ```

    每个容器都配置结束后，我们再通过 ssh 进入其他容器，便可以使用 `jps` 或 `hadoop fs xx` 指令了。

  - 我们可以使用指令 `hadoop dfsadmin -report` 查看其他容器有没有启动

    ```bash
    [root@Master sbin]# hadoop dfsadmin -report
    DEPRECATED: Use of this script to execute hdfs command is deprecated.
    Instead use the hdfs command for it.
    
    Configured Capacity: 40943976448 (38.13 GB)
    Present Capacity: 3759964160 (3.50 GB)
    DFS Remaining: 3759906816 (3.50 GB)
    DFS Used: 57344 (56 KB)
    DFS Used%: 0.00%
    Under replicated blocks: 0
    Blocks with corrupt replicas: 0
    Missing blocks: 0
    Missing blocks (with replication factor 1): 0
    
    Live datanodes (2):
    
    Name: 172.17.0.4:50010 (Slave2)
    Hostname: Slave2
    Decommission Status : Normal
    Configured Capacity: 20471988224 (19.07 GB)
    DFS Used: 28672 (28 KB)
    Non DFS Used: 17528487936 (16.32 GB)
    DFS Remaining: 1879953408 (1.75 GB)
    DFS Used%: 0.00%
    DFS Remaining%: 9.18%
    Configured Cache Capacity: 0 (0 B)
    Cache Used: 0 (0 B)
    Cache Remaining: 0 (0 B)
    Cache Used%: 100.00%
    Cache Remaining%: 0.00%
    Xceivers: 1
    Last contact: Wed Oct 14 14:51:21 UTC 2020
    
    Name: 172.17.0.3:50010 (Slave1)
    Hostname: Slave1
    Decommission Status : Normal
    Configured Capacity: 20471988224 (19.07 GB)
    DFS Used: 28672 (28 KB)
    Non DFS Used: 17528487936 (16.32 GB)
    DFS Remaining: 1879953408 (1.75 GB)
    DFS Used%: 0.00%
    DFS Remaining%: 9.18%
    Configured Cache Capacity: 0 (0 B)
    Cache Used: 0 (0 B)
    Cache Remaining: 0 (0 B)
    Cache Used%: 100.00%
    Cache Remaining%: 0.00%
    Xceivers: 1
    Last contact: Wed Oct 14 14:51:21 UTC 2020
    
    [root@Master sbin]# 
    ```


到此，我们的 docker 环境下的 hadoop 集群已经搭建成功！开酒庆祝！

