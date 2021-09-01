


# 系统服务及其分类
系统服务是在后台运行的应用程序，并且可以提供一些本地系统或网络的功能。我们把这些应用程序称作服务，也就是`Service`。不过，我们有时会看到`Daemon`的叫法，`Daemon`的英文原意是"守护神"，在这里是"守护进程"的意思。

守护进程就是为了实现服务、功能的进程。比如，我们的 apache 服务就是服务，它是用来实现 Web 服务的。那么，启动 apache 服务的进程是哪个进程呢？就是`httpd`这个守护进程（`Daemon`）。也就是说，守护进程就是服务在后台运行的真实进程。

在 Linux 中就是通过启动`httpd`进程来启动 apache 服务的，你可以把`httpd`进程当作 apache 服务的别名来理解。
## 服务的分类
Linux 中的服务按照安装方法不同可以分为 RPM 包默认安装的服务和源码包安装的服务两大类。其中，RPM 包默认安装的服务又因为启动与自启动管理方法不同分为独立的服务和基于 xinetd 的服务。

{% asset_img 1.jpg 服务分类的关系图 %}

Linux 中常见的软件包有两种：一种是 RPM 包；另一种是源码包。那么，通过 RPM 包安装的系统服务就是 RPM 包默认安装的服务（因为 Linux 光盘中全是 RPM 包，Linux 系统也是通过 RPM 包安装的，所以我们把 RPM 包又叫作系统默认包），通过源码包安装的系统服务就是源码包安装的服务。

源码包是开源的，自定义性强，通过编译安装更加适合系统，但是安装速度较慢，编译时容易报错。RPM 包是经过编译的软件包，安装更快速，不易报错，但不再是开源的。

以上这些特点都是软件包本身的特点，但是软件包一旦安装到 Linux 系统上，它们的区别是什么呢？

最主要的区别就是安装位置不同，源码包安装到我们手工指定的位置当中，而 RPM 包安装到系统默认位置当中（可以通过"rpm -ql 包名"命令查询）。也就是说，RPM 包安装到系统默认位置，可以被服务管理命令识别；但是源码包安装到手工指定位置，当然就不能被服务管理命令识别了（可以手工修改为被服务管理命令识别）。

所以，RPM 包默认安装的服务和源码包安装的服务的管理方法不同，我们把它们当成不同的服务分类。服务分类说明如下。

RPM 包默认安装的服务。这些服务是通过 RPM 包安装的，可以被服务管理命令识别。

这些服务又可以分为两种：
* 独立的服务：就是独立启动的意思，这种服务可以自行启动，而不用依赖其他的管理服务。因为不依赖其他的管理服务，所以，当客户端请求访问时，独立的服务响应请求更快速。目前，Linux 中的大多数服务都是独立的服务，如 apache 服务、FTP 服务、Samba 服务等。
& 基于 xinetd 的服务：这种服务就不能独立启动了，而要依靠管理服务来调用。这个负责管理的服务就是 xinetd 服务。xinetd 服务是系统的超级守护进程，其作用就是管理不能独立启动的服务。当有客户端请求时，先请求 xinetd 服务，由 xinetd 服务去唤醒相对应的服务。当客户端请求结束后，被唤醒的服务会关闭并释放资源。这样做的好处是只需要持续启动 xinetd 服务，而其他基于 xinetd 的服务只有在需要时才被启动，不会占用过多的服务器资源。但是这种服务由于在有客户端请求时才会被唤醒，所以响应时间相对较长。

源码包安装的服务。这些服务是通过源码包安装的，所以安装位置都是手工指定的。由于不能被系统中的服务管理命令直接识别，所以这些服务的启动与自启动方法一般都是源码包设计好的。每个源码包的启动脚本都不一样，一般需要查看说明文档才能确定。
## 查询已经安装的服务和区分服务
首先要区分 RPM 包默认安装的服务和源码包安装的服务。源码包安装的服务是不能被服务管理命令直接找到的，而且一般会安装到`/usr/local/`目录中。

也就是说，在`/usr/local/`目录中的服务都应该是通过源码包安装的服务。RPM 包默认安装的服务都会安装到系统默认位置，所以是可以被服务管理命令（如`service、chkconfig`）识别的。

其次，在 RPM 包默认安装的服务中怎么区分独立的服务和基于 xinetd 的服务？这就要依靠`chkconfig`命令了。`chkconfig`是管理 RPM 包默认安装的服务的自启动的命令。使用这条命令还能看到 RPM 包默认安装的所有服务。
```
[root@localhost ~]# chkconfig --list [服务名]
```
选项：
* `--list`：列出 RPM 包默认安装的所有服务的自启动状态；

```
[root@localhost ~]# chkconfig -list
#列出系统中RPM包默认安装的所有服务的自启动状态
abrt-ccpp 0:关闭 1:关闭 2:关闭 3:启用 4:关闭 5:启用 6:关闭
abrt-oops 0:关闭 1:关闭 2:关闭 3:启用 4:关闭 5:启用 6:关闭
…省略部分输出…
udev-post 0:关闭 1:启用 2:启用 3:启用 4:启用 5:启用 6:关闭
ypbind 0:关闭 1:关闭 2:关闭 3:关闭 4:关闭 5:关闭 6:关闭
```
这条命令的第一列为服务的名称，后面的 0~6 代表在不同的运行级别中这个服务是否开启时自动启动。这些服务都是独立的服务，因为它们不需要依赖其他任何服务就可以在相应的运行级别启动或自启动。但是没有看到基于 xinetd 的服务，那是因为系统中默认没有安装 xinetd 这个超级守护进程，需要我们手工安装。 

