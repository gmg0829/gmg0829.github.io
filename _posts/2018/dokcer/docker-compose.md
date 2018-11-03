# 简介
Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。

通过之前的介绍，我们知道使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

Compose 中有两个重要的概念：

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

使用微服务架构的系统一般包含若干个微服务，每个微服务一般部署多个实例。如果每个服务都要手动启停，那么效率低，维护量大。

# 命令
## docker-compose.yml常用命令
docker-compose.yml格式为：
```
version: "3"
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```
注意每个服务都必须通过 image 指令指定镜像或 build 指令（需要 Dockerfile）等来自动构建生成镜像。
- build

    指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。
    ```
    version: '3'
    services:
    webapp:
        build: ./dir
    ```
    你也可以使用 context 指令指定 Dockerfile 所在文件夹的路径。
    使用 dockerfile 指令指定 Dockerfile 文件名。
    ```
    version: '3'
    services:
    webapp:
        build:
        context: ./dir
        dockerfile: Dockerfile-alternate
    ```
- command

    覆盖容器启动后默认执行的命令。
    ```
    command: echo "hello world"
    ```
- dns

    自定义 DNS 服务器。可以是一个值，也可以是一个列表。
    ```
    dns: 8.8.8.8

    dns:
    - 8.8.8.8
    - 114.114.114.114
    ```
- dns_search

    配置 DNS 搜索域。可以是一个值，也可以是一个列表。
    ```
    dns_search: example.com

    dns_search:
    - domain1.example.com
    - domain2.example.com
    ```
- expose

    暴露端口，但不映射到宿主机，只被连接的服务访问。
    ```
    expose:
    - "3000"
    - "8000"
    ```
- image

    指定为镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。
    ```
    image: ubuntu
    image: orchardup/postgresql
    image: a4bc65fd
    ```
- links

    连接其他服务的容器，可以指定服务名称。
    ```
    web:
    links:
        -db
        -redis
    ```
- ports

    暴露端口信息。
    使用宿主端口：容器端口格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。
    ```
    ports:
    - "3000"
    - "8000:8000"
    - "49100:22"
    - "127.0.0.1:8001:8001"
    ```
- volumes

    数据卷所挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式。
    加载本地目录下的配置文件到容器目标地址下。
    ```
    volumes:
    - /var/lib/mysql
    - cache/:/tmp/cache
    - ~/configs:/etc/configs/:ro
    ```
- environment
    设置环境变量。你可以使用数组或字典两种格式。

    只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。  
    ```
    environment:
    RACK_ENV: development
    SESSION_SECRET:

    environment:
    - RACK_ENV=development
    - SESSION_SECRET
    ```  
 - networks   

    配置容器连接的网络。
    ```
    version: "3"
    services:
    some-service:
        networks:
        - some-network
        - other-network
    ```
 - depends_on

    解决容器的依赖、启动先后的问题。以下例子中会先启动 redis db 再启动 web
    ```
    version: '3'
    services:
        web:
         build: .
         depends_on:
            - db
            - redis
        redis:
          image: redis
        db:
          image: postgres
    ```
    注意：web 服务不会等待 redis db 「完全启动」之后才启动。
 - container_name

    指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式。
    ```
    container_name: docker-web-container
    ```
    注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。
 - restart

     restart: always 表示如果服务启动不成功会一直尝试。   
 - working_dir   
     
     指定容器中工作目录。
     ```
     working_dir: /code
     ```

## docker-compose常用命令
docker-compose 命令的基本的使用格式是
```
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```
命令选项:
```
-f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。

-p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。

--x-networking 使用 Docker 的可拔插网络后端特性

--x-network-driver DRIVER 指定网络后端的驱动，默认为 bridge

--verbose 输出更多调试信息。

-v, --version 打印版本并退出。
```
- build

    格式为 docker-compose build [options] [SERVICE...]。

    构建（重新构建）项目中的服务容器。

    服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。

    可以随时在项目目录下运行 docker-compose build 来重新构建服务。

    选项包括：
    ```
    --force-rm 删除构建过程中的临时容器。

    --no-cache 构建镜像过程中不使用 cache（这将加长构建过程）。

    --pull 始终尝试通过 pull 来获取更新版本的镜像。
    ```
- config

    验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。

- down

    此命令将会停止 up 命令所启动的容器，并移除网络

- exec

    进入指定的容器。

- help

    获得一个命令的帮助。

- images

    列出 Compose 文件中包含的镜像。
