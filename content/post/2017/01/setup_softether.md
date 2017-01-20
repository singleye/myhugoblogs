+++
date = "2017-01-10T15:22:39Z"
title = "搭建SoftEther VPN 服务"
categories = ["Technology"]
tags = ["VPN", "SoftEther"]
draft = false
summary = "最近在公司内部架了台服务器给团队的同学们使用，可是在公司外面的时候就没办法再继续访问上面的资源了实在不太方便，虽然利用公司路由器的可以将服务器端口映射出去，但这样做只能访问部分资源，还是搭建一套VPN服务器更加适合所有人的需求。"
+++
最近在公司内部架了台服务器给团队的同学们使用，可是在公司外面的时候就没办法再继续访问上面的资源了实在不太方便，虽然利用公司路由器的可以将服务器端口映射出去，但这样做只能访问部分资源，还是搭建一套VPN服务器更加适合所有人的需求。

研究了一下PPP/L2TP，但PPP已经在iOS设备上见不到了，L2TP又对CentOS 7.3支持不友好，后来找到了SoftEther，发现不仅协议支持全面而且支持命令行／图形界面的管理工具，使用和管理都很方便。

# 搭建方法
## 通过Local bridge的方式访问远程网络
具体过程可以参照[官方文档](https://www.softether.org/4-docs/2-howto/1.VPN_for_On-premise/2.Remote_Access_VPN_to_LAN)，这种方法也是我最开始使用的方法，但搭建好之后发现这种连接的设备无法访问搭建VPN的服务器本身，但是我们这台服务器上提供了需要大家访问的资源，因此这对与我们公司来说是无法接受的。原因也可以理解，因为Linux不允许访问Local bridge网络接口对应的IP。具体可以参考官方文档的[解释](https://www.softether.org/4-docs/1-manual/3._SoftEther_VPN_Server_Manual/3.6_Local_Bridges)：

> Limitations within the Linux or UNIX operating system prevent communication with IP addresses assigned to the network adapter locally bridged from the VPN side (Virtual Hub side). The cause of this restriction lies with OS's internal kernel codes rather than with the SoftEther VPN. When wishing to communicate in any form with a UNIX computer used for local bridging from the VPN side (Virtual Hub side), (for instance, when running both the VPN Server / VPN Bridge service & the HTTP Server service and wishing to grant access to the server service from the VPN side as well), prepare and connect a local bridge network adapter and physically connect both it and the existing network adapter to the same segment (as explained in 3.6 Local Bridges, it is recommended to prepare a network adapter for exclusive use in local bridging for this and other situations).

## 通过tap网络接口
后来发现可以通过tap设备来解决访问VPN服务器本身的问题，原理就是创建出一个tap设备并将它桥接到真实设备上。tap设备的配置过程和Local bridge方式基本一致，可以参考下图：
![Tap device](https://wordpress.youran.me/wp-content/uploads/2014/12/create-local-bridge.png)

## CentOS系统设置
完成tap设备配置后，在CentOS上还需要完成下面的设置才可以使用：

1. 防火墙配置，允许VPN服务器的配置和客户连接：
{{< highlight shell "linenos=inline,style=manni" >}}
$ sudo firewall-cmd –zone=public –add-port=500/udp –permanent
$ sudo firewall-cmd –zone=public –add-port=4500/udp –permanent
$ sudo firewall-cmd –zone=public –add-port=1701/udp –permanent
$ sudo firewall-cmd –zone=public –add-port=443/tcp –permanent
$ sudo firewall-cmd –reload
{{< /highlight >}}

2. 配置tap device：
{{< highlight shell "linenos=inline,style=manni" >}}
$ sudo brctl addif br0 tap_soft
$ sudo brctl show
bridge name bridge id STP enabled interfaces
br0 8000.00acc3a0fd4d yes p4p2
tap_soft
{{< /highlight >}}
