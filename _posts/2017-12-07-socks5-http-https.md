---
layout:     post
title:      私人专线PROXY的搭建
subtitle:   使用google cloud设置socks5,http/https
date:       2017-12-07
author:     Oscar Zhang
header-style: text
catalog:      true
tags:
    - Proxy
    - Socks5
---
科学上网对于科研工作者格外重要。我在网上买过科学服务，最基础的每月30块，还不稳定。借着google cloud的活动，可以免费使用1年的服务，就算过了1年活动没了，相较于网上的价格也只有几分之一。自己搭建的服务可以自己掌管，非常可靠高，是为久长之际。

科学上网的关键有两点：

1. 找到一个公网ip的主机。这部分依靠google cloud上租借vm实例实现。
2. 在该主机上设置代理(转发信息)。这里采用socks5类型，通过ShadowSocks实现。
3. 有些特殊情况可能只允许http/https类型的代理，这里也做了一个简要介绍。

# 一、创建google cloud账号 
该填啥填啥，需要绑定一张信用卡     
![img][1]  

创建VM实例（做代理最小型机器就可以）    
![img][2]   

防火墙设置      
![img][3]     
关键项如下      
![img][4]    

# 二、SOCKS5       
通过VM实例SSH进入服务器系统
![img][5]         
![img][6]      

#### 手动设置SS服务器    

    sudo apt-get update  
    sudo apt-get install python-pip     
    sudo pip install shadowsocks     
    sudo ssserver -p id -k pwd -m aes-256-cfb -d start  #id、pwd自行设置     

在`/etc/rc.local`中正确设置开机启动：   
1.`/etc/rc.local`中不需要`sudo`(默认root); 2.需要写全路径，可通过`which ssserver`查看

    /usr/local/bin/ssserver -p 8388 -k mypassword -m aes-256-cfb -d start
    /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start  

ps. 输出诊断信息到log，写在执行语句之前。切不要盲目修改`/etc/rc.local`：

    exec 2> /tmp/rc.local.log      # send stderr from rc.local to a log file
    exec 1>&2                      # send stdout to the same log file
    set -x                         # tell sh to display commands before execution

在`/etc/crontab`中设置系统定时重启，增加`ssserver`稳定性：

    30 22 * * 1,4 root sleep 70 || /sbin/reboot          # 照猫画虎，注意时区换算(uptime, last reboot, 服务器22:3即北京6:30, 每周1,4) 

#### 通过`.json`启动

编写`vim ~/shadowsocks-server.json`文件        

    -------- single usr --------
    {
        "server":"0.0.0.0",
        "server_port":8388,
        "local_address": "127.0.0.1",
        "local_port":1080,
        "password":"mypassword",
        "timeout":300,
        "method":"aes-256-cfb"
    }
    -------- multiple usr --------
    {  
         "server":"0.0.0.0"，  
         "local_address": "127.0.0.1",  
         "local_port":1080,  
          "port_password": {  
             "8388": "password",  
             "8387": "password",  
             "8386": "password",  
             "8385": "password"  
         },  
         "timeout":300,  
         "method":"aes-256-cfb",  
         "fast_open": false  
        }  

启动

    ssserver -c ~/shadowsocks-server.json -d start
        
#### 设置服务

编辑
        
    vim /etc/supervisord.conf
    
添加内容

    [program:shadowsocks]
    command=ssserver -c /etc/shadowsocks.json
    autostart=true
    autorestart=true
    user=root
    log_stderr=true
    logfile=/var/log/shadowsocks.log

在`/etc/rc.local`中添加

    service supervisord start
    
#### 批量操作

下载server模版，修改配置

    wget -P ~ https://raw.githubusercontent.com/zbhoscar/storage/master/shadowsocks-server-multiple.json
    
下载服务模版，修改配置

    wget -P /etc https://raw.githubusercontent.com/zbhoscar/storage/master/supervisord.conf

设置开机启动服务，在`/etc/rc.local`中添加

    service supervisord start

#### 使用[大神脚本](https://teddysun.com/486.html)设置SS服务器

    wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
    chmod +x shadowsocks-all.sh
    sudo ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log

直接进入设置界面，记好自己的设置

#### windows配置客户端
![img][7]      
![img][8]       
该怎么连就怎么连   
Enjoy！   
#### [Linux/Ubuntu配置客户端](https://zbhoscar.github.io/2016/12/08/ubuntu-base/)  



# 三、HTTP/HTTPS

配置相关软件并编辑

    sudo apt-get install squid3     
    sudo vim /etc/squid3/squid.conf
            
对文件做相应修改

                    --- 文件内容修改部分 ---
    # http_access deny all          # 位于1060行, 注视掉
    http_port 10086                 # 再翻几页，个性化端口id
                    --- 文件末尾添加部分 ---
    acl allcomputers src 0.0.0.0/0.0.0.0
    auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/passwords
    auth_param basic realm proxy
    acl authenticated proxy_auth REQUIRED
    http_access allow authenticated allcomputers
    
设置用户名密码认证
 
    sudo apt-get install apache2-utils
    sudo htpasswd -c -d /etc/squid3/passwords <自定义用户名>
    > 设置密码，连输两次 <
    sudo chmod o+r /etc/squid3/passwords
    sudo service squid3 restart                 # or start, stop
    
验证代理是否起作用

    tail -f /var/log/squid3/access.log
    > 另台linux机器打开shell <
    export http_proxy="http://用户名:密码@代理IP:代理端口"
    curl -l "http://www.baidu.com"
    
如果代理配置正确，会输出html，同时代理服务器上的`access.log`会记录这次请求

[1]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/1.png
[2]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/2.png
[3]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/3.png
[4]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/4.png
[5]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/5.png
[6]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/6.png
[7]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/7.png
[8]: https://raw.githubusercontent.com/zbhoscar/zbhoscar.github.io/master/img/in-post/post-proxy/8.png















