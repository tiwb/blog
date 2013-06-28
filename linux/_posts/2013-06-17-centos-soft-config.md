---
vim: wrap expandtab
layout: blog
title: CentOS常用软件的安装和配置
---

装上CentOS以后，马上开始安装一些常用的软件，这些软件很多都是在epel这个源里的，如果你还不知道如何加入epel这个源，请参考[yum源镜像]这篇文章。

一般情况下，安装一个软件可以直接使用`yum install 软件名称`即可，但是也有一些时候，软件名称和软件所在包的名称是不一样的，这个时候要想找到安装软件所需要安装的包其实很简单， 只需要用`yum provides */执行名称`（把执行名称换成你要查找的软件名称）就可以出所有提供了某个程序的包。

## 终端管理软件 tmux
首先就是tmux，这个软件和screen是一类的，可以在linux终端上同时开多个虚拟的终端，可以自由的切换，还可以拷贝，查看历史等等…… 总之绝对是必备软件。

我的tmux配置文件如下：
{% highlight bash %}
# file: ~/.tmux.conf

# set correct term
set -g default-terminal screen-256color

# copy mode
set-option -gw mode-keys vi
set-option -gw mode-mouse on
bind Escape copy-mode 
bind-key -t vi-copy 'v' begin-selection
bind-key -t vi-copy 'y' copy-selection

# status bar
set-option -g status-keys vi
set-option -g status-right "#22T"

set-option -g mouse-select-pane on

# make hotkeys like C-b,C-n work
bind-key C-l last-window
bind-key C-n next-window
{% endhighlight %}

## 文本编辑软件 vim
接下来就是vim了，目前centos带的vim版本还不够高，最好能够自己编译出最新版。vim有很多的插件和配置，所以我以后会单独写篇文章来说明。


## 网络辅助相关软件
要验证dns工作是否正常，我喜欢够用dig这个软件。用iftop能够查看网络使用情况，用tcpdump能够查出更底层的封包发送。
{% highlight bash %}
$ yum install bind-utils iftop tcpdump
{% endhighlight %}

## 其他辅助工具
bash-completion可以在让bash的自动完成能更好的工作，可以省去很多麻烦。
{% highlight bash %}
$ yum install bash-completion
{% endhighlight %}

ncdu可以更直观的查看硬盘的使用情况，也是我常用的工具之一。
{% highlight bash %}
$ yum install ncdu
{% endhighlight %}

## 开发与调试工具
用Linux，免不了要自己编译，所以gcc是必须要装的了，一般的软件包都把自动配置脚本写好了，所以装个编译器就可以了。编译器可以直接装c++，会帮你把依赖的gcc也装上。
{% highlight bash %}
$ yum install gcc-c++ make
{% endhighlight %}

当然如果要开发，最好还是装下手册：
{% highlight bash %}
$ yum install man-pages
{% endhighlight %}


调试我喜欢用cgdb，因为同时有gdb的强大，同事看代码又比较方便。
{% highlight bash %}
$ yum install cgdb
{% endhighlight %}

cgdb 也需要配置一下来符合我的使用习惯：
{% highlight bash %}
# file ~/.cgdb/cgdbrc
set tabstop=4
set autosourcereload=on
set as=short
{% endhighlight %}

暂时这样应该就能应付一下了，如果要开发，还需要装很多库之类的，还需要装一些其他的开发工具，根据具体的个人爱好和开发用途。




