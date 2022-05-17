


# 安装与卸载
```
$ sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 国内用户可以使用以下方式加快下载
$ sudo curl -L https://download.fastgit.org/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```
## bash 补全命令
```
$ curl -L https://raw.githubusercontent.com/docker/compose/1.27.4/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```
## 卸载
如果是二进制包方式安装的，删除二进制文件即可。
```
$ rm /usr/local/bin/docker-compose
```
# 使用
## 术语
* 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。
* 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。

可见，一个项目可以由多个服务（容器）关联而成，`Compose`面向项目进行管理。
## 场景
最常见的项目是 web 网站，该项目应该包含 web 应用和缓存。
下面我们用 Python 来建立一个能够记录页面访问次数的 web 网站。
web 应用
新建文件夹，在该目录中编写 app.py 文件
```
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```
Dockerfile

编写 Dockerfile 文件，内容为
```
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
docker-compose.yml
编写 docker-compose.yml 文件，这个是 Compose 使用的主模板文件。
version: '3'
services:

  web:
    build: .
    ports:
     - "5000:5000"

  redis:
    image: "redis:alpine"
```
运行 compose 项目
```
$ docker-compose up
```
此时访问本地 5000 端口，每次刷新页面，计数就会加 1。


# Compose 模板文件
模板文件是使用 Compose 的核心，涉及到的指令关键字也比较多。这里面大部分指令跟`docker run`相关参数的含义都是类似的。

默认的模板文件名称为`docker-compose.yml`，格式为 YAML 格式。
```
version: "3"
​
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```
注意每个服务都必须通过`image`指令指定镜像或`build`指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用`build`指令，在 Dockerfile 中设置的选项(例如：`CMD, EXPOSE, VOLUME, ENV`等) 将会自动被获取，无需在`docker-compose.yml`中重复设置。
## varsion
```
version: '3'
```
表示使用第三代语法来构建`docker-compose.yaml`文件。
## build
指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。
```
version: '3'
services:
​
  webapp:
    build: ./dir
```
你也可以使用`context`指令指定 Dockerfile 所在文件夹的路径。

使用`dockerfile`指令指定 Dockerfile 文件名。

使用`arg`指令指定构建镜像时的变量。
```
version: '3'
services:
​
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```
使用`cache_from`指定构建镜像的缓存
```
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```
## command
覆盖容器启动后默认执行的命令。
```
command: echo "hello world"
```
## container_name
指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式。
```
container_name: docker-web-container
```
注意: 指定容器名称后，该服务将无法进行扩展，因为 Docker 不允许多个容器具有相同的名称。
## depends_on
解决容器的依赖、启动先后的问题。以下例子中会先启动`redis db`再启动`web`。
```
version: '3'
​
services:
  web:
    build: .
    depends_on:
      - db
      - redis
​
  redis:
    image: redis
​
  db:
    image: postgres
```
注意：web 服务不会等待`redis db`「完全启动」之后才启动。
## tmpfs
挂载一个`tmpfs`文件系统到容器。
```
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```
## env_file
从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过`docker-compose -f FILE`方式来指定`Compose`模板文件，则`env_file`中变量的路径会基于模板文件路径。

如果有变量名称与`environment`指令冲突，则按照惯例，以后者为准。
```
env_file: .env
​
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```
环境变量文件中每一行必须符合格式，支持`#`开头的注释行。
```
# common.env: Set development environment
PROG_ENV=development
```
## environment
设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。
```
environment:
  RACK_ENV: development
  SESSION_SECRET:
​
environment:
  - RACK_ENV=development
  - SESSION_SECRET
```
如果变量名称或者值中用到`true|false，yes|no`等表达  含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括
```
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```
## expose
暴露端口，但不映射到宿主机，只被连接的服务访问。

仅可以指定内部端口为参数。
```
expose:
 - "3000"
 - "8000"
```
## healthcheck
通过命令检查容器是否健康运行。
```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```
## image
指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose`将会尝试拉取这个镜像。
```
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```
## labels
为容器添加 Docker 元数据信息。例如可以为容器添加辅助说明信息。
```
labels:
  com.startupteam.description: "webapp for a startup team"
  com.startupteam.department: "devops department"
  com.startupteam.release: "rc3 for v1.0"
