---
title: OSPF实战
date: 2023-03-23 20:36:11
tags: 计算机网络
categories: 计算机网络
---


# OSPF 基础命令
掌握三条命令，就能玩转 OSPF：
* 创建 OSPF 进程，进入配置视图
* 创建 OSPF 区域
* 接口激活 OSPF

## 创建 OSPF 进程
在系统视图下，使用 OSPF 命令，创建 OSPF 进程。
```
[Router]ospf 1 Router-id 1.1.1.1
```
上面的命令，是在`Router`上创建`Process-ID`为 1 的 OSPF 进程，并进入配置视图。使用的`Router-ID`是`1.1.1.1`。

`Process-ID`是可选参数，表示 OSPF 进程的编号，只在设备本身生效，也就是说，两台设备建立 OSPF 邻居时，不要求双方的`Process-ID`一样。如果不指定`Process-ID`，会分配一个默认值作为`Process-ID`。

`router-id`也是可选参数，用来指定设备的`Router-ID`。通常，手动配置`Router-ID`，不会使用默认值。
## 创建 OSPF 区域
创建完 OSPF 进程后，就需要创建 OSPF 区域。在配置视图下，使用`area`命令创建区域，并指定区域 ID。
```
[Router]ospf 1 Router-id 1.1.1.1
[Router-ospf-1]area 1
```
在 OSPF 进程 1 中，创建`Area1`。
## 接口激活 OSPF
默认状态下，所有接口都没有激活 OSPF，要在接口激活 OSPF，有两种方法：
### 在区域视图激活 OSPF
在区域视图下，使用`network`命令，再加上 IP 地址和通配符掩码，满足条件的接口激活 OSPF。
```
[Router]ospf 1 Router-id 1.1.1.1
[Router-ospf-1]area 0
[Router-ospf-1-area-0.0.0.0]network 192.168.1.0 0.0.0.255
```
举个栗子：`network 192.168.1.0 0.0.0.255`，IP 地址是`192.168.1.0`，通配符掩码是`0.0.0.255`。通配符掩码中，比特位为 0 的需要匹配，比特位为 1 的不需要匹配。命令中匹配的 IP 地址是`192.168.1.0`至`192.168.1.255`。

{% asset_img 1.png %}

计算方法是，把`192.168.1.0`用二进制表示，把通配符掩码`0.0.0.255`也换算成二进制，每个比特位对应。前 24 位全为 0，后 8 位全为 1。匹配的 IP 地址，以`192.168.1`开头，后面是 0 至 255 的任意值。接口 IP 地址在这个范围内，且 IP 地址掩码长度大于或等于`network`命令的 0 比特位数，就在接口上激活 OSPF。

有两个特殊的`network`命令，一个是`network x.x.x.x 0.0.0.0`，比如`network 192.168.1.1 0.0.0.0`，IP 地址是`192.168.1.1`的接口激活 OSPF，无论网络掩码长度多少。另一个是`network 0.0.0.0 255.255.255.255`，匹配任意 IP 地址，所有配置了 IP 地址的接口，都会激活 OSPF。
### 在指定接口激活 OSPF
上面的方法，可以使用一条命令，在多个接口激活 OSPF。另一个方法就是在指定接口激活 OSPF。先创建 OSPF 进程和区域，然后进入接口视图，使用`ospf enable`命令激活。
```bash
[Router]ospf 1 Router-id 1.1.1.1 #创建OSPF进程
[Router-ospf-1]area 0 #创建Area0
[Router-ospf-1-area-0.0.0.0]quit
[Router-ospf-1]quit
[Router]interface GigabitEthernet 0/0 #进入GE0/0接口视图
[Router-GigabitEthernet0/0]ospf enable 1 area 0 #激活OSPF
```
# OSPF 单区域实验

{% asset_img 2.png %}

路由器 RT1 的两个接口分别连接`172.16.1.0/24`和`172.16.2.0/24`网段，另一个接口连接路由器 RT2。RT2 创建 Loopback 接口，配置 IP 地址`172.16.255.2/24`，模拟 RT2 的直连网段。在 RT1 和 RT2 上运行 OSPF，让 PC 可以访问全部网段。