安装命令如下：
```
[root@localhost ~]# rpm -ivh /mnt/cdrom/Packages/ xinetd-2.3.14-34.el6.i686.rpm
Preparing...
###############
[100%]
1:xinetd
###############
[100%]
#xinetd超级守护进程
```
这里需要注意的是，在 Linux 中基于 xinetd 的服务越来越少，原先很多基于 xinetd 的服务在新版本的 Linux 中已经变成了独立的服务。安装完 xinetd 超级守护进程之后，我们再查看一下，命令如下：
```
[root@localhost ~]# chkconfig --list
abrt-ccpp 0:关闭 1:关闭 2:关闭 3:启用 4:关闭 5:启用 6:关闭
abrt-oops 0:关闭 1:关闭 2:关闭 3:启用 4:关闭 5:启用 6:关闭
…省略部分输出…
udev-post 0:关闭 1:启用 2:启用 3：启用 4:启用 5:启用 6:关闭
xinetd 0:关闭 1:关闭 2:关闭 3:启用 4:启用 5:启用 6:关闭
ypbind 0:关闭 1:关闭 2:关闭 3:关闭 4:关闭 5:关闭 6:关闭
基于 xinetd 的服务：
chargen-dgram：关闭
chargen-stream：关闭
cvs：关闭
daytime-dgram：关闭
daytime-stream：关闭
discard-dgram：关闭
discard-stream：关闭
echo-dgram：关闭
echo-stream：关闭
rsync：关闭
tcpmux-server：关闭
time-dgram：关闭
time-stream：关闭
```
在刚刚的独立的服务之下出现了一些基于 xinetd 的服务，这些服务没有自己的运行级别，因为它们不是独立的服务，到底在哪个运行级别可以自启动，则要看 xinetd 服务是在哪个运行级别自启动的。
# linux端口
服务是给系统提供功能的，在系统中除了有系统服务，还有网络服务。而每个网络服务都有自己的端口，一般端口号都是固定的。

为了统一整个互联网的端口和网络服务的对应关系，以便让所有的主机都能使用相同的机制来请求或提供服务，同一个服务使用相同的端口，这就是协议。

计算机中的协议主要分为两大类：
* 面向连接的可靠的TCP协议（`Transmission Control Protocol`，传输控制协议）；
* 面向无连接的不可靠的UDP协议（`User Datagram Protocol`，用户数据报协议）；

这两种协议都支持 216，也就是 65535 个端口。系统给我们提供了服务与端口的对应文件`/etc/services`。
## 查询系统中已经启动的服务
既然每个网络服务对应的端口是固定的，那么是否可以通过查询服务器中开启的端口，来判断当前服务器开启了哪些服务？

当然是可以的。虽然判断服务器中开启的服务还有其他方法（如通过`ps`命令），但是通过端口的方法查看最为准确。命令格式如下：
```
[root@localhost ~]# netstat 选项
```
选项：
* `-a`：列出系统中所有网络连接，包括已经连接的网络服务、监听的网络服务和 Socket 套接字；
* `-t`：列出 TCP 数据；
* `-u`：列出 UDF 数据；
* `-l`：列出正在监听的网络服务（不包含已经连接的网络服务）；
* `-n`：用端口号来显示而不用服务名；
* `-p`：列出该服务的进程 ID (PID)；

```
[root@localhost ~]# netstat -tlunp
#列出系统中所有已经启动的服务（已经监听的端口），但不包含已经连接的网络服务
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      1204/redis-server 1 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1257/nginx: master  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1191/sshd           
tcp6       0      0 :::3306                 :::*                    LISTEN      1606/mysqld         
tcp6       0      0 ::1:6379                :::*                    LISTEN      1204/redis-server 1 
tcp6       0      0 :::8080                 :::*                    LISTEN      1253/java           
tcp6       0      0 :::22                   :::*                    LISTEN      1191/sshd           
udp        0      0 127.0.0.1:323           0.0.0.0:*                           898/chronyd         
udp6       0      0 ::1:323                 :::*                                898/chronyd     
```
执行这条命令会看到服务器上所有已经开启的端口，也就是说，通过这些端口就可以知道当前服务器上开启了哪些服务。

解释一下命令的执行结果：
* `Proto`：数据包的协议。分为 TCP 和 UDP 数据包；
* `Recv-Q`：表示收到的数据已经在本地接收缓冲，但是还没有被进程取走的数据包数量；
* `Send-Q`：对方没有收到的数据包数量；或者没有 Ack 回复的，还在本地缓冲区的数据包数量；
* `Local Address`：本地 IP : 端口。通过端口可以知道本机开启了哪些服务；
* `Foreign Address`：远程主机：端口。也就是远程是哪个 IP、使用哪个端口连接到本机。由于这条命令只能查看监听端口，所以没有 IP 连接到到本机；
* `State`：连接状态。主要有已经建立连接（`ESTABLISED`）和监听（`LISTEN`）两种状态，当前只能查看监听状态；
* `PID/Program name`：进程 ID 和进程命令；

