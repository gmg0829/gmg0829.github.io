# Spring Cloud和Docker
## 基于Dockerfile构建镜像
1、执行如下命令打jar包

```
mvn clean package
```
2、创建Dockerfile文件
```
#基于那个镜像
FROM frolvlad/alpine-oraclejdk8:slim
#将本地文件挂载到当前容器
VOLUME /tmp
#复制文件到容器
ADD eureka-server-0.0.1-SNAPSHOT.jar app.jar
#声明需要暴露的端口
EXPOSE 8761
#配置容器启动后执行的命令
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
3、使用docker build构建镜像

格式为：
```
docker build -t 仓库名称/镜像名称:标签 Dockerfile的相对位置
```
```
$ docker build -t 192.168.124.131/eureka-server:0.0.1 .
Sending build context to Docker daemon  40.28MB
Step 1/5 : FROM frolvlad/alpine-oraclejdk8:slim
slim: Pulling from frolvlad/alpine-oraclejdk8
4fe2ade4980c: Pull complete 
a0290d5a7317: Pull complete 
1d8a043e07b3: Pull complete 
Digest: sha256:a51161fd28d21add32482e3852c6fa2344ff64bcc6472aaccf02a047cfcc1171
Status: Downloaded newer image for frolvlad/alpine-oraclejdk8:slim
 ---> 3ee5e1ce00fc
Step 2/5 : VOLUME /tmp
 ---> Running in a72a05d4ac7d
Removing intermediate container a72a05d4ac7d
 ---> 8dd5f25ea1ce
Step 3/5 : ADD eureka-server-0.0.1-SNAPSHOT.jar app.jar
 ---> c6ce6f3376d6
Step 4/5 : ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
 ---> Running in 2d84ce0c22b5
Removing intermediate container 2d84ce0c22b5
 ---> 572384c27261
Step 5/5 : EXPOSE 8761
 ---> Running in 1ba0bdbdc241
Removing intermediate container 1ba0bdbdc241
 ---> cb1d11ff17f2
Successfully built cb1d11ff17f2
Successfully tagged 192.168.124.131/eureka-server:0.0.1

```
4、启动镜像

```
docker run -d -p 8761:8761 192.168.124.131/eureka-server:0.0.1
```
访问 http://192.168.124.131:8761/ 可以正常访问Eureka Server首页。

5、推送镜像
### 推送到Docker Hub
gmg0829为Docker Hub用户名。
```
$ docker push gmg0829/eureka-server
The push refers to repository [docker.io/gmg0829/eureka-server]
6165efb54d3b: Pushed 
6404ee1467b0: Mounted from frolvlad/alpine-oraclejdk8 
75f9e8b2e367: Pushed 
df64d3292fd6: Pushed 
0.0.1: digest: sha256:21e61ddbcb4752a3577e460b246db0f78943c69b272883bb1cc0d43ca806ee9f size: 1163
```
查看远程仓库
![](https://github.com/gmg0829/Img/blob/master/dockerImg/hub-eureka.png?raw=true)

### 推送到Harbor
执行命令 

```
$ docker push 192.168.124.131/docker/eureka-server:0.0.1
The push refers to repository [192.168.124.131/docker/eureka-server]
6165efb54d3b: Pushed 
6404ee1467b0: Pushed 
75f9e8b2e367: Pushed 
df64d3292fd6: Pushed 
0.0.1: digest: sha256:21e61ddbcb4752a3577e460b246db0f78943c69b272883bb1cc0d43ca806ee9f size: 1163
```
查看远程仓库
![](https://github.com/gmg0829/Img/blob/master/dockerImg/harbor-eureka.png?raw=true)

## maven插件构建镜像及推送

1、修改Maven全局配置文件setting.xml，在其中添加一下内容，配置Docker Hub的用户信息。
```
<server>
    <id>docker-hub</id>
    <username>gmg0829</username>
    <password>xxxx</password>
    <configuration>
    <email>xxxx@163.com</email>
    </configuration>
</server>
```
2、修改pom.xml
```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.4.3</version>
    <configuration>
        <imageName>gmg0829/${project.artifactId}:0.0.1</imageName>
        <dockerDirectory>src/main/docker</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
        <serverId>docker-hub</serverId>
    </configuration>
</plugin>
```
- imageName 镜像名称 gmg0829是仓库名称，${project.artifactId}镜像名称，0.0.1标签名称
- dockerDirectory Dockerfile所在的路径

- resources.resource.directory 指定需要复制的根目录

- resources.resource.include 需要复制的文件 

- serverId与第一步中的id相同。

3、执行一下命令
```
 mvn clean package docker:build -DpushImage
```
发现镜像已推送到docker hub。

![](https://github.com/gmg0829/Img/blob/master/dockerImg/hub-eureka.png?raw=true)


