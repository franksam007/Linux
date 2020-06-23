## 开始之前

更新系统：
```
sudo yum update
```
注意本指南是为非root用户编写的。需要提升特权的命令以开头sudo。要利用复制，至少需要两个节点。

## 安装Redis
在本部分中，将添加EPEL存储库，然后使用它来安装Redis。

1. 添加EPEL存储库，并更新YUM以确认更改：
```
sudo yum install epel-release
sudo yum update
```

2. 安装Redis：
```
sudo yum install redis
```

3. 启动Redis：
```
sudo systemctl start redis
```
可选：在启动时自动启动Redis：
```

sudo systemctl enable redis
```

## 验证安装
验证Redis是否在运行`redis-cli`：
```
redis-cli ping
```
如果Redis正在运行，它将返回：
```
PONG
```

## 配置Redis
在本节中，将为Redis配置一些基本的持久性和调整选项。

### 持久性选项
Redis提供了两种磁盘持久性选项：

* 以指定的时间间隔制作的数据集的时间点快照（RDB）。
* 服务器执行的所有写操作的仅追加日志（AOF）。

每个选项都有其自身的优缺点，Redis文档中对此进行了详细介绍。为了最大程度地提高数据安全性，请考虑同时运行两种持久性方法。

由于默认情况下启用了时间点快照持久性，因此只需要设置AOF持久性：

1. 确保在redis.conf中的appendonly和appendfsync设置中设置了以下值：
* /etc/redis.conf
```
appendonly yes
appendfsync everysec
```
2. 重新启动Redis：
```
sudo systemctl restart redis
```

### 基本系统调整
为了提高Redis的性能，请将Linux内核的过量使用内存设置为1：
```
sudo sysctl vm.overcommit_memory=1
```
这将立即更改过量使用的内存设置，但更改不会在重新启动后持续存在。要使其永久存在，须将vm.overcommit_memory = 1添加到/etc/sysctl.conf：

* /etc/sysctl.conf
```
vm.overcommit_memory = 1
```

### 额外交换

根据使用情况，可能发现有必要添加额外的交换磁盘空间。可以通过在Linode Manager中调整磁盘大小来添加交换。Redis文档建议交换磁盘的大小与系统可用的内存量匹配。

## 分布式Redis
Redis提供了几种设置分布式数据存储的选项。下面介绍的最简单的选项是主/从复制，它创建数据的副本。只要所有写入操作均由主服务器处理，它还将允许在从属副本组之间分配读取。

可以使用Redis Sentinel使上述主/从设置变得高度可用。Sentinel可以配置为监视主实例和从属实例，并且如果主节点未按预期工作，它将执行自动故障转移。这意味着从节点之一将被选为主节点，而所有其他从节点将被配置为使用新的主节点。

在Redis 3.0及更高版本中，可以使用Redis Cluster，这是一种自动管理复制和故障转移的数据分片解决方案。使用Redis Cluster，可以将数据集自动拆分到多个节点中，这在数据集大于单个服务器的RAM时非常有用。它还使您能够在部分节点出现故障或无法与群集的其余部分通信时继续操作。

按照以下步骤可完成主/从复制，并将从属设置为只读模式。

### 设置Redis主/从复制
在本节中，将使用两个Linode，一个主节点和一个从节点。

注意要通过专用网络进行通信，您的主节点Linode和从节点Linode必须位于同一数据中心。

#### 准备Linnode
1. 使用本指南中的“安装和配置”步骤，用Redis实例设置两个Linode。也可以使用Linode Manager中的Clone选项将最初配置的磁盘复制到另一个Linode。
2. 在两个Linode上配置专用IP地址，并确保可以从属服务器访问主Linode的专用IP地址。出于安全原因，仅将私有地址用于复制流量。

#### 配置主Linode
1. 通过更新redis.conf中的bind配置选项，将主Redis实例配置为侦听私有IP地址。替换192.0.2.100为主Linode的专用IP地址：
* /etc/redis.conf
```
 bind 127.0.0.1 192.0.2.100
```
2. 重新启动Redis以应用更改：
```
sudo systemctl restart redis
```
#### 配置从节点
1. 通过向redis.conf添加slaveof指令来配置复制来配置从属实例。再次用主节点Linode的专用IP地址替换192.0.2.100。该slaveof指令有两个参数：第一个是主节点的IP地址，第二个是主机配置中指定的Redis端口。
* /etc/redis.conf
```
slaveof 192.0.2.100 6379
```
2. 重新启动从属Redis实例：
```
sudo systemctl restart redis
```
重新启动后，从属Linode将尝试将其数据集同步到主服务器，然后传播更改。

#### 确认复制
测试复制是否有效。在主Linode上，运行redis-cli并执行命令`set 'a' 1`
```
redis-cli
127.0.0.1:6379> set 'a' 1
OK
```
键入exit或按Ctrl-C退出redis-cli提示。

接下来，redis-cli在从属节点Linode上运行并执行get 'a'，这应该返回与主节点相同的值：
```
redis-cli
127.0.0.1:6379> get 'a'
"1"
```
则主/从复制设置正常运行。

## Redis安全
由于Redis旨在在受信任的环境中和受信任的客户端一起工作，因此应该控制对Redis实例的访问。建议的一些安全步骤包括：

* 使用iptables设置防火墙。
* 使用SSH隧道或Redis安全性文档中描述的方法对Redis通信进行加密。

此外，为确保没有外部流量访问您的Redis实例，建议仅侦听localhost接口或Linode的专用IP地址上的连接。

### 使用密码验证
为了增加安全性，请使用密码身份验证来保护主Linode和从属Linode之间的连接。

1. 在主Linode上，取消注释requirepassRedis配置中的行，并master_password用安全密码替换：
* /etc/redis.conf
```
requirepass master_password
```

2. 保存更改，然后通过在主Linode上重新启动Redis来应用更改：
```
sudo systemctl restart redis
```
3. 在从属Linode上，将主密码添加到Redis配置中masterpass，然后使用以下命令为从属Linode创建一个唯一密码requirepass：替换master_password为在主节点上配置的密码，然后替换slave_password为用于从属Linode的密码。
* /etc/redis.conf
```
masterpass master_password
requirepass slave_password
```
4. 保存更改，然后在从属Linode上重新启动Redis：
```
sudo systemctl restart redis
```
5. 连接到redis-cli主Linode上，并使用AUTH主密码进行身份验证：
```
redis-cli
127.0.0.1:6379> AUTH master_password
```
6. 通过身份验证后，可以通过运行查看有关Redis配置的详细信息INFO。这提供了很多信息，因此可以在命令中专门请求“Replication”部分：
```
127.0.0.1:6379> INFO replication
```
输出应类似于以下内容：
```
# Replication
role:master
connected_slaves:1
slave0:ip=192.0.2.105,port=6379,state=online,offset=1093,lag=1
```
它应确认Linode的主角色，以及连接了多少个从属Linode。 。
7. 从您的从属Linode，redis-cli使用您的从属密码连接并进行身份验证：

```
redis-cli
127.0.0.1:6379> AUTH slave_password

```
8. 通过身份验证后，可INFO用于确认从属Linode的角色及其与主服务器的连接：
```
127.0.0.1:6379> INFO replication
# Replication
role:slave
master_host:192.0.2.100
master_port:6379
master_link_status:up
```
