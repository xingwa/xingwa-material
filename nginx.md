### Nginx配置中的log_format用法梳理
```

nginx服务器日志相关指令主要有两条：一条是log_format，用来设置日志格式；另外一条是access_log，用来指定日志文件的存放路径、格式和缓存大小，可以参加ngx_http_log_module。一般在nginx的配置文件中日记配置(/usr/local/nginx/conf/nginx.conf)。

log_format指令用来设置日志的记录格式，它的语法如下：
log_format name format {format ...}
其中name表示定义的格式名称，format表示定义的格式样式。
 
log_format有一个默认的、无须设置的combined日志格式设置，相当于Apache的combined日志格式，其具体参数如下：
log_format combined '$remote_addr-$remote_user [$time_local]'
‘"$request"$status $body_bytes_sent’
 ‘"$http_referer" "$http_user_agent"’
 
也可以自定义一份日志的记录格式，不过要注意，log_format指令设置的名称在配置文件中是不能重复的。
 
假设将Nginx服务器作为Web服务器，位于负载均衡设备、Squid、Nginx反向代理之后，不能获取到客户端的真实IP地址了。
原因是经过反向代理后，由于在客户端和Web服务器之间增加了中间层，因此Web服务器无法直接拿到客户端的IP。
通过$remote_addr变量拿到的将是反向代理服务器的IP地址。
 
但是，反向代理服务器在转发请求的HTTP头信息中，可以增加X-Forwarded-For信息，用以记录原有的客户端IP地址和原来客户端请求的服务器地址。
这时候，要用log_format指令设置日志格式，让日志记录X-Forearded-For信息中的IP地址，即客户的真实IP。
 
例如，创建一个名为mylogformat的日志格式，再$http_x_forwarded_forlog_for变量记录用户的X_Forwarded-For IP 地址：
log_format mylogformat '$http_x_forwarded_for_$remote_user [$time_local]'
‘"$request"$status $body_bytes_sent’
 ‘"$http_referer" "$http_user_agent"’
在日志格式样式中，变量$remote_addr和$http_x_forwarded_for用于记录IP地址；
$remote_user用于记录远程客户端用户名称；
$time_local用于记录访问时间与时区；
$request用于记录请求URL与HTTP协议；
$status用于记录请求状态，例如成功时状态为200，页面找不到时状态为404；
$body_bytes_sent用于记录发送客户端的文件主体内容大小；
$http_referer用于记录是从哪个页面链接访问过来的；
$http_user_agent用于记录客户浏览器的相关信息。
一般来说：nginx的log_format有很多可选的参数用于指示服务器的活动状态，默认的是：


log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
想要记录更详细的信息需要自定义设置log_format，具体可设置的参数格式及说明如下：

参数                      说明                                         示例
$remote_addr             客户端地址                                    211.28.65.253
$remote_user             客户端用户名称                                --
$time_local              访问时间和时区                                18/Jul/2012:17:00:01 +0800
$request                 请求的URI和HTTP协议                           "GET /article-10000.html HTTP/1.1"
$http_host               请求地址，即浏览器中你输入的地址（IP或域名）     www.wang.com 192.168.100.100
$status                  HTTP请求状态                                  200
$upstream_status         upstream状态                                  200
$body_bytes_sent         发送给客户端文件内容大小                        1547
$http_referer            url跳转来源                                   https://www.baidu.com/
$http_user_agent         用户终端浏览器等信息                           "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C;
$ssl_protocol            SSL协议版本                                   TLSv1
$ssl_cipher              交换数据中的算法                               RC4-SHA
$upstream_addr           后台upstream的地址，即真正提供服务的主机地址     10.10.10.100:80
$request_time            整个请求的总时间                               0.205
$upstream_response_time  请求过程中，upstream响应时间                    0.002
如下是在nginx的LB代理层使用过的一个配置（nginx.conf中配置）：


log_format  main  '$remote_addr $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '$http_user_agent $http_x_forwarded_for $request_time $upstream_response_time $upstream_addr $upstream_status';
然后在nginx.conf文件或vhosts/*.conf文件中的access_log日志中指定级别为main。如下：


access_log  logs/wiki_access.log main;
error_log   logs/wiki_error.log;
重启nginx服务后生效。日志截取如下（可以从日志中看到代理到后端哪台机器上的哪个端口上，负载访问的状态值等都能看到）：

[root@lb-ng01 logs]# tail -f /data/nginx/logs/wiki_access.log
........
110.156.114.121 - [11/Aug/2017:09:57:19 +0800] "GET /rest/mywork/latest/status/notification/count?_=1502416641768 HTTP/1.1" 200 67 "http://wiki.wang-inc.com/pages/viewpage.action?pageId=11174759" Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.78 Safari/537.36 - 0.006 0.006 12.129.120.121:8090 200
112.116.25.18 - [11/Aug/2017:09:57:24 +0800] "POST /json/startheartbeatactivity.action HTTP/1.1" 200 234 "http://wiki.wang-inc.com/pages/resumedraft.action?draftId=12058756&draftShareId=014b0741-df00-4fdc-90ca-4bf934f914d1" Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36 - 0.023 0.023 12.129.120.121:8090 200

***************当你发现自己的才华撑不起野心时，就请安静下来学习吧***************

```
