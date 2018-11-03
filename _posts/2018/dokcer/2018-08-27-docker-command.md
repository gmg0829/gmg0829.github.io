---
layout:     post
title:      "docker command"
subtitle:   " \"docker command\""
date:       2018-08-27 14:24:00
author:     "gmg"
header-img: "img/post/sayBye2017.jpg"
catalog: true
tags:
    - 工作
---

> “docker study ”

## 镜像命令
搜索镜像：docker search java  
下载镜像：docker pull java  
列出镜像：docker images  
删除指定镜像：docker rmi hello  
删除所有镜像 docker rmi -f $(docker images)
## 容器命令
新建并启动容器：docker run  
``` 
1、 docker run -d --name nginx nginx:latest  
后台启动并运行一个名为nginx的容器，运行前它会自动去docker镜像站点下载最新的镜像文件  
2、 docker run -d -P 80:80 nginx:latest  
后台启动并运名为nginx的容器，然后将容器的80端口映射到物理机的80端口
3、docker run -d -v /docker/data:/docker/data -P 80:80 nginx:latest  
后台启动并运名为nginx的容器，然后将容器的80端口映射到物理机的80端口,并且将物理机的/docker/data目录映射到容器的/docker/data
3、docker run -it  nginx:latest /bin/bash  
以交互式模式运行容器，然后在容器内执行/bin/bash命令
 ```
 删除容器 docker rm
```
1、docker rm -f mydocker
强制删除容器mydocker
2、 docker rm -f dockerA dockerB
强制删除容器dockerA，dockerB
3、docker rm -v mydocker
#删除容器，并删除容器挂载的数据卷
````
列出容器 docker ps  
停止容器 docker stop  
强制停止容器 docker kill  
重启容器 docker restart  
进入容器 docker attach   
获取容器的元数据  docker inspect  
获取容器的日志  docker logs
```
-f        #跟踪日志输出
-t        #显示时间戳
--tail    #只显示最新n条容器日志
--since   #显示某个开始时间的所有日志
```
显示指定容器的端口映射  docker port
## Dockerfile常用命令
ADD 复制文件  
COPY 复制文件  （不支持URL和压缩包）  
ARG 设置构建参数  
ENV 设置环境变量
CMD 容器启动命令  
ENTRYPOINT 入口点  
EXPOSE 生命暴露的接口  
FROM指定基础镜像  
LABEL 为镜像添加元数据  
MAINTAINER 指定维护者信息  
RUN 执行命令  
USER 设置用户  
VOLUME 指定挂载点  
WORKDIR 指定工作目录

## Docker Hub命令
登录  docker login  
推送镜像 docker push

## maven的docker插件推送镜像步骤
第一步：设置maven的setting配置
 
    <server>
        <id>docker-hub</id>
        <username>用户名</username>
        <password>密码</password>
        <configuration>
            <email>邮箱</email>
        </configuration>
    </server>
第二步：修改pom

    增加<serverId>docker-hub</serverId>
第三步：执行命令

    mvn clean package docker:build -DpushImage
    
# docker-compose 常用命令：
docker-compose 
   build 构建或重新构建项目中的服务容器  
   kill   停止指定服务的容器  
   logs 查看服务日志输出   
   down 停止up命令所启动的容器  
   exec 进入指定的容器  
   port 打印绑定的公共端口  
   ps 列出所有容器  
   pull 下载镜像   
   rm 删除镜像   
   stop 停止已运行打得容器  
   up 启动
 # Docker Machine 常用命令：
docker-machine  
    create  创建虚拟机   
    rm 移除虚拟机   
    ssh登录虚拟机  
    env 客户端配置环境变量  
    inspect  检查机子信息  
    ls 查看虚拟机列表  
    status  查看虚拟机状态  
    start 启动虚拟机   
    stop  停止虚拟机        
    restart 重启虚拟机

 

