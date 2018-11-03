# 安装、更新、卸载

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费），EE 即企业版，强调安全，付费使用。Docker支持在主流的操作系统平台上使用，包括Ubuntu、Centos、Windows、MacOS系统等。  
Docker CE 分为 stable, test, 和 nightly 三个更新频道。每六个月发布一个 stable 版本 (18.09, 19.03, 19.09...)。

## 1、 Ubuntu
### 1.1 安装、更新

>警告：切勿在没有配置 Docker APT 源的情况下直接使用 apt 命令安装 Docker.

### 准备工作

#### 系统要求

Docker CE 支持以下版本的 [Ubuntu](https://www.ubuntu.com/server) 操作系统：

* Bionic 18.04 (LTS)
* Xenial 16.04 (LTS)
* Trusty 14.04 (LTS)

Docker CE 可以安装在 64 位的 x86 平台或 ARM 平台上。Ubuntu 发行版中，LTS（Long-Term-Support）长期支持版本，会获得 5 年的升级维护支持，这样的版本会更稳定，因此在生产环境中推荐使用 LTS 版本,当前最新的 LTS 版本为 Ubuntu 18.04。

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```bash
$ sudo apt-get remove docker \
               docker-engine \
               docker.io
```

#### Ubuntu 14.04 可选内核模块

从 Ubuntu 14.04 开始，一部分内核模块移到了可选内核模块包 (`linux-image-extra-*`) ，以减少内核软件包的体积。正常安装的系统应该会包含可选内核模块包，而一些裁剪后的系统可能会将其精简掉。`AUFS` 内核驱动属于可选内核模块的一部分，作为推荐的 Docker 存储层驱动，一般建议安装可选内核模块包以使用 `AUFS`。

如果系统没有安装可选内核模块的话，可以执行下面的命令来安装可选内核模块包：

```bash
$ sudo apt-get update

$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```

#### Ubuntu 16.04 +

Ubuntu 16.04 + 上的 Docker CE 默认使用 `overlay2` 存储层驱动,无需手动配置。

### 1.1.1 使用 APT 安装、升级

由于 `apt` 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。

```bash
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

为了确认所下载软件包的合法性，需要添加软件源的 `GPG` 密钥。

```bash
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -


# 官方源
# $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

然后，我们需要向 `source.list` 中添加 Docker 软件源

```bash
$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"


# 官方源
# $ sudo add-apt-repository \
#    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) \
#    stable"    
```

>以上命令会添加稳定版本的 Docker CE APT 镜像源，如果需要测试或每日构建版本的 Docker CE 请将 stable 改为 test 或者 nightly。

#### 安装

更新 apt 软件包缓存，并安装 `docker-ce`：

```bash
$ sudo apt-get update

$ sudo apt-get install docker-ce
```
指定版本安装
```
版本列表
apt-cache madison docker-ce

docker-ce | 18.03.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
```
```
执行安装命令
sudo apt-get install docker-ce=<VERSION>
```
#### 升级
```
sudo apt-get update
```


### 1.1.2 使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu 系统上可以使用这套脚本安装：

```bash
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中。

### 1.1.3 使用deb包安装 
- 下载.deb包,[下载地址](https://download.docker.com/linux/ubuntu/dists/)。
- 执行命令
```
sudo dpkg -i /path/to/package.deb
```

### 启动 Docker CE

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

Ubuntu 14.04 请使用以下命令启动：

```bash
$ sudo service docker start
```

### 建立 docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```bash
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```bash
$ sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

### 测试 Docker 是否安装正确

```bash
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

若能正常输出以上信息，则说明安装成功。

### 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置国内镜像加速(accelerate-mirror.pdf)。

### 1.2 卸载
```
sudo apt-get purge docker-ce
sudo rm -rf /var/lib/docker
```

### 参考文档

* [Docker 官方 Ubuntu 安装文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)  

## 2、Centos
### 2.1、CentOS 安装 Docker CE

>警告：切勿在没有配置 Docker YUM 源的情况下直接使用 yum 命令安装 Docker.

### 准备工作

#### 系统要求

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 `overlay2` 存储层驱动）无法使用，并且部分功能可能不太稳定。

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

### 2.1.1 使用 yum 安装、升级

执行以下命令安装依赖包：

```bash
$ sudo yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

执行下面的命令添加 `yum` 软件源：

```bash
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo


# 官方源
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo    
```

如果需要测试版本的 Docker CE 请使用以下命令：

```bash
$ sudo yum-config-manager --enable docker-ce-test
```

如果需要每日构建版本的 Docker CE 请使用以下命令：

```bash
$ sudo yum-config-manager --enable docker-ce-nightly
```

#### 安装 Docker CE

更新 `yum` 软件源缓存，并安装 `docker-ce`。

```bash
$ sudo yum makecache fast
$ sudo yum install docker-ce
```
指定版本安装,通过以下命令查看可用的安装版本：
```
yum list docker-ce.x86_64  --showduplicates |sort -r
```
在安装时可以指定版本号来安装指定版本的Docker：
```
sudo yum -y install docker-ce-<VERSION_STRING>
```
#### 升级
```
 sudo yum makecache fast
```


### 2.1.2 使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装：

```bash
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中。  
### 2.1.3 使用rpm安装、升级
#### 安装   
去[rpm地址](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)下载.rpm文件。
执行如下命令：
```
 sudo yum -y install xxx.rpm
```
#### 升级
 使用.rpm包安装的Docker不支持在线升级，升级时同样需要下载新的安装文件并使用yum -y upgrade命令进行升级安装：
 ```
  sudo yum -y upgrade xxx.rpm
 ```

### 启动 Docker CE

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

### 建立 docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```bash
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```bash
$ sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

### 测试 Docker 是否安装正确

```bash
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

若能正常输出以上信息，则说明安装成功。

### 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置 国内镜像加速(accelerate-mirror.pdf)。

### 添加内核参数

默认配置下，如果在 CentOS 使用 Docker CE 看到下面的这些警告信息：

```bash
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

请添加内核配置参数以启用这些功能。

```bash
$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后重新加载 `sysctl.conf` 即可

```bash
$ sudo sysctl -p
```
### 2.2 卸载 Docker CE
```
 sudo yum remove docker-ce
 sudo rm -rf /var/lib/docker
```

### 参考文档

* [Docker 官方 CentOS 安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)。

## 3、macOS 安装 Docker

### 系统要求

[Docker for Mac](https://docs.docker.com/docker-for-mac/) 要求系统最低为 macOS 10.10.3 Yosemite。如果系统不满足需求，可以安装 [Docker Toolbox](https://docs.docker.com/toolbox/overview/)。

### 安装

#### 使用 Homebrew 安装

[Homebrew](http://brew.sh/) 的 [Cask](https://caskroom.github.io/) 已经支持 Docker for Mac，因此可以很方便的使用 Homebrew Cask 来进行安装：

```bash
$ brew cask install docker
```

#### 手动下载安装

如果需要手动下载，请点击以下链接下载 [Stable](https://download.docker.com/mac/stable/Docker.dmg) 或 [Edge](https://download.docker.com/mac/edge/Docker.dmg) 版本的 Docker for Mac。

如同 macOS 其它软件一样，安装也非常简单，双击下载的 `.dmg` 文件，然后将那只叫 [Moby](https://blog.docker.com/2013/10/call-me-moby-dock/) 的鲸鱼图标拖拽到 `Application` 文件夹即可（其间需要输入用户密码）。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-mac-dmg.png?raw=true)

### 运行

从应用中找到 Docker 图标并点击运行。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-mac-apps.png?raw=true)

运行之后，会在右上角菜单栏看到多了一个鲸鱼图标，这个图标表明了 Docker 的运行状态。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-mac-menubar.png?raw=true)

第一次点击图标，可能会看到这个安装成功的界面，点击 "Got it!" 可以关闭这个窗口。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-mac-success.png?raw=true)

以后每次点击鲸鱼图标会弹出操作菜单。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-mac-menu.png?raw=true)

启动终端后，通过命令可以检查安装后的 Docker 版本。

```bash
$ docker --version
Docker version 17.10.0-ce, build f4ffd25
$ docker-compose --version
docker-compose version 1.17.0-rc1, build a0f95af
$ docker-machine --version
docker-machine version 0.13.0, build 9ba6da9
```

如果 `docker version`、`docker info` 都正常的话，可以尝试运行一个 [Nginx 服务器](https://store.docker.com/images/nginx/)：

```bash
$ docker run -d -p 80:80 --name webserver nginx
```

服务运行后，可以访问 <http://localhost>，如果看到了 "Welcome to nginx!"，就说明 Docker for Mac 安装成功了。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-mac-example-nginx.png?raw=true)

要停止 Nginx 服务器并删除执行下面的命令：

```bash
$ docker stop webserver
$ docker rm webserver
```

### 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置国内镜像加速，参考(accelerate-mirror.pdf)

## 4、Windows 10 PC 安装 Docker CE

### 系统要求

[Docker for Windows](https://docs.docker.com/docker-for-windows/install/) 支持 64 位版本的 Windows 10 Pro，且必须开启 Hyper-V。

### 安装

点击以下链接下载 [Stable](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe) 或 [Edge](https://download.docker.com/win/edge/Docker%20for%20Windows%20Installer.exe) 版本的 Docker for Windows。

下载好之后双击 Docker for Windows Installer.exe 开始安装。

### 运行

在 Windows 搜索栏输入 Docker 点击 Docker for Windows 开始运行。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-win-docker-app-search.png?raw=true)

Docker CE 启动之后会在 Windows 任务栏出现鲸鱼图标。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-win-taskbar-circle.png?raw=true)

等待片刻，点击 Got it 开始使用 Docker CE。

![](https://github.com/yeasy/docker_practice/blob/master/install/_images/install-win-success-popup-cloud.png?raw=true)

### 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置 国内镜像加速,参考(accelerate-mirror.pdf)。

