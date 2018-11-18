---
layout:     post
title:      "Docker三剑客之docker-swarm"
subtitle:   " \"docker-swarm\""
date:       2018-11-12 14:24:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> “docker study ”

## 简介

Swarm是Docker官方提供的一款集群管理工具，其主要作用是把若干台Docker主机抽象为一个整体，并且通过一个入口统一管理这些Docker主机上的各种Docker资源。Swarm和Kubernetes比较类似，但是更加轻，具有的功能也较kubernetes更少一些。

swarm集群提供给用户管理集群内所有容器的操作接口与使用一台Docker主机基本相同。

## Swarm一些概念说明

### 1、节点

运行 Docker 的主机可以主动初始化一个 Swarm 集群或者加入一个已存在的 Swarm 集群，这样这个运行 Docker 的主机就成为一个 Swarm 集群的节点 (node) 。

节点分为管理 (manager) 节点和工作 (worker) 节点。

管理节点用于 Swarm 集群的管理，docker swarm 命令基本只能在管理节点执行（节点退出集群命令 docker swarm leave 可以在工作节点执行）。一个 Swarm 集群可以有多个管理节点，但只有一个管理节点可以成为 leader，leader 通过 raft 协议实现。

工作节点是任务执行节点，管理节点将服务 (service) 下发至工作节点执行。管理节点默认也作为工作节点。你也可以通过配置让服务只运行在管理节点。

