---
title: 运输层——TCP的传输连接管理
date: 2020-10-17 11:51:33
tags: 计算机网络
categories: 计算机网络
---

TCP 连接有三个阶段：连接建立、数据传送、连接释放。

TCP 连接的管理就是使 TCP 连接的建立和释放都能正常地进行。

TCP 连接建立过程中要解决的三个问题：
1. 要使每一方能够确知对方的存在。
2. 要允许双方协商一些参数（如最大窗口值、是否使用窗口扩大选项和时间戳选项以及服务质量等）。
3. 能够对运输实体资源（如缓存大小、连接表中的项目等）进行分配。

TCP 连接的建立采用客户服务器方式。主动发起连接建立的应用进程叫做客户，被动等待建立连接的应用进程叫服务器。
# TCP的连接建立
TCP 建立连接的过程叫做握手。握手需要在客户和服务器之间交换三个 TCP 报文段。称之为三报文握手。

{% asset_img img1.png 用三次握手建立TCP连接的各状态 %}

假定主机 A 运行的是 TCP 客户程序，B 运行 TCP 服务器程序。最初两端的 TCP 进程都处于`CLOSED`（关闭）状态。图中在主机下面的方框分别是 TCP 进程所处的状态。注意，A 主动打开连接，B 被动打开连接。

B 的 TCP 服务器进程先创建传输控制块 TCB，准备接受客户进程的连接请求。然后服务器进程就处于`LISTEN`（收听）状态，等待客户的连接请求。

A 的 TCP 客户进程也是首先创建传输控制块 TCB，然后向 B 发出连接请求报文段，这时，首部中的同部位`SYN = 1`，同时选择一个初始序号`seq = x`。TCP 规定`SYN`报文段（即`SYN = 1`的报文段）不能携带数据，但要消耗掉一个序号。这时，TCP 客户进程进入`SYN-SENT`（同步已发送）状态。

B 收到连接请求报文段后，如同意建立连接，则向 A 发送确认。在确认报文段中，`SYN = 1, ACK = 1`，确认号是`ack = x + 1`，同时也为自己选择一个初始序号`seq = y`。这个报文段也不能携带数据，但同样要消耗掉一个序号。这时，TCP 服务器进程进入`SYN-RCVD`（同步收到）状态。

TCP 客户进程收到 B 的确认后，还要向 B 给出确认。`ACK = 1, ack = y + 1`，而自己的序号`seq = x + 1`。TCP 规定，`ACK`报文段可以携带数据。但如果不携带数据则不消耗序号，在这种情况下，下一个数据报文段的序号仍是`seq = x + 1`。这时 TCP 连接已经建立，A 进入`ESTABLISHED`（已建立连接）状态。

当 B 收到 A 的确认后，也进入`ESTABLISHED`状态。

从上⾯的过程可以发现第三次握手是可以携带数据的，前两次握手是不可以携带数据的。

⼀旦完成三次握手，双方都处于`ESTABLISHED`态，此时连接就已建立完成，客户端和服务端就可以相互发送数据了。
## 为什么是三次握手？不是两次、四次？
三次握手的原因：
* 三次握手才可以阻止重复历史连接的初始化（主要原因）
* 三次握手才可以同步双方的初始序列号
* 三次握手才可以避免资源浪费

### 原因⼀：避免历史连接
简单来说，三次握手的首要原因是为了防止旧的重复连接初始化造成混乱。

我们考虑一个场景，客户端先发送了`SYN(seq = 90)`报文，然后客户端宕机了，而且这个`SYN`报文还被网络阻塞了，服务端并没有收到，接着客户端重启后，又重新向服务端建立连接，发送了`SYN(seq = 100)`报文（注意！不是重传`SYN`，重传的`SYN`的序列号是一样的）。

看看三次握手是如何阻止历史连接的：

{% asset_img 10.jpg %}

客户端连续发送多次`SYN`建立连接的报文，在网络拥堵情况下：
* 一个「旧`SYN`报文」比「最新的`SYN`」 报文早到达了服务端，那么此时服务端就会回一个`SYN + ACK`报文给客户端，此报文中的确认号是 91（90+1）。
* 客户端收到后，发现自己期望收到的确认号应该是 100 + 1，而不是 90 + 1，于是就会回`RST`报文。
* 服务端收到`RST`报文后，就会释放连接。
* 后续最新的`SYN`抵达了服务端后，客户端与服务端就可以正常的完成三次握手了。

