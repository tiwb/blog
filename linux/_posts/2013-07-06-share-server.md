---
vim: wrap expandtab ft=markdown
layout: blog
title: 搭建资料共享服务器
---

有了虚拟机环境以后，就可以开始架设各种各样的服务了， 最常用的就是文件共享服务了，我准备把文件共享放到一台服务器上，名字就叫"share"吧，附加一块100G的虚拟盘，之前在配置svn服务器的时候已经介绍过如何附加硬盘了，这里就再说明了。


## 用samba做Windows共享
先搭个windows共享，可以放一些软件的安装程序之类的，先做一个简单的samba共享吧。

首先安装samba4

{% highlight bash %}
$ yum install samba3
{% endhighlight %}

让smb服务自动启动：

{% highlight bash %}
$ chkconfig smb on
{% endhighlight %}

修改samba的配置文件：
{% highlight bash %}
# file: /etc/samba/smb.conf

[global]
  workgroup = TIWB
  server string = Samba Server Version %v

  # log
  log file = /var/log/samba/log.%m
  max log size = 50

  # security
  security = user
  passdb backend = tdbsam

  # printer
  load printers = no
  cups options = raw

[public]
  comment = Public Stuff
  path = /data/samba
  public = yes
  writable = yes
  printable = no
  write list = root

{% endhighlight %}

配置好以后就可以启动samba服务了：

{% highlight bash %}
$ service smb start
{% endhighlight %}

最后可以用smbclient命令检查samba服务，如果你之前安装过这个命令：
{% highlight bash %}
$ smbclient -L share -U%
{% endhighlight %}
