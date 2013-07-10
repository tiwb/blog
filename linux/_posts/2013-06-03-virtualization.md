---
vim: wrap expandtab ft=markdown
layout: blog
title: 虚拟化软件的安装和使用
---

虚拟化是个好东西，现在服务器上基本上都是直接上虚拟化的，除了对性能要求特别高的情况。前两天装虚拟化，只记得最后为了省事用的是virt-manager管理，却忘了这么安装了，又是查了N久资料才搞定…… 所以果断记下来。

要使用kvm，首先需要安装软件， 在centos6.4上，需要安装qemu-kvm这个软件包，当然顺便也要装个qemu-img来做硬盘镜像。
{% highlight bash %}
$ yum install qemu-kvm qemu-img
{% endhighlight %}

光装了软件包还不行， kvm还需要加载一个kernel模块，记得有这么一步，可是忘记是什么名字了…… 看了官方手册才发现原来是kvm-intel，如果时amd的cpu就是kvm-amd，还有就是别忘了在bios中打开虚拟化的支持。
{% highlight bash %}
$ modprobe kvm-intel
{% endhighlight %}

modprobe其实是我不太理解的一个命令，也不用改什么配置文件， 就这么运行一下就行了……难怪会被忘记。如果一切正常，现在就可以用qemu-kvm命令启动kvm虚拟机了，你可以用lsmod检查下以确保kvm模块被正确加载。

记得最早的时候， 我就是直接用qemu-kvm命令来运行虚拟机的，写好启动脚本，然后在tmux中运行后deteach…… 直到后来有5台以上虚拟机要管理时，才觉得有点太麻烦了，于是找到了libvirt。

## 安装虚拟化软件
在需要跑虚拟机的机器上装上libvirt这个软件包，这个包听起来像是个库，其实它提供了libvirtd这个服务，装上以后请确保这个服务会随着系统自动启动。
{% highlight bash %}
$ yum install libvirt
{% endhighlight %}


## 使用virt-manager管理虚拟机
我们的第一台虚拟机要用virt-manager来创建。
<div class="note info">
<p>virt-manager 是一个桌面工具，所以尽量不然安装在服务器上。你可以先在windows上安装一个centos的虚拟机， 然后再安装这个软件。</p>
</div>

{% highlight bash %}
$ yum install virt-manager
{% endhighlight %}

安装了virt-manager以后在桌面上启动，然后用ssh连接到服务器上，就可以远程管理服务器上的虚拟机了，用起来跟一般的虚拟化软件差不多。

## 使用virsh

很多情况下不方便使用使用图形化的工具管理，所以还要掌握使用命令行管理虚拟机的基本方法，首先先安装virsh：

{% highlight bash %}
$ yum install libvirt-client
{% endhighlight %}

### 打开console
有的时候需要解决一些虚拟机的网络问题， 当然可以使用vnc连接，当无法使用vnc连接的时候，还可以使用串口。

在虚拟机里的XML配置里增加下面的设备：`$ virsh edit centos6`
{% highlight xml %}
<serial type='pty'>
  <target port='0' />
</serial>
<console type='pty'>
  <target type='serial' port='0' />
</console>
{% endhighlight %}

然后还需要系统支持console连接，最简单的办法就是在grub.conf的启动参数里增加：
{% highlight bash %}
console=ttyS0,115200
{% endhighlight %}


### 使用差分镜像
现在我们该以之前做好的系统为模板创建多个虚拟机了。这次我准备使用差分镜像，在使用了差分以后，基础镜像就不能改动了，所以我们的模板也要使用差分。先把模板的镜像文件移动到一个安全的地方：
{% highlight bash %}
$ mv centos6.qcow2 /home/iso/centos6_base.qcow2
{% endhighlight %}

然后重新创建原来的镜像文件：
{% highlight bash %}
$ qemu-img create -f qcow2 \
    -o backing_file=/home/iso/centos6_base.qcow2 \
    centos6.qcow2
{% endhighlight %}

这样就好了，目前仅当作使用差分可以节省空间吧，以后再研究看看有什么高级功能。


### 克隆虚拟机
我不准备从头开始写一个配置文件，因为太长了，之前已经用virt-manager创建了一个虚拟机模板，我们就用它来克隆吧：
{% highlight bash %}
$ virsh dumpxml centos6 >centos6.xml
{% endhighlight %}

然后编辑刚才dump的xml文件，修改下虚拟机的名字, uuid删掉，改下硬盘镜像的路径，再改下网卡的mac地址，不要与之前冲突了。

接着再为这个虚拟机创建一个硬盘镜像：
{% highlight bash %}
$ qemu-img create -f qcow2 \
    -o backing_file=/home/iso/centos6_base.qcow2 \
    gateway.qcow2
{% endhighlight %}

然后导入虚拟机：
{% highlight bash %}
$ virsh define centos6.xml
{% endhighlight %}

比如我们刚才把虚拟机的名字改成了gateway， 现在就可以启动虚拟机了：
{% highlight bash %}
$ virsh start gateway
{% endhighlight %}

下面就可以用console连接虚拟机了：
{% highlight bash %}
$ virsh console gateway
{% endhighlight %}