如果服务端在收到`RST`报文之前，先收到了「新`SYN`报文」，也就是服务端收到客户端报文的顺序是：「旧`SYN`报文」->「新`SYN`报文」，此时会发生什么?

当服务端第一次收到`SYN`报文，也就是收到 「旧`SYN`报文」时，就会回复`SYN + ACK`报文给客户端，此报文中的确认号是 91（90+1）。

然后这时再收到「新`SYN`报文」时，就会回`Challenge Ack`报文给客户端，这个`ack`报文并不是确认收到「新`SYN`报文」的，而是上一次的`ack`确认号，也就是 91（90+1）。所以客户端收到此`ACK`报文时，发现自己期望收到的确认号应该是 101，而不是 91，于是就会回`RST`报文。

如果是两次握手连接，就无法阻止历史连接，那为什么 TCP 两次握手为什么无法阻止历史连接呢？

主要是因为在两次握手的情况下，服务端没有中间状态给客户端来阻止历史连接，导致服务端可能建立一个历史连接，造成资源浪费。

在两次握手的情况下，服务端在收到`SYN`报文后，就进入`ESTABLISHED`状态，意味着这时可以给对方发送数据，但是客户端此时还没有进入`ESTABLISHED`状态，假设这次是历史连接，客户端判断到此次连接为历史连接，那么就会回`RST`报文来断开连接，而服务端在第一次握手的时候就进入`ESTABLISHED`状态，所以它可以发送数据的，但是它并不知道这个是历史连接，它只有在收到`RST`报文后，才会断开连接。

{% asset_img 9.png %}

可以看到，如果采用两次握手建立 TCP 连接的场景下，服务端在向客户端发送数据前，并没有阻止掉历史连接，导致服务端建立了一个历史连接，又白白发送了数据，妥妥地浪费了服务端的资源。

因此，要解决这种现象，最好就是在服务端发送数据前，也就是建立连接之前，要阻止掉历史连接，这样就不会造成资源浪费，而要实现这个功能，就需要三次握手。

所以，TCP 使用三次握手建立连接的最主要原因是防止「历史连接」初始化了连接。
### 原因二：同步双方初始序列号
TCP 协议的通信双方， 都必须维护⼀个序列号， 序列号是可靠传输的⼀个关键因素，它的作用： 
* 接收方可以去除重复的数据；
* 接收方可以根据数据包的序列号按序接收；
* 可以标识发送出去的数据包中， 哪些是已经被对方收到的；

可⻅，序列号在 TCP 连接中占据着⾮常重要的作用，所以当客户端发送携带「初始序列号」的`SYN`报文的时候，需要服务端回⼀个`ACK`应答报文，表示客户端的`SYN`报文已被服务端成功接收，那当服务端发送「初始序列号」给客户端的时候，依然也要得到客户端的应答回应，这样⼀来⼀回，才能确保双方的初始序列号能被可靠的同步。

{% asset_img 12.jpg %}

四次握手其实也能够可靠的同步双方的初始化序号，但由于第二步和第三步可以优化成⼀步，所以就成了三次握手。

而两次握手只保证了⼀方的初始序列号能被对方成功接收，没办法保证双方的初始序列号都能被确认接收。
### 原因三：避免资源浪费
如果只有两次握手，当客户端的`SYN`请求连接在网络中阻塞，客户端没有接收到`ACK`报文，就会重新发送`SYN`，由于没有第三次握手，服务器不清楚客户端是否收到了自己发送的建立连接的`ACK`确认信号，所以每收到⼀个`SYN`就只能先主动建立⼀个连接，这会造成什么情况呢？

如果客户端的`SYN`阻塞了，重复发送多次`SYN`报文，那么服务器在收到请求后就会建立多个冗余的⽆效链接，造成不必要的资源浪费。

{% asset_img 13.jpg %}

即两次握手会造成消息滞留情况下，服务器重复接受⽆用的连接请求`SYN`报文，而造成重复分配资源。
### 小结
TCP 建立连接时，通过三次握手能防止历史连接的建立，能减少双方不必要的资源开销，能帮助双方同步初始化序列号。序列号能够保证数据包不重复、不丢弃和按序传输。

不使用两次握手和四次握手的原因：
* 两次握手：⽆法防止历史连接的建立，会造成双方资源的浪费，也⽆法可靠的同步双方序列号；
* 四次握手：三次握手就已经理论上最少可靠连接建立，所以不需要使用更多的通信次数。

