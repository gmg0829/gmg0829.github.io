# 仓库
## Docker Hub

目前 Docker 官方维护了一个公共仓库 Docker Hub，其中已经包括了数量超过 15,000 的镜像。大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

1、注册

你可以在 https://cloud.docker.com 免费注册一个 Docker 账号。

2、登录

可以通过执行 docker login 命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。
你可以通过 docker logout 退出登录。

3、拉取镜像

你可以通过 docker search 命令来查找官方仓库中的镜像，并利用 docker pull 命令来将它下载到本地。

4、推送镜像

用户也可以在登录后通过 docker push 命令来将自己的镜像推送到 Docker Hub。

## 搭建私有仓库   
### Harbor 搭建
1、下载Harbor安装文件 

从 github harbor 官网 [release ](https://github.com/goharbor/harbor/releases) 页面下载指定版本的安装包。
```
1、在线安装包
    $ wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.0.tgz
    $ tar xvfharbor-offline-installer-v1.6.0.tgz
2、离线安装包
    $ wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-online-installer-v1.6.0.tgz
    $ tar xvf harbor-online-installer-v1.6.0.tgz
```
2、配置Harbor 

解压缩之后，目录下回生成harbor.conf文件，该文件就是Harbor的配置文件。
```
## Configuration file of Harbor

# hostname设置访问地址，可以使用ip、域名，不可以设置为127.0.0.1或localhost
hostname = docker.bksx.com

# 访问协议，默认是http，也可以设置https，如果设置https，则nginx ssl需要设置on
ui_url_protocol = http

# mysql数据库root用户默认密码root123，实际使用时修改下
db_password = root123

max_job_workers = 3 
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA

# 邮件设置，发送重置密码邮件时使用
email_identity = 
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false

# 启动Harbor后，管理员UI登录的密码，默认是Harbor12345
harbor_admin_password = 1qaz@WSX

# 认证方式，这里支持多种认证方式，如LADP、本次存储、数据库认证。默认是db_auth，mysql数据库认证
auth_mode = db_auth

# LDAP认证时配置项
#ldap_url = ldaps://ldap.mydomain.com
#ldap_searchdn = uid=searchuser,ou=people,dc=mydomain,dc=com
#ldap_search_pwd = password
#ldap_basedn = ou=people,dc=mydomain,dc=com
#ldap_filter = (objectClass=person)
#ldap_uid = uid 
#ldap_scope = 3 
#ldap_timeout = 5

# 是否开启自注册
self_registration = on

# Token有效时间，默认30分钟
token_expiration = 30

# 用户创建项目权限控制，默认是everyone（所有人），也可以设置为adminonly（只能管理员）
project_creation_restriction = everyone

verify_remote_cert = on
```
3、启动 Harbor 

修改完配置文件后，在的当前目录执行./install.sh，Harbor服务就会根据当期目录下的docker-compose.yml开始下载依赖的镜像，检测并按照顺序依次启动。

启动完成后，我们访问刚设置的hostname即可。默认是80端口，如果端口占用，我们可以去修改docker-compose.yml文件中，对应服务的端口映射。

输入用户名admin，默认密码 Harbor12345 （或已修改密码）登录系统。   

![首页](https://github.com/gmg0829/Img/blob/master/dockerImg/harbor.png?raw=true)

我们可以看到系统各个模块如下：
- 项目：新增/删除项目，查看镜像仓库，给项目添加成员、查看操作日志、复制项目等
- 日志：仓库各个镜像create、push、pull等操作日志
- 系统管理 
  - 用户管理：新增/删除用户、设置管理员等
  - 复制管理：新增/删除从库目标、新建/删除/- 启停复制规则等
  - 配置管理：认证模式、复制、邮箱设置、系统设置等
- 其他设置 
  - 用户设置：修改用户名、邮箱、名称信息
  - 修改密码：修改用户密码

新建项目完毕后，我们就可以用admin账户提交本地镜像到Harbor仓库了。例如我们提交本地hello-world镜像
```
1、admin登录
$ docker login 192.168.124.131
Username: admin
Password: 
Login Succeeded

2、给镜像打tag
$ docker tag hello-world 192.168.124.131/docker/hello-world:latest
$ docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
192.168.124.131/docker/hello-world   latest              4ab4c602aa5e        7 weeks ago         1.84kB
hello-world                          latest              4ab4c602aa5e        7 weeks ago         1.84kB

3、push到仓库
$ docker push 192.168.124.131/docker/hello-world
The push refers to repository [192.168.124.131/docker/hello-world]
428c97da766c: Pushed 
latest: digest: sha256:1a6fd470b9ce10849be79e99529a88371dff60c60aab424c077007f6979b4812 size: 524

```
查看远程镜像仓库：
![herllo-world](https://github.com/gmg0829/Img/blob/master/dockerImg/docker-helloworld.png?raw=true)

注意：

配置并启动Harbor之后，本地执行登录操作，报错：
```
docker login 192.168.124.131
Username: admin
Password:
Error response from daemon: Get https://192.168.124.131/v2/users/: dial tcp  192.168.124.131:443: getsockopt: connection refused
```
解决方法：

修改docker启动文件，设置信任的主机与端口：
```
#vim /usr/lib/systemd/system/docker.service  修改如下一行
ExecStart=/usr/bin/dockerd --insecure-registry=192.168.124.131
```
重新启动docker：
```
systemctl daemon-reload
systemctl restart docker.service
```
