---
title: Docker入门
date: 2022-01-18 16:24:41
tags: [Docker]
categories: Docker
---

# 安装Docker
## CentOS
### 使用 yum 安装
```
// 安装需要的软件包， yum-utils 提供yum-config-manager功能
$ yum install -y yum-utils
// 添加yum软件源
// 阿里云的
$ yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
// 或ustc的
$ yum-config-manager --add-repo http://mirrors.ustc.edu.cn/docker- ce/linux/centos/docker-ce.repo
// 安装docker，出现输入的界面都按 y
$ yum install -y docker-ce
// 启动docker，刚下载会自动运行不用再启动
$ systemctl enable docker
$ systemctl start docker
// 测试docker是否安装成功
$ docker -v
```
## 镜像加速器
国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。
* 阿里云加速器(点击管理控制台 -> 登录账号(淘宝账号) -> 右侧镜像工具 -> 镜像加速器 -> 复制加速器地址)
* 网易云加速器`https://hub-mirror.c.163.com`
* 百度云加速器`https://mirror.baidubce.com`
* ustc镜像加速器`https://docker.mirrors.ustc.edu.cn`
```
// 编辑文件/etc/docker/daemon.json
$ mkdir /etc/docker
$ vi /etc/docker/daemon.json
// 在文件中加入下面内容
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
// 重新启动服务
$ systemctl daemon-reload
$ systemctl restart docker
```
# 使用镜像
Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜像。
## 拉取镜像
可以通过`docker search`命令来查找官方仓库中的镜像，并利用`docker pull`命令来将它下载到本地。
```bash
$ docker search centos
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                             The official build of CentOS.                   6449      [OK]
ansible/centos7-ansible            Ansible on Centos7                              132                  [OK]
consol/centos-xfce-vnc             Centos container with "headless" VNC session…   126                  [OK]
jdeathe/centos-ssh                 OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   117                  [OK]
centos/systemd                     systemd enabled base container.                 96                   [OK]
```
可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、收藏数（表示该镜像的受关注程度）、是否官方创建（`OFFICIAL`）、是否自动构建 （`AUTOMATED`）。

根据是否是官方提供，可将镜像分为两类。
* 一种是类似 centos 这样的镜像，被称为基础镜像或根镜像。这些基础镜像由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。
* 还有一种类型，比如`ansible/centos7-ansible`镜像，它是由 Docker Hub 的注册用户创建并维护的，往往带有用户名称前缀。可以通过前缀`username/`来指定使用某个用户提供的镜像，比如`ansible`用户。

另外，在查找的时候通过`--filter=stars=N`参数可以指定仅显示收藏数量为`N`以上的镜像。
## 获取镜像
从 Docker 镜像仓库获取镜像的命令是`docker pull`。
```bash
docker pull [选项] [Docker 镜像仓库地址[:端口号]/]仓库名[:标签]
```
具体的选项可以通过`docker pull --help`命令看到。

镜像名称的格式。
* Docker 镜像仓库地址：地址的格式一般是`<域名/IP>[:端口号]`。默认地址是`Docker Hub(docker.io)`。
* 仓库名：仓库名是两段式名称，即`<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为`library`，也就是官方镜像。

```bash
$ docker pull centos:7
7: Pulling from library/centos
2d473b07cdd5: Pull complete 
Digest: sha256:c73f515d06b0fa07bb18d8202035e739a494ce760aa73129f60f4bf2bd22b407
Status: Downloaded newer image for centos:7
docker.io/library/centos:7
```
上面的命令中没有给出 Docker 镜像仓库地址，因此将会从`Docker Hub （docker.io）`获取镜像。而镜像名称是`centos:7`，因此将会获取官方镜像`library/centos`仓库中标签为 7 的镜像。`docker pull`命令的输出结果最后一行给出了镜像的完整名称，即：`docker.io/library/centos:7`。

从下载过程中可以看到镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。
## 列出镜像
列出已经下载下来的镜像，可以使用`docker image ls`命令或`docker images`。
```bash
// 或docker images
$ docker image ls

REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        5.7       4181d485f650   4 days ago     448MB
redis        latest    f1b6973564e9   3 weeks ago    113MB
nginx        latest    c316d5a335a5   3 weeks ago    142MB
centos       7         eeb6ee3f44bd   5 months ago   204MB
```
列表包含了仓库名、标签、镜像 ID、创建时间以及所占用的空间。

镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个标签。
### 镜像体积
如果仔细观察，会注意到，这里标识的所占用空间和在 Docker Hub 上看到的镜像大小不同。比如，`centos:7`镜像大小，在这里是 204MB，但是在 Docker Hub 显示的却是 103.35MB。这是因为 Docker Hub 中显示的体积是压缩后的体积。在镜像下载和上传过程中镜像是保持着压缩状态的，因此 Docker Hub 所显示的大小是网络传输中更关心的流量大小。而`docker image ls`显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小。

另外一个需要注意的问题是，`docker image ls`列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

可以通过`docker system df`命令来便捷的查看镜像、容器、数据卷所占用的空间。
```bash
$ docker system df

TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          4         0         826MB     826MB (100%)
Containers      0         0         0B        0B
Local Volumes   2         0         398.7MB   398.7MB (100%)
Build Cache     0         0         0B        0B
```
### 列出部分镜像
不加任何参数的情况下，`docker image ls`会列出所有顶层镜像，但是有时候我们只希望列出部分镜像。`docker image ls`有好几个参数可以帮助做到这个事情。

根据仓库名列出镜像
```bash
$ docker image ls ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               329ed837d508        3 days ago          63.3MB
ubuntu              bionic              329ed837d508        3 days ago          63.3MB
```
列出特定的某个镜像，也就是说指定仓库名和标签
```
$ docker image ls ubuntu:18.04
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               329ed837d508        3 days ago          63.3MB
```
除此以外，`docker image ls`还支持强大的过滤器参数`--filter`，或者简写`-f`。比如，我们希望看到在`mongo:3.2`之后建立的镜像，可以用下面的命令：
```
$ docker image ls -f since=mongo:3.2
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              5f515359c7f8        5 days ago          183 MB
nginx               latest              05a60462f8ba        5 days ago          181 MB
```
想查看某个位置之前的镜像也可以，只需要把`since`换成`before`即可。

此外，如果镜像构建时，定义了`LABEL`，还可以通过`LABEL`来过滤。
```
$ docker image ls -f label=com.example.version=0.1
...
```
## 删除本地镜像
删除本地的镜像，可以使用`docker image rm`命令，其格式为：
```
$ docker image rm [选项] <镜像1> [<镜像2> ...]
// 或docker rmi [选项] <镜像1> [<镜像2> ...]
```
### 用 ID、镜像名、摘要删除镜像
其中，`<镜像>`可以是镜像短 ID、镜像长 ID、镜像名或者镜像摘要。

比如我们有这么一些镜像：
```bash
$ docker image ls
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
centos                      latest              0584b3d2cf6d        3 weeks ago         196.5 MB
redis                       alpine              501ad78535f0        3 weeks ago         21.03 MB
docker                      latest              cf693ec9b5c7        3 weeks ago         105.1 MB
nginx                       latest              e43d811ce2f4        5 weeks ago         181.5 MB
```
我们可以用镜像的完整 ID，也称为长 ID，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用短 ID 来删除镜像。`docker image ls`默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。

比如这里，如果我们要删除`redis:alpine`镜像，可以执行：
```bash
//或docker rmi 501
$ docker image rm 501
Untagged: redis:alpine
Untagged: redis@sha256:f1ed3708f538b537eb9c2a7dd50dc90a706f7debd7e1196c9264edeea521a86d
Deleted: sha256:501ad78535f015d88872e13fa87a828425117e3d28075d0c117932b05bf189b7
Deleted: sha256:96167737e29ca8e9d74982ef2a0dda76ed7b430da55e321c071f0dbff8c2899b
Deleted: sha256:32770d1dcf835f192cafd6b9263b7b597a1778a403a109e2cc2ee866f74adf23
Deleted: sha256:127227698ad74a5846ff5153475e03439d96d4b1c7f2a449c7a826ef74a2d2fa
Deleted: sha256:1333ecc582459bac54e1437335c0816bc17634e131ea0cc48daa27d32c75eab3
Deleted: sha256:4fc455b921edf9c4aea207c51ab39b10b06540c8b4825ba57b3feed1668fa7c7
```
我们也可以用镜像名，也就是`<仓库名>:<标签>`，来删除镜像。
```bash
$ docker image rm centos
Untagged: centos:latest
Untagged: centos@sha256:b2f9d1c0ff5f87a4743104d099a3d561002ac500db1b9bfa02a783a46e0d366c
Deleted: sha256:0584b3d2cf6d235ee310cf14b54667d889887b838d3f3d3033acd70fc3c48b8a
Deleted: sha256:97ca462ad9eeae25941546209454496e1d66749d53dfa2ee32bf1faabd239d38
```
当然，更精确的是使用镜像摘要删除镜像。
```bash
$ docker image ls --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```
## 使用 Dockerfile 定制镜像
镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是`Dockerfile`。

`Dockerfile`是一个文本文件，其内包含了一条条的指令，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

以定制 nginx 镜像为例。在一个空白目录中，建立一个文本文件，并命名为 Dockerfile：
```bash
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```
其内容为：
```bash
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
这个 Dockerfile 很简单，一共就两行。涉及到了两条指令，`FROM`和`RUN`。
### FROM 指定基础镜像
所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个`nginx`镜像的容器，再进行修改一样，基础镜像是必须指定的。而`FROM`就是指定基础镜像，因此一个 Dockerfile 中`FROM`是必备的指令，并且必须是第一条指令。

