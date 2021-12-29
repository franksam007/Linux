### 前提
宿主机可以访问外网

### （虚拟机）网卡配置
编辑网卡配置， `vi /etc/sysconfig/network-scripts/ifcfg-eth0`，其中eth0为网卡设备名，须根据具体设备进行修改。
```
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.122.63
NETMASK=255.255.255.0
GATEWAY=192.168.122.1
```
此处使用静态配置，也可使用DHCP配置：
```
ONBOOT=yes
BOOTPROTO=dhcp
...
```

### （虚拟机）重启网络服务
`systemctl restart network`

### （虚拟机）设置默认路由，指向主机ip
`ip route add default via 192.168.122.1 dev eth0`

其中`192.168.122.1`是宿主机的虚拟网卡的IP，作为虚拟机的默认网关，通过此IP将虚拟机的包转发到主机的**真正**网卡IP上。

### （虚拟机）增加DNS服务器
默认情况下，虚拟机中将默认网关`192.168.122.1`作为DNS服务器，但该IP并不提供DNS服务，所以需要增加真正的DNS服务器。
`vi  /etc/resolv.conf`，添加以下内容（可以使用任何合法的DNS服务器IP）：
```
nameserver 202.106.0.20
nameserver 8.8.8.8
```
