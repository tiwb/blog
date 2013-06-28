---
vim: wrap expandtab
layout: blog
title: Linux 防火墙配置
---

在配置防火墙时，最重要的就是要把已经连接上的连接加入到规则中，前期配置就是因为没有加这一项，导致防火墙总是失败的。因为连接是双向的，必须2个方向都允许了才可以通信，而我在配置的时候一般只写单方向的。

{% highlight sh %}
# file: /etc/sysconfig/iptables
# eth0: internet connection with ip: 1.1.1.1
# eth1: local connection with ip 192.168.0.1

*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:INTERNET - [0:0]

# proxy public port to local machine
-A PREROUTING -d 1.1.1.1 -p tcp --dport 12345 -j DNAT --to-destination 192.168.0.1:12345

# default go internet
-A POSTROUTING -j INTERNET

# nat to without local address
-A INTERNET -d 0.0.0.0/8 -j RETURN
-A INTERNET -d 10.0.0.0/8 -j RETURN
-A INTERNET -d 127.0.0.0/8 -j RETURN
-A INTERNET -d 169.254.0.0/16 -j RETURN
-A INTERNET -d 172.16.0.0/12 -j RETURN
-A INTERNET -d 192.168.0.0/16 -j RETURN
-A INTERNET -d 224.0.0.0/4 -j RETURN
-A INTERNET -d 240.0.0.0/4 -j RETURN
-A INTERNET -j MASQUERADE

COMMIT


*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]

# enable all established connections.
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# http enables on all interface.
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT

# loopback
-A INPUT -i lo -j ACCEPT

# input from internet
-A INPUT -i eth0 -p tcp -m tcp --dport 58422 -j ACCEPT
-A INPUT -i eth0 -p icmp -j DROP
-A INPUT -i eth0 -j REJECT

# others
-A INPUT -p icmp -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p udp -m udp --dport 53 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 53 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 67 -j ACCEPT
-A INPUT -p udp -m udp --dport 67 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited


# connected state of forward is allowed.
-A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT

# forward to local net is allowed.
-A FORWARD -d 10.0.0.0/24 -j ACCEPT
-A FORWARD -d 192.168.0.0/16 -j ACCEPT

# forward of a mac address.
-A FORWARD -m mac --mac-source 00:00:00:00:00:00 -j ACCEPT

# from 192.168.0.x to all addresses is allowed.
-A FORWARD -s 192.168.0.0/8 -j ACCEPT

# from any host to a public address is allowed.
-A FORWARD -d tiwb.com -j ACCEPT

# disable others
-A FORWARD -j REJECT --reject-with icmp-host-prohibited

COMMIT


*mangle
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]

COMMIT
{% endhighlight %}