RT1 配置如下：
```shell
[RT1]ospf 1 router-id 1.1.1.1
[RT1-ospf-1]area 0
[RT1-ospf-1-area-0.0.0.0]network 172.16.1.0 0.0.0.255
[RT1-ospf-1-area-0.0.0.0]network172.16.2.0 0.0.0.255
[RT1-ospf-1-area-0.0.0.0]network 172.16.12.0 0.0.0.3
```
RT2 配置如下：
```
[RT2]ospf 1 router-id 2.2.2.2
[RT2-ospf-1]area 0
[RT2-ospf-1-area-0.0.0.0]network 172.16.12.0 0.0.0.3
[RT2-ospf-1-area-0.0.0.]network 172.16.255.0 0.0.0.255
```
配置完成后，RT1 和 RT2 建立邻接关系，交换 LSA。查看 RT1 的邻居表：

{% asset_img 3.png %}

邻接表中，RT1 在`Area0`发现邻居，`Router-ID`是`2.2.2.2`，接口 IP 地址是`172.16.12.2`，邻居状态是`Full`，对端是`Master`。

再看 RT1 的 OSPF 路由表：

{% asset_img 4.png %}

命令`display ospf routing`，是查看 OSPF 路由表，并不是路由器的全局路由器。这些 OSPF 路由表能否加载到全局路由表，还要看路由表优先级等因素。这里发现 R2 的 Loopback0 接口路由，是一条主机路由，实际上，Loopback0 接口的掩码长度是 24，而不是 32。这是因为 OSPF 把 Loopback 接口作为末梢网络，无论实际掩码是多少，Type-1 LSA 中，都以`255.255.255.255`进行通告。

查看 RT2 的 Type-1 LSA ：

{% asset_img 5.png %}

如果希望 RT2 的 Type-1 LSA 描述 Loopback 接口的实际掩码信息，可以把接口的网络类型改成 Broadcast 或 NBMA，比如：
```
[RT2]interface LoopBack 0
[RT2-LoopBack0]ospf network-type broadcast
```
再看 RT2 的 Type-1 LSA：

{% asset_img 6.png %}

RT2 以实际掩码信息通告`Loopback0`接口。现在看下 RT1 的路由表中 OSPF 路由：

{% asset_img 7.png %}

RT1 把`172.16.255.0/24`网段加载到路由表。同时，RT2 也获得到达`172.16.1.0/24`和`172.16.2.0/24`的路由。这样 PC 就能访问网络中的各个网段。

# Silent-Interface
{% asset_img 8.png %}

上个实验的拓扑图中，RT1 的`GE0/0/0`和`GE0/0/1`接口连接终端网段，只有终端 PC，没有 OSPF 路由器。然而，接口已经激活了 OSPF，会周期性的发送`Hello`报文，但是 PC 无法识别、也不需要识别`Hello`报文。这时，可以把 RT1 的`GE0/0/0`和`GE0/0/1`配置成静默接口（`Silent-Interface`），接口就会禁止收发`Hello`报文。

R1 的配置如下：
```
[RT1]ospf 1
[RT1-ospf-1]silent-interface GigabitEthernet 0/0/0
[RT1-ospf-1]silent-interface GigabitEthernet 0/0/1
```
虽然两个接口指定为`Silent-Interface`，但是已经使用 network 命令激活 OSPF，因此 RT2 还是能通过 OSPF 学习到这两个接口网段的路由。

{% asset_img 9.png %}

# OSPF 多区域实验

{% asset_img 10.png %}

RT1 和 RT2 是两台汇聚交换机，各自下挂两个终端网段，同时上连核心交换机 RT3 。在三台路由器上部署 OSPF ，使用多区域 OSPF 的设计，实现全网各个网段的数据互通。