## 第一次握手丢失了，会发生什么？
当客户端想和服务端建立 TCP 连接的时候，首先第一个发的就是`SYN`报文，然后进入到`SYN_SENT`状态。

在这之后，如果客户端迟迟收不到服务端的`SYN-ACK`报文（第二次握手），就会触发「超时重传」机制，重传`SYN`报文，而且重传的`SYN`报文的序列号都是一样的。

不同版本的操作系统可能超时时间不同，有的 1 秒的，也有 3 秒的，这个超时时间是写死在内核里的，如果想要更改则需要重新编译内核，比较麻烦。

当客户端在 1 秒后没收到服务端的`SYN-ACK`报文后，客户端就会重发`SYN`报文，那到底重发几次呢？

在 Linux 里，客户端的 SYN 报文最大重传次数由`tcp_syn_retries`内核参数控制，这个参数是可以自定义的，默认值一般是 5。
```
cat /proc/sys/net/ipv4/tcp_syn_retries
5
```
通常，第一次超时重传是在 1 秒后，第二次超时重传是在 2 秒，第三次超时重传是在 4 秒后，第四次超时重传是在 8 秒后，第五次是在超时重传 16 秒后。没错，每次超时的时间是上一次的 2 倍。

当第五次超时重传后，会继续等待 32 秒，如果服务端仍然没有回应`ACK`，客户端就不再发送`SYN`包，然后断开 TCP 连接。

所以，总耗时是`1+2+4+8+16+32=63`秒，大约 1 分钟左右。

举个例子，假设`tcp_syn_retries`参数值为 3，那么当客户端的 SYN 报文一直在网络中丢失时，会发生下图的过程：

{% asset_img 14.png %}

具体过程：
当客户端超时重传 3 次 SYN 报文后，由于`tcp_syn_retries`为 3，已达到最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到服务端的第二次握手（`SYN-ACK`报文），那么客户端就会断开连接。
## 第二次握手丢失了，会发生什么？
当服务端收到客户端的第一次握手后，就会回`SYN-ACK`报文给客户端，这个就是第二次握手，此时服务端会进入`SYN_RCVD`状态。

第二次握手的`SYN-ACK`报文其实有两个目的：
* 第二次握手里的`ACK`，是对第一次握手的确认报文；
* 第二次握手里的`SYN`，是服务端发起建立 TCP 连接的报文；

所以，如果第二次握手丢了，就会发生比较有意思的事情，具体会怎么样呢？

因为第二次握手报文里是包含对客户端的第一次握手的`ACK`确认报文，所以，如果客户端迟迟没有收到第二次握手，那么客户端就觉得可能自己的`SYN`报文（第一次握手）丢失了，于是客户端就会触发超时重传机制，重传`SYN`报文。

然后，因为第二次握手中包含服务端的`SYN`报文，所以当客户端收到后，需要给服务端发送`ACK`确认报文（第三次握手），服务端才会认为该`SYN`报文被客户端收到了。

那么，如果第二次握手丢失了，服务端就收不到第三次握手，于是服务端这边会触发超时重传机制，重传`SYN-ACK`报文。

在 Linux 下，`SYN-ACK`报文的最大重传次数由`tcp_synack_retries`内核参数决定，默认值是 5。
```
# cat /proc/sys/net/ipv4/tcp_synack_retries
5
```
因此，当第二次握手丢失了，客户端和服务端都会重传：
* 客户端会重传`SYN`报文，也就是第一次握手，最大重传次数由`tcp_syn_retries`内核参数决定；
* 服务端会重传`SYN-ACK`报文，也就是第二次握手，最大重传次数由`tcp_synack_retries`内核参数决定。

举个例子，假设`tcp_syn_retries`参数值为 1，`tcp_synack_retries`参数值为 2，那么当第二次握手一直丢失时，发生的过程如下图：

{% asset_img 15.png %}

具体过程：
* 当客户端超时重传 1 次`SYN`报文后，由于`tcp_syn_retries`为 1，已达到最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到服务端的第二次握手（`SYN-ACK`报文），那么客户端就会断开连接。
* 当服务端超时重传 2 次`SYN-ACK`报文后，由于`tcp_synack_retries`为 2，已达到最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到客户端的第三次握手（`ACK`报文），那么服务端就会断开连接。
## 第三次握手丢失了，会发生什么？
客户端收到服务端的`SYN-ACK`报文后，就会给服务端回一个`ACK`报文，也就是第三次握手，此时客户端状态进入到`ESTABLISH`状态。

