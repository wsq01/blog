---
title: 应用层——P2P应用
date: 2020-10-28 14:30:16
tags: 计算机网络
categories: 计算机网络
---


P2P应用就是指具有 P2P 体系结构的网络应用。所谓 P2P 体系结构就是在这样的网络应用中，没有或只有极少数的固定服务器，绝大多数的交互都是使用对等方式（P2P方式）进行的。

在 P2P 工作方式下，所有的音频/视频文件都是在普通的互联网用户之间传输。这种工作方式解决了集中式媒体服务器可能出现的瓶颈问题。
# 具有集中目录服务器的 P2P 工作方式
Napster 最早使用 P2P 技术，提供免费下载 MP3 音乐。Napster 将所有音乐文件的索引信息都集中存放在 Napster 目录服务器中。使用者只要查找目录服务器，就可知道应从何处下载所要的MP3文件。用户要及时向 Napster 的目录服务器报告自己存有的音乐文件。

Napster 的文件传输是分散的，文件的定位则是集中的。
## Napster 的工作过程
{% asset_img img1.png %}
{% asset_img img2.png %}
{% asset_img img3.png %}
{% asset_img img4.png %}

集中式目录服务器的缺点：可靠性差、会成为性能的瓶颈。
# 具有全分布式结构的 P2P 文件共享程序
Gnutella 是第二代 P2P 文件共享程序，采用全分布方法定位内容的 P2P 文件共享应用程序。

Gnutella 与 Napster 最大的区别是：不使用集中式的目录服务器，而是使用洪泛法在大量 Gnutella 用户之间进行查询。

为了不使查询的通信量过大，Gnutella 设计了一种有限范围的洪泛查询，以减少倾注到互联网的查询流量，但也影响到查询定位的准确性。
## 使用 P2P 的比特洪流 BT
BitTorrent 所有对等方集合称为一个洪流，下载文件的数据单元为长度固定的文件块。基础设施结点，叫做追踪器。

A 就和这些对等方建立了 TCP 连接。所有与 A 建立了 TCP 连接的对等方为“相邻对等方”。
# P2P 文件分发的分析
从互联网传送数据到主机，叫做下载(`download`)，从主机向互联网传送，则称为上传(`upload`)或上载。

有 N 台主机从服务器下载一个大文件，其长度为`F bit`。假定主机与互联网连接的链路的上传速率和下载速率分别为 ui 和 di ，单位都是`bit/s`。

{% asset_img img9.png %}

## C/S 方式下分发的最短时间
从服务器端考虑，所有主机分发完毕的最短时间 Tcs 不可能小于 NF/us；

下载速率最慢的主机的下载速率为 dmin，则 Tcs 不可能小于 F/dmin。

由此可得出所有主机都下载完文件 F 的最少时间是：Tcs=max（ NF/us，F/dmin ）
## P2P 方式下分发的最短时间
初始服务器文件分发的最少时间不可能小于 F/us；
下载文件分发的最少时间不可能小于 F/dmin；
上载文件分发的最少时间不可能小于 NF/uT，其中是 uT 是上传速率之和。
所有主机都下载完文件 F 的最少时间的下限是：
Tp2p >= max（F/us， F/dmin， NF/uT）
## 时间比较
设所有的对等方的上传速率都是 u，并且 F/u = 1 小时。

设服务器的上传速率 us = 10u。

当 N = 30 时，
P2P方式：最少时间的下限是 0.75 小时 < 1 小时（不管 N 多大）。
客户−服务器方式：最少时间是 3 小时。
# 在 P2P 对等方中搜索对象
Napster在一个集中式目录服务器中构建的查找数据库虽然很简单，但性能上却有瓶颈。

Gnutella是一种采用全分布方法定位内容的P2P文件共享应用程序，它解决了集中式目录服务器所造成的瓶颈问题。然而Gnutella是在非结构化的覆盖网络中采用查询洪泛的方法来进行查找的，因此查找的效率较低。

现在广泛使用的索引和查找技术叫做分布式散列表 DHT(`Distributed Hash Table`)。DHT 也可译为分布式哈希表，它是由大量对等方共同维护的散列表。

分布式散列表 DHT 利用散列函数，把资源名 K 及其存放的结点 IP 地址 N 都分别映射为资源名标识符 KID 和结点标识符 NID。

Chord 把结点按标识符数值从小到大沿顺时针排列成一个环形覆盖网络。
## 基于 DHT 的 Chord 环
每个资源由 Chord 环上与其标识符值最接近的下一个结点提供服务。

{% asset_img img7.png %}

为了加速查找，在 Chord 环上可以增加一些指针表，它又称为路由表或查找器表。

对于结点`N4`， 第 1 列第`i`行计算`N4 + 2i – 1`，得出后继结点。

{% asset_img img8.png %}