RT1 的配置如下：
```bash
[RT1]ospf 1 router-id 1.1.1.1
[RT1-ospf-1]area 1
[RT1-ospf-1-area-0.0.0.1]network 172.16.1.0 0.0.0.255
[RT1-ospf-1-area-0.0.0.1]network 172.16.2.0 0.0.0.255
[RT1-ospf-area-0.0.0.1]quit
[RT1-ospf-1]area 0
[RT1-ospf-1-area-0.0.0.o]network 172.16.0.0 0.0.0.3
```
RT2 的配置如下：
```
[RT2]ospf 1 router-id 2.2.2.2
[RT2-ospf-1]area 2
[RT2-ospf-1-area-0.0.0.2]network 172.16.9.0 0.0.0.255
[RT2-ospf-1-area-0.0.0.2]network 172.16.10.0 0.0.0.255
[RT2-ospf-1-area-0.0.0.2]quit
[RT2-ospf-1]area 0
[RT2-ospf-1-area-0.0.0.o]network 172.16.0.4 0.0.0.3
```
RT3 的配置如下：
```
[RT3]ospf 1 router-id 3.3.3.3
[RT3-ospf-1]area 0
[RT3-ospf-1-area-0.0.0.0]network 172.16.0.0 0.0.0.3
[RT3-ospf-1-area-0.0.0.0]network 172.16.0.4 0.0.0.3
```
查看 RT3 的 OSPF 邻居表：

{% asset_img 11.png %}

RT3 和 RT1、RT2 建立的 FULL 的邻接关系，我们再看看 RT3 的路由表：

{% asset_img 12.png %}

`Area0`的路由器 RT3，已经学到了 RT1 和 RT2 下连的终端网段路由。这些路由都是区域间路由，根据 RT1 和 RT2 在`Area0`泛洪的 Type-3 LSA 计算得出，而 RT3 要计算到达`Area1`和`Area2`的区域间路由，除了这些网段的 Type-3 LSA，还需要指定 ABR 的位置。作为 ABR，RT1 和 RT2 在泛洪 Type-1 LSA 时，会把 B 比特位设置为 1。因此，通过 Area0 内泛洪的`Type-1`、`Type-2`LSA ，RT3 能计算到达 ABR 的最佳路径。使用`display ospf abr-asbr`命令查看 ABR 和 ASBR 信息：

{% asset_img 13.png %}

在 RT1 和 RT2 上查看路由表，看到两台路由器都学到了全网的路由，RT1 和 RT2 下挂的 PC 就可以到达全网各个网段。
# OSPF Cost 值

{% asset_img 14.png %}

R1 和 R2 连接到相同的一个网段：`192.168.100.0/24`，同时下连 R3。R1、R2、R3 都激活 OSPF，在相同的 Area 中，接口的 Cost 又是默认值，这时 R3 的路由表中，到达`192.168.100.0/24`会有两条等价路由：

{% asset_img 15.png %}

如果两条链路以主备方式工作，该如何实现呢？一个最简单的方法就是调整接口 Cost 值。比如把 R3 的`G0/0/2`接口 Cost 值调大，到达`192.168.100.0/24`的报文会转发给 R1，当 R1 故障时，R3 自动把流量切到 R2。

R3 的配置：
```
[R3]interface GigabitEthernet 0/0/2
[R3-GigabitEthernet0/0/2]ospf cost 100
```
查看接口参数：

{% asset_img 16.png %}

在查看 R3 的路由表：

{% asset_img 17.png %}

配置生效。

# OSPF 特殊区域
{% asset_img 18.png %}

R1 、R2 、R3 运行 OSPF，R3 把自己的静态路由引入 OSPF，让域内的路由器学习到外部路由。
## 1、基本配置
R1 配置：
```
[Rl]ospf i router-id 1.1.1.1
[R1-ospf-l]area 1
[R1-ospf-1-area-0.0.0.l]network 10.1.12.0 0.0.0.255
```
R2 配置：
```
[R2]ospf  router-id 2.2.2.2
[R2-ospf-l]area 1
[R2-ospf-1-area-0.0.0.]network 10.1.12.0 0.0.0.255
[R2-ospf-1-area-0.0.0.1]quit
[R2-ospf-1]area 0
[R2-ospf-1-area-0.0.0.0]network 10.1.23.0 0.0.0.255
```
R3 配置：
```
[R3]ip route-static 10.3.1.0 24 NULL 0
[R3]ip route-static 10.3.2.0 24 NULL O
[R3]ospf 1 router-id 3.3.3.3
[R3-ospf-1]area 0
[R3-ospf-1-area-0.0.0.o]network 10.1.23.0 0.0.0.255
[R3-ospf-1-area-0.0.0.0]quit
[R3-ospf-1]import-route static
```
观察 R1 的路由表：

