 ```
 系统版本：contos7_64

kafka简介：


1、安装JDK

http://www.cnblogs.com/xxoome/p/7112637.html

2、下载kafka

下载页面：http://kafka.apache.org/downloads.html

2.1、解压

tar zxvf kafka_2.11-1.1.0.tgz


2.2、启动和停止

配置server.properties

# 配置外网访问
listeners = PLAINTEXT://192.168.1.232:9092
启动zookeeper server:

./zookeeper-server-start.sh ../config/zookeeper.properties &
启动kafka server:

./kafka-server-start.sh ../config/server.properties &
停止kafka server:

./kafka-server-stop.sh
停止zookeeper server:

./zookeeper-server-stop.sh
3、单机连通性测试

运行producer：

./kafka-console-producer.sh --broker-list localhost:9092 --topic test
运行consumer：

./kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
在producer端输入字符串并回车，查看consumer端是否显示。


 
 ```
