---
vim: wrap expandtab
layout: blog
title: 用Linux做网关和路由器
---

不用想都知道用linux能当路由器和网关，不光能控制上网，还能开点高级服务什么的，比如做个透明http代理。很多需要的软件都需要自己编译和配置，所以用标准的linux发行版当路由器的需求还是有的。

这个几年前就配过，前几天想再装一台网关时，居然忘记怎么配置了…… 所以还是记下来吧。

## IP转发
Linux IP 转发很简单，打开ip_forward就可以了：
{% highlight sh %}
echo 1 >/proc/sys/net/ipv4/ip_forward
{% endhighlight %}

或者：

{% highlight sh %}
sysctl -w net.ipv4.ip_forward=1
{% endhighlight %}

在centos中设置：

{% highlight sh %}
# file: /etc/sysctl.conf
net.ipv4.ip_forward = 1
{% endhighlight %}


## 用iptables做NAT
如果只是打开了IP转发，但是没有任何NAT规则的话，linxu现在还只有路由功能，还不能让机器共享上网，要想共享上网，还需要设置NAT规则。

NAT可以用iptables或者ip指令来做， ip命令我还不会用，先用iptables来配：
{% highlight sh %}
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 192.168.0.1
{% endhighlight %}

也可以这样:
{% highlight sh %}
iptables -t nat -A POSTROUTING -j MASQUERADE
{% endhighlight %}