{% asset_img 19.png %}

R1 学习到了区域间路由和外部路由，外部路由标记位 O_ASE（ OSPF AS External ）。再看看 R1 的 LSDB：

{% asset_img 20.png %}

R1 的 LSDB，有 Type-1、Type-2、Type-3、Type-4、Type-5 LSA。Type-3 LSA 描述到达`10.1.23.0/24`的区域间路由。Type-4 LSA 描述到达 ASBR，也就是 R3 的路由，是由 ABR R2 产生。Type-5 LSA 描述外部路由`10.3.1.0/24`和`10.3.2.0/24`，并在整个 OSPF 域内泛洪，这时 R1 有到达各个网段的路由。

## 2、Area1 配置为 Stub 区域
先把 Area1 配置为 Stub 区域，R1 和 R2 的配置如下：
```
[Rl]ospf 1
[R1-ospf-l]area 1
[R1-ospf-1-area-0.0.0.1]stub
```
```
[R2]ospf 1
[R2-ospf-l]area 1
[R2-ospf-1-area-0.0.0.1]stub
```
某个区域为 Stub 区域，区域内的路由器都要配置成 Stub 区域，否则无法正确建立邻居关系。Stub 区域的 ABR，即 R2，会阻挡 Type-4 、Type-5 LSA 进入区域内，减少 LSA 泛洪的数量，从而减小路由表规模，降低设备负担。

现在 R1 无法学到 OSPF 外部路由，同时 R2 会下发一条用 Type-3 LSA 描述的外部路由，让 Area1 内的路由器访问域外的网络。观察 R1 的路由表和 LSDB：

{% asset_img 21.png %}
{% asset_img 22.png %}

R1 的路由表减少，不再有 Type-4 和 Type-5 LSA ，只有 Type-1、Type-2、Type-3 LSA。

## 3、Area1 配置为 Totally-Stub 区域
如果要进一步减少 LSA 泛洪，可以把区域间的路由也阻挡。在上个实验的基础上，R2 配置如下：
```
[R2]ospf 1
[R2-ospf-l]area 1
[R2-ospf-1-area-0.0.0.1]stub no-summary
```
这时，R2 阻挡 Type-3、Type-4、Type-5 LSA 进入 Area1，同时自动下发一条默认路由，使用 Type-3 LSA 描述。这样当 R1 访问区域外的网络时，就使用默认路由转发数据。

查看 R1 路由表和 LSDB：

{% asset_img 23.png %}
{% asset_img 24.png %}

R1 路由表只有一条 0.0.0.0/0 的默认路由，极大简化了路由表。同时，R1 的 LSDB 也非常简洁。

## 4、Area1 配置为 NSSA

{% asset_img 25.png %}

网络发生变化，Area1 的 R1 连着一个外部路由，需要引入 OSPF，让域内路由器获得外部路由，但又希望保持 Stub 区域特性，那么可以把 Area1 配置为 NSSA。在上个实验的基础上，R1 配置如下：
```bash
[R1]ip route-static 10.1.1.0 24 NULL 0 #模拟外部路由
[R1]ospf 1
[Rl-ospf-l]area 1
[R1-ospf-1-area-0.0.0.1]undo stub
[R1-ospf-1-area-0.0.0.]nssa
[R1-ospf-1-area-0.0.0.1]quit
[R1-ospf-1]import-route static
```
在 R1 创建静态路由，模拟成外部路由，先取消 Stub 配置，然后配置 NSSA，再把外部路由引入 OSPF。

