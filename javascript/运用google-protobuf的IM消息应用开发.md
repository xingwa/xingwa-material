Protocol Buffers是谷歌定义的一种跨语言、跨平台、可扩展的数据传输及存储的协议，因为将字段协议分别放在传输两端，传输数据中只包含数据本身，不需要包含字段说明，所以传输数据量小，解析效率高。一条消息用protobuf序列化后的大小是json的10分之一。类似的序列化框架还有Thrift、avro。thrift和avro都提供rpc服务和序列化，而protocol buffer只是提供序列化功能。

```
https://www.cnblogs.com/1wen/p/6509253.html

https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/index.html

https://www.cnblogs.com/naci/p/5341433.html(PHP实例)

https://blog.csdn.net/njweiyukun/article/details/69388227（node.js）
```