在 Docker Hub 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如`nginx、redis、mongo、mysql、httpd、php、tomcat`等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如`ubuntu、debian、centos、fedora、alpine`等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为`scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。
```bash
FROM scratch
...
```
如果你以`scratch`为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接`FROM scratch`会让镜像体积更加小巧。使用 Go 语言 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。
### RUN 执行命令
`RUN`指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：
* `shell`格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的`RUN`指令就是这种格式。
```
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
* `exec`格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

既然`RUN`就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个`RUN`呢？比如这样：
```bash
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```
Dockerfile 中每一个指令都会建立一层，`RUN`也不例外。每一个`RUN`的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit`这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。

Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。

上面的 Dockerfile 正确的写法应该是这样：
```bash
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```
首先，之前所有的命令只有一个目的，就是编译、安装`redis`可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个`RUN`一一对应不同的命令，而是仅仅使用一个`RUN`指令，并使用`&&`将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加`\`的命令换行方式，以及行首`#`进行注释的格式。

此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了`apt`缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。
### 构建镜像
让我们再回到之前定制的 nginx 镜像的 Dockerfile 来。在 Dockerfile 文件所在目录执行：
```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
 ---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 9cdc27646c7b
 ---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```
从命令的输出结果中，我们可以清晰的看到镜像的构建过程。在`Step 2`中，如同我们之前所说的那样，`RUN`指令启动了一个容器`9cdc27646c7b`，执行了所要求的命令，并最后提交了这一层`44aa4490ce2c`，随后删除了所用到的这个容器`9cdc27646c7b`。

