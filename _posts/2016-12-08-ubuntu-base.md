---
layout:     post
title:      Ubuntu科研用机指南
subtitle:   常用设置和软件安装
date:       2016-12-08
author:     Oscar Zhang
header-style: text
catalog:      true
tags:
    - IT
    - Ubuntu
    - Linux
---
常用Ubuntu做科研，这是常用的一些配置指南。

### ShadowSocks配置

#### 安装ShadowSocks

    sudo apt-get install python-pip
    sudo pip install shadowsocks

设置`.json`文件

    {
     "server":"my_server_ip",
     "local_address": "127.0.0.1",
     "local_port":1080,
     "server_port":my_server_port,
     "password":"my_password",
     "timeout":300,
     "method":"aes-256-cfb"
    }

启动ShadowSocks

    /usr/bin/sslocal -c /DIR/shadowsocks.json

`DIR`是`.json`的路径，`sslocal`可以通过`which sslocal`找到

#### 添加开机服务

先写一个`.service`文件

    sudo pluma /etc/systemd/system/shadowsocks.service

填写如下内容：

    [Unit]
    Description=Shadowsocks Client Service
    After=network.target
    
    [Service]
    Type=simple
    User=root
    ExecStart=/usr/bin/sslocal -c /DIR/shadowsocks.json
    
    [Install]
    WantedBy=multi-user.target                

启动服务
    
    systemctl enable /etc/systemd/system/shadowsocks.service 

`.service`放在`/lib/systemd/system`也可以，直接`service`调用

最终效果是给本机 ip `127.0.0.1`的`1080`端口科学加成，可以配置浏览器和软件用来科学。    
比如系统自带firefox浏览器中，设置手动代理，选择socks5 DNS，加入科学端口就可以了。   
[更全面的讲解](https://blog.huihut.com/2017/08/25/LinuxInstallConfigShadowsocksClient/)

### Proxychains

通过`ShadowSocks`的建立的本地端口给指令科学加成

    sudo apt install proxychains

配置`/etc/proxychains.conf`               

    socks5 127.0.0.1 1080

示例: 通过`proxychains`启动`Dropbox`

    proxychains dropbox start -i   #启动后可能提示缺少python-gpgme, install即可

[更全面的讲解](http://einverne.github.io/post/2017/02/terminal-sock5-proxy.html)

### readline插件

解决使用Shell时⬆⬇⬅➡乱码的问题，**在Ubuntu16中貌似不需要**

    sudo apt-get install libreadline-dev    ， 不然安装python3的方向键会乱码

### terminator终端

很好用的终端，戒不掉

    sudo apt-get install terminator

### 右键打开终端选项

很实用，需要重启，**在Ubuntu16中貌似不需要**

    sudo apt-get install nautilus-open-terminal

[终端配色](https://ubuntugenius.wordpress.com/2011/07/11/how-to-change-the-command-line-prompt-colour-in-the-ubuntulinux-terminal/)

    sudo gedit ~/.bashrc
   替换相似`PS1`行，并设置`force_color_prompt=yes`     
   这是一种配色方案     
   
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;36m\]\u\[\033[01;36m\]@\[\033[01;36m\]\h\[\033[01;36m\]:\[\033[01;33m\]\w\[\033[01;36m\]\$ \[\e[m\] '

### 超级解压缩

    apt install pigz
    tar --use-compress-program=pigz -xvpf ucfTrainTestlist.tar.gz -C ./test

### Ubuntu环境文件

- `/etc/profile`为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行，并从/etc/profile.d目录的配置文件中搜集shell的设置。   
- `/etc/environment`在登录时操作系统使用的第二个文件,系统在读取你自己的profile前,设置环境文件的环境变量。   
- `/etc/bash.bashrc`为每一个运行bash shell的用户执行此文件，当bash shell被打开时,该文件被读取。  
- `~/.profile`:每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次，默认情况下,他设置一些环境变量,执行用户的.bashrc文件，`每个用户应当在这里添加自己的环境配置`。  
- `~/.bashrc`包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该文件被读取，`终端配色在这里修改`。    
- `~/.bash_logout`当每次退出系统(退出bash shell)时，执行该文件。
相互关系    
- `/etc/profile`中设定的变量(全局)的可以作用于任何用户,而`~/.bashrc`等中设定的变量(局部)只能继承`/etc/profile`中的变量,他们是"父子"关系。     

每次登录之后加载profile和bashrc的流程       
1) `/etc/profil` --> `/etc/profile.d/*.sh`      
2) `$HOME/.profile` --> `$HOME/.bashrc`    

生效不用重启  
`source profile`, `source bash`

环境变量·自定义

    export PATH=/usr/local/cuda-8.0/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
    export LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LIBRARY_PATH
    export PYTHONPATH=/home/zbh/Desktop/caffe-ssd/python:$PYTHONPATH

### GCC/G++ multi

    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    gcc -v
    ls /usr/bin/gcc*
    sudo apt-get install gcc-4.5 gcc-4.5-multilib g++-4.5 g++-4.5-multilib
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 50
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.5 40
    sudo update-alternatives --config gcc
    sudo update-alternatives --remove gcc /usr/bin/gcc-4.5

### FFMPEG
解决编译OpenCV出错的问题

    sudo apt-get install librubberband-dev libx264-dev libvpx-dev
    ./configure --enable-gpl --enable-version3 --enable-nonfree --enable-libx264 --enable-librubberband --enable-libvpx

### U盘设置exFat格式Enable
可能需要重启，反正Ubuntu16不需要

    sudo apt-get install exfat-utils 

### Firefox使用本地账户

[在附加组件(addons)中添加附加组件管理器(addons manager)](http://mozilla.com.cn/thread-343905-1-1.html)

### 安装rar
解压RAR出`Parsing Filters is unsupported`

    sudo apt-get install unrar

### 设置工作区
在`software center`中设置`unity tweak tool`