```
[root@localhost ~]# netstat -an
#查看所有的网络连接，包括已连接的网络服务、监听的网络服务和Socket套接字
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      1204/redis-server 1 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1257/nginx: master  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1191/sshd           
tcp6       0      0 :::3306                 :::*                    LISTEN      1606/mysqld         
tcp6       0      0 ::1:6379                :::*                    LISTEN      1204/redis-server 1 
tcp6       0      0 :::8080                 :::*                    LISTEN      1253/java           
tcp6       0      0 :::22                   :::*                    LISTEN      1191/sshd           
udp        0      0 127.0.0.1:323           0.0.0.0:*                           898/chronyd         
udp6       0      0 ::1:323                 :::*                                898/chronyd         

[root@localhost ~]# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0    244 192.168.10.10:22        192.168.10.2:55812      ESTABLISHED
tcp        0      0 127.0.0.1:6379          127.0.0.1:53630         ESTABLISHED
tcp6       0      0 :::3306                 :::*                    LISTEN     
tcp6       0      0 ::1:6379                :::*                    LISTEN     
tcp6       0      0 :::8080                 :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 127.0.0.1:3306          127.0.0.1:60784         ESTABLISHED
tcp6       0      0 127.0.0.1:3306          127.0.0.1:60782         ESTABLISHED
tcp6       0      0 127.0.0.1:60780         127.0.0.1:3306          ESTABLISHED
...省略部分输出...
```
执行`netstat -an`命令能査看更多的信息，在`Stated`中也看到了已经建立的连接（`ESTABLISED`）。这是`ssh`远程管理命令产生的连接，`ssh`对应的端口是 22。

而且我们还看到了`Socket`套接字。在服务器上，除网络服务可以绑定端口，用端口来接收客户端的请求数据外，系统中的网络程序或我们自己开发的网络程序也可以绑定端口，用端口来接收客户端的请求数据。这些网络程序就是通过`Socket`套接字来绑定端口的。也就是说，网络服务或网络程序要想在网络中传递数据，必须利用`Socket`套接字绑定端口，并进行数据传递。

使用`netstat -an`命令查看到的这些`Socket`套接字虽然不是网络服务，但是同样会占用端口，并在网络中传递数据。

解释一下`Socket`套接字的输出：
* `Proto`：协议，一般是unix；
* `RefCnt`：连接到此`Socket`的进程数量；
* `Flags`：连接标识；
* `Type`：Socket访问类型；
* `State`：状态，`LISTENING`表示监听，`CONNECTED`表示已经建立连接；
* `I-Node`：程序文件的`i`节点号；
* `Path`：`Socket`程序的路径，或者相关数据的输出路径；

# 独立服务管理（RPM包的启动与自启动）
RPM 包默认安装的服务分为独立的服务和基于 xinetd 的服务。
## 独立服务的启动管理
独立的服务要想启动，主要有两种方法。
### 1.使用/etc/init.d/目录中的启动脚本来启动独立的服务
既然所有独立服务的启动脚本都存放在`/etc/init.d/`目录中，那么，调用这些脚本就可以启动独立的服务了。这种启动方式是推荐启动方式，命令格式如下：
```
[root@localhost ~]#/etc/init.d独立服务名 start| stop|status|restart|...
```
参数：
* `start`：启动服务；
* `stop`：停止服务；
* `status`：查看服务状态；
* `restart`：重启动服务；

我们以启动 RPM 包默认安装的`httpd`服务为例，命令如下：
```
[root@localhost ~]# /etc/init.d/httpd start
正在启动httpd:
[确定]
#启动httpd服务
[root@localhost ~]# /etc/init.d/httpd status
httpd (pid 13313)正在运行…
#查询httpd服务状态，并能够看到httpd服务的PID
[root@localhost ~]#/etc/init.d/httpd stop
停止 httpd:
[确定]
#停止httpd服务
[root@localhost ~]#/etc/init.d/httpd restart
停止httpd:
[失败]
正在启动httpd:
[确定]
重启动httpd服务
```
### 2.使用service命令来启动独立的服务
在 CentOS 系统中，我们还可以依赖`service`命令来启动独立的服务。`service`命令实际上只是一个脚本，这个脚本仍然需要调用`/etc/init.d/`中的启动脚本来启动独立的服务。而且`service`命令是红帽系列 Linux 的专有命令，其他的 Linux 发行版本不一定拥有这条命令。

`service`命令格式如下：
```
[root@localhost ~]# service 独立服务名 start|stop|restart|...
```
例如：
```
[root@localhost ~]# service httpd restart
停止httpd:
[确定]
正在启动httpd:
[确定]
```
命令比输入`/etc/init_d/`目录要稍微简单。`service`命令还可以查看所有独立服务的启动状态，这是一个常用功能，命令格式如下：
```
[root@localhost ~]# service --status-all
```
选项：
* `--status-all`：列出所有独立服务的启动状态;