这里我们使用了`docker build`命令进行镜像构建。其格式为：
```
docker build [选项] <上下文路径/URL/->
```
在这里我们指定了最终镜像的名称`-t nginx:v3`，构建成功后，我们可以像之前运行`nginx:v2`那样来运行这个镜像，其结果会和`nginx:v2`一样。
### 镜像构建上下文（Context）
如果注意，会看到`docker build`命令最后有一个`.`。`.`表示当前目录，而`Dockerfile`就在当前目录，因此不少初学者以为这个路径是在指定 Dockerfile 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。那么什么是上下文呢？

首先我们要理解`docker build`的工作原理。Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过`COPY`指令、`ADD`指令等。而`docker build`命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

如果在 Dockerfile 中这么写：
```bash
COPY ./package.json /app/
```
这并不是要复制执行`docker build`命令所在的目录下的`package.json`，也不是复制 Dockerfile 所在目录下的`package.json`，而是复制 上下文（context） 目录下的`package.json`。

因此，`COPY`这类指令中的源文件的路径都是相对路径。这也是初学者经常会问的为什么`COPY ../package.json /app`或者`COPY /opt/xxxx /app`无法工作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

现在就可以理解刚才的命令`docker build -t nginx:v3 .`中的这个`.`，实际上是在指定上下文的目录，`docker build`命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

如果观察`docker build`输出，我们其实已经看到了这个发送上下文的过程：
```
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
...
```
理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现`COPY /opt/xxxx /app`不工作后，于是干脆将 Dockerfile 放到了硬盘根目录去构建，结果发现`docker build`执行后，在发送一个几十 GB 的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让`docker build`打包整个硬盘，这显然是使用错误。

一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用`.gitignore`一样的语法写一个`.dockerignore`，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

那么为什么会有人误以为`.`是指定 Dockerfile 所在目录呢？这是因为在默认情况下，如果不额外指定 Dockerfile 的话，会将上下文目录下的名为 Dockerfile 的文件作为 Dockerfile。

这只是默认行为，实际上 Dockerfile 的文件名并不要求必须为 Dockerfile，而且并不要求必须位于上下文目录中，比如可以用`-f ../Dockerfile.php`参数指定某个文件作为 Dockerfile。

当然，一般大家习惯性的会使用默认的文件名 Dockerfile，以及会将其置于镜像构建上下文目录中。
## Dockerfile 指令详解
### COPY 复制文件
```bash
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```
和`RUN`指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

`COPY`指令将从构建上下文目录中`<源路径>`的文件/目录复制到新的一层的镜像内的`<目标路径>`位置。
```bash
COPY package.json /usr/src/app/
```
`<源路径>`可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的`filepath.Match`规则，如：
```bash
COPY hom* /mydir/
COPY hom?.txt /mydir/
```
`<目标路径>`可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用`WORKDIR`指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用`COPY`指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上`--chown=<user>:<group>`选项来改变文件的所属用户及所属组。
```bash
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```
如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。
### ADD 更高级的复制文件
`ADD`指令和`COPY`的格式和性质基本一致。但是在`COPY`基础上增加了一些功能。

比如`<源路径>`可以是一个 URL，这种情况下，Docker 引擎会试图去下载这个链接的文件放到`<目标路径>`去。下载后的文件权限自动设置为 600，如果这并不是想要的权限，那么还需要增加额外的一层`RUN`进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层`RUN`指令进行解压缩。所以不如直接使用`RUN`指令，然后使用 wget 或者 curl 工具下载，处理权限、解压缩、然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。

如果 <源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，`ADD`指令将会自动解压缩这个压缩文件到 <目标路径> 去。
在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 ubuntu 中：
```bash
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
...
```
但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用`ADD`命令了。

在 Docker 官方的 Dockerfile 最佳实践文档中要求，尽可能的使用`COPY`，因为`COPY`的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用`ADD`的场合，就是所提及的需要自动解压缩的场合。

另外需要注意的是，`ADD`指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。

