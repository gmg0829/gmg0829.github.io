# 容器
1、新建容器 
docker create命令创建一个容器，例如：
```
 docker create -it hello-world
```
使用docker create命令新建的容器处于停止状态。可以使用docker start命令启动它。

2、新建并启动容器
docker run等价于先执行docker create,后执行docker start。
``` 
1、 docker run -d --name nginx nginx:latest  
后台启动并运行一个名为nginx的容器，运行前它会自动去docker镜像站点下载最新的镜像文件  
2、 docker run -d -p 80:80 nginx:latest  
后台启动并运名为nginx的容器，然后将容器的80端口映射到物理机的80端口
3、docker run -d -v /docker/data:/docker/data -p 80:80 nginx:latest  
后台启动并运名为nginx的容器，然后将容器的80端口映射到物理机的80端口,并且将物理机的/docker/data目录映射到容器的/docker/data
3、docker run -it  nginx:latest /bin/bash  
以交互式模式运行容器，然后在容器内执行/bin/bash命令
 ```
3、终止容器  
使用docker stop来停止运行中的容器。
```
docker stop 607fbd4ef4fb
```
查看处于终止状态的容器的ID信息。
```
docker ps -a -q
```
终止所有容器：
```
docker stop $(docker ps -aq)
```
4、进入容器   

某些时候需要进入容器进行操作，包括使用 docker attach 命令或 docker exec 命令，推荐大家使用 docker exec 命令。
- docker attach
```
docker attach 607fbd4ef4fb
```
注意： 如果从这个 stdin 中 exit，会导致容器的停止。
- docker exec
```
docker exec -it 607f bash
```
只用 -i 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 -i -t 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 docker exec 的原因。

5、删除容器  
可以使用 docker container rm 来删除一个处于终止状态的容器。例如
```
 docker rm  607f
```
如果要删除一个运行中的容器，可以添加 -f 参数。Docker 会发送 SIGKILL 信号给容器。

删除所有容器
```
docker rm $(docker ps -aq)
```
6、导入和导出容器   

- 导出容器
```
docker export 7691a814370e > ubuntu.tar
```
- 导入容器
```
cat ubuntu.tar | docker import - test/ubuntu:v1.0
```
