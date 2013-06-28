---
vim: wrap expandtab
layout: blog
title: CentOS虚拟机模板的一些配置
---

装好了虚拟机，并且安装了一些软件后，需要对虚拟机进行一定的设置能让它更好的工作。

## 禁用ipv6
首先我们可以禁用ipv6，因为在内网里用不到……
{% highlight bash %}
# file /etc/modprobe.d/disable-ipv6.conf
install ipv6 /bin/true
{% endhighlight %}

{% highlight bash %}
# file /etc/sysconfig/network
NETWORKING=yes
NETWORKING_IPV6=no
DHCP_HOSTNAME=centos
{% endhighlight %}


{% highlight bash %}
# file /etc/sysctl.conf
...
# Disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
{% endhighlight %}

{% highlight bash %}
$ yum remove iptables-ipv6
{% endhighlight %}


## 禁用selinux
很多软件配置问题都是由于selinux引起的，为了方便一般用户使用，可以把selinux禁用掉，对于在内网内的机器安全性影响不是很大。

{% highlight bash %}
# file: /etc/selinux/config
SELINUX=disabled
{% endhighlight %}

使用`getenforce`命令可以查询当前selinux的状态，想用命令关闭或者打开selinux，可以使用:
{% highlight bash %}
$ setenforce 0
{% endhighlight %}


## 配置源服务器
由于我们在宿主机器上已经配置过源了， 所以只需要把源配置文件考过来就行了：
{% highlight bash %}
$ rm /etc/yum.repos.d/*
$ scp root@vm1.tiwb.com:/etc/yum.repos.d/* /etc/yum.repos.d/
{% endhighlight %}

## 关闭防火墙
由于虚拟机全部是跑在网关防火墙后面的，所以就没必要再自己开个防火墙了，可以直接把防火墙关闭掉，省去管理它的麻烦…… 当然也可以把防火墙规则清空。
{% highlight bash %}
$ chkconfig iptables off
$ service iptables stop
{% endhighlight %}


## 清理工作
下面要做一些清理工作，以便我们以后直接复制虚拟机镜像后能直接正常工作，首先就是要把网卡的信息删掉，否则eth0这个网卡名称会被占用，当换了mac地址以后会多出一块新的网卡eth1，还需要重新配置。
{% highlight bash %}
$ rm /etc/udev/rules.d/70-persistent-net.rules
{% endhighlight %}

清除dhcp缓存
{% highlight bash %}
$ rm /var/lib/dhclient/dhclient-eth0.leases
{% endhighlight %}

清理yum缓存
{% highlight bash %}
$ yum clean all
{% endhighlight %}


清除log文件
根据需要可以清理下log文件，注意在清理是不要直接删掉log文件，可以用下面的方法来清理:
{% highlight bash %}
$ cat /dev/null >/var/log/boot.log
{% endhighlight %}

还可以删掉命令记录…… 怎么感觉像干过什么坏事一样。。
{% highlight bash %}
$ rm ~/.bash_history
$ rm ~/.viminfo
$ rm ~/.vimundodir/*
{% endhighlight %}

然后就可以poweroff了，现在可以安全的把镜像备份好，供以后快速复制使用，或者用来做差分镜像。
