---
title: Mac使用Docker搭建Ubuntu环境
author: Yimeng
date: '2020-09-30'
permalink: /posts/2020/09/mac-docker-ubuntu/
categories:
  - Tools
tags:
  - Ubuntu
  - vertue-machine
  - linux
---

主要参考资料：  
1. [mac 下使用 Docker 搭建 ubuntu 环境
](https://www.smslit.top/2018/12/20/docker_ubuntu_learn/)  
2. [Linux Mac之间文件传输](https://developer.aliyun.com/article/7949)  
3. [Mac下启动ssh服务](https://www.jianshu.com/p/52f01e967e22)

对虚拟机和双系统不是很了解，本想着 MacOS 不需要装 Linux 虚拟机就可以完成大部分工作，谁知道老师让 MacOS 的同学也使用 Docker 安装一下 Ubuntu ，学习这个安装和使用的过程，以备日后工作中的需要。使用 Docker 搭建 Ubuntu 环境是不会污染 MacOS 的，同时还可以学习 Linux 的用法。除了安装 Ubuntu 环境，我还尝试了使用 Mac 中的 ssh 连接 Ubuntu 虚拟机，分享一下搭建环境的过程。

Most importantly, peace and love. :)

## 安装和配置 Docker

### 下载并安装 Docker

- 登录 [Docker 官网](https://hub.docker.com/) 下载安装包；

- 按照 Mac 安装软件的流程，将安装好的 Docker 拖入 Application 文件夹中；

- 成功运行时，电脑顶部栏会出现一个小鲸鱼的 LOGO；

- 如果安装成功，在终端运行版本查看的命令可以得到如下信息：

```
(base) yimeng@localhost ~ %  docker --version
Docker version 19.03.12, build 48a66213fe
(base) yimeng@localhost ~ %  docker-compose --version
docker-compose version 1.27.2, build 18f557f9
(base) yimeng@localhost ~ %  docker-machine --version
docker-machine version 0.14.0, build 89b8332
```

### 配置 Docker

由于国内直接访问 Docker 官方默认的镜像源比较慢，可以在 Docker desktop 中更换国内的镜像源进行加速。这里使用官方提供的一个镜像仓库地址：`https://registry.docker-cn.com`。

- 在顶栏中的小鲸鱼 logo 中打开 Preferences；

- 选择 Docker Engine，修改配置文件（这里基于我安装的 Docker 来说，应该不同的界面修改方式是类似的）；

![docker_config](https://tva1.sinaimg.cn/large/007S8ZIlly1gj8gyb6r6nj318s0u0tnj.jpg)

- 给 `registry-mirrors` 参数设置国内镜像，点击 `Apply & Restart`（更多配置文件参数请参考 [Docker daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/)）。

## 定制 Ubuntu 镜像

### 获取 Ubuntu 镜像

在 Mac OS Terminal 运行：

```
docker pull ubuntu
```

该命令从 ubuntu 官网拉取最新的镜像，作为定制 ubuntu 镜像的基础。

运行命令 `docker image ls` 可以查看当前安装的 Docker 镜像。

```
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
ubuntu-ssh               latest              9c0f16f0d16a        About an hour ago   351MB
ubuntu-ssh-yimeng        latest              51f97460be28        13 hours ago        346MB
ubuntu                   latest              9140108b62dc        4 days ago          72.9MB
docker/getting-started   latest              1f32459ef038        2 months ago        26.8MB
```

### ubuntu 容器

#### 创建一个 ubuntu 容器

运行命令 `docker run -i -t --name mineos ubuntu bash` 可以创建并运行一个可以使用终端交互的 ubuntu 容器，参数含义如下：

| 参数     | 值      | 含义             |
|--------|--------|----------------|
| \-i    | 无      | 可以输入进行交互       |
| \-t    | 无      | 终端交互           |
| –name  | mineos | 指定容器名称为 mineos |
| ubuntu | 无      | 指定使用镜像         |
| bash   | 无      | 指定容器启动使用的应用    |


上面的命令执行后，就会登陆 ubuntu 容器的 bash 中，执行命令 `cat /etc/issue` 可以查看系统版本，我的ubuntu版本是 `Ubuntu 20.04.1 LTS`。使用快捷键 `ctrl` + `d` 退出 ubuntu 容器，此时就会停止容器的运行。

#### 查看已有的容器

运行命令 `docker ps` 可以查看当前运行的容器，运行命令 `docker ps -a` 可以查看所有的容器信息，包括已经关闭的。若上一步已经停止 mineos 容器的运行，运行后者就可以看到已经关闭的 mineos 容器信息。

```
(base) yimeng@localhost ~ % docker ps -a
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                        PORTS                   NAMES
f1f0fe3c7d7a        ubuntu-ssh               "/usr/sbin/sshd -D"      About an hour ago   Exited (255) 29 minutes ago   0.0.0.0:26122->22/tcp   yimeng-learn
14c6626650af        ubuntu-ssh-yimeng        "/usr/sbin/sshd -D"      10 hours ago        Created                                               rym
0f863d7f183f        ubuntu-ssh-yimeng        "/usr/sbin/sshd -D"      13 hours ago        Exited (255) 3 hours ago      0.0.0.0:26122->22/tcp   learn
d5dd6461cd7a        ubuntu                   "bash"                   17 hours ago        Exited (0) 6 seconds ago                              mineos
00f99784b701        ubuntu                   "bash"                   24 hours ago        Created                                               sad_cerf
4a3dcd80f7ae        ubuntu                   "bash"                   24 hours ago        Exited (255) 3 hours ago                              ecstatic_wiles
cf6a0e5c227f        docker/getting-started   "/docker-entrypoint.…"   24 hours ago        Exited (255) 3 hours ago      0.0.0.0:80->80/tcp      eloquent_hypatia

```

#### 以交互的形势启动容器

运行命令 `docker start mineos` 可以启动容器，此时会发现无法像创建容器时登录容器的bash，此时加入参数 `-i` 就可以了：

```
(base) yimeng@localhost ~ % docker start -i mineos
root@d5dd6461cd7a:/# 
```

## ubuntu 容器的基本配置

登录到 ubuntu 的 bash 之后就可以当做正常的 ubuntu 使用了。

- 更新软件源信息：`apt-get update`；

- 首先安装 vim：`apt-get install vim`；

- 之所以先安装编辑工具，是为了可以编辑 `/etc/apt/sources.list`，更换为国内访问更快的软件源，比如可以将文件中的内容替换为如下阿里云的：

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

- 重新更新软件源信息：`apt-get update`，可以发现更新的速度明显加快；

- 安装 git 和 python3：`apt-get install git python3`；

## 配置 SSH

这一部分主要是为了 Mac 可以 ssh 连接 ubuntu 容器。

### 安装 openssh-server

```
apt-get install openssh-server
```

用于开启 ssh 服务供外部连接。

### 配置 sshd

需要更改 sshd 的默认配置，编辑文件 `/etc/ssh/sshd_config`，大概从 29 行开始主要更改三处，更改后内容如下：

```
PermitRootLogin yes # 可以登录 root 用户
PubkeyAuthentication yes # 可以使用 ssh 公钥许可
AuthorizedKeysFile	.ssh/authorized_keys # 公钥信息保存到文件 .ssh/authorized_keys 中
```

### 重启 sshd

为了让扇面的配置生效，使用命令 `/etc/init.d/ssh restart` 重启 sshd。

### 添加主机的 ssh 公钥

此时应当在 ubuntu 容器中，添加主机 Mac OS 的公钥。

- 在 HOME 路径下创建 .ssh 目录：`mkdir ~/.ssh`；

- 新建文件 `~/.ssh/authorized_keys`：`touch ~/.ssh/authorized_keys`；

- 新开一个 Mac OS terminal，运行命令 `cat ~/.ssh/id_rsa.pub`，复制打印的公钥信息；

- 回到 ubuntu 容器中，将上一步中的公钥信息粘贴到 `~/.ssh/authorized_keys` 中保存。

> 注意：如果主机没有公钥信息，需要先[生成 ssh 公钥](https://blog.csdn.net/iAilu/article/details/89790105?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)。

- 此时完成了 ssh 访问支持的添加，退出容器。

## 提交修改到镜像

现在已经退出到 mac terminal 窗口，容器的修改并不会影响到源镜像，以上的步骤完成了 ubuntu 的基本配置，并且添加了 ssh 支持，这一步是产生新的镜像版本。

- 查看刚才操作的容器信息，运行命令 `docker ps -a`，可以看到 mineos 的状态是退出（exited）的，复制 mineos 的 `CONTAINER ID`，比如为 `d5dd6461cd7a`；

- 运行命令提交产生 ubuntu 新版本的镜像：

```
docker commit -m 'add ssh' -a '5km' d5dd6461cd7a ubuntu-ssh
```

1. -m，指定提交信息

2. -a，指定提交者

3. 你需要把 d5dd6461cd7a 替换为您的容器的 CONTAINER ID

4. ubuntu-ssh 是新镜像的名称，可以随意指定

- 使用命令 `docker image ls` 可以查看当前安装的镜像，上述操作正常的话就会看到 `ubuntu-ssh` 的镜像信息；

- 此时之前创建的容器就没用了，可以通过命令 `docker rm mineos` 进行删除。

## 最终的 Ubuntu 容器

有了具有 SSH 支持的 ubuntu 镜像，我们就可以创建新的 ubuntu 容器，通过以下命令进行创建：

```
docker run -d -p 26122:22 --name yimeng-learn ubuntu-ssh /usr/sbin/sshd -D
```

| 参数                 | 值        | 含义                                               |
|--------------------|----------|--------------------------------------------------|
| \-d                | 无        | 后台运行                                             |
| \-p                | 26122:22 | 绑定主机的 26122 端口到ubuntu容器的 22 端口\(ssh服务的默认端口为 22\) |
| –name              | learn    | 指定容器名称为 learn                                    |
| ubuntu\-ssh        | 无        | 使用镜像 ubuntu\-ssh 创建容器                            |
| /usr/sbin/sshd \-D | 无        | 指定容器启动使用的应用及参数                                   |


在 mac terminal 运行 `ssh -p 26122 root@localhost` 即可连接已经启动的 ubuntu 容器 yimeng-learn。

> 注意：连接时需要输入公钥密码，该密码为设置 mac ssh 公钥时自己设定的密码。

为了更方便的连接，可以为容器创建 ssh 连接的主机短名，往 macOS 的 `~/.ssh/config` 中添加以下内容：

```
Host yimeng-learn
	HostName localhost
	User     root
	Port     26122
```

此时就可以通过命令 `ssh yimeng-learn` 连接 ubuntu 容器 yimeng-learn 了。

## Ubuntu 和 Mac 之间的文件传输

从 Mac 上传输文件到 Linux 主机上，这个过程可以使用 FTP 客户端，如 Transmit for Mac、FileZilla ，虽然使用客户端操作起来比较方便，但需要下载安装等，可能遇到下载不流畅等问题。所以直接在终端运行命令来实现文件传输更加方便。

**主要使用 scp 命令。**

```
scp [可选参数] file_source file_target 
```

### 复制文件

```
scp local_file remote_username@remote_ip:remote_folder 
scp local_file remote_username@remote_ip:remote_file 
scp local_file remote_ip:remote_folder 
scp local_file remote_ip:remote_file 
```

- 第1,2个指定了用户名，命令执行后需要再输入密码，第1个仅指定了远程的目录，文件名字不变，第2个指定了文件名； 

- 第3,4个没有指定用户名，命令执行后需要输入用户名和密码，第3个仅指定了远程的目录，文件名字不变，第4个指定了文件名； 

### 复制目录

```
scp -r local_folder remote_username@remote_ip:remote_folder 
scp -r local_folder remote_ip:remote_folder 
```

- 第1个指定了用户名，命令执行后需要再输入密码； 

- 第2个没有指定用户名，命令执行后需要输入用户名和密码； 

### 查看 Mac OS 主机 ip 地址

在 terminal 运行 `ifconfig`，找到主机的 ip 地址：

![MAC_ip](https://tva1.sinaimg.cn/large/007S8ZIlly1gj8p5wv6isj315g0qik8q.jpg)

连接 ubuntu 容器，进行文件的复制：

```
root@d50ed9079017:~# scp yimeng@10.48.1.240:/Users/yimeng/Desktop/dbgen /root/renyimeng/dbgen
Password:
dbgen                                                100%  120KB  25.2MB/s   00:00    

```

完成文件在 Mac 和 Ubuntu 之间的传输。
