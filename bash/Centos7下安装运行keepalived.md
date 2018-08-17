### Centos7下安装运行keepalived

```
原文地址：https://www.cnblogs.com/xxoome/p/8621677.html

keepalived下载地址：http://www.keepalived.org/download.html

 

使用wget命令下载，下载位置/usr/local/

wget http://www.keepalived.org/software/keepalived-1.4.2.tar.gz
解压：

tar zxvf keepalived-1.4.2.tar.gz
安装依赖插件：

yum install -y gcc openssl-devel popt-devel
编译安装：

cd keepalived-1.4.2

#指定安装目录
./configure --prefix=/usr/local/keepalived

make && make install
运行前配置

复制代码
#
cp /usr/local/keepalived-1.4.2/keepalived/etc/init.d/keepalived /etc/init.d/

#
mkdir /etc/keepalived

#
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/

#
cp /usr/local/keepalived-1.4.2/keepalived/etc/sysconfig/keepalived /etc/sysconfig/

#
cp /usr/local/keepalived/sbin/keepalived /usr/sbin/
复制代码
修改配置文件：

vim /etc/keepalived/keepalived.conf
具体配置如下：

master服务器配置：

复制代码
! Configuration File for keepalived

global_defs {
   router_id xxoome_master #起一个没重复的名字即可
}

# 检测nginx是否运行
vrrp_script chk_nginx {
        script "/etc/keepalived/nginx_check.sh"
        interval 2
        weight -20
}

vrrp_instance VI_1 {
    state MASTER

    # 网卡名字，文章下方会给出如何获取网卡名字的方法
    interface enp0s3
    virtual_router_id 106
    
    # 本机ip
    mcast_src_ip 192.168.0.132
    priority 100 #权重，master要大于slave
    nopreempt
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    # 与上方nginx运行状况检测呼应
    track_script {
        chk_nginx
    }

    virtual_ipaddress {
        #虚拟ip（VIP，一个尚未占用的内网ip即可）
        192.168.0.133
    }
}
复制代码
slave服务器配置：

复制代码
! Configuration File for keepalived

global_defs {
   router_id xxoome_backup #起一个没重复的名字即可
}

# 检测nginx是否运行
vrrp_script chk_nginx {
        script "/etc/keepalived/nginx_check.sh"
        interval 2
        weight -20
}

vrrp_instance VI_1 {
    state BACKUP

    # 网卡名字，文章下方会给出如何获取网卡名字的方法
    interface enp0s3
    virtual_router_id 106

    # 本机ip
    mcast_src_ip 192.168.0.183
    priority 80 #权重，slave要小于master
    nopreempt
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    # 与上方nginx运行状况检测呼应
    track_script {
        chk_nginx
    }

    virtual_ipaddress {
        #虚拟ip（VIP，一个尚未占用的内网ip即可，与master配置一致）
        192.168.0.133
    }
}
复制代码
nginx监听脚本：

#! /bin/bash
pidof nginx
if [ $? -ne 0 ];then
/etc/init.d/keepalived stop
fi
启动服务：

service keepalived start


查看服务启动情况：

ps -aux |grep keepalived


查看启动日志：

journalctl -xe
 配置成功后的效果。enp0s3是网卡名字；192.168.0.133是虚拟ip，已经成功绑定到网卡上。



 

```
