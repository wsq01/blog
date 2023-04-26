---
title: Wireshark 使用教程
date: 2023-03-04 16:51:12
tags: 计算机网络
categories: 计算机网络
---

wireshark 是非常流行的网络封包分析软件，简称小鲨鱼，功能十分强大。可以截取各种网络封包，显示网络封包的详细信息。

wireshark 是开源软件，可以放心使用。对应的，linux 下的抓包工具是`tcpdump`。
# Wireshark抓包原理
Wireshark 使用 WinPCAP 作为接口，直接与网卡进行数据报文交换。

Wireshark 使用的环境大致分为两种，一种是电脑直连网络的单机环境，另外一种就是应用比较多的网络环境，即连接交换机的情况。
* 单机情况下，Wireshark 直接抓取本机网卡的网络流量；
* 交换机情况下，Wireshark 通过端口镜像、ARP 欺骗等方式获取局域网中的网络流量。
 * 端口镜像：利用交换机的接口，将局域网的网络流量转发到指定电脑的网卡上。
 * ARP 欺骗：交换机根据 MAC 地址转发数据，伪装其他终端的 MAC 地址，从而获取局域网的网络流量。

# Wireshark软件安装
软件下载路径：`https://www.wireshark.org/`。

按照系统版本选择下载，下载完成后，按照软件提示一路`Next`安装。
# Wireshark抓包示例
先介绍一个使用 wireshark 工具抓取`ping`命令操作的示例，可以上手操作感受一下抓包的具体过程。

1、打开 wireshark，主界面如下：

{% asset_img 1.png %}

2、选择菜单栏上「捕获 -> 选项」，勾选 WLAN 网卡。这里需要根据各自电脑网卡使用情况选择，简单的办法可以看使用的 IP 对应的网卡。点击开始，启动抓包。

{% asset_img 2.png %}

3、wireshark 启动后，wireshark 处于抓包状态中。

{% asset_img 3.png %}

4、执行需要抓包的操作，如在 cmd 窗口下执行`ping www.baidu.com`。

5、操作完成后相关数据包就抓取到了，可以点击「停止捕获分组」按钮。

{% asset_img 4.png %}

6、为避免其他无用的数据包影响分析，可以通过在过滤栏设置过滤条件进行数据包列表过滤，获取结果如下。说明：`ip.addr == 183.232.231.172 and icmp`表示只显示 ICPM 协议且主机 IP 为`183.232.231.172`的数据包。说明：协议名称`icmp`要小写。

{% asset_img 5.png %}

# Wireshakr抓包界面介绍
{% asset_img 6.png %}

Wireshark 的主界面包含6个部分：
* 菜单栏：用于调试、配置
* 工具栏：常用功能的快捷方式
* 过滤栏：指定过滤条件，过滤数据包
* 数据包列表：核心区域，每一行就是一个数据包
* 数据包详情：数据包的详细数据
* 数据包字节：数据包对应的字节流，二进制

说明：数据包列表区中不同的协议使用了不同的颜色区分。协议颜色标识定位在菜单栏「视图 --> 着色规则」。

{% asset_img 7.png %}

## WireShark 主要分为这几个界面
### 1. Display Filter(显示过滤器)
用于设置过滤条件进行数据包列表过滤。菜单路径：分析 --> Display Filters。

{% asset_img 8.png %}

### 2. Packet List Pane(数据包列表)
显示捕获到的数据包，每个数据包包含编号，时间戳，源地址，目标地址，协议，长度，以及数据包信息。不同协议的数据包使用了不同的颜色区分显示。

{% asset_img 9.png %}

### 3. Packet Details Pane(数据包详细信息)
在数据包列表中选择指定数据包，在数据包详细信息中会显示数据包的所有详细信息内容。数据包详细信息面板是最重要的，用来查看协议中的每一个字段。各行信息分别为
1. Frame:   物理层的数据帧概况
2. Ethernet II: 数据链路层以太网帧头部信息
3. Internet Protocol Version 4: 互联网层IP包头部信息
4. Transmission Control Protocol:  传输层T的数据段头部信息，此处是TCP
5. Hypertext Transfer Protocol:  应用层的信息，此处是HTTP协议

{% asset_img 10.png %}

从下图可以看到 wireshark 捕获到的 TCP 包中的每个字段。

{% asset_img 11.png %}
{% asset_img 12.png %}

### 4. Dissector Pane(数据包字节区)
报文原始内容。

{% asset_img 13.png %}

# Wireshark过滤器设置
wireshark 工具中自带了两种类型的过滤器，学会使用这两种过滤器会帮助我们在大量的数据中迅速找到我们需要的信息。
## 1.抓包过滤器
捕获过滤器的菜单栏路径为「捕获 --> 捕获过滤器」。用于在抓取数据包前设置。

{% asset_img 14.png %}

设置如下。

{% asset_img 15.png %}

`ip host 183.232.231.172`表示只捕获主机 IP 为`183.232.231.172`的数据包。获取结果如下：

