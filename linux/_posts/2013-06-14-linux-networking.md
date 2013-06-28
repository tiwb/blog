---
vim: wrap expandtab
layout: blog
title: Linux高级网络配置
---

大多数情况下，仅配置物理网卡就可以满足需求了，但是还有特殊情况，有些时候，我们就需要VLAN和桥接这类复杂的网络结构了，这里的情况是：为了开虚拟机，需要使用桥接，如果想把虚拟机的网络和物理机器完全分开，那就需要划分VLAN了，在linux中管理vlan需要交换机的端口能够被设置成trunk模式。

我自己对于网桥的理解是网桥就像个HUB一样，能够把多个物理网卡或者虚拟网卡连接在一起，要注意的是配置完以后使用网桥时需要在网桥上配置IP地址， 而不是在原来的网卡上配。

在centos上配置网桥很简单，新建一个配置文件：
{% highlight sh %}
# file: /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
TYPE=Bridge
BOOTPROTO=dhcp
ONBOOT=yes
STP=on
{% endhighlight %}

如果想把一个端口加入到网桥里，也只需要改一下配置文件就好：
{% highlight sh %}
# file: /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
TYPE=Ethernet
BRIDGE=br0
{% endhighlight %}

## 配置VLAN
首先要加载802.1q模块：
{% highlight sh %}
modprobe 8021q
{% endhighlight %}

然后就可以写配置文件了，在centos中，只需要在一个网卡的名字后买再加上vlan的ID就可以了……比如我们要配置vlan的ID时10:
{% highlight sh %}
# file: /etc/sysconfig/network-scripts/eth0.10
DEVICE=eth0.10
ONBOOT=yes
BOOTPROTO=dhcp
VLAN=yes
{% endhighlight %}

让然vlan的接口也可以被加到网桥里。