例如：
```
abrtd(pid 1505)正在运行…
abrt-dumpoops(pid 1513)正在运行…
acpid(pid 1312)正在运行...
…省略部分输出…
```
随着`httpd`服务的启动和停止，使用`netstat -tlun`命令就会看到 80 端口出现和消失。这也就说明 apache 服务绑定的口就是 80，所以我们可以端口是否在服务器中出现来判断 apache 服务是否启动。
## 独立服务的自启动管理
自启动指的是在系统之后，服务是否随着系统的启动而自动启动。如果启动了某个服务，那么这个服务会在系统重启之后启动吗？

答案是不知道，因为启动命令只负责启动服务，而和服务的自启动完全没有关系。同样地，自启动命令只管服务是否会在系统重启之后启动，而和当前系统中的服务是否启动没有关系。

独立服务的自启动方法有三种。
### 1.使用 chkconfig 服务自启动管理命令
第一种方法是利用`chkconfig`服务自启动管理命令来管理独立服务的自启动，这条命令的用法如下：
```
[root@localhost ~]# chkconfig --list
```
使用`chkconfig`命令除了可以查看所有 RPM 包默认安装服务的自启动状态，也可以修改和设置 RPM 包默认安装服务的自启动状态，只是独立的服务和基于 xinetd 的服务的设定方法稍有不同。我们先来看看独立的服务如何设置。命令格式如下：
```
[root@localhost ~]# chkconfig [--level 运行级别][独立服务名][on|off]
```
选项：
* `--level`: 设定在哪个运行级别中开机自启动（`on`），或者关闭自启动（`off`）；

例如：
```
[root@localhost ~]# chkconfig --list | grep httpd
httpd 0:关闭 1:关闭 2:关闭 3:关闭 4:关闭 5:关闭 6:关闭
#查询httpd的自启动状态。所有的级别都是不自启动的
[root@localhost ~]# chkconfig --level 2345 httpd on
#设置apache服务在进入2、3、4、5级别时自启动
[root@localhost ~]# chkconfig --list | grep httpd
httpd 0:关闭 1:关闭 2:启用 3:启用 4:启用 5:启用 6:关闭
#查询apache服务的自启动状态。发现在2、3、4、5这4个运行级别中变为了"启用"
```
还记得 0~6 这 7 个 Linux 的运行级别吗？如果在 0~6 这 7 个运行级别中服务都显示"关闭"，则该服务不自启动。如果在某个运行级别中显示"启用"，则代表在进入这个运行级别时，该服务开机自启动。

服务的自启动方法和服务的启动方法是不通用的，我们做一个实验验证一下。命令如下：
```
[root@localhost ~]# /etc/init.d/httpd status
httpd已停
#查询apache服务状态，是已经停止的
[root@localhost ~]# chkconfig --level 2345 httpd on
#设置apache服务在进入2、3、4、5级别时自启动
[root@localhost ~]# chkconfig --list|grep httpd
httpd 0:关闭 1:关闭 2:启用 3:启用 4:启用 5:启用 6:关闭
#查看一下，自启动已经生效
[root@localhost ~]#/etc/init.d/httpd status
httpd已停
#但是apache服务在当前系统中还是关闭的
```
虽然 apache 被设置为自启动，但是当前系统中的 apache 是没有启动的，所以启动和自启动是独立的。
### 2.修改 /etc/rc.d/rc.local 文件，设置服务自启动
第二种方法就是修改`/etc/rc.d/rc.local`文件，在文件中加入服务的启动命令。这个文件是在系统启动时，在输入用户名和密码之前最后读取的文件（注意：`/etc/rc.d/rc.loca`和`/etc/rc.local`文件是软链接，修改哪个文件都可以）。这个文件中有什么命令，都会在系统启动时调用。

如果我们把服务的启动命令放入这个文件，这个服务就会在开机时自启动。命令如下：
```
[root@localhost ~]#vi /etc/rc.d/rc.local
#!/bin/sh
#
#This script will be executed *after* all the other init scripts.
#You can put your own initialization stuff in here if you don't want to do the full Sys V style init stuff.
touch /var/lock/subsys/local
/etc/rc.d/init.d/httpd start
#在文件中加入apache的启动命令
```
这样，只要重启之后，apache 服务就会开机自启动了。推荐大家使用这种方法管理服务的自启动，有两点好处：
第一，如果大家都采用这种方法管理服务的自启动，当我们碰到一台陌生的服务器时，只要查看这个文件就知道这台服务器到底自启动了哪些服务，便于集中管理。
第二，`chkconfig`命令只能识别 RPM 包默认安装的服务，而不能识别源码包安装的服务。 源码包安装的服务的自启动也是通过`/etc/rc.d/rc.local`文件实现的，所以不会出现同一台服务器自启动了两种安装方法的同一个服务。

还要注意一下，修改`/etc/rc.d/rc.local`配置文件的自启动方法和`chkconfig`命令的自启动方法是两种不同的自启动方法。所以，就算通过修改`/etc/rc.d/rc.local`配置文件的方法让某个独立的服务自启动了，执行`chkconfig --list`命令并不到有什么变化。
### 3.使用 ntsysv 命令管理自启动
第三种方法是使用`ntsysv`命令调用窗口模式来管理服务的自启动。命令格式如下：
```
[root@localhost ~]# ntsysv [--level 运行级别]
```
选项：
* `--level`运行级别：可以指定设定自启动的运行级别；

