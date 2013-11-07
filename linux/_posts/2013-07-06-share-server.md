---
vim: wrap expandtab ft=markdown
layout: blog
title: 搭建资料共享服务器
---

有了虚拟机环境以后，就可以开始架设各种各样的服务了， 最常用的就是文件共享服务了，我准备把文件共享放到一台服务器上，名字就叫"share"吧，附加一块100G的虚拟盘，之前在配置svn服务器的时候已经介绍过如何附加硬盘了，这里就再说明了。


## 配置Samba匿名共享
先搭个windows共享，可以放一些软件的安装程序之类的，先做一个简单的samba共享吧。

安装samba：

{% highlight bash %}
$ yum install samba
{% endhighlight %}

让smb服务自动启动：

{% highlight bash %}
$ chkconfig smb on
{% endhighlight %}

修改samba的配置文件`/etc/samba/smb.conf`
{% highlight bash %}
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

  # allow guest
  map to guest = bad user
  guest account = nobody

[public]
  comment = Public Stuff
  path = /data/samba/public
  public = yes
  writable = yes

{% endhighlight %}

然后建立public目录，并把所有者给nobody:
{% highlight bash %}
$ mkdir -p /data/samba/public
$ chown nobody:nobody /data/samba/public
{% endhighlight %}


配置好以后就可以启动samba服务了：

{% highlight bash %}
$ service smb start
{% endhighlight %}

最后可以用smbclient命令检查samba服务，如果你之前安装过这个命令：
{% highlight bash %}
$ smbclient -L share -U%
{% endhighlight %}

如果在windows上要输入用户名， 那么输入nobody就可以了。


## Samba集成LDAP验证

首先， 要导入samba的schema文件，samba的文档里有一个转换好的ldif文件，所以直接拷贝过去：
{% highlight bash %}
$ scp /usr/share/doc/samba-3.6.9/LDAP/samba.ldif root@ldap.tiwb.net:~
{% endhighlight %}

然后到ldap的服务器上导入这个文件：
{% highlight bash %}
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f cn\=samba.ldif
{% endhighlight %}


接下来，要配置服务器用ldap登录：

安装nss-pam-ldapd
{% highlight bash %}
$ yum install nss-pam-ldapd
{% endhighlight %}

然后运行authconfig-tui，根据向导填写信息。

在`/etc/samba/smb.conf`中加入ldap的配置：
{% highlight ini %}
[global]
  passdb backend = ldapsam:"ldap://ldap.tiwb.net"
  ldap ssl = no
  ldap suffix = dc=tiwb,dc=net
  ldap user suffix = ou=users
  ldap group suffix = ou=groups
  ldap machine suffix = ou=machines
  ldap admin dn = cn=Manager,dc=tiwb,dc=net
{% endhighlight %}

重启服务，然后修改密码：

{% highlight bash %}
$ service smb restart
$ smbpasswd lijia
{% endhighlight %}