来自 Docker 官网的这张图片形象的展示了集群中管理节点与工作节点的关系。
![](https://github.com/gmg0829/Img/blob/master/dockerImg/swarm-diagram.png?raw=true)

### 2、服务和任务

- 任务 （Task）是 Swarm 中的最小的调度单位，目前来说就是一个单一的容器。

- 服务 （Services） 是指一组任务的集合，服务定义了任务的属性。服务有两种模式：

- replicated services 按照一定规则在各个工作节点上运行指定个数的任务。

- global services 每个工作节点上运行一个任务

两种模式通过 docker service create 的 --mode 参数指定。

来自 Docker 官网的这张图片形象的展示了容器、任务、服务的关系。
![](https://github.com/gmg0829/Img/blob/master/dockerImg/services-diagram.png?raw=true)
## Swarm 调度策略
Swarm在scheduler节点（leader 节点）运行容器的时候，会根据指定的策略来计算最适合运行容器的节点，目前支持的策略有：spread, binpack, random.

1）Random

顾名思义，就是随机选择一个 Node 来运行容器，一般用作调试用，spread 和 binpack 策略会根据各个节点的可用的 CPU, RAM 以及正在运行的容器的数量来计算应该运行容器的节点。

2）Spread

在同等条件下，Spread 策略会选择运行容器最少的那台节点来运行新的容器，binpack 策略会选择运行容器最集中的那台机器来运行新的节点。使用 Spread 策略会使得容器会均衡的分布在集群中的各个节点上运行，一旦一个节点挂掉了只会损失少部分的容器。

3）Binpack

Binpack 策略最大化的避免容器碎片化，就是说 binpack 策略尽可能的把还未使用的节点留给需要更大空间的容器运行，尽可能的把容器运行在一个节点上面。
## Swarm命令行说明
```
docker swarm：集群管理
init          #初始化集群
join          #将节点加入集群
join-token    #管理加入令牌
leave         #从集群中删除某个节点，强制删除加参数--force 
update        #更新集群
unlock        #解锁集群

docker node：节点管理，
demote      #将集群中一个或多个节点降级
inspect     #显示一个或多个节点的详细信息
ls          #列出集群中的节点
promote     #将一个或多个节点提升为管理节点
rm          #从集群中删除停止的节点，--force强制删除参数
ps          #列出一个或多个节点上运行的任务
update      #更新节点

docker service：服务管理，
create      #创建一个新的服务
inspect     #列出一个或多个服务的详细信息
ps          #列出一个或多个服务中的任务信息
ls          #列出服务
rm          #删除一个或多个服务
scale       #扩展一个或多个服务
update      #更新服务
```

## 安装布署swarm集群服务


- manager：192.168.124.129 

- node：192.168.124.132

1、修改主机名，配置hosts文件
```
[root@manager ~]# cat >>/etc/hosts<<EOF
192.168.124.129  manager
192.168.124.132 node1
EOF
[root@manager ~]# tail -3 /etc/hosts
192.168.124.129  manager
192.168.124.132 node1
#node1 node2配置同上一致即可
```
2、配置docker

编辑docker文件：/usr/lib/systemd/system/docker.service

```
vim /usr/lib/systemd/system/docker.service
```
修改ExecStart行为下面内容
```
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock \
```

重新加载docker配置
```
systemctl daemon-reload // 1，加载docker守护线程
systemctl restart docker // 2，重启docker
```
所有节点加上上面标记的部分，开启2375端口

3、所有节点下载swarm镜像文件
```
$ docker pull swarm
```

4、创建swarm并初始化
```
$ docker swarm init --advertise-addr 192.168.124.129
Swarm initialized: current node (4c70fdpk3ip083rg7nnuk5stw) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5jubbodkfxlp96pg2w9sihqdvtruhkdje3bls1nb9ujiig0t3n-1d9l7t4bssnglbuemx9v06r3x 192.168.124.129:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
#执行上面的命令后，当前的服务器就加入到swarm集群中，同时会产生一个唯一的token值，其它节点加入集群时需要用到这个token。
#--advertise-addr 表示swarm集群中其它节点使用后面的IP地址与管理节点通讯，上面也提示了其它节点如何加入集群的命令。
```
5、将node1加入到集群中

在node1下执行
```
$ docker swarm join --token SWMTKN-1-5jubbodkfxlp96pg2w9sihqdvtruhkdje3bls1nb9ujiig0t3n-1d9l7t4bssnglbuemx9v06r3x 192.168.124.129:2377
This node joined a swarm as a worker.
```
6、管理节点查看集群节点状态
```
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
4c70fdpk3ip083rg7nnuk5stw *   manager             Ready               Active              Leader              18.09.0
5viloj6u950gkilsl689aonyn     node1               Ready               Active              Reachable           18.09.0

#swarm集群中node的AVAILABILITY状态有两种：Active、drain。其中actice状态的节点可以接受管理节点的任务指派；drain状态的节点会结束任务，也不会接受管理节点的任务指派，节点处于下线状态。
```
7、Swarm 的Web管理

```
$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```
浏览器访问

![](https://github.com/gmg0829/Img/blob/master/dockerImg/portainer.png?raw=true)

## docker-swarm布署服务
1、布署服务前创建于个用于集群内不同主机之间容器通信的网络
```
$ docker network create -d overlay dockernet
5lhuzjkx36j40na59gmu400op
```
2、创建服务(nginx为例)
```
$docker service create --replicas 1 --network dockernet --name nginx-cluster -p 80:80 nginx
klpwtncehp0vkh0d1gqqvicf6
#--replicas 指定副本数量
$docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
klpwtncehp0v        nginx-cluster       replicated          1/1                 nginx:latest        *:80->80/tcp
$  docker service ps nginx-cluster
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
j1y2blg3pa7j        nginx-cluster.1     nginx:latest        manager             Running             Running 53 seconds ago   

$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED              STATUS              PORTS                    NAMES
9a2e361f535b        nginx:latest          "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp                   nginx-cluster.1.j1y2blg3pa7j9mtg54e2csr7f
```
3、在线动态扩容服务

```
docker service scale nginx-cluster=5
nginx-cluster scaled to 5

$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
klpwtncehp0v        nginx-cluster       replicated          5/5                 nginx:latest        *:80->80/tcp

$ docker service ps nginx-cluster
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
j1y2blg3pa7j        nginx-cluster.1     nginx:latest        manager             Running             Running 3 minutes ago                            
y5ib98y3rr5i        nginx-cluster.2     nginx:latest        node1               Running             Running 35 seconds ago                           
wpiydfv0j2w5        nginx-cluster.3     nginx:latest        node1               Running             Running 35 seconds ago                           
ibl73haatpvc        nginx-cluster.4     nginx:latest        manager             Running             Running about a minute ago                       
a6oa1h83ba3c        nginx-cluster.5     nginx:latest        node1               Running             Running 35 seconds ago 
#从输出结果可以看出已经将服务动态扩容至5个，也就是5个容器运行着相同的服务
```
4、节点故障
```
$ docker service ps nginx-cluster
ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
j1y2blg3pa7j        nginx-cluster.1       nginx:latest        manager             Running             Running 45 minutes ago                        
ugx002mtbfmp        nginx-cluster.2       nginx:latest        manager             Running             Running 7 seconds ago                         
y5ib98y3rr5i         \_ nginx-cluster.2   nginx:latest        node1               Shutdown            Shutdown 11 seconds ago                       
q1f5jhhx7kcy        nginx-cluster.3       nginx:latest        manager             Running             Running 7 seconds ago                         
wpiydfv0j2w5         \_ nginx-cluster.3   nginx:latest        node1               Shutdown            Shutdown 11 seconds ago                       
ibl73haatpvc        nginx-cluster.4       nginx:latest        manager             Running             Running 43 minutes ago                        
a6f7zpclrpm4        nginx-cluster.5       nginx:latest        manager             Running             Running 7 seconds ago                         
a6oa1h83ba3c         \_ nginx-cluster.5   nginx:latest        node1               Shutdown            Shutdown 11 seconds ago 
#如果集群中节点发生故障，会从swarm集群中被T除，然后利用自身的负载均衡及调度功能，将服务调度到其它节点上 
```
5、其它常用命令介绍

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
klpwtncehp0v        nginx-cluster       replicated          5/5                 nginx:latest        *:80->80/tcp
$ docker service update --replicas 2 nginx-cluster
nginx-cluster
#将服务缩减到2个
$ docker service  ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
klpwtncehp0v        nginx-cluster       replicated          2/2                 nginx:latest        *:80->80/tcp
$ docker service update --image nginx:new nginx-cluster
#更新服务的镜像版本

$ docker rm nginx-cluster
#将所有节点上的所有容器全部删除，任务也将全部删除
```

参考地址链接：
- [容器技术｜Docker三剑客之docker-swarm](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247487131&idx=1&sn=ab26a01288355ca29fe16a50346ec8cd&chksm=e91b6b87de6ce291af3ad05d9842b6ad45e62d9f9d86d7474a3f38f7ba8cc4feea9817d22060&scene=21#wechat_redirect)

- [Docker入门到实践-Swarm ](https://yeasy.gitbooks.io/docker_practice/swarm_mode/overview.html)