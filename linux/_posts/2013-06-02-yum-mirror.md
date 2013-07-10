---
vim: wrap expandtab ft=markdown
layout: blog
title: 本地yum源镜像
---

CentOS直接连着网上的源装软件，少了还可以，多了就觉得慢了……所以还是在本地做个源同步吧，写个cron脚本就好了：

文件: /etc/cron.daily/sync-mirrors.cron
{% highlight bash %}
#!/bin/bash
date=`date +%Y%m%d`
mirrors_path="/home/www/mirrors"
log_file="/home/www/mirrors/sync_log/$date.log"
mirror_url="mirrors.ustc.edu.cn"

# create log directory
mkdir -p `dirname $log_file`

# sync function
function sync() {
    remote_path=$1
    local_path=$2

    # create local directory
    mkdir -p $local_path

    # sync
    /usr/bin/rsync -avrt --delete \
        --exclude=debug/ \
        --exclude=isos/ \
        --exclude=SRPMS/ \
        --exlucde=drpms/ \
        --exclude=i386/ \
        --exclude=ppc64/ \
        --exclude=.~tmp~/ \
        $remote_path $local_path >>$log_file
}

echo "---- $Date `date` Begin ----" >>$log_file

# sync only we need.
sync "rsync://$mirror_url/centos/6" "$mirrors_path/centos"
sync "rsync://$mirror_url/centos/6.4" "$mirrors_path/centos"
sync "rsync://$mirror_url/fedora/epel/6" "$mirrors_path/epel"

echo "---- $Date `date` End ----" >>$log_file
{% endhighlight %}


然后需要架个http服务器，首先安装nginx：

{% highlight bash %}
$ yum install nginx
$ chkconfig nginx on
{% endhighlight %}

临时给自己分个域名，在`/etc/hosts`中加入：
{% highlight bash %}
192.168.1.1   mirrors.tiwb.net
{% endhighlight %}

下面配置nginx服务器，增加一个配置文件`/etc/nginx/conf.d/mirrors.conf`，内容如下：
{% highlight bash %}
server {
  server_name mirrors.tiwb.net;

  root /home/www/mirrors;

  location / {
    autoindex on;
  }

  # 目前我们的源还没有同步好，所以先代理到其他的网站上
  # 等本地数据同步好了就可以删除下面的配置了
  location /centos {
    proxy_pass http://mirrors.163.com/centos;
  }
  location /epel {
    proxy_pass http://mirrors.ustc.edu.cn/fedora/epel;
  }
}
{% endhighlight %}


最后启动nginx服务器：
{% highlight bash %}
$ service nginx start
{% endhighlight %}


## 第一次使用epel时的安装：
epel如果只是配置了源，会由于缺少证书而报错， 所以需要先安装epel：
{% highlight bash %}
$ rpm -Uvh http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm
{% endhighlight %}

注意随着epel版本的升级， 这个地址中的文件名也是会变的。


## 本地源配置：
有了本地源以后，就可以修改yum的源配置来使用本地源了。
{% highlight bash %}
# file: /etc/yum.repos.d/centos.repo
[base]
name=CentOS-$releasever - Base
baseurl=http://mirrors.tiwb.net/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=http://mirrors.tiwb.net/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=http://mirrors.tiwb.net/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=http://mirrors.tiwb.net/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
baseurl=http://mirrors.tiwb.net/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
{% endhighlight %}

还有epel的源配置：
{% highlight bash %}
# file: /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux 6 - $basearch
baseurl=http://mirrors.tiwb.net/epel/6/$basearch
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 6 - $basearch - Debug
baseurl=http://mirrors.tiwb.net/epel/6/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 6 - $basearch - Source
baseurl=http://mirrors.tiwb.net/epel/6/SRPMS
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
gpgcheck=1
{% endhighlight %}