```
## logging
配置日志选项。
```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```
目前支持三种日志驱动类型。
```
driver: "json-file"
driver: "syslog"
driver: "none"
```
`options`配置日志驱动的相关参数。
```
options:
  max-size: "200k"
  max-file: "10"
```
## network_mode
设置网络模式。使用和`docker run`的`--network`参数一样的值。
```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```
## networks
配置容器连接的网络。
```
version: "3"
services:
​
  some-service:
    networks:
     - some-network
     - other-network
​
networks:
  some-network:
  other-network:
```
## pid
跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程 ID 来相互访问和操作。
```
pid: "host"
```
## ports
暴露端口信息。

使用宿主端口：容器端口 (`HOST:CONTAINER`) 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。
```
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```
注意：当使用`HOST:CONTAINER`格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果。
## secrets
存储敏感数据，例如 mysql 服务密码。
```
version: "3.1"
services:
​
mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret
​
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
## ulimits
指定容器的`ulimits`限制值。
例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。
```
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```
## volumes
数据卷所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）。
该指令中路径支持相对路径。
```
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```
如果路径为数据卷名称，必须在文件中配置数据卷。
```
version: "3"
​
services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql
​
volumes:
  mysql_data:
```
## 其它指令
此外，还有包括`domainname, entrypoint, hostname, ipc, mac_address, privileged, read_only, shm_size, restart, stdin_open, tty, user, working_dir`等指令，基本跟`docker run`中对应参数的功能一致。

指定服务容器启动后执行的入口文件。
```
entrypoint: /code/entrypoint.sh
```
指定容器中运行应用的用户名。
```
user: nginx
```
指定容器中工作目录。
```
working_dir: /code
```
指定容器中搜索域名、主机名、mac 地址等。
```
domainname: your_website.com
hostname: test
mac_address: 08-00-27-00-0C-0A
```
允许容器中运行一些特权命令。
```
privileged: true
```
指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为`always`或者`unless-stopped`。
```
restart: always
```
以只读模式挂载容器的`root`文件系统，意味着不能对容器内容进行修改。
```
read_only: true
```
打开标准输入，可以接受外部输入。
```
stdin_open: true
```
模拟一个伪终端。
```
tty: true
```
## 读取变量
`Compose`模板文件支持动态读取主机的系统环境变量和当前目录下的`.env`文件中的变量。
例如，下面的 Compose 文件将从运行它的环境中读取变量 ${MONGO_VERSION} 的值，并写入执行的指令中。
```
version: "3"
services:
​
db:
  image: "mongo:${MONGO_VERSION}"
```
如果执行`MONGO_VERSION=3.2 docker-compose up`则会启动一个 mongo:3.2 镜像的容器；如果执行`MONGO_VERSION=2.8 docker-compose up`则会启动一个`mongo:2.8`镜像的容器。
若当前目录存在`.env`文件，执行 docker-compose 命令时将从该文件中读取变量。
在当前目录新建`.env`文件并写入以下内容。
```
# 支持 # 号注释
MONGO_VERSION=3.6
```
执行`docker-compose up`则会启动一个`mongo:3.6`镜像的容器。





自定义配置文件redis.conf
下载redis获取对应版本的redis.conf，地址https://redis.io/download（注意：这里下载的版本与后续docker-compose.yml中版本需一致）
必须修改redis.conf的配置：
bind 127.0.0.1   改为：#bind 127.0.0.1 

protected-mode yes 改为：protected-mode no

#requirepass password 改为：requirepass 自定义密码

 hostname: redis



# 关于网络,如果用到mysql或者redis，并且希望在同一个网络，那么就可以直接使用同一个网络名
# docker network create project-demo_bridge
networks:
  project-demo_bridge:
    driver: bridge

services:
   project-demo:
     container_name: project-demo
     image: project-demo:1.0
     restart: always
     volumes:
       # 同步时间
       - /etc/localtime:/etc/localtime:ro
       # 如果项目有些日志或者写文件，需要同步到宿主机器，也需要定义相应的卷
       - ./data:/etc/project-demo/data
       - ./log:/etc/project-demo/log
     ports:
       - 8080:8080
     networks:
       - project-demo_bridge