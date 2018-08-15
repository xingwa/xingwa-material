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
 
 ```