因为这个第三次握手的`ACK`是对第二次握手的`SYN`的确认报文，所以当第三次握手丢失了，如果服务端那一方迟迟收不到这个确认报文，就会触发超时重传机制，重传`SYN-ACK`报文，直到收到第三次握手，或者达到最大重传次数。

注意，`ACK`报文是不会有重传的，当`ACK`丢失了，就由对方重传对应的报文。

举个例子，假设`tcp_synack_retries`参数值为 2，那么当第三次握手一直丢失时，发生的过程如下图：

{% asset_img 16.png %}

具体过程：
当服务端超时重传 2 次`SYN-ACK`报文后，由于`tcp_synack_retries`为 2，已达到最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到客户端的第三次握手（`ACK`报文），那么服务端就会断开连接。
## 初始序列号 ISN 是如何随机产生的？
起始 ISN 是基于时钟的，每 4 毫秒`+1`，转⼀圈要 4.55 个小时。

RFC1948 中提出了⼀个较好的初始化序列号 ISN 随机生成算法。
```
ISN = M + F(localhost, localport, remotehost, remoteport)
```
M 是⼀个计时器，这个计时器每隔 4 毫秒加 1。 F 是⼀个 Hash 算法，根据源 IP、⽬的 IP、源端口、⽬的端口生成⼀个随机数值。要保证 Hash 算法不能被外部轻易推算得出，用 MD5 算法是⼀个比较好的选择。

# TCP的连接释放
TCP 连接释放过程是通过四次挥手方式。

{% asset_img img2.png %}

数据传输结束后，通信的双方都可释放连接。现在 A 和 B 都处于`ESTABLISHED`状态。A 的应用进程先向其 TCP 发出连接释放报文段，并停止再发送数据，主动关闭 TCP 连接。A 把连接释放报文段首部的`FIN`置1，其序号`seq = u`，它等于前面已传送过的数据的最后一个字节的序号加1。这时 A 进入`FIN-WAIT-1`（终止等待1）状态，等待 B 的确认。TCP 规定，`FIN`报文段即使不携带数据也消耗掉一个序号。

B 收到连接释放报文段后立即发出确认，确认号是`ack = u + 1`，而这个报文段自己的序号是`v`，等于 B 前面已传送过的数据的最后一个字节的序号加1。然后 B 进入`CLOSE-WAIT`（关闭等待）状态。TCP 服务器进程这时应通知高层应用进程，因而从 A 到 B 这个方向的连接就释放了，这时的 TCP 连接处于半关闭状态，即 A 已经没有数据要发送了，但 B 若发送数据，A 仍要接收。也就是说，从 B 到 A 这个方向的连接并未关闭。这个状态可能会持续一些时间。

A 收到来自 B 的确认后，就进入`FIN-WAIT-2`（终止等待2）状态，等待 B 发出的连接释放报文段。

若 B 已经没有要向 A 发送的数据，其应用进程就通知 TCP 释放连接。这时 B 发出的连接释放报文段必须使`FIN = 1`。现假定 B 的序号为`w`（半关闭状态 B 可能又发送了一些数据）。B 还必须重复上次已发送过的确认号`ack = u + 1`。这时 B 就进入`LAST-ACK`（最后确认）状态，等待 A 的确认。

A 在收到 B 的连接释放报文段后，必须对此发出确认。在确认报文段中，`ACK = 1, ack = w + 1`，而自己的序号是`seq = u + 1`。然后进入到`TIME-WAIT`（时间等待）状态。现在 TCP 连接还没有释放掉。必须经过时间等待计时器设置的时间 2MSL 后，A 才进入到`CLOSED`状态。时间 MSL 叫最长报文段寿命(`Maximum Segment Lifetime`)。当 A 撤销相应的传输控制块 TCB 后，就结束了这次 TCP 连接。

B 只要收到了 A 的确认，就进入`CLOSED`状态。同样，B 在撤销相应的传输控制块 TCB 后，就结束了这次的 TCP 连接。B 结束 TCP 连接的时间要比 A 早。

可以看到，每个方向都需要一个 FIN 和一个 ACK，因此通常被称为四次挥手。

需要注意是：主动关闭连接的，才有`TIME_WAIT`状态。
## 为什么挥手需要四次？
关闭连接时，客户端向服务端发送`FIN`时，仅仅表示客户端不再发送数据了但是还能接收数据。

