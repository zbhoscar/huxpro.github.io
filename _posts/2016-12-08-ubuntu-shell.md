---
layout:     post
title:      Ubuntu Shell常用指令
subtitle:   命令行下的操作
date:       2016-12-08
author:     Oscar Zhang
header-style: text
catalog:      true
tags:
    - Linux
    - Shell
---
Ubuntu Shell常用到的指令
## 状态查看

    cat /proc/cpuinfo |grep MHz|uniq        # cpu信息，grep表示只抓取字段
    nmcli dev list iface eth1 | grep DNS    # Real DNS
    watch -n 10 nvidia-smi                  # 每10s屏显nvidia-smi结果
    sudo fuser -v /dev/nvidia*              # 检查nvidia-smi看不到的显存占用，然后kill pid_num
    ps -o stime,etime pid_num               # 查看pid_num的起始、持续时间
    openssl version -a
    top | grep usr                          # cpu使用，grep表示只抓取字段

## 文件操作
    
    sudo cp /home/wally/test.iso /local/arm # -R 递归；-f：no ask ；-i：ask if cover
    sudo rm -rf /local/arm                  # 递归删除文件夹，慎用
    sudo mkdir /media/matlab
    ln -s /data3_alpha/datasets ~/Desktop/  # 建立软连接
    chmod u+x var.sh                        # 给var.sh添加可执行权限        
    cd ./../opt                             # 注意.和..的用法 
    mount -o loop /dir/a.iso /media/matlab  # 挂载iso到目录
    unmount /media/matlab
    rename 's/$/.txt/' *                    # 当前文件夹下给所有文件添加后缀名扩展名
    rename 's/\.JPG/.jpg/' *.JPG            # 删除：把.jpg替代为空
    sudo nautilus                           # 打开带sudo的资源管理器
    chmod -R 777 /opt/opencvx64             # -R 递归
    chown -R zbh.zbh /home/zbh/IPT          # 路径必须为真实路径，软连接没反应
    chgrp -R zbh /data2_alpha/zbh/          # 修改组
    chown -R zbh /data2_alpha/zbh/          # 修改用户
    wget -P ~/Downloads ftp://example_link  # 修改用户

## 用户与组
    
0、查看用户UID、GID                    
        
    cat /etc/passwd                 # 所有用户
    id myftpfinger                  # 查看制定用户
    
1、创建用户组 
    
    groupadd -g 600 groupftp        # 创建一个组groupftp，其GID为600
    
2、为组添加用户

    gpasswd -a user1 groupftp       # 把user1加入groupftp组
    gpasswd -d user1 groupftp       # 把user1退出groupftp组
    
3、修改组
    
    groupmod -n user groupftp       # 修改组名user为groupftp

4、删除组

    groupdel groupftp               # 删除组groupftp

5、创建用户

    useradd -u 700 myftp -g 600     # 创建myftp用户，指定uid为700，所属组uid600

6、为新建用户设置密码

    passwd myftp

7、修改用户账户

    usermod -l myftp2 myftp         # 将用户myftp的登录名改为myftp2
    usermod -g groupftp myftp       # 将用户myftp加入到groupftp组中    # 等同于 /etc/sudoers 添加myftp   ALL=(ALL)     ALL
    usermod -d /ftp1/ftp2 ftp       # 将用户ftp目录改为/ftp1/ftp2
    
8、删除用户账户

    userdel myftp                   # 删除用户myftp
    userdel -r myftp                # 删除用户myftp，同时删除他的工作目录    

**当多个linux主机共享存储阵列时**
注意`/etc/group`和`/etc/passwd`中的`groupmod`和`usermod`。    
因为不同主机系统显示的文件`属组用户名`是相对的，唯有磁盘中文件的`GID`是绝对的。    
eg. 在 `/etc/group` 文件中：

    alpha主机：   zbh:x:1027 ; zpf:x:1001
    beta主机：    zbh:x:1001 ; 无1027
    
在两个主机在自身系统下给文件刷zbh.zbh时：alpha刷完，beta显示为1027；beta刷完，alpha显示zpf，造成同一用户在两个服务器共享的存储设备中权限不同的情况。  
必须在两套主机系统中给zbh.zbh设置相同的GID，eg.=1080，才能使得访问可以同步更新。更新完GID之后，原有的文件属性并不会改变。  
在`/home/zbh/`下有一些用户执行文件，在新GID下无法读写，需要重刷zbh.zbh(eg. vncserver)。

## [: XXXX: unexpected operator
运行.sh文件报错 `[: XXXX: unexpected operator...`     
原因：Ubuntu默认.sh是连接到dash或者bash，默认与当前.sh不匹配。     
当前.sh默认连接查看：

    ps | grep $$ | awk '{print $4}'
    sudo dpkg-reconfigure dash
    选择 > NO


## .sh demo
#### 按行随机抽取

    shuf -n5 main.txt               #shuf .txt 随机按行打乱txt并输出
    sort -R main.txt | head -5
    awk -vN=5 -vC="`wc -l file`" 'BEGIN{srand();while(n<N){i=int(rand()*C+1);if(!(i in a)){a[i]++;n++}}}NR in a' file
    while read line;do echo "$line $RANDOM";done < main.txt | sort -k2,2n| awk 'NR<=5{print $1}'

#### 求文件行数
    
    line=`sed -n '$=' $txt`
    
#### 得到随机数

    n=$(python -c 'import random;print [random.randint(1,6000) for _ in range(16)]') # 用python得到返回结果，不能在循环中引用因为python不识别命令行中的变量
    
#### 随机排序？

         awk 'BEGIN{ 100000*srand();}{ printf "%s %s\n", rand(), $0}' train_160309-train.txt |sort -k1n | awk '{gsub($1FS,""); print $0}'                   # 终端打印
    sudo awk 'BEGIN{ 100000*srand();}{ printf "%s %s\n", rand(), $0}' train_160309-train.txt |sort -k1n | awk '{gsub($1FS,""); print $0}' | tee train.txt   # 打印并保存
    sudo awk 'BEGIN{ 100000*srand();}{ printf "%s %s\n", rand(), $0}' train_160309-train.txt |sort -k1n | awk '{gsub($1FS,""); print $0}' > train.txt       # 不打印仅保存




