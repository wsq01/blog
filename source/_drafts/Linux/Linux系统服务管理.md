



# linux端口
服务是给系统提供功能的，在系统中除了有系统服务，还有网络服务。而每个网络服务都有自己的端口，一般端口号都是固定的。

为了统一整个互联网的端口和网络服务的对应关系，以便让所有的主机都能使用相同的机制来请求或提供服务，同一个服务使用相同的端口，这就是协议。

计算机中的协议主要分为两大类：
面向连接的可靠的TCP协议（Transmission Control Protocol，传输控制协议）；
面向无连接的不可靠的UDP协议（User Datagram Protocol，用户数据报协议）；

这两种协议都支持 216，也就是 65535 个端口。这么多端口怎么记忆呢？系统给我们提供了服务与端口的对应文件 /etc/services。

查询系统中已经启动的服务
既然每个网络服务对应的端口是固定的，那么是否可以通过查询服务器中开启的端口，来判断当前服务器开启了哪些服务？

当然是可以的。虽然判断服务器中开启的服务还有其他方法（如通过ps命令），但是通过端口的方法查看最为准确。命令格式如下：
[root@localhost ~]# netstat 选项

选项：
-a：列出系统中所有网络连接，包括已经连接的网络服务、监听的网络服务和 Socket 套接字；
-t：列出 TCP 数据；
-u：列出 UDF 数据；
-l：列出正在监听的网络服务（不包含已经连接的网络服务）；
-n：用端口号来显示而不用服务名；
-p：列出该服务的进程 ID (PID)；

举个例子：
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
Proto：数据包的协议。分为 TCP 和 UDP 数据包；
Recv-Q：表示收到的数据已经在本地接收缓冲，但是还没有被进程取走的数据包数量；
Send-Q：对方没有收到的数据包数量；或者没有 Ack 回复的，还在本地缓冲区的数据包数量；
Local Address：本地 IP : 端口。通过端口可以知道本机开启了哪些服务；
Foreign Address：远程主机：端口。也就是远程是哪个 IP、使用哪个端口连接到本机。由于这条命令只能查看监听端口，所以没有 IP 连接到到本机；
State:连接状态。主要有已经建立连接（ESTABLISED）和监听（LISTEN）两种状态，当前只能查看监听状态；
PID/Program name：进程 ID 和进程命令；

再举个例子：
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

执行"netstat -an"命令能査看更多的信息，在 Stated 中也看到了已经建立的连接（ESTABLISED）。这是 ssh 远程管理命令产生的连接，ssh 对应的端口是 22。

而且我们还看到了 Socket 套接字。在服务器上，除网络服务可以绑定端口，用端口来接收客户端的请求数据外，系统中的网络程序或我们自己开发的网络程序也可以绑定端口，用端口来接收客户端的请求数据。这些网络程序就是通过 Socket 套接字来绑定端口的。也就是说，网络服务或网络程序要想在网络中传递数据，必须利用 Socke 套接字绑定端口，并进行数据传递。

使用"netstat -an"命令查看到的这些 Socke 套接字虽然不是网络服务，但是同样会占用端口，并在网络中传递数据。

解释一下 Socket 套接字的输出：
Proto：协议，一般是unix；
RefCnt：连接到此Socket的进程数量；
Flags：连接标识；
Type：Socket访问类型；
State：状态，LISTENING表示监听，CONNECTED表示已经建立连接；
I-Node：程序文件的 i 节点号；
Path：Socke程序的路径，或者相关数据的输出路径；
# 独立服务管理（RPM包的启动与自启动）
我们知道，RPM 包默认安装的服务分为独立的服务和基于 xinetd 的服务。
## 独立服务的启动管理
独立的服务要想启动，主要有两种方法。
### 1.使用/etc/init.d/目录中的启动脚本来启动独立的服务
既然所有独立服务的启动脚本都存放在 /etc/init.d/ 目录中，那么，调用这些脚本就可以启动独立的服务了。这种启动方式是推荐启动方式，命令格式如下：
```
[root@localhost ~]#/etc/init.d独立服务名 start| stop|status|restart|...
```
参数：
* `start`：启动服务；
* `stop`：停止服务；
* `status`：查看服务状态；
* `restart`：重启动服务；

我们以启动 RPM 包默认安装的 httpd 服务为例，命令如下：
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
命令比输入 /etc/init_d/ 目录要稍微简单。service 命令还可以查看所有独立服务的启动状态，这是一个常用功能，命令格式如下：
```
[root@localhost ~]# service --status-all
```
选项：
--status-all：列出所有独立服务的启动状态;

例如：
```
abrtd(pid 1505)正在运行…
abrt-dumpoops(pid 1513)正在运行…
acpid(pid 1312)正在运行...
…省略部分输出…
```
随着 httpd 服务的启动和停止，使用"netstat -tlun"命令就会看到 80 端口出现和消失。这也就说明 apache 服务绑定的口就是 80，所以我们可以端口是否在服务器中出现来判断 apache 服务是否启动。
## 独立服务的自启动管理
自启动指的是在系统之后，服务是否随着系统的启动而自动启动。如果启动了某个服务，那么这个服务会在系统重启之后启动吗？