```
[root@localhost ~]# ntsysv --level 235
#只设定2、3、5级别的服务自启动
[root@localhost ~]# ntsysv
#按默认的运行级别设置服务自启动
```
执行命令后，会和 setup 命令类似，出现命令界面，如图 1 所示。


图 1 ntsysv命令界面

这个命令的操作是这样的：
上下键：在不同服务之间移动；
空格键：选定或取消服务的自启动。也就是在服务之前是否输入"*"；
Tab键：在不同项目之间切换；
F1键：显示服务的说明；

需要注意的是，ntsysv 命令不仅可以管理独立服务的自启动，也可以管理基于 xinetd 服务的自启动。也就是说，只要是 RPM 包默认安装的服务都能被 ntsysv 命令管理。但是源码包安装的服务不行。

`ntsysv`命令虽然简单，但它是红帽系列 Linux 的专有命令，其他的 Linux 发行版本不一定拥有这条命令，而且条命令也不能管理源码包安装的服务，所以我们推荐大家使用`/etc/rc.d/rc.local`文件来管理服务的自启动。
# 基于xinetd服务的管理方法详解
# 源码包服务管理
源码包服务的启动管理
源码包服务中所有的文件都会安装到指定目录当中，并且没有任何垃圾文件产生（Linux 的特性），所以服务的管理脚本程序也会安装到指定目录中。源码包服务的启动管理方式就是在服务的安装目录中找到管理脚本，然后执行这个脚本。

问题来了，每个服务的启动脚本都是不一样的，我们怎么确定每个服务的启动脚本呢？还记得在安装源码包服务时，我们强调需要査看每个服务的说明文档吗（一般是 INSTALL 或 READEM）？在这个说明文档中会明确地告诉大家服务的启动脚本是哪个文件。

我们用 apache 服务来举例。一般 apache 服务的安装位置是`/usr/local/apache2/`目录，那么 apache 服务的启动脚本就是 /usr/local/apache2/bin/apachectl 文件（查询 apache 说明文档得知）。启动命令如下：
```
[root@localhost ~]# /usr/local/apache2/bin/apachectl start|stop|restart|...
#源码包服务的启动管理
```
例如：
[root@localhost ~]# /usr/local/apache2/bin/apachectl start
#会启动源码包安装的apache服务

注意，不管是源码包安装的 apache，还是 RPM 包默认安装的 apache，虽然在一台服务器中都可以安装，但是只能启动一因为它们都会占用 80 端口。

源码包服务的启动方法就这一种，比 RPM 包默认安装的服务要简单一些。
## 源码包服务的自启动管理
源码包服务的白启动管理也不能依靠系统的服务管理命令，而只能把标准启动命令写入 /etc/rc.d/rc.local 文件中。系统在启动过程中读取 /etc/rc.d/rc.local 文件时，就会调用源码包服务的启动脚本，从而让该服务开机自启动。命令如下：
```
[root@localhost ~]# vi /etc/rc.d/rc.local
#修改自启动文件
#!/bin/sh
#This script will be executed *after* all the other init scripts.
#You can put your own initialization stuff in here if you don11
#want to do the full Sys V style init stuff.
touch /var/lock/subsys/local /usr/local/apache2/bin/apachectl start
#加入源码包服务的标准启动命令，保存退出，源码包安装的apache服务就被设为自启动了
```
## 让源码包服务被服务管理命令识别
在默认情况下，源码包服务是不能被系统的服务管理命令所识别和管理的，但是如果我们做一些设定，则也是可以让源码包服务被系统的服务管理命令所识别和管理的。不过笔者并不推荐大家这样做，因为这会让本来区别很明确的源码包服务和 RPM 包服务变得容易混淆，不利于系统维护和管理。

我们做一个实验，看看如何把源码包安装的 apache 服务变为和 RPM 包默认安装的 apache 服务一样，可以被 service、chkconfig、ntsysv 命令所识别。实验如下：
1) 卸载RPM包默认安装的apache服务
```
[root@localhost ~]# yum -y remove httpd
#卸载RPM包默认安装的apache服务,避免对实验产生影响（在生产服务器上慎用yum卸载，因为这有可能造成服务器崩溃)
[root@localhost ~]# service httpd start httpd:未被识别的服务
#因为服务被卸载,所以service命令不能识别httpd服务
```
2) 安装源码包的apache服务，并启动
```
#安装源码包的apache服务
[root@localhost ~]# /usr/local/apache2/bin/apachect1 start
[root@localhost ~]# netstat -tlun | grep 80
tcp 0 0 :::80 :::* LISTEN
#启动源码包安装的apache服务，查看端口确定已经启动
```
3) 让源码包安装的apache服务能被service命令管理启动
```
[root@localhost ~]# ln -s /usr/local/apache2/bin/apachectl /etc/±nit.d/apache
#service命令其实只是在/etc/init.d/目录中查找是否有服务的启动脚本，所以我们只需要做一个软链接,把源码包的启动脚本链接到/etc/init.d/目录中,就能被service命令所管理了。为了照顾大家的习惯，我把软链接文件命名为apache,注意这不是RPM包默认安装的apache服务
[root@localhost ~]# service apache restart
#虽然RPM包默认安装的apache服务被卸载了,但是service命令也能够生效
```
4) 让源码包安装的apache服务能被chkconfig命令管理自启动
```
[root@localhost ~]# vi /etc/init.d/apache
#修改源码包安装的apache服务的启动脚本(注意此文件是软链接,所以修改的还是源码包启动脚本)
#!/bin/sh
#
#chkconfig: 35 86 76
#指定httpd脚本可以被chkconfig命令所管理
#格式是：chkconfig：运行级别 启动顺序 关闭顺序
#这里我们让apache服务在3和5级别中能被chkconfig命令所管理，启动顺序是S86，关闭顺序是K76
#(自定顺序，不要和系统中已有的启动顺序冲突)
#description: source package apache
#说明，内容随意
#以上两句话必须加入,才能被chkconfig命令所识别 ...省略部分输出...
[root@localhost ~]# chkconfig --add apache
#让chkconfig命令能够管理源码包安装的apache服务
[root01ocalhost ~]# chkconfig --list | grep apache
apache 0:关闭 1:关闭 2:关闭 3:关闭 4:关闭 5:关闭 6:关闭
#很神奇吧,虽然RPM包默认安装的apache服务被删除了,但是chkconfig命令可以管理源码包安装的tapache服务
```
5) 让ntsysv命令可以管理源码包安装的apache服务
#ntsysv 命令其实和 chkconfig 命令使用同样的管理机制,也就是说,ntsysv 已经可以进行源码包安装 apache 服务的自启动管理了,如图 1 所示


