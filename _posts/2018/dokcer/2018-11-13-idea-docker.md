---
layout:     post
title:      "idea集成Dokcer实战"
subtitle:   " \"idea集成Dokcer实战\""
date:       2018-11-13 14:24:00
author:     "gmg"
catalog: true
tags:
    - 工作
---


# idea集成Dokcer实战

## 1、下载Dokcer插件
![](https://github.com/gmg0829/Img/blob/master/dockerImg/docker-intergetion.png?raw=true)
## 2、修改docker.service配置文件
```
$ vi /usr/lib/systemd/system/docker.service
在 ExecStart=/usr/bin/dockerd
后添加 -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
修改后：
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```
## 3、idea配置连接
![](https://github.com/gmg0829/Img/blob/master/dockerImg/idea-docker-set.png?raw=true)

## 4、idea配置启动项
![](https://github.com/gmg0829/Img/blob/master/dockerImg/idea-docker-run.png?raw=true)

## 5、启动
![](https://github.com/gmg0829/Img/blob/master/dockerImg/idea-run-success.png?raw=true)

查看远程服务器
```
 $ docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
eureka-server                           0.0.1               17c2a6cbcb05        24 minutes ago      204MB
frolvlad/alpine-oraclejdk8              slim                3ee5e1ce00fc        13 days ago         164MB

docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
720b58448621        17c2a6cbcb05        "java -Djava.securit…"   25 minutes ago      Up 25 minutes       0.0.0.0:8761->8761/tcp   eureka-server

```


