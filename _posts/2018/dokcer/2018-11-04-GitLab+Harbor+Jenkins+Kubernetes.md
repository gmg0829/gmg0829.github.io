---
layout:     post
title:      "GitLab+Harbor+Jenkins+Kubernetes构建持续交付系统"
subtitle:   " \"GitLab+Harbor+Jenkins+Kubernetes构建持续交付系统\""
date:       2018-11-04 14:24:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> “docker study ”

# GitLab+Harbor+Jenkins+Kubernetes构建持续交付系统
![](https://github.com/gmg0829/Img/blob/master/dockerImg/dokcer+k8s.png?raw=true)

**流程说明**

应用构建和发布流程说明。

    1、用户向Gitlab提交代码，代码中必须包含Dockerfile

    2、将代码提交到远程仓库

    3、用户在发布应用时需要填写git仓库地址和分支、服务类型、服务名称、资源数量、实例个数，确定后触发Jenkins自动构建

    4、Jenkins的CI流水线自动编译代码并打包成docker镜像推送到Harbor镜像仓库

    5、Jenkins的CI流水线中包括了自定义脚本，根据我们已准备好的kubernetes的YAML模板，将其中的变量替换成用户输入的选项

    6、生成应用的kubernetes YAML配置文件

    7、更新Ingress的配置，根据新部署的应用的名称，在ingress的配置文件中增加一条路由信息

    8、更新PowerDNS，向其中插入一条DNS记录，IP地址是边缘节点的IP地址。关于边缘节点，请查看边缘节点配置

    9、Jenkins调用kubernetes的API，部署应用