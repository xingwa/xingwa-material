```
https://www.cnblogs.com/hujiapeng/p/7352904.html

通过yum等软件仓库都可以直接安装RabbitMQ，但版本一般都较为保守。
RabbitMQ官网提供了新版的rpm包（http://www.rabbitmq.com/download.html），但是安装的时候会提示需要erlang版本>=19.3，然而默认yum仓库中的版本较低。
其实RabbitMQ在github上有提供新的erlang包（https://github.com/rabbitmq/erlang-rpm）
也可以直接加到yum源中

#vim /etc/yum.repos.d/rabbitmq-erlang.repo
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/20/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1

#yum clean all
#yum makecache
然后下载RabbitMQ的RPM包(http://www.rabbitmq.com/download.html)

这里是centos7的版本
#wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.4/rabbitmq-server-3.7.4-1.el7.noarch.rpm
#yum install rabbitmq-server-3.7.4-1.el7.noarch.rpm
yum会自动去源里安装依赖包
安装到这里就完成了，下面进行简单的配置

启动RabbitMQ服务
#service rabbitmq-server start
状态查看
#rabbitmqctl status
启用插件
#rabbitmq-plugins enable rabbitmq_management
重启服务
#service rabbitmq-server restart
添加帐号:name 密码:passwd
#rabbitmqctl add_user name passwd
赋予其administrator角色
#rabbitmqctl set_user_tags name administrator
设置权限
#rabbitmqctl set_permissions -p / name ".*" ".*" ".*"

作者：薇文文
链接：https://www.jianshu.com/p/f54dc259a9ed
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```