服务器收到客户端的`FIN`报文时，先回一个`ACK`应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送`FIN`报文给客户端来表示同意现在关闭连接。

从上面过程可知，服务端通常需要等待完成数据的发送和处理，所以服务端的`ACK`和`FIN`一般都会分开发送，从而比三次握手导致多了一次。
## 第一次挥手丢失了，会发生什么？
当客户端（主动关闭方）调用`close`函数后，就会向服务端发送`FIN`报文，试图与服务端断开连接，此时客户端的连接进入到`FIN_WAIT_1`状态。

正常情况下，如果能及时收到服务端（被动关闭方）的`ACK`，则会很快变为`FIN_WAIT2`状态。

如果第一次挥手丢失了，那么客户端迟迟收不到被动方的`ACK`的话，也就会触发超时重传机制，重传`FIN`报文，重发次数由`tcp_orphan_retries`参数控制。

当客户端重传`FIN`报文的次数超过`tcp_orphan_retries`后，就不再发送`FIN`报文，则会在等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到第二次挥手，那么直接进入到`close`状态。

举个例子，假设`tcp_orphan_retries`参数值为 3，当第一次挥手一直丢失时，发生的过程如下图：

{% asset_img 17.png %}

具体过程：
当客户端超时重传 3 次`FIN`报文后，由于`tcp_orphan_retries`为 3，已达到最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到服务端的第二次挥手（`ACK`报文），那么客户端就会断开连接。
## 第二次挥手丢失了，会发生什么？
当服务端收到客户端的第一次挥手后，就会先回一个`ACK`确认报文，此时服务端的连接进入到`CLOSE_WAIT`状态。

`ACK`报文是不会重传的，所以如果服务端的第二次挥手丢失了，客户端就会触发超时重传机制，重传`FIN`报文，直到收到服务端的第二次挥手，或者达到最大的重传次数。

举个例子，假设`tcp_orphan_retries`参数值为 2，当第二次挥手一直丢失时，发生的过程如下图：

{% asset_img 18.png %}

具体过程：
当客户端超时重传 2 次`FIN`报文后，由于`tcp_orphan_retries`为 2，已达到最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到服务端的第二次挥手（`ACK`报文），那么客户端就会断开连接。

这里提一下，当客户端收到第二次挥手，也就是收到服务端发送的`ACK`报文后，客户端就会处于`FIN_WAIT2`状态，在这个状态需要等服务端发送第三次挥手，也就是服务端的`FIN`报文。

对于`close`函数关闭的连接，由于无法再发送和接收数据，所以`FIN_WAIT2`状态不可以持续太久，而`tcp_fin_timeout`控制了这个状态下连接的持续时长，默认值是 60 秒。

这意味着对于调用`close`关闭的连接，如果在 60 秒后还没有收到`FIN`报文，客户端（主动关闭方）的连接就会直接关闭，如下图：

{% asset_img 19.png %}

但是注意，如果主动关闭方使用`shutdown`函数关闭连接，指定了只关闭发送方向，而接收方向并没有关闭，那么意味着主动关闭方还是可以接收数据的。

此时，如果主动关闭方一直没收到第三次挥手，那么主动关闭方的连接将会一直处于`FIN_WAIT2`状态（`tcp_fin_timeout`无法控制`shutdown`关闭的连接）。如下图：

{% asset_img 20.png %}

## 第三次挥手丢失了，会发生什么？
当服务端（被动关闭方）收到客户端（主动关闭方）的`FIN`报文后，内核会自动回复`ACK`，同时连接处于`CLOSE_WAIT`状态，顾名思义，它表示等待应用进程调用`close`函数关闭连接。

此时，内核是没有权利替代进程关闭连接，必须由进程主动调用`close`函数来触发服务端发送`FIN`报文。

服务端处于`CLOSE_WAIT`状态时，调用了`close`函数，内核就会发出`FIN`报文，同时连接进入`LAST_ACK`状态，等待客户端返回`ACK`来确认连接关闭。

如果迟迟收不到这个`ACK`，服务端就会重发`FIN`报文，重发次数仍然由`tcp_orphan_retries`参数控制，这与客户端重发`FIN`报文的重传次数控制方式是一样的。

举个例子，假设`tcp_orphan_retries = 3`，当第三次挥手一直丢失时，发生的过程如下图：

