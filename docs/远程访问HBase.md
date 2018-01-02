# 远程访问HBase

在开发HBase程序时，需要在本地IDE进行调试。然而HBase却部署在云端，因此需要使用Java客户端程序远程访问HBase，客户端需要进行相应的配置。而为了让客户端程序通过外网访问顺利访问，服务器也要进行相应配置。然而在实际操作过程中遇到了很多的问题，这些问题大都与HBase的启动过程有关系。

## Java客户端配置

`${HBASE_HOME}/conf/hbase-site.xml`默认是没有任何配置的，HBase的默认配置都是从`hbase-common-x.x.x.jar`中的`hbase-default.xml`中读取出来的。HBase会默认调用监控着`127.0.0.1`上`2182`端口的zookeeper来获得HMaster的地址与端口，从而与HMaster进行更进一步的交互。为了与远程的zookeeper进行交互，需要配置`hbase.zookeeper.quorum`指定远程地址，`hbase.zookeeper.property.clientPort`指定远程端口。有两种方法进行配置：

* 新建`src/main/resource/hbase-site.xml`文件，进行配置：
```
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>远程ip</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
```

* 写死到代码里面：
```
configuration.set("hbase.zookeeper.quorum","远程ip");  //hbase 服务地址
configuration.set("hbase.zookeeper.property.clientPort","2181"); //端口号
```

第二种比较方便，但不利用类文件的迁移，推荐使用第一种方法。

## 服务器端配置

### 服务器的hostname

zookeeper会返回给客户端HMaster的地址。然而离奇的是，zookeeper返回的并不是`ip地址`，而是`hostname`。这给编写客户端的初学者造成了很大困扰，因为服务器的`hostname`并没有在往往都没有在客户端`/etc/hosts`文件中进行配置，DNS也无法进行解析。因此客户端出现的是一次又一次地重试，最后出现无法解析`hostname`的错误。因此，

* 需要客户端的`/etc/hosts`文件中对`hostname`进行配置，解析到服务器的ip。

当然，如果想要自己配置服务器的`hostname`，需要更改服务器端的`/etc/hostname`文件中进行配置，重启生效。

### HMaster绑定的ip和端口

HMaster绑定ip和端口的时候，会读取`/etc/hosts`文件，解析出服务器对应的`hostname`对应的ip地址，绑定特定端口。最初的时候`hostname`对应的很可能是`127.0.0.1`上的端口，是服务器本地环路，外界无法访问。这时候，客户端在拿到HMaster地址之后，访问服务器的HMaster端口，会出现拒绝连接的错误。所以需要对服务器端的`/etc/hosts`文件进行配置：

* 将`hostname`对应的ip改成实际的ip地址。