- kill

    格式为 docker-compose kill [options] [SERVICE...]。

    通过发送 SIGKILL 信号来强制停止服务容器。

    支持通过 -s 参数来指定发送的信号，例如通过如下指令发送 SIGINT 信号。

    ```
    $ docker-compose kill -s SIGINT
    ```
- logs

    格式为 docker-compose logs [options] [SERVICE...]。

    查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 --no-color 来关闭颜色。

    该命令在调试问题的时候十分有用。    
- pause

    格式为 docker-compose pause [SERVICE...]。

    暂停一个服务容器。
- ps

    格式为 docker-compose ps [options] [SERVICE...]。

    列出项目中目前的所有容器。

    选项：

    -q 只打印容器的 ID 信息。
- pull

    格式为 docker-compose pull [options] [SERVICE...]。

    拉取服务依赖的镜像。

    选项：

    --ignore-pull-failures 忽略拉取镜像过程中的错误。
    push
    推送服务依赖的镜像到 Docker 镜像仓库。
- restart

    格式为 docker-compose restart [options] [SERVICE...]。

    重启项目中的服务。

    选项：

    -t, --timeout TIMEOUT 指定重启前停止容器的超时（默认为 10 秒）。
- rm

    格式为 docker-compose rm [options] [SERVICE...]。

    删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器。

    选项：

    -f, --force 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。

    -v 删除容器所挂载的数据卷。

- start

    格式为 docker-compose start [SERVICE...]。

    启动已经存在的服务容器。
- stop

    格式为 docker-compose stop [options] [SERVICE...]。

    停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器。

    选项：

    -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。    
- up

    格式为 docker-compose up [options] [SERVICE...]。

    该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

    链接的服务都将会被自动启动，除非已经处于运行状态。

    可以说，大部分时候都可以直接通过该命令来启动一个项目。

    默认情况，docker-compose up 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。

    当通过 Ctrl-C 停止命令时，所有容器将会停止。

    如果使用 docker-compose up -d，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。

    选项：

    -d 在后台运行服务容器。

    --no-color 不使用颜色来区分不同的服务的控制台输出。

    --no-deps 不启动服务所链接的容器。

    --force-recreate 强制重新创建容器，不能与 --no-recreate 同时使用。

    --no-recreate 如果容器已经存在了，则不重新创建，不能与 --force-recreate 同时使用。

    --no-build 不自动构建缺失的服务镜像。

    -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
- version

    格式为 docker-compose version。

    打印版本信息。
- port

    格式为 docker-compose port [options] SERVICE PRIVATE_PORT。

    打印某个容器端口所映射的公共端口。
- scale

    格式为 docker-compose scale [options] [SERVICE=NUM...]。
    设置指定服务运行的容器个数。    

    通过 service=num 的参数来设置数量。例如：

    $ docker-compose scale web=3 db=2

    将启动 3 个容器运行 web 服务，2 个容器运行 db 服务。

## 使用Docker Compose编排Spring Cloud 服务。
编排时用到的微服务项目：

    discovery ：注册中心

    server：服务提供者

    client:服务消费者

    zuul:网关


1、由于Docker 默认的网络模式是bridge,各个容器的IP都不同。因此在Eureka Server配置一个主机名(discovery),让各个微服务使用主机名访问Eureka Server：
```
eureka:
  instance:
    hostname: discovery
```
然后将如下配置
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
替换为：
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://discovery:8761/eureka/
```
2、编写构建镜像脚本

bash脚本的用途是分别执行几个module下的Dockerfile文件，分别打成镜像，不用一个个手工执行，文件如下：

buildDockerImage.sh:
```
#!/usr/bin/env bash
set -eo pipefail
modules=( discovery server client zuul )
for module in "${modules[@]}"; do
    docker build -t "microservice/${module}:latest" ${module}
done
```
tree查看microservice文件夹下模块目录：
```
 $ tree
.
├── buildDockerImage.sh
├── client
│   ├── client-0.0.1-SNAPSHOT.jar
│   └── Dockerfile
├── discovery
│   ├── Dockerfile
│   └── eureka-server-0.0.1-SNAPSHOT.jar
├── docker-compose.yml
├── server
│   ├── Dockerfile
│   └── server-0.0.1-SNAPSHOT.jar
└── zuul
    ├── Dockerfile
    └── zuul-0.0.1-SNAPSHOT.jar

