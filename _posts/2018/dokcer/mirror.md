# 镜像命令
## 1、获取镜像
Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜像。
```
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```
```
$ docker pull ubuntu:16.04
16.04: Pulling from library/ubuntu
bf5d46315322: Pull complete
9f13e0ac480c: Pull complete
e8988b5b3097: Pull complete
40af181810e7: Pull complete
e6f7c7e5c03e: Pull complete
Digest: sha256:147913621d9cdea08853f6ba9116c2e27a3ceffecf3b492983ae97c3d643fbbe
Status: Downloaded newer image for ubuntu:16.04
```
上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub 获取镜像。而镜像名称是 ubuntu:16.04，因此将会获取官方镜像 library/ubuntu 仓库中标签为 16.04 的镜像。 相当于
 docker pull registry.hub.docker.com/ubuntu:16.04命令。 

从下载过程中可以看到我们之前提及的分层存储的概念，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。
## 2、搜寻镜像
docker search命令可以搜索远程仓库中共享的镜像，默认搜索Docker Hub官方仓库中的镜像。
```
[root@node02 ~]# docker search java
NAME                                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
node                                         Node.js is a JavaScript-based platform for...   6428      [OK]       
tomcat                                       Apache Tomcat is an open source implementa...   2105      [OK]       
java                                         Java is a concurrent, class-based, and obj...   1872      [OK]       
openjdk                                      OpenJDK is an open-source implementation o...   1297      [OK]       
ghost                                        Ghost is a free and open source blogging p...   860       [OK]       
anapsix/alpine-java                          Oracle Java 8 (and 7) with GLIBC 2.28 over...   361                  [OK]
jetty                                        Jetty provides a Web server and javax.serv...   278       [OK]       
groovy                                       Apache Groovy is a multi-faceted language ...   58        [OK]       
ibmjava                                      Official IBM® SDK, Java™ Technology Editio...   56        [OK]  
```
返回了很多包括关键字的镜像，其中包括名称、描述、星级（标识该镜像的受欢迎度）、是否官方创建、是否自动创建。
## 3、查看镜像信息  
```
$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
ubuntu               16.04               1e0c3dd64ccd        4 weeks ago         188 MB
ubuntu               latest              1e0c3dd64ccd        4 weeks ago         127 MB
redis                alpine              501ad78535f0        3 weeks ago         21.03MB
```
列表包含了 仓库名、标签、镜像 ID、创建时间 以及 所占用的空间

镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个标签。因此，在上面的例子中，我们可以看到 ubuntu:16.04 和 ubuntu:latest 拥有相同的 ID，因为它们对应的是同一个镜像,只是别名不同而已。标签在这里起到引用和快捷方式的作用。

列出特定的某个镜像，也就是说指定仓库名和标签
```
docker image ls ubuntu:16.04
```
查看该镜像的详细信息，返回的是一个JSON格式的消息。
 ```
 docker inspect 1e0c3dd64ccd
 ```
## 4、删除镜像
更多的时候是用 短 ID 来删除镜像,一般取IMAGE ID 前3个字符以上，只要足够区分于别的镜像就可以了。
```
docker rmi 501
```
我们也可以用镜像名，也就是 <仓库名>:<标签>，来删除镜像。
```
docker rmi redis
```
强制删除镜像(不推荐使用-f参数来强制删除一个存在容器依赖的镜像，这样会造成一些遗留问题。)
```
docker rmi -f redis
```
需要删除所有仓库名为 redis 的镜像：
```
docker rmi $(docker image ls -q redis)
```
删除所有镜像 
```
docker rmi $(docker images -q)
```
## 5、镜像创建
创建镜像的方式有三种：基于已有镜像的容器创建、基于Dockerfile创建。
### 5.1 基于已有镜像的容器创建 
该方式主要用docker commit命令，其命令格式为：
```
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```
主要选项为：

- -a,  --author="" 作者信息 
- -m, --message="" 提交信息
- -p,  --pause=true 提交时暂停容器运行

对已启动的镜像进行修改，我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。
我们可以用下面的命令将容器保存为镜像：
```
docker commit \
    --a "gmg" \
    --m "update page" \
    webserver \
    nginx:v2
```
至此，我们第一次完成了定制镜像，使用的是 docker commit 命令，手动操作给旧的镜像添加了新的一层，形成新的镜像，对镜像多层存储应该有了更直观的感觉。 

### 5.2  基于Dockerfile创建

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