因此在`COPY`和`ADD`指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用`COPY`指令，仅在需要自动解压缩的场合使用`ADD`。
在使用该指令的时候还可以加上`--chown=<user>:<group>`选项来改变文件的所属用户及所属组。
```bash
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```
### CMD 容器启动命令
`CMD`指令的格式和`RUN`相似，也是两种格式：
* shell 格式：`CMD <命令>`
* exec 格式：`CMD ["可执行文件", "参数1", "参数2"...]`

参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了`ENTRYPOINT`指令后，用`CMD`指定具体的参数。

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。`CMD`指令就是用于指定默认的容器主进程的启动命令的。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，ubuntu 镜像默认的 CMD 是`/bin/bash`，如果我们直接`docker run -it ubuntu`的话，会直接进入`bash`。我们也可以在运行时指定运行别的命令，如`docker run -it ubuntu cat /etc/os-release`。这就是用 cat /etc/os-release 命令替换了默认的`/bin/bash`命令了，输出了系统版本信息。

在指令格式上，一般推荐使用`exec`格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。
如果使用 shell 格式的话，实际的命令会被包装为`sh -c`的参数的形式进行执行。比如：
```
CMD echo $HOME
```
在实际执行中，会将其变更为：
```
CMD [ "sh", "-c", "echo $HOME" ]
```
这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。

提到 CMD 就不得不提容器中应用在前台执行和后台执行的问题。。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 systemd 去启动后台服务，容器内没有后台服务的概念。
一些初学者将 CMD 写为：
```
CMD service nginx start
```
然后发现容器执行后就立即退出了。甚至在容器内去使用`systemctl`命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

而使用`service nginx start`命令，则是希望`upstart`来以后台守护进程形式启动 nginx 服务。而刚才说了`CMD service nginx start`会被理解为`CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 sh。那么当`service nginx start`命令结束后，sh 也就结束了，sh 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：
```
CMD ["nginx", "-g", "daemon off;"]
```
### ENTRYPOINT 入口点
`ENTRYPOINT`的格式和`RUN`指令格式一样，分为`exec`格式和`shell`格式。

`ENTRYPOINT`的目的和`CMD`一样，都是在指定容器启动程序及参数。`ENTRYPOINT`在运行时也可以替代，不过比`CMD`要略显繁琐，需要通过`docker run`的参数`--entrypoint`来指定。

当指定了`ENTRYPOINT`后，`CMD`的含义就发生了改变，不再是直接的运行其命令，而是将`CMD`的内容作为参数传给`ENTRYPOINT`指令，换句话说实际执行时，将变为：`<ENTRYPOINT> "<CMD>"`

那么有了 CMD 后，为什么还要有 ENTRYPOINT 呢？这种`<ENTRYPOINT> "<CMD>"`有什么好处么？让我们来看几个场景。

场景一：让镜像变成像命令一样使用
假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 CMD 来实现：
```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```
假如我们使用`docker build -t myip .`来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：
```
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
```
嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的`CMD`中可以看到实质的命令是 curl，那么如果我们希望显示 HTTP 头信息，就需要加上`-i`参数。那么我们可以直接加`-i`参数给`docker run myip`么？
```
$ docker run myip -i
docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".
```
我们可以看到可执行文件找不到的报错，`executable file not found`。之前我们说过，跟在镜像名后面的是 command，运行时会替换 CMD 的默认值。因此这里的 -i 替换了原来的 CMD，而不是添加在原来的 curl -s http://myip.ipip.net 后面。而 -i 根本不是命令，所以自然找不到。
那么如果我们希望加入 -i 这参数，我们就必须重新完整的输入这个命令：
```
$ docker run myip curl -s http://myip.ipip.net -i
```
这显然不是很好的解决方案，而使用`ENTRYPOINT`就可以解决这个问题。现在我们重新用`ENTRYPOINT`来实现这个镜像：
```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```
这次我们再来尝试直接使用 docker run myip -i：
```
$ docker run myip
```
当前 IP：61.148.226.66 来自：北京市 联通
```
$ docker run myip -i
HTTP/1.1 200 OK
Server: nginx/1.8.0
Date: Tue, 22 Nov 2016 05:12:40 GMT
Content-Type: text/html; charset=UTF-8
Vary: Accept-Encoding
X-Powered-By: PHP/5.6.24-1~dotdeb+7.1
X-Cache: MISS from cache-2
X-Cache-Lookup: MISS from cache-2:80
X-Cache: MISS from proxy-2_6
Transfer-Encoding: chunked
Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006
Connection: keep-alive