图 1 ntsysv 命令识别 apache

总结一下，如果想让源码包服务被service命令所识别和管理，则只需做一个软链接把启动脚本链接到 /etc/init.d/ 目录中即可。要想让源码包服务被 chkconfig 命令所是被，除了需要把服务的启动脚本链接到 /etc/init.d/ 目录中，还要修改这个启动脚本，在启动脚本的开头加入如下内容：
#chkconfig:运行级别 启动顺序 关闭
#description:说明

然后需要使用"chkconfig--add 服务名"的方式把服务加入 chkconfig 命令的管理中。命令格式如下：
```
[root@localhost ~]# chkconfig [选项][服务名]
```
选项：
* `-add`：把服务加入 chkconfig 命令的管理中；
* `-del`：把服务从 chkconfig 命令的管理中删除；

例如：
```
[root@localhost ~]# chkconfig -del httpd
#把apache服务从chkconfig命令的管理中删除
```
# 常见服务类别及功能
Linux 中的服务数量非常多，我们在学习时一直使用 apache 服务作为实例。很多人会产生困惑：其他的服务都是干什么的呢？它们有什么作用呢？是不是必须启动的呢？

在生产服务器上，安装完 Linux 之后有一步重要的工作，就是服务优化。也就是关闭不需要的服务，只开启需要的服务。因为服务启动得越多，占用的系统资源就越多，而且被攻击的可能性也増加了。如果要进行服务优化，就需要知道这些服务都有什么作用，如表 1 所示。