```
modules为各个模块目录，模块目录下存放该模块的jar包和Dockerfile文件。

执行sh buildDockerImage.sh命令，执行完，使用docker images 查看镜像。
```
$ docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
microservice/zuul            latest              d0fcb94e203e        30 seconds ago      204MB
microservice/client          latest              a3ff8e192ee6        33 seconds ago      200MB
microservice/server          latest              8f04580f6063        35 seconds ago      200MB
microservice/discovery       latest              d8b9347a8229        38 seconds ago      204MB
frolvlad/alpine-oraclejdk8   slim                3ee5e1ce00fc        9 days ago          164MB
```

3、编写docker-compose-yml
```
version: "3"
services:
  discovery:
    image: microservice/discovery
    hostname: discovery
    ports:
    - "8761:8761"
  server:
    image: microservice/server
    ports:
    - "9000:9000"
  client:
    image: microservice/client
    ports:
    - "9001:9001"
  zuul:
    image: microservice/zuul
    ports:
    - "9002:9002"
```
执行以下命令启动项目。
```
[root@node02 microservice]# docker-compose up -d
Creating microservice_discovery_1 ... 
Creating microservice_discovery_1 ... done
Creating microservice_client_1 ... 
Creating microservice_server_1 ... 
Creating microservice_client_1
Creating microservice_client_1 ... done
Creating microservice_zuul_1 ... 
Creating microservice_zuul_1 ... done
```
查看镜像：
```
$ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                    NAMES
ef16f3cccfdb        microservice/zuul        "java -Djava.securit…"   38 minutes ago      Up 38 minutes       0.0.0.0:9002->9002/tcp   microservice_zuul_1
657b28a5f17b        microservice/server      "java -Djava.securit…"   38 minutes ago      Up 38 minutes       0.0.0.0:9000->9000/tcp   microservice_server_1
e048e4d69f49        microservice/client      "java -Djava.securit…"   38 minutes ago      Up 38 minutes       0.0.0.0:9001->9001/tcp   microservice_client_1
66f9c53ec2b5        microservice/discovery   "java -Djava.securit…"   38 minutes ago      Up 38 minutes       0.0.0.0:8761->8761/tcp   microservice_discovery_1

```
访问注册中心：

![](https://github.com/gmg0829/Img/blob/master/dockerImg/%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83.png?raw=true)

访问网关服务

zuul->client->server
```
http://192.168.124.131:9002/api-a/getUser?name=gmg

hi gmg,i am from 
```

## 编排高可用的Spring cloud微服务集群及动态伸缩

1、将所有的服务的注册中心改为
```
eureka:
  client:
    serviceUrl:
      defaultZone : http://peer1:8761/eureka/,http://peer2:8762/eureka/
```
2、docker-compose-yml修改为
```
version: "3"
services:
  peer1:
    image: microservice/discovery
    ports:
    - "8761:8761"
    environment:
    - spring.profiles.active=peer1
  peer2:
    image: microservice/discovery
    hostname: peer2
    ports:
    - "8762:8762"
    environment:
    - spring.profiles.active=peer2
  server:
    image: microservice/server
    ports:
    - "9000:9000"
  client:
    image: microservice/client
    ports:
    - "9001:9001"
  zuul:
    image: microservice/zuul
    ports:
    - "9002:9002"
```
3、构建镜像

执行sh buildDockerImage.sh命令构建镜像。

4、创建容器

执行 docker-compose up -d  
查看镜像：
```
docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED              STATUS              PORTS                    NAMES
8dfff5cf705f        microservice/discovery   "java -Djava.securit…"   About a minute ago   Up About a minute   0.0.0.0:8762->8762/tcp   microserviceha_peer2_1
a89594944490        microservice/client      "java -Djava.securit…"   About a minute ago   Up About a minute   0.0.0.0:9001->9001/tcp   microserviceha_client_1
29e941b18623        microservice/zuul        "java -Djava.securit…"   About a minute ago   Up About a minute   0.0.0.0:9002->9002/tcp   microserviceha_zuul_1
18580a21ab2f        microservice/discovery   "java -Djava.securit…"   About a minute ago   Up About a minute   0.0.0.0:8761->8761/tcp   microserviceha_peer1_1
1bcaaadac4a4        microservice/server      "java -Djava.securit…"   About a minute ago   Up About a minute   0.0.0.0:9000->9000/tcp   microserviceha_server_1

```

5、查看注册中心：
![](https://github.com/gmg0829/Img/blob/master/dockerImg/%E9%AB%98%E5%8F%AF%E7%94%A8%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83.png?raw=true)

