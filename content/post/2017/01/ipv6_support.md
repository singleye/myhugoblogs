+++
tags = ["IPv6", "Network", "阿里云"]
date = "2017-01-18T18:56:51+08:00"
title = "阿里云IPv6支持方案"
categories = ["Technology"]
draft = false
summary = "最近团队开发的APP提交到苹果APP store时被拒了，原因是不支持IPv6的访问。原来苹果App store从2016年6月开始强制新上线APP支持IPv6网络，但由于IPv6基础设施在国内的推广非常缓慢，因此导致了该问题。"
+++
最近团队开发的APP提交到苹果APP store时被拒了，原因是不支持IPv6的访问。原来苹果App store从2016年6月开始强制新上线APP支持IPv6网络，但由于IPv6基础设施在国内的推广非常缓慢，因此导致了该问题。


# IPv6介绍
为了理解IPv6需要先了解其产生的原因，就是IPv4地址资源的问题。IPv4的网络使用32位的地址空间（XX.XX.XX.XX），因此最大支持的数量是4,294,967,296（2^32个），其中还有1800多万个[私有地址](https://zh.wikipedia.org/wiki/专用网络)和2.7亿个[多播地址](https://zh.wikipedia.org/wiki/多播)。互联网的发展显然超出了普通的32位地址空间的容量，IPv6地址使用128位的地址空间，这意味着几乎取之不尽的地址空间。另外IPv6比IPv4还进行了很多的改进与扩充。

## IPv6地址
1. 冒分16进制表示法（XXXX:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX），每个部分中的0可以省略。比如：2001:0DB8:0000:0023:0008:0800:200C:417A 可以缩写为2001:DB8:0:23:8:800:200C:417A

2. 0位压缩。如果地址中包含很多连续的0，可以把0压缩为"::"，并且"::"只能出现1次。
比如 FF01:0:0:0:0:0:0:1101 可以缩略为 FF01::1101

3. 内嵌IPv4地址表示法。为了实现IPv4-IPv6互通，IPv4地址会嵌入IPv6地址中，此时地址常表示为：X:X:X:X:X:X:d.d.d.d，前96b采用冒分十六进制表示，而最后32b地址则使用IPv4的点分十进制表示，例如::192.168.0.1与::FFFF:192.168.0.1就是两个典型的例子，注意在前96b中，压缩0位的方法依旧适用。

## IPv6地址分类
| 地址类型 | IPv4 | IPv6 |
| -------- | ---- | ---- |
| 单播(unicast) | Yes | Yes |
| 组播(multicast) | Yes | Yes |
| 任播(anycast) | No | Yes|
| 广播 | Yes | No (通过组播来达到类似目的) |

IPv6的地址类型通过地址的前缀进行区别

| IPv6地址类型 | 前缀标识 |
| ------------ | -------- |
| Loopback (unicast)     | ::1/128  |
| Link local (unicast)   | FE80::/10 |
| Site local (unicast)   | FEC0::/10 |
| Global (unicast)       |           |
| multicast              | FF00::/8  |
| anycast                | 从单播地址空间中分配 |

## IPv4 vs IPv6
IPv6比IPv4的优势：
1. IPv6具有更大的地址空间。IPv4中规定IP地址长度为32，最大地址个数为2^32；而IPv6中IP地址的长度为128，即最大地址个数为2^128。与32位地址空间相比，其地址空间增加了2^128-2^32个。

2. IPv6使用更小的路由表。IPv6的地址分配一开始就遵循聚类（Aggregation）的原则，这使得路由器能在路由表中用一条记录（Entry）表示一片子网，大大减小了路由器中路由表的长度，提高了路由器转发数据包的速度。

3. IPv6增加了增强的组播（Multicast）支持以及对流的控制（Flow Control），这使得网络上的多媒体应用有了长足发展的机会，为服务质量（QoS，Quality of Service）控制提供了良好的网络平台。

4. IPv6加入了对自动配置（Auto Configuration）的支持。这是对DHCP协议的改进和扩展，使得网络（尤其是局域网）的管理更加方便和快捷。

5. IPv6具有更高的安全性。在使用IPv6网络中用户可以对网络层的数据进行加密并对IP报文进行校验，在IPV6中的加密与鉴别选项提供了分组的保密性与完整性。极大的增强了网络的安全性。

6. 允许扩充。如果新的技术或应用需要时，IPV6允许协议进行扩充。

7. 更好的头部格式。IPV6使用新的头部格式，其选项与基本头部分开，如果需要，可将选项插入到基本头部与上层数据之间。这就简化和加速了路由选择过程，因为大多数的选项不需要由路由选择。

8. 新的选项。IPV6有一些新的选项来实现附加的功能[14]  。


# IPv6支持方法

## 6in4隧道方式

1. 创建tunnel
到![tunnelbroker](https://www.tunnelbroker.net)注册账号，并且创建一个新的常规(Regular) tunnel。创建时候需要在'IPv4 Endpoint'栏填入服务器的公网IPv4地址，并在'Available Tunnel Servers'中选择一个适合自己的服务器区域，过程如下图：
![创建tunnel](/media/2017/01/create_tunnel.jpg)

创建完成后的tunnel包含了几个重要的信息：
* Server IPv4 Address: 这个是tunnel服务端的IPv4地址，创建tunnel的时候需要用到。
* Server IPv6 Address: 这个是tunnel的服务端IPv6地址。
* Client IPv4 Address: 这个是tunnel客户端的IPv4地址。
* Client IPv6 Address: 这个地址需要设置在CentOS服务器的tunnel上面，也是后面DNS服务器需要设置的AAAA记录对应的地址。
![tunnel详情](/media/2017/01/tunnel_detail.jpg)

2. 解除阿里云主机IPv6限制
阿里云的CentOS主机默认状态下是把IPv6给禁掉的，可以使用下面的脚本先把系统的IPv6功能打开。

{{< highlight shell "linenos=inline,style=manni" >}}
#!/bin/sh

mkdir ipv6
mv /etc/modprobe.d/disable_ipv6.conf ipv6/
modprobe ipv6

cp /etc/sysctl.conf ipv6/
cat /etc/sysctl.conf | sed -e 's/disable_ipv6 = 1/disable_ipv6 = 0/' >
/etc/.sysctl.conf.bak
mv -f /etc/.sysctl.conf.bak /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
{{< /highlight >}}

完成上面配置后可以用'ifconfig'检验一下网络接口，如果出现'inet6'类型的信息说明配置已经生效。

3. 配置CentOS服务器端tunnel
这一步需要用到上面创建tunnel时的'Server IPv4 Address'/'Client IPv4 Address'/'Client IPv6 Address'

{{< highlight shell "linenos=inline,style=manni" >}}
ip tunnel add he-ipv6 mode sit remote [Server IPv4 Address] local [Client IPv4 Address] ttl 255
ip link set he-ipv6 up
ip addr add [Client IPv6 Address]/64 dev he-ipv6
ip route add ::/0 dev he-ipv6
ip -f inet6 addr
{{< /highlight >}}

完成这一步后已经可以对'[Client IPv6 Address]'进行访问了，可以通过'ping6'或者'curl'进行验证。

{{< highlight shell "linenos=inline,style=manni" >}}
# ping6 [Client IPv6 Address]
# curl --globoff -6 [Client IPv6 Address]
{{< /highlight >}}

4. 设置DNS AAAA记录
大家熟悉的A记录是DNS中IPv4的对应地址，相应的IPv6地址叫AAAA记录。设置成功后就可以直接用DNS进行访问了。


# 附录
## IPv4私有地址
| RFC1918 规定区块名 | IP地址区段 | IP数量 | 分类网络说明 | 最大CIDR区块（子网络遮罩） | 主机端位长|
| ------ | ------ | ------ | ------ | ------ | ------ |
| 24位区块 | 10.0.0.0 – 10.255.255.255 | 16,777,216 | 单个A类网络 | 10.0.0.0/8(255.0.0.0) | 24位 |
| Shared Address Space | 100.64.0.0 - 100.127.255.255 | 4,194,304 | 64个连续B类网络 | 100.64.0.0/10 (255.192.0.0) | 22位 |
| 20位区块 | 172.16.0.0 – 172.31.255.255 | 1,048,576 | 16个连续B类网络 | 172.16.0.0/12(255.240.0.0) | 20位 |
| 16位区块 | 192.168.0.0 – 192.168.255.255 | 65,536 | 256个连续C类网络 | 192.168.0.0/16(255.255.0.0) | 16位 |

# 参考资料
* https://zh.wikipedia.org/wiki/IPv6
* http://baike.baidu.com/item/IPv6
* http://test-ipv6.com/faq_6to4.html