{% asset_img 21.png %}

具体过程：
* 当服务端重传第三次挥手报文的次数达到了 3 次后，由于`tcp_orphan_retries`为 3，达到了重传最大次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到客户端的第四次挥手（`ACK`报文），那么服务端就会断开连接。
* 客户端因为是通过`close`函数关闭连接的，处于`FIN_WAIT_2`状态是有时长限制的，如果`tcp_fin_timeout`时间内还是没能收到服务端的第三次挥手（`FIN`报文），那么客户端就会断开连接。

## 第四次挥手丢失了，会发生什么？
当客户端收到服务端的第三次挥手的`FIN`报文后，就会回`ACK`报文，也就是第四次挥手，此时客户端连接进入`TIME_WAIT`状态。

在 Linux 系统，`TIME_WAIT`状态会持续 2MSL 后才会进入关闭状态。

然后，服务端（被动关闭方）没有收到`ACK`报文前，还是处于`LAST_ACK`状态。

如果第四次挥手的`ACK`报文没有到达服务端，服务端就会重发`FIN`报文，重发次数仍然由`tcp_orphan_retries`参数控制。

举个例子，假设`tcp_orphan_retries`为 2，当第四次挥手一直丢失时，发生的过程如下：

{% asset_img 22.png %}

具体过程：
* 当服务端重传第三次挥手报文达到 2 时，由于`tcp_orphan_retries`为 2， 达到了最大重传次数，于是再等待一段时间（时间为上一次超时时间的 2 倍），如果还是没能收到客户端的第四次挥手（`ACK`报文），那么服务端就会断开连接。
* 客户端在收到第三次挥手后，就会进入`TIME_WAIT`状态，开启时长为 2MSL 的定时器，如果途中再次收到第三次挥手（`FIN`报文）后，就会重置定时器，当等待 2MSL 时长后，客户端就会断开连接。

## 为什么 TIME_WAIT 等待的时间是 2MSL？
MSL 是`Maximum Segment Lifetime`，报文最大生存时间，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。因为 TCP 报文基于是 IP 协议的，而 IP 头中有一个 TTL 字段，是 IP 数据报可以经过的最大路由数，每经过一个处理他的路由器此值就减 1，当此值为 0 则数据报将被丢弃，同时发送 ICMP 报文通知源主机。

MSL 与 TTL 的区别：MSL 的单位是时间，而 TTL 是经过路由跳数。所以 MSL 应该要大于等于 TTL 消耗为 0 的时间，以确保报文已被自然消亡。

`TIME_WAIT`等待 2 倍的 MSL，比较合理的解释是：网络中可能存在来自发送方的数据包，当这些发送方的数据包被接收方处理后又会向对方发送响应，所以一来一回需要等待 2 倍的时间。

比如如果被动关闭方没有收到断开连接的最后的`ACK`报文，就会触发超时重发`FIN`报文，另一方接收到`FIN`后，会重发`ACK`给被动关闭方，一来一去正好 2 个 MSL。

2MSL 的时间是从客户端接收到`FIN`后发送`ACK`开始计时的。如果在`TIME-WAIT`时间内，因为客户端的`ACK`没有传输到服务端，客户端又接收到了服务端重发的`FIN`报文，那么 2MSL 时间将重新计时。

在 Linux 系统里 2MSL 默认是 60 秒，那么一个 MSL 也就是 30 秒。Linux 系统停留在`TIME_WAIT`的时间为固定的 60 秒。

其定义在 Linux 内核代码里的名称为`TCP_TIMEWAIT_LEN`：
```
#define TCP_TIMEWAIT_LEN (60*HZ) /* how long to wait to destroy TIME-WAIT 
                                    state, about 60 seconds  */
```
如果要修改`TIME_WAIT`的时间长度，只能修改 Linux 内核代码里`TCP_TIMEWAIT_LEN`的值，并重新编译 Linux 内核。
## A 必须等待 2MSL 的时间
第一，为了保证 A 发送的最后一个`ACK`报文段能够到达 B。这个`ACK`报文段有可能丢失，因而使处在`LAST-ACK`状态的 B 收不到对已发送的`FIN + ACK`报文段的确认。B 会超时重传这个`FIN + ACK`报文段，而 A 就能在 2MSL 时间内收到这个重传的`FIN + ACK`报文段。接着 A 重传一次确认，重新启动 2MSL 计时器。最后，A 和 B 都正常进入到`CLOSED`状态。如果 A 在`TIME_WAIT`状态不等待一段时间，而是在发送完`ACK`报文段后立即释放连接，那么就无法收到 B 重传的`FIN + ACK`报文段，因而也不会再发送一次确认报文段。这样，B 就无法按照正常步骤进入`CLOSED`状态。

