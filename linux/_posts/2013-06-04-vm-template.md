---
vim: wrap expandtab ft=markdown
layout: blog
title: 配置虚拟机模板
---

装好了虚拟机，并且安装了一些软件后，我们对这台虚拟机进行一些配置，以后就不用再重新安装系统了，需要新的系统时只要克隆这个虚拟机就好。

## 禁用ipv6
首先我们可以禁用ipv6，因为在内网里用不到…… 

创建文件`/etc/modprobe.d/disable-ipv6.conf`，内容如下：
{% highlight bash %}
install ipv6 /bin/true
{% endhighlight %}

在文件`/etc/sysconfig/network`中加入：
{% highlight bash %}
NETWORKING_IPV6=no
{% endhighlight %}


在文件`/etc/sysctl.conf`中加入：
{% highlight bash %}
net.ipv6.conf.all.disable_ipv6 = 1
{% endhighlight %}

最后可以删除iptables-ipv6这个软件包：
{% highlight bash %}
$ yum remove iptables-ipv6
{% endhighlight %}


## 禁用selinux
很多软件配置问题都是由于selinux引起的，为了方便一般用户使用，可以把selinux禁用掉，对于在内网内的机器安全性影响不是很大。

修改文件`/etc/selinux/config`
{% highlight bash %}
SELINUX=disabled
{% endhighlight %}

使用`getenforce`命令可以查询当前selinux的状态，想用命令关闭或者打开selinux，可以使用:
{% highlight bash %}
$ setenforce 0
{% endhighlight %}

我们还可以删掉多余的软件包：
{% highlight bash %}
$ yum remove selinux*
{% endhighlight %}


## 配置源服务器
由于我们在宿主机器上已经配置过源了， 所以只需要把源配置文件考过来就行了：
{% highlight bash %}
$ rm /etc/yum.repos.d/*
$ scp root@mirrors.tiwb.net:/etc/yum.repos.d/* /etc/yum.repos.d/
{% endhighlight %}

## 关闭防火墙
由于虚拟机全部是跑在网关防火墙后面的，所以就没必要再自己开个防火墙了，可以直接把防火墙关闭掉，省去管理它的麻烦…… 当然也可以把防火墙规则清空。
{% highlight bash %}
$ chkconfig iptables off
$ service iptables stop
{% endhighlight %}

## 安装acpid
虚拟机要能正常地响应电源事件，才能在主系统里直接重起或关机，所以要安装acpi
{% highlight bash %}
$ yum install acpid
{% endhighlight %}

## 修改网络配置
把修改一下网络配置，这样在我们更换了网卡以后就不用再次修改它了：

文件`/etc/sysconfig/network-scripts/ifcfg-eth0`的内容：
{% highlight bash %}
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=dhcp
{% endhighlight %}

如果不想在路由表中看到link-local，可以加入设置NOZEROCONF，我们默认的主机名是centos，如果dhcp失败，就没有主机名。

完整的`/etc/sysconfig/network`配置文件如下：
{% highlight bash %}
NOZEROCONF=yes
NETWORKING=yes
NETWORKING_IPV6=no
DHCP_HOSTNAME=centos
{% endhighlight %}

## 清理工作
下面要做一些清理工作，以便我们以后直接复制虚拟机镜像后能直接正常工作，首先就是要把网卡的信息删掉，否则eth0这个网卡名称会被占用，当换了mac地址以后会多出一块新的网卡eth1，还需要重新配置。

执行脚本:
{% highlight bash %}
#!/bin/sh

# clean net rules.
rm /etc/udev/rules.d/70-persistent-net.rules

# clean dhcp leases
rm /var/lib/dhclient/dhclient-eth0.leases

# clean yum cache
yum clean all

# empty logs
for i in `find /var/log -type f` do
  cat /dev/null >$i
done

# remove histories
rm ~/.bash_history
rm ~/.viminfo
rm ~/.vimundodir/*

{% endhighlight %}

然后就可以poweroff了，现在可以安全的把镜像备份好，供以后快速复制使用，或者用来做差分镜像。
