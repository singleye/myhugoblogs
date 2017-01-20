+++
date = "2017-01-10T15:22:39Z"
title = "搭建SoftEther VPN 服务"
categories = ["Technology"]
tags = ["VPN", "SoftEther"]
draft = true
summary = "最近在公司内部架了台服务器给团队的同学们使用，可是在公司外面的时候就没办法再继续访问上面的资源了实在不太方便，虽然利用公司路由器的可以将服务器端口映射出去，但这样做只能访问部分资源，还是搭建一套VPN服务器更加适合所有人的需求。"
+++
最近在公司内部架了台服务器给团队的同学们使用，可是在公司外面的时候就没办法再继续访问上面的资源了实在不太方便，虽然利用公司路由器的可以将服务器端口映射出去，但这样做只能访问部分资源，还是搭建一套VPN服务器更加适合所有人的需求。

研究了一下PPP/L2TP，但PPP已经在iOS设备上见不到了，L2TP又对CentOS 7.3支持不友好，后来找到了SoftEther，发现不仅协议支持全面而且支持命令行／图形界面的管理工具，使用和管理都很方便。


1. 防火墙配置
<pre><code>
firewall-cmd –zone=public –add-port=500/udp –permanent
firewall-cmd –zone=public –add-port=4500/udp –permanent
firewall-cmd –zone=public –add-port=1701/udp –permanent
firewall-cmd –zone=public –add-port=443/tcp –permanent
firewall-cmd –reload
</code></pre>

2. 配置tap device
<pre><code>
$ sudo  brctl addif br0 tap_vpn
</code></pre>
<pre><code>
$ sudo brctl show
bridge name bridge id STP enabled interfaces
br0 8000.00acc3a0fd4d yes p4p2
tap_vpn
</code></pre>