第二，防止 “已失效的连接请求报文段”出现在本连接中。A 在发送完最后一个`ACK`报文段后，再经过时间 2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失。这样就可以使下一个新的连接中不会出现这种旧的连接请求报文段。

B 只要收到了 A 发岀的确认，就进入`CLOSED`状态。同样，B 在撤销相应的传输控制块 TCB 后，就结束了这次的 TCP 连接。B 结束 TCP 连接的时间要比 A 早一些。
## 为什么需要 TIME_WAIT 状态？
主动发起关闭连接的一方，才会有`TIME-WAIT`状态。

需要`TIME-WAIT`状态，主要是两个原因：
* 防止具有相同「四元组」的旧数据包被收到；
* 保证「被动关闭连接」的一方能被正确的关闭，即保证最后的`ACK`能让被动关闭方接收，从而帮助其正常关闭；

### 原因一：防止旧连接的数据包
假设`TIME-WAIT`没有等待时间或时间过短，被延迟的数据包抵达后会发生什么呢？

{% asset_img 23.jpg %}

* 如上图黄色框框服务端在关闭连接之前发送的`SEQ = 301`报文，被网络延迟了。
* 这时有相同端口的 TCP 连接被复用后，被延迟的`SEQ = 301`抵达了客户端，那么客户端是有可能正常接收这个过期的报文，这就会产生数据错乱等严重的问题。

所以，TCP 就设计出了这么一个机制，经过 2MSL 这个时间，足以让两个方向上的数据包都被丢弃，使得原来连接的数据包在网络中都自然消失，再出现的数据包一定都是新建立连接所产生的。
### 原因二：保证连接正确关闭
`TIME-WAIT`另一个重要的作用是等待足够的时间以确保最后的`ACK`能让被动关闭方接收，从而帮助其正常关闭。

假设`TIME-WAIT`没有等待时间或时间过短，断开连接会造成什么问题呢？

{% asset_img 24.jpg %}

* 如上图红色框框客户端四次挥手的最后一个`ACK`报文如果在网络中被丢失了，此时如果客户端`TIME-WAIT`过短或没有，则就直接进入了`CLOSED`状态了，那么服务端则会一直处在`LASE_ACK`状态。
* 当客户端发起建立连接的`SYN`请求报文后，服务端会发送`RST`报文给客户端，连接建立的过程就会被终止。

如果`TIME-WAIT`等待足够长的情况就会遇到两种情况：
* 服务端正常收到四次挥手的最后一个`ACK`报文，则服务端正常关闭连接。
* 服务端没有收到四次挥手的最后一个`ACK`报文时，则会重发`FIN`关闭连接报文并等待新的`ACK`报文。

所以客户端在`TIME-WAIT`状态等待 2MSL 时间后，就可以保证双方的连接都可以正常的关闭。
## 保活计时器
除时间等待计时器外，TCP还设有一个保活计时器(`keepalive timer`)，用来防止在 TCP 连接出现长时期的空闲。

设想有这样的情况：客户己主动与服务器建立了 TCP连接。但后来客户端的主机突然出故障。显然，服务器以后就不能再收到客户发来的数据。因此，应当有措施使服务器不要再白白等待下去。这就是使用保活计时器。服务器每收到一次客户的数据，就重新设置保活计时器。

保活计时器通常设置为2小时。若服务器过了2小时还没有收到客户的信息，它就发送探测报文段。若发送了10个探测报文段（每一个相隔75秒）还没有响应，就假定客户出了故障，因而就终止该连接。 
# TCP的有限状态机

{% asset_img img3.png TCP 的有限状态机 %}

图中每一个方框都是 TCP 可能具有的状态。

每个方框中的大写英文字符串是 TCP 标准所使用的 TCP 连接状态名。状态之间的箭头表示可能发生的状态变迁。

箭头旁边的字，表明引起这种变迁的原因，或表明发生状态变迁后又出现什么动作。

图中有三种不同的箭头：
* 粗实线箭头表示对客户进程的正常变迁。
* 粗虚线箭头表示对服务器进程的正常变迁。
* 另一种细线箭头表示异常变迁。 