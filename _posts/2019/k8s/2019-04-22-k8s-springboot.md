---
layout:     post
title:      "部署springboot镜像到k8s"
subtitle:   " \"部署springboot镜像到k8s\""
date:       2019-04-22 14:24:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> k8s study ”



# 部署springboot镜像到k8s
## 推送镜像到镜像仓库
这里使用的镜像仓库是阿里云仓库。镜像名称为 
```
registry.cn-hangzhou.aliyuncs.com/gmg/hello:0.0.1
```
## 编写hello-rc.yaml
> 注意：生成secret，不然会出现镜像拉去失败

执行如下命令
```
$ kubectl create secret docker-registry regsecret 
-n namespace --docker-server=registry.cn-hangzhou.aliyuncs.com --docker-username=gmg0829  --docker-password=xxx --docker-email=xxx@163.com
```
其中：

- regsecret： 指定密钥的键名称，可自行定义。
- n：指定namespace，如不指定则默认创建在default命名空间下。
- --docker-server：指定 Docker 仓库地址。
- --docker-username: 指定 Docker 仓库用户名。
- --docker-password：指定 Docker 仓库登录密码，即容器镜像registry独立登录密码。
- --docker-email：指定邮件地址。

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: hello
  labels:
    name: hello
spec:
  replicas: 1
  selector:
    name: hello
  template:
    metadata:
      labels:
        name: hello
    spec:
      containers:
      - image: registry.cn-hangzhou.aliyuncs.com/gmg/hello:0.0.1
        name: hello
        ports:
        - containerPort: 8080
      imagePullSecrets:
        - name: regsecret
```
其中：
- imagePullSecrets 是声明拉取镜像时需要指定密钥。
- regsecret 必须和上面生成密钥的键名一致。
- image 远程仓库镜像

执行下面命令创建rc
```
$ kubectl create -f hello-rc.yaml 
replicationcontroller/hello created
```
```
$ kubectl get rc
NAME    DESIRED   CURRENT   READY   AGE
hello   1         1         1       2m16s
$ kubectl get pod
NAME          READY   STATUS    RESTARTS   AGE
hello-p96bm   1/1     Running   0          2m8s
```
## 编写hello-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: hello
  name: hello
spec:
  ports:
  - port: 8080
    nodePort: 30008
  selector:
    name: hello
  type: NodePort
```