当前 IP：61.148.226.66 来自：北京市 联通
```
可以看到，这次成功了。这是因为当存在`ENTRYPOINT`后，`CMD`的内容将会作为参数传给`ENTRYPOINT`，而这里`-i`就是新的`CMD`，因此会作为参数传给 curl，从而达到了我们预期的效果。

场景二：应用运行前的准备工作
启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。
比如 mysql 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。

此外，可能希望避免使用 root 用户去启动服务，从而提高安全性，而在启动服务前还需要以 root 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 root 身份执行，方便调试等。

这些准备工作是和容器 CMD 无关的，无论 CMD 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 ENTRYPOINT 中去执行，而这个脚本会将接到的参数（也就是`<CMD>`）作为命令，在脚本最后执行。比如官方镜像 redis 中就是这么做的：
```bash
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```
可以看到其中为了`redis`服务创建了`redis`用户，并在最后指定了`ENTRYPOINT`为`docker-entrypoint.sh`脚本。
```bash
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    find . \! -user redis -exec chown redis '{}' +
    exec gosu redis "$0" "$@"
fi
```
exec "$@"
该脚本的内容就是根据 CMD 的内容来判断，如果是 redis-server 的话，则切换到 redis 用户身份启动服务器，否则依旧使用 root 身份执行。比如：
```bash
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```

# 操作容器
简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。
## 查看容器
查看正在运行的容器使用命令：`docker ps`。

查看所有容器使用命令：`docker ps -a`。
## 启动
启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`exited`）的容器重新启动。

因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。
### 新建并启动
所需要的命令主要为`docker run`。

参数说明：
* `-i`：表示运行容器
* `-t`：表示容器启动后会进入其命令行。加入这个参数后，容器创建就能登录进去。即分配一个伪终端。
* `--name`:为创建的容器命名。
* `-v`：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个`－v`做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
* `-d`：在`run`后面加上`-d`参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加`-i -t`两个参数，创建后就会自动进去容器）。
* `-p`：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个`-p`做多个端口映射

### 交互式容器
以交互式方式创建并启动容器，启动完成后，直接进入当前容器。使用`exit`命令退出容器。需要注意的是以此种方式启动容器，如果退出容器，则容器会进入停止状态。
```
$ docker run -it --name=ubuntu ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
```
在交互模式下，用户可以通过所创建的终端来输入命令，例如
```
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```
当利用`docker run`来创建容器时，Docker 在后台运行的标准操作包括：
* 检查本地是否存在指定的镜像，不存在就从仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个`ip`地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

### 守护式容器
创建一个守护式容器；如果对于一个需要长期运行的容器来说，我们可以创建一个守护式容器。
```
# 创建并启动守护式容器
$ docker run -di --name=mycentos2 centos:7
# 登录进入容器命令为：docker exec -it container_name (或者 container_id) /bin/bash（exit退出 时，容器不会停止）
$ docker exec -it mycentos2 /bin/bash
```
### 启动已终止容器
可以利用`docker container start 容器名称或者ID`或`docker start 容器名称或者ID`命令，直接将一个已经终止（`exited`）的容器启动运行。

容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用`ps`或`top`来查看进程信息。
```
root@ba267838cc1b:/# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```
可见，容器中仅运行了指定的`bash`应用。这种特点使得 Docker 对资源的利用率极高。
## 终止
可以使用`docker container stop 容器名称或者ID`或`docker stop 容器名称或者ID`来终止一个运行中的容器。

