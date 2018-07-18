### CentOS设置系统时间与网络时间同步

```
1.  安装ntpdate工具

# yum -y install ntp ntpdate

2.  设置系统时间与网络时间同步

# ntpdate cn.pool.ntp.org

3.  将系统时间写入硬件时间

# hwclock --systohc
```