答案是不知道，因为启动命令只负责启动服务，而和服务的自启动完全没有关系。同样地，自启动命令只管服务是否会在系统重启之后启动，而和当前系统中的服务是否启动没有关系。

独立服务的自启动方法有三种。
### 1.使用 chkconfig 服务自启动管理命令
第一种方法是利用`chkconfig`服务自启动管理命令来管理独立服务的自启动，这条命令的用法如下：
```
[root@localhost ~]# chkconfig --list
```
使用 chkconfig 命令除了可以查看所有 RPM 包默认安装服务的自启动状态，也可以修改和设置 RPM 包默认安装服务的自启动状态，只是独立的服务和基于 xinetd 的服务的设定方法稍有不同。我们先来看看独立的服务如何设置。命令格式如下：
[root@localhost ~]# chkconfig [--level 运行级别][独立服务名][on|off]

#选项：
--level: 设定在哪个运行级别中开机自启动（on），或者关闭自启动（off）；

例如：
[root@localhost ~]# chkconfig --list | grep httpd
httpd 0:关闭 1:关闭 2:关闭 3:关闭 4:关闭 5:关闭 6:关闭
#查询httpd的自启动状态。所有的级别都是不自启动的
[root@localhost ~]# chkconfig --level 2345 httpd on
#设置apache服务在进入2、3、4、5级别时自启动
[root@localhost ~]# chkconfig --list | grep httpd
httpd 0:关闭 1:关闭 2:启用 3:启用 4:启用 5:启用 6:关闭
#查询apache服务的自启动状态。发现在2、3、4、5这4个运行级别中变为了"启用"

还记得 0~6 这 7 个 Linux 的运行级别吗？如果在 0~6 这 7 个运行级别中服务都显示"关闭"，则该服务不自启动。如果在某个运行级别中显示"启用"，则代表在进入这个运行级别时，该服务开机自启动。

服务的自启动方法和服务的启动方法是不通用的，我们做一个实验验证一下。命令如下：
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

大家看到了吗？虽然 apach 被设置为自启动，但是当前系统中的 apache 是没有启动的，所以启动和自启动是独立的。
### 2.修改 /etc/rc.d/rc.local 文件，设置服务自启动
第二种方法就是修改 /etc/rc.d/rc.local 文件，在文件中加入服务的启动命令。这个文件是在系统启动时，在输入用户名和密码之前最后读取的文件（注意：/etc/rc.d/rc.loca和/etc/rc.local 文件是软链接，修改哪个文件都可以）。这个文件中有什么命令，都会在系统启动时调用。

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
第二，chkconfig 命令只能识别 RPM 包默认安装的服务，而不能识别源码包安装的服务。 源码包安装的服务的自启动也是通过 /etc/rc.d/rc.local 文件实现的，所以不会出现同一台服务器自启动了两种安装方法的同一个服务。

还要注意一下，修改 /etc/rc.d/rc.local 配置文件的自启动方法和 chkconfig 命令的自启动方法是两种不同的自启动方法。所以，就算通过修改 /etc/rc.d/rc.local 配置文件的方法让某个独立的服务自启动了，执行"chkconfig --list"命令并不到有什么变化。
### 3.使用 ntsysv 命令管理自启动
第三种方法是使用 ntsysv 命令调用窗口模式来管理服务的自启动，非常简单。命令格式如下：
```
[root@localhost ~]# ntsysv [--level 运行级别]
```
选项：
--level 运行级别：可以指定设定自启动的运行级别；
例如：
[root@localhost ~]# ntsysv --level 235
#只设定2、3、5级别的服务自启动
[root@localhost ~]# ntsysv
#按默认的运行级别设置服务自启动

执行命令后，会和 setup 命令类似，出现命令界面，如图 1 所示。


图 1 ntsysv命令界面

这个命令的操作是这样的：
上下键：在不同服务之间移动；
空格键：选定或取消服务的自启动。也就是在服务之前是否输入"*"；
Tab键：在不同项目之间切换；
F1键：显示服务的说明；

需要注意的是，ntsysv 命令不仅可以管理独立服务的自启动，也可以管理基于 xinetd 服务的自启动。也就是说，只要是 RPM 包默认安装的服务都能被 ntsysv 命令管理。但是源码包安装的服务不行。

这样管理服务的自启动多么方便，为什么还要学习其他的服务自启动管理命令呢？ ntsysv 命令虽然简单，但它是红帽系列 Linux 的专有命令，其他的 Linux 发行版本不一定拥有这条命令，而且条命令也不能管理源码包安装的服务，所以我们推荐大家使用 /etc/rc.d/rc.local 文件来管理服务的自启动。