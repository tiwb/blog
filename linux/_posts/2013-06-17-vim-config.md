---
vim: wrap expandtab
layout: blog
title: vim的编译与配置
---
CentOS 6.4上的vim的版本还是7.2， 目前vim已经升级到7.3了， 我最喜欢7.3里的持久化undo这个功能，可以说已经离不开了…… 所以需要自己编译vim

安装依赖和编译环境：
{% highlight bash %}
$ yum install gcc make ncurses-devel python-devel
{% endhighlight %}

下载并编译安装vim，如果你不喜欢默认装到/usr/local下或者您没有root权限，可以在配置的地方加上`--prefix=你要安装的目录`
{% highlight bash %}
$ mkdir /tmp/vim
$ cd /tmp/vim
$ wget http://www.vim.org/vim-7.3.tar.bz2
$ tar -xvf vim-7.3.tar.bz2
$ cd vim73
$ ./configure --disable-nls \
              --enable-multibyte \
              --enable-pythoninterp=yes \
              --with-features=huge
$ make
$ make install
$ cd
$ rm -rf /tmp/vim
{% endhighlight %}


我的vim配置文件：
{% highlight vim %}
" file: ~/.vimrc
syntax on
set nocompatible
set hlsearch
set incsearch
set nobk
set noswapfile
set mouse=a
set tabstop=2
set softtabstop=2
set shiftwidth=2
set expandtab
set autowriteall
set nowrap
set modeline
set textwidth=0
set autoindent

filetype plugin indent on

set undodir=~/.vimundodir
set undofile

" Gui options
set guioptions-=T
set guioptions-=r
set guifont=Inconsolata:h16

" goto file
function _MyGotoFile()
  try
    cs find f <cfile>
  catch
    find <cfile>
  endtry
endfunction

nmap <silent> gf :call _MyGotoFile()<CR>

" quick fix map
nmap <C-n> :cn<CR>
nmap <C-p> :cp<CR>

{% endhighlight %}