此外，当 Docker 容器中指定的应用终结时，容器也自动终止。

例如对于只启动了一个终端的容器，用户通过`exit`命令或`Ctrl+d`来退出终端时，所创建的容器立刻终止。

终止状态的容器可以用`docker container ls -a`或`docker ps -a`命令看到。例如
```bash
$ docker container ls -a
CONTAINER ID        IMAGE                    COMMAND                CREATED             STATUS                          PORTS               NAMES
ba267838cc1b        ubuntu:18.04             "/bin/bash"            30 minutes ago      Exited (0) About a minute ago                       trusting_newton
```
处于终止状态的容器，可以通过`docker container start`或`docker start`命令来重新启动。

此外，`docker container restart`或`docker restart`命令会将一个运行态的容器终止，然后再重新启动它。
## 进入容器
在使用`-d`参数时，容器启动后会进入后台。某些时候需要进入容器进行操作，使用`docker exec`命令。

`docker exec`后边可以跟多个参数，这里主要说明`-i -t`参数。
* 只用`-i`参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。
* 当`-i -t`参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bash
$ docker run -di --name=ubuntu ubuntu:18.04
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

$ docker exec -i 69d1 /bin/bash
ls
bin
boot
dev
...

#登录进入容器命令为：docker exec -it container_name (或者 container_id) /bin/bash（exit退出时，容器不会停止）
$ docker exec -it 69d1 /bin/bash
root@69d137adef7a:/#
```
## 删除
### 删除容器
可以使用`docker container rm 容器名称（容器ID）`或`docker rm 容器名称（容器ID）`来删除一个处于终止状态的容器。
```
$ docker container rm trusting_newton
trusting_newton
```
如果要删除一个运行中的容器，可以添加`-f`参数。Docker 会发送`SIGKILL`信号给容器。

## 清理所有处于终止状态的容器
用`docker container ls -a`命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以删除所有处于终止状态的容器。
```
$ docker container prune
// 或
$ docker rm `docker ps -a -q`
```
# 数据管理
在容器中管理数据主要有两种方式：
* 数据卷（`Volumes`）
* 挂载主机目录 (`Bind mounts`)

## 数据卷
数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UnionFS，可以提供很多有用的特性：
* 数据卷可以在容器之间共享和重用
* 对数据卷的修改会立马生效
* 对数据卷的更新，不会影响镜像
* 数据卷默认会一直存在，即使容器被删除

注意：数据卷的使用，类似于 Linux 下对目录或文件进行`mount`，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）。
### 创建一个数据卷
```bash
$ docker volume create my-vol

