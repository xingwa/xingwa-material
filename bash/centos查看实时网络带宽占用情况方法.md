```
Linux中查看网卡流量工具有iptraf、iftop以及nethogs等，iftop可以用来监控网卡的实时流量(可以指定网段)、反向解析IP、显示端口信息等。


centos安装iftop的命令如下：

yum install iftop -y
复制代码

常用参数说明：
-i设定监测的网卡，如：

iftop -i eth1

```