表 1 Linux中常见服务的作用
服务名称	功能简介	建议
acpid	电源管理接口。如果是笔记本电脑用户，则建议开启，可以监听内核层的相关电源事件	开启
anacron	系统的定时任务程序。是 cron 的一个子系统，如果定时任务错过了执行时间，则可以通过 anacron 继续唤醒执行	关闭
alsasound	alsa 声卡驱动。如果使用 alsa 声卡，则开启	关闭
apmd	电源管理模块。如果支持 acpid，就不需要 apmd，可以关闭	关闭
atd	指定系统在特定时间执行某个任务，只能执行一次。如果需要则开启，但我们一般使用 crond 来执行循环定时任务	关闭
auditd	审核子系统。如果开启了此服务，那么 SELinux 的审核信息会写入 /var/log/audit/ audit.log 文件；如果不开启，那么审核信息会记录在 syslog 中	开启
autofs	让服务器可以自动挂载网络中其他服务器的共享数据,一般用来自动挂载 NFS 服务。如果没有 NFS 服务，则建议关闭	关闭
avahi-daemon	avahi 是 zeroconf 协议的实现，它可以在没有 DNS 服务的局域网里发现基于 zeroconf 协议的设备和服务。除非有兼容设备或使用 zeroconf 协议，否则关闭	关闭
bluetooth	蓝牙设备支持。一般不会在服务器上启用蓝牙设备，关闭它	关闭
capi	仅对使用 ISND 设备的用户有用	关闭
chargen-dgram	使用 UDP 协议的 chargen server。其主要提供类似远程打字的功能	关闭
chargen-stream	同上	关闭
cpuspeed	可以用来调整 CPU 的频率。当闲置时，可以自动降低 CPU 频率来节省电量	开启
crond	系统的定时任务，一般的 Linux 服务器都需要定时任务来协助系统维护。建议开启	开启
cvs	一个版本控制系统	关闭
daytime-dgram	使用 TCP 协议的 daytime 守护进程，该协议为客户机实现从远程服务器获取日期和时间的功能	关闭
daytime-slream	同上	关闭
dovecot	邮件服务中 POP3/IMAP 服务的守护进程，主要用来接收信件。如果启动了邮件服务则开启：否则关闭	关闭
echo-dgram	服务器回显客户服务的进程	关闭
echo-stream	同上	关闭
firstboot	系统安装完成后，有一个欢迎界面，需要对系统进行初始设定，这就是这个服务的作用。既然不是第一次启动了，则建议关闭	关闭
gpm	在字符终端 (ttyl~tty6) 中可以使用鼠标复制和粘贴，这就是这个服务的功能	开启
haldaemon	检测和支持 USB 设备。如果是服务器则可以关闭，个人机则建议开启	关闭
hidd	蓝牙鼠标、键盘等蓝牙设备检测。必须启动 bluetooth 服务	关闭
hplip	HP 打印机支持，如果没有 HP 打印机则关闭	关闭
httpd	apache 服务的守护进程。如果需要启动 apache，就开启	开启
ip6tables	IPv6 的防火墙。目前 IPv6 协议并没有使用，可以关闭	关闭
iptables	防火墙功能。Linux 中的防火墙是内核支持功能。这是服务器的主要防护手段，必须开启	开启
irda	IrDA 提供红外线设备（笔记本电脑、PDA’s、手机、计算器等）间的通信支持。建议关闭	关闭
irqbalance	支持多核处理器，让 CPU 可以自动分配系统中断（IRQ)，提高系统性能。目前服务器多是多核 CPU，请开启	开启
isdn	使用 ISDN 设备连接网络。目前主流的联网方式是光纤接入和 ADSL，ISDN 己经非常少见，请关闭	关闭
kudzu	该服务可以在开机时进行硬件检测，并会调用相关的设置软件。建议关闭，仅在需要时开启	关闭
lvm2-monitor	该服务可以让系统支持LVM逻辑卷组，如果分区采用的是LVM方式，那么应该开启。建议开启	开启
mcstrans	SELinux 的支持服务。建议开启	开启
mdmonitor	该服务用来监测 Software RAID 或 LVM 的信息。不是必需服务，建议关闭	关闭
mdmpd	该服务用来监测 Multi-Path 设备。不是必需服务，建议关闭	关闭
messagebus	这是 Linux 的 IPC (Interprocess Communication，进程间通信）服务，用来在各个软件中交换信息。建议关闭	关闭
microcode _ctl	Intel 系列的 CPU 可以通过这个服务支持额外的微指令集。建议关闭	关闭
mysqld	MySQL 数据库服务器。如果需要就开启；否则关闭	开启
named	DNS 服务的守护进程，用来进行域名解析。如果是 DNS 服务器则开启；否则关闭	关闭
netfs	该服务用于在系统启动时自动挂载网络中的共享文件空间，比如 NFS、Samba 等。 需要就开启，否则关闭	关闭
network	提供网络设罝功能。通过这个服务来管理网络，建议开启	开启
nfs	NFS (Network File System) 服务，Linux 与 Linux 之间的文件共享服务。需要就开启，否则关闭	关闭
nfslock	在 Linux 中如果使用了 NFS 服务，那么，为了避免同一个文件被不同的用户同时编辑，所以有这个锁服务。有 NFS 时开启，否则关闭	关闭
ntpd	该服务可以通过互联网自动更新系统时间.使系统时间永远准确。需要则开启，但不是必需服务	关闭
pcscd	智能卡检测服务，可以关闭	关闭
portmap	用在远程过程调用 (RPC) 的服务，如果没有任何 RPC 服务，则可以关闭。主要是 NFS 和 NIS 服务需要	关闭
psacct	该守护进程支持几个监控进程活动的工具	关闭
rdisc	客户端 ICMP 路由协议	关闭
readahead_early	在系统开启的时候，先将某些进程加载入内存整理，可以加快启动速度	关闭
readahead_later	同上	关闭
restorecond	用于给 SELinux 监测和重新加载正确的文件上下文。如果开启 SELinux，则需要开启	关闭
rpcgssd	与 NFS 有关的客户端功能。如果没有 NFS 就关闭	关闭
rpcidmapd	同上	关闭
rsync	远程数据备份守护进程	关闭
sendmail	sendmail 邮件服务的守护进程。如果有邮件服务就开启；否则关闭	关闭
setroubleshoot	该服务用于将 SELinux 相关信息记录在日志 /var/log/messages 中。建议开启	开启
smartd	该服务用于自动检测硬盘状态。建议开启	开启
smb	网络服务 samba 的守护进程。可以让 Linux 和 Windows 之间共享数据。如果需要则开启	关闭
squid	代理服务的守护进程。如果需要则开启：否则关闭	关闭
sshd	ssh 加密远程登录管理的服务。服务器的远程管理必须使用此服务，不要关闭	开启
syslog	日志的守护进程	开启
vsftpd	vsftp 服务的守护进程。如果需要 FTP 服务则开启；否则关闭	关闭
xfs	这是 X Window 的字体守护进程，为图形界面提供字体服务。如果不启动图形界面，就不用开启	关闭
xinetd	超级守护进程。如果有依赖 xinetd 的服务，就必须开启	开启
ypbind	为 NIS (网络信息系统）客户机激活 ypbind 服务进程	关闭
yum-updatesd	yum 的在线升级服务	关闭
# sar命令详解：分析系统性能
sar 命令很强大，是分析系统性能的重要工具之一，通过该命令可以全面地获取系统的 CPU、运行队列、磁盘读写（I/O）、分区（交换区）、内存、CPU 中断和网络等性能数据。

sar 命令的基本格式如下：
```
[root@localhost ~]# sar [options] [-o filename] interval [count]
```
此命令格式中，各个参数的含义如下：
-o filename：其中，filename 为文件名，此选项表示将命令结果以二进制格式存放在文件中；
interval：表示采样间隔时间，该参数必须手动设置；
count：表示采样次数，是可选参数，其默认值为 1；
options：为命令行选项，由于 sar 命令提供的选项很多，这里不再一一介绍，仅列举出常用的一些选项及对应的功能，如表 1 所示。

表 1 sar 命令行选项及功能
sar命令选项	功能
-A	显示系统所有资源设备（CPU、内存、磁盘）的运行状况。
-u	显示系统所有 CPU 在采样时间内的负载状态。
-P	显示当前系统中指定 CPU 的使用情况。
-d	显示系统所有硬盘设备在采样时间内的使用状态。
-r	显示系统内存在采样时间内的使用情况。
-b	显示缓冲区在采样时间内的使用情况。
-v	显示 inode 节点、文件和其他内核表的统计信息。
-n	显示网络运行状态，此选项后可跟 DEV（显示网络接口信息）、EDEV（显示网络错误的统计数据）、SOCK（显示套接字信息）和 FULL（等同于使用 DEV、EDEV和SOCK）等，有关更多的选项，可通过执行 man sar 命令查看。
-q	显示运行列表中的进程数、进程大小、系统平均负载等。
-R	显示进程在采样时的活动情况。
-y	显示终端设备在采样时间的活动情况。
-w	显示系统交换活动在采样时间内的状态。
有关 sar 命令更多可用的选项及功能，可通过执行 man sar 命令查看。


【例 1】
如果想要查看系统 CPU 的整理负载状况，每 3 秒统计一次，统计 5 次，可以执行如下命令：
```
[root@localhost ~]# sar -u 3 5
Linux 2.6.32-431.el6.x86_64 (localhost)     10/25/2019     _x86_64_    (1 CPU)

06:18:23 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
06:18:26 AM     all     12.11      0.00      2.77      3.11      0.00     82.01
06:18:29 AM     all      6.55      0.00      2.07      0.00      0.00     91.38
06:18:32 AM     all      6.60      0.00      2.08      0.00      0.00     91.32
06:18:35 AM     all     10.21      0.00      1.76      0.00      0.00     88.03
06:18:38 AM     all      8.71      0.00      1.74      0.00      0.00     89.55
Average:        all      8.83      0.00      2.09      0.63      0.00     88.46
```
此输出结果中，各个列表项的含义分别如下：
%user：用于表示用户模式下消耗的 CPU 时间的比例；
%nice：通过 nice 改变了进程调度优先级的进程，在用户模式下消耗的 CPU 时间的比例；
%system：系统模式下消耗的 CPU 时间的比例；
%iowait：CPU 等待磁盘 I/O 导致空闲状态消耗的时间比例；
%steal：利用 Xen 等操作系统虚拟化技术，等待其它虚拟 CPU 计算占用的时间比例；
%idle：CPU 空闲时间比例。

【例 2】
如果想要查看系统磁盘的读写性能，可执行如下命令：
```
[root@localhost ~]# sar -d 3 5
Linux 2.6.32-431.el6.x86_64 (localhost)     10/25/2019     _x86_64_    (1 CPU)

06:36:52 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
06:36:55 AM    dev8-0      3.38      0.00    502.26    148.44      0.08     24.11      4.56      1.54

06:36:55 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
06:36:58 AM    dev8-0      1.49      0.00     29.85     20.00      0.00      1.75      0.75      0.11

06:36:58 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
06:37:01 AM    dev8-0     68.26      6.96  53982.61    790.93      3.22     47.23      3.54     24.17

06:37:01 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
06:37:04 AM    dev8-0    111.69   3961.29    154.84     36.85      1.05      9.42      3.44     38.43

06:37:04 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
06:37:07 AM    dev8-0      1.67    136.00      2.67     83.20      0.01      6.20      6.00      1.00

Average:          DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:       dev8-0     34.45    781.10   9601.22    301.36      0.78     22.74      3.50     12.07
```
此输出结果中，各个列表头的含义如下：
tps：每秒从物理磁盘 I/O 的次数。注意，多个逻辑请求会被合并为一个 I/O 磁盘请求，一次传输的大小是不确定的；
rd_sec/s：每秒读扇区的次数；
wr_sec/s：每秒写扇区的次数；
avgrq-sz：平均每次设备 I/O 操作的数据大小（扇区）；
avgqu-sz：磁盘请求队列的平均长度；
await：从请求磁盘操作到系统完成处理，每次请求的平均消耗时间，包括请求队列等待时间，单位是毫秒（1 秒=1000 毫秒）；
svctm：系统处理每次请求的平均时间，不包括在请求队列中消耗的时间；
%util：I/O 请求占 CPU 的百分比，比率越大，说明越饱和。

除此之外，如果想要查看系统内存使用情况，可以执行sar -r 5 3命令；如果要想查看网络运行状态，可执行sar -n DEV 5 3命令，等等。有关其它参数的用法，这里不再给出具体实例，有兴趣的读者可自行测试，观察运行结果。