R2 配置如下：
```
[R2]ospf 1
[R2-ospf-l]area 1
[R2-ospf-1-area-0.0.0.]undo stub
[R2-ospf-1-area-0.0.0.1]nssa
```
某个区域配置为 NSSA，则区域内的所有路由器都要进行相应配置，否则建立邻居关系会出现问题。Area1 区域成为 NSSA 后，会阻挡 Type-4、Type-5 LSA 进入区域。但是 ABR R2 会下发一条 Type-7 LSA 的默认路由，让区域内的路由器，通过默认路由到达域外网络。同时，会向 Area0 通告 Type-5 LSA 描述`10.1.1.0/24`路由，让 OSPF 其它区域的路由器都学习到这条路由。

查看 R1 的 LSDB ：

{% asset_img 26.png %}

R1 的 LSDB 中，有 Type-1、Type-2、Type-3、Type-7 LSA，其中两条 Type-7 LSA，一条是 R1 生成的，描述引入的外部路由`10.1.1.0/24`，另一条是 R2 生成的，是一条默认路由。再看看 R1 的路由表：

{% asset_img 27.png %}

再看看 R3 的 LSDB:

{% asset_img 28.png %}

R3 有三条 Type-5 LSA，其中两条是自己生成的，描述外部路由`10.3.1.0/24`和`10.3.2.0/24`，另一条是 R2 生成的，描述外部路由`10.1.1.0/24`。R2 把从 Area1 收到的 Type-7 LSA 转换成 Type-5 LSA，并通告到 Area0 中。查看 R3 路由表：

{% asset_img 29.png %}

## 5、Area1 配置为 Totally NSSA
为了进一步减少 Area1 的 LSA，把 Area1 配置成 Totally NSSA 实现。在上个实验的基础上，R2 配置如下：
```
[R2]ospf 1
[R2-ospf-l]area 1
[R2-ospf-1-area-0.0.0.1]nssa no-summary
```
这样 Area1 内不会有 Type-3 LSA 泛洪，R1 也学不到区域间路由。

看下 R1 的 LSDB：

{% asset_img 30.png %}

R1 的 LSDB 中，只有 Type-1、Type-2、Type-7 LSA 和一条描述的默认路由 Type-3 LSA。当 NSSA 内同时存在 Type-3 LSA 和 Type-7 LSA 描述的默认路由时，路由器优先使用 Type-3 LSA 的默认路由，忽略 Type-7 LSA 的默认路由。

# Virtual Link
{% asset_img 31.png %}

R1、R2、R3 运行 OSPF，规划两个区域`Area0`和`Area23`。R3 有两条路由到达`192.168.2.0/24`网段，因为 R3 不能使用非 0 区域的 Type-3 LSA 来计算区域间路由，因此无论路径的 Cost 如何，R3 都会选择 R1 到达目的网段。查看 R3 的 OSPF 路由表：

{% asset_img 32.png %}

如果向让 R3 从 R2 到达`192.168.2.0/24`，即使用高带宽链路转发，一个简单的方法是，在 R2 和 R3 之间跨越`Area23`建立一条`Virtual Link`，通过这条`Virtual Link`，R2 直接把 Type-1 LSA 发送给 R3。

R2 配置如下：
```
[R2]ospf 1
[R2-ospf-l]area 23
[R2-ospf-1-area-0.0.0.23]vlink-peer 3.3.3.3
```
R3 配置如下：
```
[R3]ospf 1
[R3-ospf-l]area 23
[R3-ospf-1-area-0.0.0.23]vlink-peer 2.2.2.2
```
配置完成后，R2 和 R3 建立一条`Virtual Link`，`Virtual Link`穿过 Area23，在 R3 查看`Virtual Link`信息：

{% asset_img 33.png %}

`Virtual Link`建立完成后，状态为`Full`，Cost 为 1，再看下 R3 的 OSPF 路由表：

{% asset_img 34.png %}

`192.168.2.0/24`路由的下一跳变成了`192.168.23.2`，说明到达这个网段的下一跳切换到了 R2，达到预期目标。

