## 一、下载

官网：http://zookeeper.apache.org/releases.html

镜像：https://www.apache.org/dyn/closer.cgi/zookeeper

## 二、安装
ZooKeeper是运行在Java环境下的，所以需要先安装JDK，注意需要Oracle JDK，Open JDK 1.8存在问题。

* 解压
```
shell> tar -xvf zookeeper-3.5.3-beta.tar.gz -C /usr/local/   #自选目标目录
```

* 修改配置
```
shell> cd /usr/local/zookeeper-3.5.3-beta/conf
shell> cp zoo_sample.cfg zoo.cfg
shell> vim zoo.cfg
dataDir=/usr/local/zookeeper-3.6.1/data   #自选数据目录
dataLogDir=/usr/local/zookeeper-3.6.1/logs  #自选日志目录
```
## 三、启动&停止
### 1、启动
```
shell> bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
### 2、查看状态
```
shell> bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone
```
### 3、停止
```
shell> bin/zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```
### 4、重启
```
shell> bin/zkServer.sh restart
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
## 四、测试
### 1、jps，查看QuorumPeerMain是否存在
```
shell> jps
19748 Jps
19159 QuorumPeerMain
```
### 2、zkServer.sh status
```
shell> bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.3-beta/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone
```
### 3、zkClient
```
shell> bin/zkCli.sh -server localhost:2181
Connecting to localhost:2181
2020-03-26 23:41:43,303 [myid:] - INFO  [main:Environment@109] - Client environment:zookeeper.version=3.5.3-beta-8ce24f9e675cbefffb8f21a47e06b42864475a60, built on 04/03/2017 16:19 GMT
2020-03-26 23:41:43,306 [myid:] - INFO  [main:Environment@109] - Client environment:host.name=VM_2_24_centos
2020-03-26 23:41:43,306 [myid:] - INFO  [main:Environment@109] - Client environment:java.version=1.8.0_241
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:java.vendor=Oracle Corporation
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:java.home=/usr/local/java/jdk1.8.0_241/jre
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:java.class.path=/usr/local/zookeeper-3.5.3-beta/bin/../build/classes:/usr/local/zookeeper-3.5.3-beta/bin/../build/lib/*.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/slf4j-log4j12-1.7.5.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/slf4j-api-1.7.5.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/netty-3.10.5.Final.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/log4j-1.2.17.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jline-2.11.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jetty-util-9.2.18.v20160721.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jetty-servlet-9.2.18.v20160721.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jetty-server-9.2.18.v20160721.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jetty-security-9.2.18.v20160721.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jetty-io-9.2.18.v20160721.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jetty-http-9.2.18.v20160721.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/javax.servlet-api-3.1.0.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jackson-mapper-asl-1.9.11.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/jackson-core-asl-1.9.11.jar:/usr/local/zookeeper-3.5.3-beta/bin/../lib/commons-cli-1.2.jar:/usr/local/zookeeper-3.5.3-beta/bin/../zookeeper-3.5.3-beta.jar:/usr/local/zookeeper-3.5.3-beta/bin/../src/java/lib/*.jar:/usr/local/zookeeper-3.5.3-beta/bin/../conf:.:/usr/local/java/jdk1.8.0_241/lib/dt.jar:/usr/local/java/jdk1.8.0_241/lib/tools.jar
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:java.io.tmpdir=/tmp
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:java.compiler=<NA>
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:os.name=Linux
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:os.arch=amd64
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:os.version=3.10.0-1062.9.1.el7.x86_64
2020-03-26 23:41:43,308 [myid:] - INFO  [main:Environment@109] - Client environment:user.name=root
2020-03-26 23:41:43,309 [myid:] - INFO  [main:Environment@109] - Client environment:user.home=/root
2020-03-26 23:41:43,309 [myid:] - INFO  [main:Environment@109] - Client environment:user.dir=/usr/local/zookeeper-3.5.3-beta
2020-03-26 23:41:43,309 [myid:] - INFO  [main:Environment@109] - Client environment:os.memory.free=52MB
2020-03-26 23:41:43,310 [myid:] - INFO  [main:Environment@109] - Client environment:os.memory.max=228MB
2020-03-26 23:41:43,310 [myid:] - INFO  [main:Environment@109] - Client environment:os.memory.total=57MB
2020-03-26 23:41:43,313 [myid:] - INFO  [main:ZooKeeper@865] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@5e8c92f4
2020-03-26 23:41:43,323 [myid:] - INFO  [main:ClientCnxnSocket@236] - jute.maxbuffer value is 4194304 Bytes
Welcome to ZooKeeper!
2020-03-26 23:41:43,335 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1113] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2020-03-26 23:41:43,401 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@948] - Socket connection established, initiating session, client: /0:0:0:0:0:0:0:1:42446, server: localhost/0:0:0:0:0:0:0:1:2181
2020-03-26 23:41:43,412 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1381] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x10009d728340001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
```