# 查看所有的数据卷
$ docker volume ls
​
DRIVER              VOLUME NAME
local               my-vol
```
在主机里使用以下命令可以查看指定数据卷的信息
```bash
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```
### 启动一个挂载数据卷的容器
在用`docker run`命令的时候，使用`--mount`标记来将数据卷挂载到容器里。在一次`docker run`中可以挂载多个数据卷。

下面创建一个名为`web`的容器，并加载一个数据卷到容器的`/usr/share/nginx/html`目录。
```bash
$ docker run -d -P --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```
### 查看数据卷的具体信息
在主机里使用以下命令可以查看 web 容器的信息
```bash
$ docker inspect web
```
数据卷信息在`"Mounts"`属性下面
```bash
"Mounts": [
    {
        "Type": "volume",
        "Name": "my-vol",
        "Source": "/var/lib/docker/volumes/my-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```
### 删除数据卷
```
$ docker volume rm my-vol
```
数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用`docker rm -v`这个命令。

无主的数据卷可能会占据很多空间，要清理请使用以下命令
```
$ docker volume prune
```
## 文件拷贝
将linux宿主机中的文件拷贝到容器内可以使用命令：
```
# docker cp 需要拷贝的文件或目录 容器名称:容器目录

# 创建一个文件abc.txt
$ touch abc.txt
# 复制abc.txt到mycentos2的容器的 / 目录下
$ docker cp abc.txt mycentos2:/
# 进入mycentos2容器
$ docker exec -it mycentos2 /bin/bash
# 查看容器 / 目录下文件
$ ll
```
将文件从容器内拷贝出来到linux宿主机使用命令：
```
# docker cp 容器名称:容器目录 需要拷贝的文件或目录
#进入容器后创建文件cba.txt
touch cba.txt
# 退出容器 exit
# 在Linux宿主机器执行复制；将容器mycentos2的/cba.txt文件复制到 宿主机器的/root目录下
docker cp mycentos2:/cba.txt /root
```

> 注意：容器在停止状态下也可以完成文件的拷贝

## 挂载主机目录
可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器。

创建容器时添加`-v`参数，后边为宿主机目录:容器目录，例如：`docker run -di -v /usr/local/test:/usr/local/test --name=mycentos3 centos:7`。
```bash
# 创建linux宿主机器要挂载的目录
mkdir /usr/local/test
# 创建并启动容器mycentos3,并挂载linux中的/usr/local/test目录到容器的/usr/local/test；
# 也就是在 linux中的/usr/local/test中操作相当于对容器相应目录操作
docker run -di -v /usr/local/test:/usr/local/test --name=mycentos3 centos:7
# 在linux下创建文件
touch /usr/local/test/def.txt
# 进入容器
docker exec -it mycentos3 /bin/bash

# 在容器中查看目录中是否有对应文件def.txt
ll /usr/local/test
```
> 注意：如果你共享的是多级的目录，可能会出现权限不足的提示。 这是因为 CentOS7 中的安全模块 selinux 把权限禁掉了，需要添加参数`--privileged=true`来解决挂载的目录没有权限的问题。

# 使用网络
Docker 允许通过外部访问容器或容器互联的方式来提供网络服务。
## 外部访问容器
容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过`-P`或`-p`参数来指定端口映射。

当使用`-P`标记时，Docker 会随机映射一个端口到内部容器开放的网络端口。

使用`docker container ls`可以看到，本地主机的 32768 被映射到了容器的 80 端口。此时访问本机的 32768 端口即可访问容器内 NGINX 默认页面。
```bash
$ docker run -d -P nginx:alpine

$ docker container ls -l
CONTAINER ID    IMAGE          COMMAND                  CREATED         STATUS       PORTS         NAMES
fae320d08268    nginx:alpine   "/docker-entrypoint.…"   24 seconds ago   
```
`-p`则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有`ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。
### 映射所有接口地址
使用`hostPort:containerPort`格式本地的 80 端口映射到容器的 80 端口，可以执行
```
$ docker run -d -p 80:80 nginx:alpine
```
此时默认会绑定本地所有接口上的所有地址。
### 映射到指定地址的指定端口
可以使用`ip:hostPort:containerPort`格式指定映射使用一个特定地址，比如`localhost`地址`127.0.0.1`
```
$ docker run -d -p 127.0.0.1:80:80 nginx:alpine
```
### 映射到指定地址的任意端口
使用`ip::containerPort`绑定`localhost`的任意端口到容器的 80 端口，本地主机会自动分配一个端口。
```
$ docker run -d -p 127.0.0.1::80 nginx:alpine
```
还可以使用`udp`标记来指定`udp`端口
```
$ docker run -d -p 127.0.0.1:80:80/udp nginx:alpine
```
### 查看映射端口配置
使用`docker port`来查看当前映射的端口配置，也可以查看到绑定的地址
```
$ docker port fa 80
0.0.0.0:32768
```
注意：
* 容器有自己的内部网络和 ip 地址（使用 docker inspect 查看，Docker 还可以有一个可变的网络配置。）
* `-p`标记可以多次使用来绑定多个端口

例如
```
$ docker run -d -p 80:80  -p 443:443 nginx:alpine
```
## 容器互联
