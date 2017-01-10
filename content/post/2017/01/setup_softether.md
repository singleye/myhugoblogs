+++
date = "2017-01-10T15:22:39Z"
title = "搭建SoftEther VPN 服务"
categories = ["Technology"]
tags = ["VPN", "SoftEther"]
draft = true

+++

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
