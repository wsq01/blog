


# RIPv2 基础配置
下面的拓扑图中有三台路由器，我们在路由器上部署 RIPv2 ，让网络中的各个网段能够实现互通。

{% asset_img 1.png %}

RT1 配置：
```
<Huawei>system-view
[Huawei]sysname RT1
[RT1]interface GigabitEthernet 0/0/0
[RT1-GigabitEthernet0/0/0]ip address 192.168.1.1 30
[RT1-GigabitEthernet0/0/0]quit
[RT1]interface GigabitEthernet 0/0/1
[RT1-GigabitEthernet0/0/1]ip address 172.16.1.254 24
[RT1-GigabitEthernet0/0/1]quit
[RT1]
[RT1]rip 1
[RT1-rip-1]version 2
[RT1-rip-1]network 192.168.1.0
[RT1-rip-1]network 172.16.0.0
```
配置说明：
* `rip 1`：数字 1 表示 RIP 的进程 ID。如果不配置，系统会自动生成一个。一个设备运行不同 RIP 进程，使用进程 ID 区分，且相互独立。
* `version 2`：配置 RIP 的版本，这里配置的是 RIPv2。
* `network 192.168.1.0`：`network`命令用于网段的激活。
* `network 172.16.0.0`：需要注意的是`network`命令指定的必须是主类网络地址，而不是子网地址。如果使用`network 172.16.1.0`命令，那么系统会报错，因为`172.16.1.0`是一个子网地址，而不是主类地址。

