---
layout:     post
title:      "容器监控"
subtitle:   " \"容器监控\""
date:       2018-11-14 14:24:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> “docker study ”

# Jenkins、Gitlab、Docker Hub与Docker的自动化CI/CD实战
## gitlab
### 1、配置yum源
```
vim /etc/yum.repos.d/gitlab-ce.repo
```
复制以下内容：
```
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
Repo_gpgcheck=0
Enabled=1
Gpgkey=https://packages.gitlab.com/gpg.key
```
### 2、更新本地yum缓存
```
sudo yum makecache
```
### 3、安装GitLab社区版
```
sudo yum intall gitlab-ce        #自动安装最新版
sudo yum install gitlab-ce-x.x.x    #安装指定版本
```
GitLab常用命令
```
sudo gitlab-ctl start    # 启动所有 gitlab 组件；
sudo gitlab-ctl stop        # 停止所有 gitlab 组件；
sudo gitlab-ctl restart        # 重启所有 gitlab 组件；
sudo gitlab-ctl status        # 查看服务状态；
sudo gitlab-ctl reconfigure        # 启动服务；
sudo vim /etc/gitlab/gitlab.rb        # 修改默认的配置文件；
gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
sudo gitlab-ctl tail        # 查看日志；
```
4、配置Gitlab

配置SSH Keys

将/root/.ssh/下的id_rsa.pub新增到gitlab设置-》SSH秘钥。如下图：
![](https://github.com/gmg0829/Img/blob/master/dockerImg/gitlab-ssh.png?raw=true)

新建项目,项目如下：
![](https://github.com/gmg0829/Img/blob/master/dockerImg/gitlab-pro.png?raw=true)


## jenkins
### 1、安装Jenkins
下载jenkins的war包放到tomcat的webapps目录下，启动tomcat
。
### 2、配置Jenkins
2.1 系统管理-》插件管理下载以下插件：	
 - Git Parameter
 - GitLab Plugin
 - Maven Integration plugin
 - Publish Over SSH
 - SSH plugin

2.2 配置Jdk、Git、Maven:
![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-Java-git-mvn.png?raw=true)
2.3 新增访问gitlab凭据
![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-g-authen.png?raw=true)
### 新建项目、配置项目
1、动态获取Git仓库tag，与用户交互选择Tag发布：【也可以设置分支】
![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-tag.png?raw=true)
2、指定项目Git仓库地址：
![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-source.png?raw=true)
修改*/master为$Tag，Tag是上面动态获取的变量名，表示根据用户选择打代码版本。

3.设置maven构建命令选项：
```
clean package -Dmaven.test.skip=true
```
![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-pre.png?raw=true)

4、在Jenkins本机镜像构建与推送到镜像仓库，Docker主机使用推送的镜像创建容器：

![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-post.png?raw=true)
上图的命令如下：
```
REPOSITORY=registry.cn-hangzhou.aliyuncs.com/gmg/eureka-server:${Tag}
cat > Dockerfile << EOF
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD target/eureka-server-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
EXPOSE 8761
EOF
docker build -t $REPOSITORY .
docker login --username=gmg0829 --password=xxx registry.cn-hangzhou.aliyuncs.com
docker push $REPOSITORY
```
```
REPOSITORY=registry.cn-hangzhou.aliyuncs.com/gmg/eureka-server:${Tag}
sudo docker rm -f eureka-server |true
sudo docker image rm $REPOSITORY |true
sudo docker container run -d --name eureka-server -p 8761:8761 $REPOSITORY
```

### 构建项目
1、点击构建

![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-build.png?raw=true)
2、点击查看构建日志

![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-console.png?raw=true)
3、显示构建成功

![](https://github.com/gmg0829/Img/blob/master/dockerImg/j-success.png?raw=true)

### 查看镜像、容器和远程仓库
```
$ docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
registry.cn-hangzhou.aliyuncs.com/gmg/eureka-server   0.0.1               f5d98ac34a75        6 minutes ago       204MB
$ docker ps
CONTAINER ID        IMAGE                                                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
cb3630774275        registry.cn-hangzhou.aliyuncs.com/gmg/eureka-server:0.0.1   "java -Djava.securit…"   19 minutes ago      Up 19 minutes       0.0.0.0:8761->8761/tcp   eureka-server


```
![](https://github.com/gmg0829/Img/blob/master/dockerImg/ali-docker.png?raw=true)