{% asset_img 16.png %}

## 2. 显示过滤器
显示过滤器是用于在抓取数据包后设置过滤条件进行过滤数据包。

通常是在抓取数据包时设置条件相对宽泛或者没有设置导致抓取的数据包内容较多时使用显示过滤器设置条件过滤以方便分析。

{% asset_img 17.png %}

同样上述场景，在捕获时未设置抓包过滤规则直接通过网卡进行抓取所有数据包。

{% asset_img 18.png %}

执行`ping www.baidu.com`获取的数据包列表如下：

{% asset_img 19.png %}

观察上述获取的数据包列表，含有大量的无效数据。这时可以通过设置显示器过滤条件进行提取分析信息。`ip.addr == 183.232.231.172`，并进行过滤。

{% asset_img 20.png %}

# wireshark过滤器表达式的规则
## 1. 抓包过滤器语法
抓包过滤器类型`Type（host、net、port）`、方向`Dir（src、dst）`、协议`Proto（ether、ip、tcp、udp、http、icmp、ftp`等）、逻辑运算符（`&&`与、`||`或、`!`非）
1. 协议过滤：直接在抓包过滤框中直接输入协议名即可。
* `tcp`，只显示 TCP 协议的数据包列表
* `http`，只查看 HTTP 协议的数据包列表
* `icmp`，只显示 ICMP 协议的数据包列表
2. IP 过滤
```
host 192.168.1.104
src host 192.168.1.104
dst host 192.168.1.104
```
3. 端口过滤
```
port 80
src port 80
dst port 80
```
4. 逻辑运算符
```
// 抓取主机地址为192.168.1.80、目的端口为80的数据包
src host 192.168.1.104 && dst port 80 
// 抓取主机为192.168.1.104或者192.168.1.102的数据包
host 192.168.1.104 || host 192.168.1.102 
// 不抓取广播数据包
!broadcast
```

## 2. 显示过滤器语法
1. 比较操作符：`==、!=、>、<、>=、<=`
2. 协议过滤：直接在`Filter`框中直接输入协议名即可。注意：协议名称需要输入小写。
{% asset_img 21.png %}
3. `ip`过滤
`ip.src ==112.53.42.42`显示源地址为`112.53.42.42`的数据包列表
`ip.dst==112.53.42.42`, 显示目标地址为`112.53.42.42`的数据包列表
`ip.addr==112.53.42.42`显示源IP地址或目标IP地址为`112.53.42.42`的数据包列表
{% asset_img 22.png %}
4. 端口过滤
* `tcp.port==80`, 显示源主机或者目的主机端口为 80 的数据包列表。
* `tcp.srcport==80`,  只显示 TCP 协议的源主机端口为 80 的数据包列表。
* `tcp.dstport==80`，只显示 TCP 协议的目的主机端口为 80 的数据包列表。
{% asset_img 23.png %}
5. `http`模式过滤
`http.request.method=="GET"`, 只显示 HTTP GET 方法的。
{% asset_img 24.png %}
6. 逻辑运算符为`and/or/not`
过滤多个条件组合时，使用`and/or`。比如获取 IP 地址为`192.168.0.104`的 ICMP 数据包表达式为`ip.addr == 192.168.0.104 and icmp`。
{% asset_img 25.png %}
7. 按照数据包内容过滤
假设我要以 ICMP 层中的内容进行过滤，可以单击选中界面中的码流，在下方进行选中数据。
{% asset_img 26.png %}
右键单击选中后出现如下界面
{% asset_img 27.png %}
选中后在过滤器中显示如下
{% asset_img 28.png %}
后面条件表达式就需要自己填写。如下我想过滤出`data`数据包中包含`abcd`内容的数据流。关键词是`contains`，完整条件表达式为`data contains "abcd"`
{% asset_img 29.png %}
## 3. 常见用显示过滤需求及其对应表达式
* 数据链路层：
```
// 筛选源mac地址为04:f9:38:ad:13:26的数据包
eth.src == 04:f9:38:ad:13:26
```
* 网络层：
```
// 筛选ip地址为192.168.1.1的数据包
ip.addr == 192.168.1.1
// 筛选192.168.1.0网段的数据
ip contains "192.168.1"
```
* 传输层：
```
// 筛选端口为80的数据包
tcp.port == 80
// 筛选12345端口和80端口之间的数据包
tcp.port == 12345 && tcp.port == 80
// 筛选从12345端口到80端口的数据包
tcp.srcport == 12345 && tcp.dstport == 80
```
* 应用层：
特别说明: `http`中`http.request`表示请求头中的第一行（如`GET index.jsp HTTP/1.1`）`http.response`表示响应头中的第一行（如`HTTP/1.1 200 OK`），其他头部都用`http.header_name`形式。
```
// 筛选url中包含.php的http数据包
http.request.uri contains ".php"
// 筛选内容包含username的http数据包
http contains "username"
```