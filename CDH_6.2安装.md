# CDH 6.2在Linux上的安装

（https://www.cloudera.com/documentation/enterprise/6/latest/topics/cm_ig_reqs_space.html）

## 1. 安装准备
### 1.1. 空间规划
#### Cloudera Manager Server
* 默认：/var/lib/cloudera-scm-server/
* 空间：主要是所用数据库的空间，取决于所管理的节点数以及多长时间删除已运行命令
* 配置：Administration > Settings：Command Eviction Age，默认2年

#### Cloudera Management Service
##### Activity Monitor Configuration
* 空间：主要是所用数据库的空间，取决于所运行的作业和任务
* 配置：Configuration > Scope > Activity Monitor or Cloudera Management Service > Category > Main.
  * Purge Activities Data at This Age
  * Purge Attempts Data at This Age
  * Purge MapReduce Service Data at This Age
  默认14天（336小时）
  
  实验观察：1000节点，200任务/作业，10分钟/作业，保留时间7天-->200G数据空间

##### Service Monitor Configuration
* 默认：/var/lib/cloudera-scm-server/
* 空间：
   * 10 GiB Services Time Series Storage
   * 1 GiB Impala Query Storage
   * 1 GiB YARN Application Storage
   共: 最少~12 GiB(无上限)
* 配置：Configuration > Scope > Service Monitor or Cloudera Management Service (Service-Wide) > Category > Main
  * Time-Series Storage
  * Impala Storage
  * YARN Storage

##### Host Monitor
* 默认：/var/lib/cloudera-host-monitor/
* 空间：
   * 10 GiB Host Time Series Storage
* 配置：Configuration > Scope > Host Monitor or Cloudera Management Service (Service-Wide) > Category > Main
  * Time-Series Storage

##### Event Server
* 默认：/var/lib/cloudera-scm-eventserver/
* 空间：
   * 5,000,000事件
* 配置：Configuration > Scope > Event Server or Cloudera Management Service  (Service-Wide) > Category > Main
  * Maximum Number of Events in the Event Server Store

##### Reports Manager
* 默认：/var/lib/cloudera-scm-headlamp/
* 空间：
   * 取决于数据库
* 配置：无直接参数控制其容量

#### Cloudera Navigator
##### Navigator Audit Server
* 默认：数据库
* 空间：
   * 保留90天审计日志
* 配置：Configuration > Scope > Navigator Audit Server or Cloudera Management Service (Service-Wide) > Category > Main
  *  Navigator Audit Server Data Expiration Period

##### Navigator Metadata Server
* 默认：/var/lib/cloudera-scm-navigator/
* 空间：
* 配置：无直接控制参数
  
#### Cluster Lifecycle Management with Cloudera Manager
##### Parcel Lifecycle Management
* 下面目录需要足够空间：
  * Local Parcel Repository Path: /opt/cloudera/parcel-repo
  * Parcel Cache: /opt/cloudera/parcel-cache
  * Host Parcel Directory: /opt/cloudera/parcels

##### Space Reclamation Tasks
* Activity Monitor (One-time)：活动监视器只对MR任务，如果所有任务迁移到Yarn，活动监视数据库将停止增长，等待一定时间后（根据保留时间），可以清空该数据库
* Service Monitor and Host Monitor (One-time)：如从4.x升级到5.x，原数据库迁移到专用时序数据库
* Ongoing Space Reclamation：Cloudera Management Services会自动在后台汇总、清除或以其他方式整合过时的数据。

### 1.2 配置网络名称
CDH需要IPv4。不支持IPv6。

按如下方式配置群集中的每个主机，以确保所有成员都可以相互通信：
1. 将主机名设置为唯一的名称（而不是localhost）。

`sudo hostnamectl set hostname foo-1.example.com`

2. 使用群集中每个主机的IP地址和完全限定的域名（fqdn），编辑/etc/hosts。也可以添加未限定的名称。
```
1.1.1.1 foo-1.example.com foo-1
2.2.2.2 foo-2.example.com foo-2
3.3.3.3 foo-3.example.com foo-3
4.4.4.4 foo-4.example.com foo-4
```
*重要：*
* /etc/hosts中每个主机的规范名称必须是fqdn（例如myhost-1.example.com），而不是非限定主机名（例如myhost-1）。规范名称是IP地址之后的第一个条目。
* 不要在/etc/hosts或配置DNS中使用别名。
* 非限定主机名（简称）在Cloudera Manager实例中必须是唯一的。例如，不能同时由同一个Cloudera Manager服务器管理host01.example.com和host01.standby.example.com。

3. 编辑/etc/sysconfig/network，仅此主机的fqdn：

`hostname=foo-1.example.com`

4. 验证每个主机是否一致地标识到网络：
  * 运行uname-a并检查主机名是否与hostname命令的输出匹配。
  * 运行/sbin/ifconfig并注意eth0（或bond0）条目中inet addr的值，例如：
    ```
    eth0-link-encap:ethernet-hwaddr 00:0c:29:a4:e8:97
    inet地址：172.29.82.176 bcast:172.29.87.255 mask:255.255.248.0
    …
    ```
  * 运行host-v-t a$（hostname）并验证输出是否与hostname命令匹配。IP地址应与ifconfig为eth0（或bond0）报告的地址相同：
    ```
    Trying “foo-1.example.com”
    …
    ；；ANSWER SECTION：
   foo-1.example.com.60 IN
   一
   172.27.82.176
   ```


 
  
