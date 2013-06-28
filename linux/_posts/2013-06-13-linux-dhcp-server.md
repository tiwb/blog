---
vim: wrap expandtab
layout: blog
title: 用dnsmasq架设DHCP和DNS服务
---

当我们配置好了网关以后，就可以用linux网关来上网了，但是为了使用方便，还需要架设一个DHCP服务，这样客户端机器就可以自动获取IP地址了。当有了DHCP服务以后，还可以做一个DNS服务，网络上的机器就可以全部用名字来访问了，而且还可以做一些DNS缓存之类的很方便的服务，下面就来架设这2个服务。

对于一般的家庭和公司，使用轻量级的dnsmasq服务就完全能满足需求，又好用配置也简单，在装好了dnsmasq服务以后，就可以开始写配置文件了。

在以下的配置中，tiwb.com是我们的本地域名，一般情况下，可以直接写一个公司的名字或者用个互联网上不存在的域名来玩。
我们本地使用192.168.0.0/8这个网段，所以反解的地址是0.168.192.in-addr.arpa。反解可是很重要的，有了它，才能从192.168.0.1这个地址对应到tiwb.com上，实际使用中，ping一下IP地址就知道是谁的电脑了～ 而且如果没有正确的设置反解服务，一些用到这个特性的程序就会变得很‘卡’。

我们架设在内网环境中，还存在另外一台DNS服务器，比如公司可能会有windows的域控制器，而且linux服务器群在另外一个网段中， 这样就需要转发另外一个dns服务器了信息了，我们架设另外一个子网是tiwb.net，它的dns服务器地址是10.0.0.1。完整的配置文件是这样子的：

{% highlight sh %}
# file: /etc/dnsmasq.conf

# local domain is 'tiwb.com'
domain=tiwb.com

# our internet connection is eth1, so no DHCP for that interface.
no-dhcp-interface=eth1

# real resolv file is here
resolv-file=/etc/resolv.dnsmasq

dhcp-authoritative
filterwin2k

# Set the NIS domain name to 'tiwb.com'
# dhcp-option=40,tiwb.com

# this is our primary dhcp range
dhcp-range=192.168.0.10,192.168.0.200,255.255.255.0,12h

# this domains are local
local=/tiwb.com./
local=/0.168.192.in-addr.arpa./

# another local domain servered by another DNS server.
server=/tiwb.net./10.0.0.1
server=/0.0.10.in-addr.arpa./10.0.0.1

# dhcp hosts with fixed ip address
dhcp-host=192.168.0.2,www.tiwb.com
dhcp-host=192.168.0.3,mail.tiwb.com

# logs
log-queries
log-dhcp

{% endhighlight %}

除了以上的配置文件以外，还需要配置resolv.dnsmasq，这个配置文件里是真正的上游dhcp服务器：
{% highlight sh %}
# file: /etc/resolv.dnsmasq
nameserver 114.114.114.114
nameserver 8.8.8.8
{% endhighlight %}

而真正本机用到的解析文件里指向本机就可以了：
{% highlight sh %}
# file: /etc/resolv.conf
nameserver 127.0.0.1
{% endhighlight %}

除了在dnsmasq的服务器中可以配置固定域名， 在hosts文件中配置也非常方便：
{% highlight sh %}
# file: /etc/hosts

127.0.0.1       localhost
192.168.0.1     tiwb.com gateway.tiwb.com
192.168.0.4     svn.tiwb.com

# you can also 'hack' on some domains with /etc/hosts
74.125.71.104   encrypted.google.com
{% endhighlight %}


## 配置dhcp客户端
至此，DHCP和DNS服务器就架设好了，其他的机器，只要用DHCP自动获取IP地址就好，对于centos，可以这么配：
{% highlight sh %}
# file: /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
{% endhighlight %}

还需要另外一个配置文件才能告诉dhcp服务本机的名称：
{% highlight sh %}
# file: /etc/sysconfig/network
NETWORKING=yes
NOZEROCONF=yes
DHCP_HOSTNAME=svn
{% endhighlight %}
