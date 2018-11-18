---
layout:     post
title:      "Docker三剑客之docker-machine"
subtitle:   " \"docker-machine\""
date:       2018-11-11 14:24:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> “docker study ”

## 简介

docker-machine就是docker公司官方提出的，用于在各种平台上快速创建具有docker服务的虚拟机的技术，甚至可以通过指定driver来定制虚拟机的实现原理（一般是virtualbox）。

Docker Machine 是安装和管理 Docker 的工具。它有命令行工具：docker-machine。

docker-machine可以在本地布署相应环境的同时完成远程docker主机相同环境的布署，减少重复的操作。

### docker-machine安装

```
$ sudo curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine
$ sudo chmod +x /usr/local/bin/docker-machine
```
完成后，查看版本信息。
```
$ docker-machine -v
docker-machine version 0.13.0, build 9ba6da9
```
### docker-machine命令
```
docker-machine active
#显示当前的活动主机

docker-machine config
#显示连接主机的配置

docker-machine create
#创建一个主机

docker-machine env
#设置当前的环境与哪个主机通信

docker-machine inspect
#查看主机的详细信息

docker-machine ip
#查看主机的IP 

docker-machine kill
#强制关闭一个主机

docker-machine ls 
#查看所有的主机信息

docker-machine provision
#重新配置现在主机

docker-machine regenerate-certs
#为主机重新生成证书

docker-machine restart
#重启主机

docker-machine rm
#删除主机

docker-machine ssh
#以SSH的方式连接到主机上

docker-machine scp
#远程复制

docker-machine status
#查看主机的状态

docker-machine stop
#停止一个正在运行的主机

docker-machine upgrade
#升级主机的docker服务到最新版本

docker-machine version
#查看docker-machine版本
```

## 实例

两台服务器

- 本地主机：192.168.124.129 

- 远程主机：192.168.124.132

1、配置免密登录

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
/root/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:k6pTKvpG2AOIyktJfAV09tkeJ/zB6aYHtpxY7Q5jzzk root@node01
The key's randomart image is:
+---[RSA 2048]----+
|  .o.o           |
|    o.. + . .    |
|+   .  o = =     |
|+o .    ..B .    |
|++o     S= =     |
|o++   ..=.B      |
|.... o.. O o     |
| .o o.  . BE.    |
|.+....     =.    |
+----[SHA256]-----+

$ ssh-copy-id 192.168.124.132
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 192.168.124.132 (192.168.124.132)' can't be established.
ECDSA key fingerprint is SHA256:isURMZPfryhzac244IpxCfFGGkdWQ32IvV6MNIkk6L0.
ECDSA key fingerprint is MD5:01:d9:a4:16:79:0f:b9:dc:98:4b:c9:5d:9e:67:25:14.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.124.132's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.124.132'"
and check to make sure that only the key(s) you wanted were added.

$ ssh 192.168.124.132
Last login: Wed Nov 14 11:18:51 2018 from 192.168.124.1

$ ip add |grep 192.168.124
    inet 192.168.124.132/24 brd 192.168.124.255 scope global ens32
```

2、创建远程主机

远程主机需要安装有docker环境
```
$ docker-machine create -d generic --generic-ip-address=192.168.124.132 --generic-ssh-user=root --engine-registry-mirror https://registry.docker-cn.com   dockerhost

Running pre-create checks...
Creating machine...
(dockerhost) No SSH key specified. Assuming an existing key at the default location.
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with centos...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env dockerhost

$ docker-machine ls
NAME         ACTIVE   DRIVER    STATE     URL                          SWARM   DOCKER     ERRORS
dockerhost   -        generic   Running   tcp://192.168.124.132:2376           v18.09.0   
```
3、配置环境变量
```
$ docker-machine env dockerhost
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.124.132:2376"
export DOCKER_CERT_PATH="/root/.docker/machine/machines/dockerhost"
export DOCKER_MACHINE_NAME="dockerhost"
# Run this command to configure your shell: 
# eval $(docker-machine env dockerhost)
$ eval $(docker-machine env dockerhost)
$ docker-machine ssh dockerhost
Last login: Thu Nov 15 05:51:29 2018 from manager
[root@dockerhost ~]# docker -v
Docker version 18.09.0, build 4d60db4
```
4、运行一个容器

```
$ docker run -d nginx:1.13
Unable to find image 'nginx:1.13' locally
1.13: Pulling from library/nginx
f2aa67a397c4: Pull complete 
3c091c23e29d: Pull complete 
4a99993b8636: Pull complete 
Digest: sha256:b1d09e9718890e6ebbbd2bc319ef1611559e30ce1b6f56b2e3b479d9da51dc35
Status: Downloaded newer image for nginx:1.13
7dfcf725aca0edce72e6b1288cfa803e6feadff8fba1216fc1811c97d83eb05d
$  docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               1.13                ae513a47849c        6 months ago        109MB

$ docker-machine ssh dockerhost
Last login: Thu Nov 15 05:52:52 2018 from manager
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               1.13                ae513a47849c        6 months ago        109MB
```
由此可以发现，利用docker-machine可以减少重复操作，便于环境的创建


参考地址链接：
- [容器技术｜Docker三剑客之docker-machine](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486291&idx=1&sn=937f1fdb7d05838e37fd256233e45b7e&chksm=e91b6e4fde6ce759f66f2435feea0bc876fcdb7b2761096ce086465a395179ef0d86804911da&scene=21#wechat_redirect)