基本结构：   
基础镜像、维护者信息、镜像操作命令、容器启动时指令。例如：
```
# 第一行必须指定基于的容器镜像
FROM ubuntu
# 维护者信息
MAINTAINER docker_user docker_user@email.com
# 镜像的操作指令
RUN echo “deb http://archive.ubuntu.com/ubuntu/ raring main universe” >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo “\ndaemon off;” >> /etc/nginx/nginx.conf
# 容器启动时执行指令
CMD /usr/sbin/nginx
```
Dockerfile命令详解:  

    1、FROM   
    ```
    格式为 FROM <image> 或 FROM <image>:<tag>
    ```
    Dockerfile 的第一条指令必须为 FROM 指令。并且，如果在同一个 Dockerfile 中创建多个镜像时，可以使用多个 FROM 指令。

    2、MAINTAINER   
    ```
    格式为 MAINTAINER <name>，指定维护者信息。
    ```
    3、RUN   
    有两种格式，分别为：
    ```
    RUN <command>
    RUN [“executable”, “param1”, “param2”]。
    ```
    前者将在 shell 终端中运行命令，即 /bin/sh -c，后者则使用 exec 执行。指定使用其他终端可以通过第二种方式实现，例如 RUN  [“/bin/bash”, “-c”, “echo hello”]。  

    每条 RUN 指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

    4、CMD   
    ```
    CMD [“executable”, “param1”, “param2”] 使用 exec 执行，推荐方式。

    CMD command  在/bin/sh 中执行，提供给需要交互的应用。

    CMD [“param1”, “param2”] 提供给 ENTRYPOINT 的默认参数。
    ```
    指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条 CMD 命令，只有最后一条会被执行。如果用户在启动容器时指定了要运行的命令，则会覆盖掉 CMD 指定的命令。

    5、EXPOSE  

    EXPOSE 指令是声明运行时容器提供服务端口。告诉 Docker 服务，容器需要暴露的端口号，供互联系统使用。
    格式为：
    ```
    EXPOSE <port> [<port>…]
    ```
    6、ENV  
    格式有两种  
    ```
    ENV <key> <value>
    ENV <key1>=<value1> <key2>=<value2>...
    ```
    指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。例如：
    ```
    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
    ```
    7、ADD  
    格式为：
    ```
    ADD src dest
    ```
    该命令将复制指定的 src 到容器中的 dest。其中 src 可以是 Dockerfile 所在目录的一个相对路径(文件或目录)；也可以是一个 URL；还可以是一个 tar 文件(自动解压为目录)。

    8、COPY  
    格式为：  
    ```
    COPY src dest
    ```
    复制本地主机的 src (为 Dockerfile 所在目录的相对路径，文件或目录) 为容器中的 dest。目标路径不存在时，会自动创建。当使用本地目录为源目录时，推荐使用 COPY。

    9、ENTRYPOINT  
    两种格式:
    ```
    ENTRYPOINT [“executable”, “param1”, “param2”]

    ENTRYPOINT command param1 param2 (shell 中执行)
    ```
    ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。

    每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个 ENTRYPOINT 时，只有最后一个生效。

    10、VOLUME   
    格式：
    ```
    VOLUME ["<路径1>", "<路径2>"...]
    VOLUME <路径>
    ```
    创建一个可以从本地或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。 

    11、USER
    格式： 
    ```
    USER <用户名>
    ```
    USER 指令和 WORKDIR 相似，都是改变环境状态并影响以后的层。WORKDIR 是改变工作目录，USER 则是改变之后层的执行 RUN, CMD 以及 ENTRYPOINT 这类命令的身份。

    USER 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。
    ```
    RUN groupadd -r redis && useradd -r -g redis redis
    USER redis
    RUN [ "redis-server" ]
    ```
    12、WORKDIR 
    格式：
    ```
    WORKDIR <工作目录路径>
    ```
    为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如：  
    ```
    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd
    ```
    则最终路径为 /a/b/c。

    13、ONBUILD 
    格式：
    ```
    ONBUILD <其它指令>
    ```
    ONBUILD 是一个特殊的指令，它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

    Dockerfile 中的其它指令都是为了定制当前镜像而准备的，唯有 ONBUILD 是为了帮助别人定制自己而准备的。
### 参考文档
- Dockerfie 官方文档：https://docs.docker.com/engine/reference/builder/

- Dockerfile 最佳实践文档：https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/

- Docker 官方镜像 Dockerfile：https://github.com/docker-library/docs

### 创建镜像
编写完成 Dockerfile 之后，可以通过 docker build 命令来创建镜像。基本的格式为 docker build [选项] 路径，该命令将读取指定路径下(包括子目录)的 Dockerfile ，并将该路径下所有内容发送给 docker 服务端，由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。

另外，可以通过 .dockerignore 文件来让 docker 忽略路径下的目录和文件。 要指定镜像的标签信息，可以通过 -t 选项来实现。
例如，指定 Dockerfile 所在路径为 /tmp/docker_builder/，并且希望生成镜像标签为 build_repo/first_image，可以使用下面的命令：
```
$ sudo docker build -t build_repo/first_image /tmp/docker_builder/
# 如果不想使用上次 build 过程中产生的 cache 可以添加 --no-cache 选项
$ sudo docker build --no-cache -t build_repo/first_image /tmp/docker_builder
```
## 6、存出和载入镜像
- 存出镜像
如果想存出文件到本地文件，可以用docker save命令。例如将本地的镜像ubuntu:14.04镜像为文件ubuntu_14.04.tar
```
docker save -o ubuntu_14.04.tar ubuntu:14.04
```
- 载入镜像
可以使用docker load 从存入的本地文件中导入到本地镜像库。例如从文件ubuntu_14.04.tar导入镜像到本地镜像列表。
```
docker load --input ubuntu_14.04.tar
```
## 7、上传镜像 
使用docker push命令上传镜像到仓库。默认上传到DockerHub官方仓库(需要登录)。命令格式：
```
docker push NAME[:TAG]
```
例如用户user上传本地的test:latest镜像，可以添加新的标签user/test:latest,启用docker push命令上传镜像。
```
docker tag test:latest user/test:latest
docker push user/test:latest
``` 
第一次使用时，会提示输入登录信息。




