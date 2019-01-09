---
layout:     post
title:      Ubuntu科研用机指南
subtitle:   常用设置和软件安装
date:       2016-12-08
author:     Oscar Zhang
header-style: text
catalog:      true
tags:
    - Linux
    - Init
---
常用Ubuntu做科研，这是常用的一些配置指南。

### NVIDIA驱动安装

#### 先禁用nouvea

    sudo gedit /etc/modprobe.d/blacklist.conf

末尾添加

    blacklist nouveau
    
更新内核

    sudo update-initramfs -u
    
重启。确认生效

    sudo reboot
    lsmod | grep nouveau

没有输出即可

#### 安装驱动

    sudo add-apt-repository ppa:graphics-drivers/ppa
    sudo apt update

识别显卡模型和推荐的驱动

    ubuntu-drivers devices
    
安装驱动
    
    sudo apt install nvidia-driver-410 nvidia-settings nvidia-prime

或者

    sudo ubuntu-drivers autoinstall
    
重启生效

    sudo reboot

### 科学上网

#### 安装ShadowSocks

    sudo apt update
    sudo apt install python-pip
    sudo pip install shadowsocks
    
或者

    sudo apt update
    sudo apt install python-pip
    sudo apt install git
    sudo pip install -U git+https://github.com/shadowsocks/shadowsocks.git@master

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
    
下载

    wget -P ~ https://raw.githubusercontent.com/zbhoscar/storage/master/shadowsocks.json

启动ShadowSocks

    /usr/bin/sslocal -c /DIR/shadowsocks.json

`DIR`是`.json`的路径，`sslocal`可以通过`which sslocal`找到

#### ShadowSocks服务

先写一个`.service`文件

    sudo pluma /lib/systemd/system/shadowsocks.service

填写如下内容：

    [Unit]
    Description=Shadowsocks Client Service
    After=network.target
    
    [Service]
    Type=simple
    User=root
    ExecStart=/usr/bin/sslocal -c /path/shadowsocks.json
    
    [Install]
    WantedBy=multi-user.target    

懒得写命令行:

    wget -P /lib/systemd/system https://raw.githubusercontent.com/zbhoscar/storage/master/shadowsocks.service

启动服务
    
    systemctl enable shadowsocks.service 

最终效果是给本机 ip `127.0.0.1`的`1080`端口科学加成，可以配置浏览器和软件用来科学。    
比如系统自带firefox浏览器中，设置手动代理，选择socks5 DNS，加入科学端口就可以了。   
[更全面的讲解](https://blog.huihut.com/2017/08/25/LinuxInstallConfigShadowsocksClient/)

#### Proxychains

通过`ShadowSocks`的建立的本地端口给指令科学加成

    sudo apt install proxychains

配置`/etc/proxychains.conf`               

    socks5 127.0.0.1 1080

示例: 通过`proxychains`启动`Dropbox`

    proxychains dropbox start -i   #启动后可能提示缺少python-gpgme, install即可

[更全面的讲解](http://einverne.github.io/post/2017/02/terminal-sock5-proxy.html)

### 服务器远程控制

#### 命令行控制
需要在受控机器与本机在同一网络下

    ssh usrname@server_ip
    
[不在同一网络下的方法](https://zbhoscar.github.io/2018/08/19/frp/)
    
#### 图形界面控制

受控机器使用VNC做图形界面控制，需要与本机在同一网络中
                                                
    sudo apt-get install vnc4server    # 最新版本看官网
    vim /home/用户名/.vnc/xstartup
    ⬇⬇⬇ 内容修改 ⬇⬇⬇
    # &x-window-manager &              # 注释掉     
    exec /usr/bin/mate-session &       # 针对mate界面
    ⬆⬆⬆ 内容修改 ⬆⬆⬆
    service vncserver restart

使用方法

    ssh usrname@server_ip               # 本机命令行登陆受控机器
    vncserver -geometry 1024x768 :id    # 受控机器开启界面，注意是x不是*
    vncviewer server_ip:id              # 本机登陆vnc登陆受控机器
    vncserver -kill :id                 # 受控机器关闭界面
    ps -ef | grep 'vnc' | grep id       # 受控机器检测vnc端口id被占用

### 硬盘挂载

#### Samba远程文件夹

[配置Samba远程连接](http://jingyan.baidu.com/article/00a07f38b9194082d028dc08.html)  
    
Windows系统
    
    任意界面 `win + R`
    > 输入 \\受控机器的ip\文件夹名 <
    > 输入 账号密码 <

Ubuntu系统
    
    文件夹界面 `Ctrl + l`
    > 输入 smb://受控机器的ip/文件夹名/
    > 输入 账号密码 <

#### [`/home`挂载](https://www.jianshu.com/p/5c12289dbcd1)

#### [NFS硬盘挂载](https://www.cnblogs.com/heruiguo/p/7998260.html)

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

### 终端相关
#### readline插件

解决使用Shell时⬆⬇⬅➡乱码的问题，**在Ubuntu16中貌似不需要**

    sudo apt-get install libreadline-dev    ， 不然安装python3的方向键会乱码

#### terminator终端

很好用的终端，戒不掉

    sudo apt-get install terminator

#### 右键打开终端选项

很实用，需要重启，**在Ubuntu16中貌似不需要**

    sudo apt-get install nautilus-open-terminal

#### [终端配色](https://ubuntugenius.wordpress.com/2011/07/11/how-to-change-the-command-line-prompt-colour-in-the-ubuntulinux-terminal/)

    sudo gedit ~/.bashrc
   替换相似`PS1`行，并设置`force_color_prompt=yes`     
   这是一种配色方案：     
   
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;36m\]\u\[\033[01;36m\]@\[\033[01;36m\]\h\[\033[01;36m\]:\[\033[01;33m\]\w\[\033[01;36m\]\$ \[\e[m\] '

### 数据格式相关
#### 超级解压缩

    apt install pigz
    tar --use-compress-program=pigz -xvpf ucfTrainTestlist.tar.gz -C ./test

#### 安装rar
解压RAR出`Parsing Filters is unsupported`

    sudo apt-get install unrar
    
#### U盘exFat格式支持
可能需要重启，反正Ubuntu16不需要

    sudo apt-get install exfat-utils 

### 其他
####  GCC/G++ multi

    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    gcc -v
    ls /usr/bin/gcc*
    sudo apt-get install gcc-4.5 gcc-4.5-multilib g++-4.5 g++-4.5-multilib
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 50
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.5 40
    sudo update-alternatives --config gcc
    sudo update-alternatives --remove gcc /usr/bin/gcc-4.5

#### FFMPEG
解决编译OpenCV出错的问题

    sudo apt-get install librubberband-dev libx264-dev libvpx-dev
    ./configure --enable-gpl --enable-version3 --enable-nonfree --enable-libx264 --enable-librubberband --enable-libvpx

#### Firefox使用本地账户

[在附加组件(addons)中添加附加组件管理器(addons manager)](http://mozilla.com.cn/thread-343905-1-1.html)

#### 设置工作区
在`software center`中设置`unity tweak tool`

### Boot引导修复Grub

#### 先决环境

U盘启动，试用ubuntu，联网。

打开终端，添加`boot-repair`所在的源：

    sudo add-apt-repository ppa:yannubuntu/boot-repair
    sudo apt-get update

安装`boot-repair`:

    sudo apt-get install -y boot-repair 
    
运行`boot-repair`:

    sudo boot-repair

一般选择`recommanded repair`即可，启动项重启生效

启动项查看

    sudo efibootmgr
    
启动项删除

    sudo efibootmgr -b 000A -B
