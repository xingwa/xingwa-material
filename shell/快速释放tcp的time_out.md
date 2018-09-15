```
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'   


今天压力测试时， 刚开始出现了很多异常， 都是 java.net.NoRouteToHostException: Cannot assign requested address. 
经网上查资料， 是由于linux分配的客户端连接端口用尽， 无法建立socket连接所致，虽然socket正常关闭，但是端口不是立即释放， 而是处于TIME_WAIT状态， 默认等待60s后才释放。
    查看linux支持的客户端连接端口范围, 也就是28232个端口： 
        cat  /proc/sys/net/ipv4/ip_local_port_range
        32768 - 61000
    
    解决方法：
    1. 调低端口释放后的等待时间， 默认为60s， 修改为15~30s
        echo 15 > /proc/sys/net/ipv4/tcp_fin_timeout
    2. 修改tcp/ip协议配置， 通过配置/proc/sys/net/ipv4/tcp_tw_resue, 默认为0， 修改为1， 释放TIME_WAIT端口给新连接使用。
        echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
    3. 修改tcp/ip协议配置，快速回收socket资源，  默认为0， 修改为1.
        echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle


    通过上面3项调整， 压力测试运行正常。
    
    
    统计在一台前端机上高峰时间TCP连接的情况，统计命令：
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

结果：



除了ESTABLISHED，可以看到连接数比较多的几个状态是：FIN_WAIT1, TIME_WAIT, CLOSE_WAIT, SYN_RECV和LAST_ACK；下面的文章就这几个状态的产生条件、对系统的影响以及处理方式进行简单描述。

 

发现存在大量TIME_WAIT状态的连接
tcp        0      0 127.0.0.1:3306              127.0.0.1:41378             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:41379             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:39352             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:39350             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:35763             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:39372             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:39373             TIME_WAIT
tcp        0      0 127.0.0.1:3306              127.0.0.1:41176             TIME_WAIT
 
 
 
通过调整内核参数解决
vi /etc/sysctl.conf


编辑文件，加入以下内容：
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
 
然后执行/sbin/sysctl -p让参数生效。
 
net.ipv4.tcp_syncookies = 1表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout修改系統默认的TIMEOUT时间
 
修改之后，再用命令查看TIME_WAIT连接数
netstat -ae|grep “TIME_WAIT” |wc –l

 发现大量的TIME_WAIT 已不存在，mysql进程的占用率很快就降下来的，网站访问正常。
 不过很多时候，出现大量的TIME_WAIT状态的连接，往往是因为网站程序代码中没有使用mysql.colse()，才导致大量的mysql  TIME_WAIT.

 

根据TCP协议定义的3次握手断开连接规定,发起socket主动关闭的一方 socket将进入TIME_WAIT状态,TIME_WAIT状态将持续2个MSL(Max Segment Lifetime),在Windows下默认为4分钟,即240秒,TIME_WAIT状态下的socket不能被回收使用. 具体现象是对于一个处理大量短连接的服务器,如果是由服务器主动关闭客户端的连接,将导致服务器端存在大量的处于TIME_WAIT状态的socket, 甚至比处于Established状态下的socket多的多,严重影响服务器的处理能力,甚至耗尽可用的socket,停止服务. TIME_WAIT是TCP协议用以保证被重新分配的socket不会受到之前残留的延迟重发报文影响的机制,是必要的逻辑保证.
      在HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters,添加名为TcpTimedWaitDelay的
DWORD键,设置为60,以缩短TIME_WAIT的等待时间

http://kerry.blog.51cto.com/172631/105233/

修改之后，再用
netstat -ae|grep mysql
tcp        0      0 aaaa:50408               192.168.12.13:mysql           ESTABLISHED nobody     3224651
tcp        0      0 aaaa:50417               192.168.12.13:mysql           ESTABLISHED nobody     3224673
tcp        0      0 aaaa:50419               192.168.12.13:mysql           ESTABLISHED nobody     3224675

发现大量的TIME_WAIT 已不存在，mysql进程的占用率很快就降下来的，各网站访问正常！！
 以上只是暂时的解决方法，最后仔细巡查发现是前天新上线的一个系统，程序代码中没有使用mysql.colse()，才导致大量的mysql  TIME_WAIT 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 CentOS 7 运维优化
一般的，我们安装CentOS mini和其他相应服务后，就能正常工作了。但工作一段时间后，服务器会出现不稳定、被入侵、甚至在突然的高并发时直接瘫痪状况。这些问题大多都是由于我们考虑其实际的抗压性和安全性的原因。所以，在这里提供一些运维优化的建议。

1.关闭不需要的服务
众所周知，服务越少，系统占用的资源就会越少， 所以应当关闭不需要的服务。建议把不需要的服务关闭掉，这样做的好处是减少内存和CPU资源占用。首先可以看下系统中 存在着哪些已经启动了的服务。

// 安装ntsysv
yum install -y ntsysv
// 设置启动的服务
ntsysv
1
2
3
4
下面列出的是需要启动的服务，未列出的服务一律关闭。

crond ： 自动计划任务。
network：Linux系统的网络服务，很重要，若不开启此服务的话，服务器就不能联网。
sshd： OpenSSH 服务器守护进程。
rsyslog：Linux的日志系统服务（CentOS5.8下此服务名称为syslog），必须要启动。
2．关闭不需要的TTY
可用vim编辑器打开文件

vim /etc/init/start-ttys.conf
// 内容如下：
start on stopped rc RUNLEVEL=[2345]
env ACTIVE_CONSOLES=/dev/tty[1-6]
env X_TTY=/dev/tty1
task
script
    . /etc/sysconfig/init
    for tty in $(echo $ACTIVE_CONSOLES) ; do
        [ "$RUNLEVEL" = "5" -a "$tty" = "$X_TTY" ] && continue
        initctl start tty TTY=$tty
    done
end script
1
2
3
4
5
6
7
8
9
10
11
12
13
这段代码使 init 打开了6个控制台，可分则用 ALT + F1 到 ALT + F6 控制台默认都驻留在内存中。用ps aux 命令即可看到，命今如下：

ps aux | grep tty | grpe -v grep
1
命令显示结果如下所示：

root         1211  0.0  0.2 115520  2048 tty1
root         1213  0.0  0.2 115520  2048 tty2
root         1214  0.0  0.2 115520  2048 tty3
root         1217  0.0  0.2 115520  2048 tty4
root         1219  0.0  0.2 115520  2048 tty5
1
2
3
4
5
事实上没有必要使用这么多，那如何关闭不需婴的进程呢？ 
通常保留两个控制台就可以了。

vim /etc/init/start-ttys.conf
1
3．对TCP／IP网络参数进行调整
调整TCPⅡP网络参数，可以加强对抗 SYN Flood 的能力，命令如下：

echo 'net.ipv4.tcp_syncookies = 1' >> /etc/sysctl.conf
sysctl -p
1
2
4．修改SHELL命令的 history 记录个数
// 用Vim编辑器打开
vim /etc/profile
// 找到HISTSIZE=1000 并改为 100；
HISTSIZE=100
// 立即生效
source /etc/profile
1
2
3
4
5
6
5.定时校正服务器的时间
yum install -y ntp
crontab -e
// 加入一行
*/5 * * * * /usr/sbin/ntpdate ntp.api.bz
1
2
3
4
ntp.api.bz是一组NTP服务器集群，之前是6台服务器，位于上海电信；现在是3台服务器，分散于上海和浙江电信，可以用 dig 命令查看

dig ntp.api.bz
1
6．停止IPV6网络服务
在 CentOS64 默认的状态下，IPv6 是被启用的。

// 可用如下命令查看：
lsmod | grep ipv6
1
2
有些网络和应用程序还不支持 IPv6 ，因此，禁用 IPv6 可以说是一个非常好的选择： 加强系统的安全性，并提高系统的整体性能。不过，首先要确认一下：IPv6是不是处于动的状态，命令如下：

// 列出全部网络接口信息
ifconfig -a

// 修改相应的配置文件，停止 IPv6 ，命令如下：
echo "install ipv6 /bin/true" > /etc/modprobe.d/disable-ipv6.conf
# 每当系统需要加载IPv6时，强制执行 /bin/true 来替代实际加载的模块
echo "IPV6INIT=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
# 禁用基于IPv6网络，使之不会被触发启动
1
2
3
4
5
6
7
8
7.调整 Linux 的最大文件打开数
要调整一下 Linux 的最大文件打开数，否则运行 Squid 诅服务的机器在高负载时执行性能将会很差；另外，在 Linux 下部署应用时，有时候会遇上 “Too many open files” 这样的问题，这个值也会影响服务器的最大并发数。其实 Linux 是有文件句柄限制的。但默认值下是很高，一般是1024，生产服务器很容易就会达到这个值，所以需要改动此值。

// 打开配置  
vim /etc/security/limit.conf
// 在最后一行添加如下
* soft nofile 65535
* hard nofile 65535
// 再打开配置
vim /etc/rc.local
// 添加如下内容
ulimit -SHn 65535 
1
2
3
4
5
6
7
8
9
另外，ulimit -n 命令并不能真正看到文件的最大文件打开数。可用如下脚本查看：

#!/bin/bash
for pid in `ps aux |grep nginx |grep -v grep|awk '{print $2}'`
do
cat /proc/${pid}/limits |grep 'Max open files'
done
1
2
3
4
5
8.启动网卡
在配置 CentOS 7 网卡 IP 地址时，容易忽略的一项是Linux在启动时未 启动网卡，其后果很明显，那就是该 Linux 机器永远也没有 IP 地址。

// 查看以太网代号(也可用ifconfig命令)
ip address
// 修改网卡配置文件
vim /etc/sysconfig/network-scripts/ifcfg-enp1s0
// 修改如下内容（如果没有，请自行添加）
# 系统启动时就启动网卡设备
ONBOOT=yes
# 允许用从DHCP处获取的DNS覆盖本地的DNS
PEERDNS=yes
# 不允许普通用户修改网卡
USERCTL=no
1
2
3
4
5
6
7
8
9
10
11
9．关闭写磁盘I/O功能
Linux文件默认有3个时间，分别如下所示。

atime：对此文件的访问时间。
ctime：此文件inOde发生变化的时间。
mtime：此文件的修改时间。
如果有多个小文件（比如 Web 服务器的页面上有多个小图片），通常是没有必要记录文件的访问时间的，这样就可以减少写磁盘的 I/O ，可这要如何配置呢？

// 修改文件系统的配置文件
vim /etc/fstab
// 然后，在包含大量小文件的分区中使用 noatime 和 nodiratime 这两个命令。例如：  
/dev/sda5 /data/pics ext3 noatime,nodiratime 0 0 
这样文件被访问时就不会再产生写磁盘的 I/O 了。 
1
2
3
4
5
10．修改SSH登录配置
SSH服务配置优化，请保持机器中至少包含一个具有sudo权限的用户，下面的配置禁止root远程登录，代码内容如下所示：

# 禁止root远程登录
sed -i 's@#PermitRootLogin yes@PermitRootLogin no@' /etc/ssh/sshd_config
# 禁止空密码登录
sed -i 's@PermitEmptyPasswords no@PermitEmptyPasswords no@' /etc/ssh/sshd_config
# 关闭SSH反向查询，以加快SSH的访问速度
sed -i 's@UseDNS yes@UseDNS no@' /etc/ssh/sshd_config /etc/ssh/sshd_config
1
2
3
4
5
6
11.增加具有SUdO权限的用户
添加用户的步骤和过程比较简单这里略过，由于系统已经禁止了root远程登录，因 此需要一个具有sudo权限的admin用户，权限跟root相当。

vim /etc/sudoers
## Allow root to run any commands anywhere
root    ALL=(ALL)   ALL
# 然后添加如下内容：
admin   ALL=(ALL)   ALL
# 如果在进行sudo切换时不想输入密码，可以做如下更改：
admin   ALL=(ALL)   NOPASSWD:ALL
1
2
3
4
5
6
7
12.优化Linux下的内核TCP参数以提高系统性能
内核的优化跟服务器的优化一样，应本着稳定安全的原则。下面以Squid服务器为例来说明，待客户端与服务器端建立 TCP/IP 连接后就会关闭Socket，服务器端连接的端口状态也就变为 TIME_WAIT 了。那是不是所有执行主动关闭的SOCket都会进入TIME_WAIT 状态呢？有没有什么情况可使主动关闭的Socket直接进入CLOSED状态呢？答案是主动关 闭的一方在发送最后一个ACK后就会进人 TIME_WAIT 状态，并停留2MSL（报文最大生存）时间，这是 TCP/IP 必不可少的，也就是说这一点是“解决”不了的。

TCP/IP 护设计者如此设计，主要原因有两个：

防止上一次连接中的包迷路后重新出现，影响新的连接经过 2MSL 时间后，上一次连接中所有重复的包都会消失。
为了可靠地关闭TCP连接。主动关闭方发送的最后一个ACKFN有可能会丢失，如果丢失，被动方会重新发送Fm，这时如果主动方处于CLOSED状态，就会q 应RST而不是ACK。所以主动方要处于TIM巳吣IT状态，而不能是CLOSED」态。另外，TIME_WAIT 并不会占用很大的资源，除非受到攻击。
// 在Squid服务器中可输入如下命令查看当前连接统计数： 
netstat -n | awk '/^tcp/ {++S[$NF]} END{for(a in S)} print a, S[a]}'  
1
2
命令显示结果如下所示：

LAST_ACK 14
SYN_RECV 348
ESTABISHED 70
FIN_WAIT1 229
FIN_WAIT2 30
CLOSING 33
TIME_WAIT 18122
1
2
3
4
5
6
7
命令中的含义分别如下。

CLOSED：无恬动的或正在进行的连接。
LISTEN：服务器正在等待进入呼叫。
SYN_RECV：一个连接请求已经到达，等待确认。
SYN_SENT：应用已经开始，打开一个连接。
ESTABLISHED；正常数据传输状态。
FIN_WAT1：应用说它已经完成。
FIN_WAT2：另一边己同意释放。
ITMED_WAIT：等待所有分组死掉。
CLOSING；两边尝试同时关闭。
TIME_WAIT：另一边已初始化一个释放。
LAST_ACK：等待所有分组死掉。 
也就是说，这条命令可以把当前系统的网络连接状态分类汇总。 
在 Linux 下高并发的 Squid 服务器中，TCP TIME_WAIT 套接字的数量经常可达到两三万，服务器很容易就会被拖死。不过，可以通过修改Linux 内核参数来减少 Squid 服务器的 TIME_WAIT 套接字数量，命令如下：
vim /etc/sysctl.conf
// 然后，增加以下参数
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
1
2
3
4
5
6
7
8
9
10
以下将简单说明上面各个参数的含义：

net.ipv4.tcp_syncookies = 1表示开启SYN Cookies。当出现SYN等待队列溢出时 启用 Cookie 旋来处理，可防范少量的SYN 攻击。该参数默认为0，表示关闭。
net.ipv4.tcp_tw_reuse = 1表示开启重用，即允许将TIME-WAIT 套接字重新用于的TCP连接。该参数默认为 0，表示关闭。
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT 套接字的快速回收，该参数默认为0，表示关闭。
net.ipv4.tcp_fin_timeout = 30 表示如果套接字由本端要求关闭，那么这个参数将决定保持在FlN-WAIT-2 状态的时间。
net.ipv4.tcp_keepalive_time = 1200 表示当 Keepalived 启用时，TCP发送Keepalived 消息的频度改为20分钟，默认值是2小时。
net.ipv4.ip_local_port_range = 10000 65000 表示CentOS 系统向外连接的端口范围。其默认值很小，这里改为10000到65000。建议不要将这里的最低值设得太低，否则可能会占用正常的端口。
net.ipv4.tcp_max_syn_backlog = 8192 表示SYN队列的长度，默认值为1024，此处加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000 表示系统同时保持TIME_WAIT 套接字的最大数量，如果超过这个数字，TlME_WAIT 套接字将立刻被清除并打印警告信息，默认值为180000，此处改为5000。对于Apache、Nginx等服务器，前面介绍的几个参数已经可以很好地减少TIME_WAIT套接字的数量，但是对于Squid来说，效果却不大，有了此参数就可以控制TME_WAIT 套接字的最大数量，避免Squid 记服务器被大量的TIME_WAIT 套接字拖死。
执行以下命令使内核配置立马生效：

/sbin/sysctl -p
1
如果是用于Apache 或 Nginx 等 Web 服务器，则只需要更改以下几项即可。

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 10000 65000

// 执行以下命令使内核配置立马生效
/sbin/sysctl -p
1
2
3
4
5
6
7
如果是Post6x邮件服务器，则建议内核优化方案如下:

net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 10000 65000
kernel.shmmax = 134217728

// 执行以下命令使内核配置立马生效
/sbin/sysctl -p
1
2
3
4
5
6
7
8
9
当然这些都只是最基本的更改，大家还可以根据自己的需求来更改内核的设置，比如我们的线上机器在高并发的情况下，经常会出现 ‘‘TCP：too many orpharned sockets ” 的报错尽量也要本着服务器稳定的最高原则。如果服务器不稳定的话，一切工作和努力就都会白费。 
如果以上优化仍无法满足工作要求，则又可能需要定制你的服务器内核或升级服务器硬件。

版权声明：本文多数内容来自互联网，只经过本人修改组合，无版权，故可随意转载。( ¯▽¯；)	https://blog.cs
 
 
 
 
 
 
 
 
 
 
 
 
 ```
