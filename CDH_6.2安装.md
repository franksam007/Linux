# CDH 6.2在Linux上的安装

（https://www.cloudera.com/documentation/enterprise/6/latest/topics/cm_ig_reqs_space.html）

## 1. 安装准备
### 1.1. 空间规划
#### 1.1.1. Cloudera Manager Server
* 默认：/var/lib/cloudera-scm-server/
* 空间：主要是所用数据库的空间，取决于所管理的节点数以及多长时间删除已运行命令
* 配置：Administration > Settings：Command Eviction Age，默认2年

#### 1.1.2. Cloudera Management Service
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

#### 1.1.3. Cloudera Navigator
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
  
#### 1.1.4. Cluster Lifecycle Management with Cloudera Manager
##### Parcel Lifecycle Management
* 下面目录需要足够空间：
  * Local Parcel Repository Path: /opt/cloudera/parcel-repo
  * Parcel Cache: /opt/cloudera/parcel-cache
  * Host Parcel Directory: /opt/cloudera/parcels

##### Space Reclamation Tasks
* Activity Monitor (One-time)：活动监视器只对MR任务，如果所有任务迁移到Yarn，活动监视数据库将停止增长，等待一定时间后（根据保留时间），可以清空该数据库
* Service Monitor and Host Monitor (One-time)：如从4.x升级到5.x，原数据库迁移到专用时序数据库
* Ongoing Space Reclamation：Cloudera Management Services会自动在后台汇总、清除或以其他方式整合过时的数据。

### 1.2. 配置网络名称
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

### 1.3. 停止防火墙

要在集群中的每个主机上禁用防火墙，请在每个主机上执行以下步骤。

* 对于iptables，保存现有规则集：
```
sudo iptables save>~/firewall.rules
```

* 禁用防火墙：
  * RHEL 7兼容：
    ```
    sudo systemctl disable firewalld
    sudo systemctl stop firewalld（停止防火墙）
    ```
  * SLES：
    ```
    sudo chkconfig SuSEfirewall2_setup off
    sudo chkconfig SuSEfirewall2_init off
    sudo rcSuSEfirewall2 stop
    ```
   * Ubuntu：
     ```
     sudo service ufw stop
     ```
### 1.4. 设置SELinux
安全增强型Linux（SELinux）允许通过策略设置访问控制。如果在使用策略部署CDH时遇到问题，请在集群上部署CDH之前，在每个主机上将SELinux设置为许可模式。

要设置SELinux模式，请在每个主机上执行以下步骤。

1. 检查SELinux状态：

`getenforce`

2. 如果输出是Permissive或Disabled，则可以跳过此任务并继续禁用防火墙。如果正在强制执行输出，请继续执行下一步。

3. 打开/etc/selinux/config文件（在某些系统中，是/etc/sysconfig/selinux文件）。

4. 将行selinux=enforcing更改为selinux=permitive。

5. 保存并关闭文件。

6. 重新启动系统或运行以下命令立即禁用SELinux：

`setenforce 0`

7. 在安装和部署了cdh之后，可以通过将/etc/selinux/config (或/etc/sysconfig/selinux)中SELINUX=permissive改回SELINUX=enforcing，重新启用selinux，然后运行以下命令立即切换到强制模式：

`setenforce 1`

 ### 1.5. 启动NTP服务
CDH要求您在集群中的每台计算机上配置网络时间协议（NTP）服务。大多数操作系统都包括用于时间同步的ntpd服务。


RHEL7兼容的操作系统默认使用chronyd而不是ntpd。如果chronid正在运行（在任何操作系统上），ClouderaManager将使用它来确定主机时钟是否同步。否则，Cloudera Manager使用ntpd。

*注意*：如果使用ntpd来同步主机时钟，但chronyd也在运行，则Cloudera Manager依赖chronyd来验证时间同步，即使时间同步不正确。这可能会导致Cloudera Manager报告时钟偏移错误，即使时间是正确的。要解决此问题，请配置并使用chronid，或者禁用它并将其从主机中删除。


要使用ntpd进行时间同步：

1. 安装NTP包：
  * RHEL兼容：
  
    `yum install ntp`
    
  * SLES：
  
    `zypper install ntp`
    
  * Ubuntu：
  
    `apt-get install ntp`

2. 编辑/etc/ntp.conf文件以添加ntp服务器，如下例所示。
  ```
  server 0.pool.ntp.org
  server 1.pool.ntp.org
  server 2.pool.ntp.org
  ```
3. 启动ntpd服务：
  * RHEL 7兼容:
  
    `sudo systemctl start ntpd`
    
  * 兼容RHEL 6，SLES，Ubuntu：
  
    `sudo service ntpd start`

4. 将ntpd服务配置为在启动时运行：
  * RHEL 7兼容：
  
    `sudo systemctl enable ntpd`
    
  * 兼容RHEL 6，SLES，Ubuntu：
  
    `chkconfig ntpd on`

5. 将系统时钟同步到NTP服务器：

  `ntpdate-u <ntp_server>`

6. 将硬件时钟与系统时钟同步：

  `hwclock --systohc`
 
### 1.6 Hue主机上安装Pytohn 2.7
CDH6中的Hue需要python 2.7，默认情况下，它包含在RHEL7兼容操作系统（OSES）中。

RHEL6兼容的操作系统包括python 2.6。在安装或升级到Cloudera Enterprise 6之前，必须在所有Hue主机上安装python 2.7：

#### RHEL 6
1. 确保可以访问软件集合库。有关详细信息，请参阅Red Hat知识库文章，如何使用Red Hat软件集合（rhscl）或Red Hat开发人员工具集（dts）？.

2. 安装python 2.7：

  `sudo yum install python27`

3. 验证是否安装了python 2.7：
  ```
  source /opt/rh/python27/enable
  python --version
  ```


#### CentOS 6

1. 启用软件集合库：
  
  `sudo yum install centos-release-scl`

2. 安装软件集合实用程序：

  `sudo yum install scl-utils`


3. 安装python 2.7：

  `sudo yum install python27`

4. 验证是否安装了python 2.7：

  ```
  source /opt/rh/python27/enable
  python --version
  ```

#### Oracle Linux 6


1. 下载软件集合库存储库：

  `sudo wget -o/etc/yum.repos.d/public-yum-ol6.repo http://yum.oracle.com/public-yum-ol6.repo`

2. 编辑/etc/yum.repos.d/public-yum-ol6.repo并确保已启用设置为1，如下所示：
  ```
  [ol6_software_collections]
  name=Software Collection Library release 3.0 packages for Oracle Linux 6 (x86_64)
  baseurl=http://yum.oracle.com/repo/OracleLinux/OL6/SoftwareCollections/x86_64/
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
  gpgcheck=1
  enabled=1
  ```

3. 安装软件集合实用程序：

  `sudo yum install scl-utils`

4. 安装python 2.7：

  `sudo yum install python27`

5. 验证是否安装了python 2.7：
  ```
  source /opt/rh/python27/enable
  python --version
  ```
#### Ubuntu 18.04
1、安装不同版本

  ```
  sudo apt install python
  sudo apt install python3
  ```

2. 注册系统配置
  ```
  # update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
  update-alternatives: using /usr/bin/python2.7 to provide /usr/bin/python (python) in auto mode
  # update-alternatives --install /usr/bin/python python /usr/bin/python3.6 2
  update-alternatives: using /usr/bin/python3.4 to provide /usr/bin/python (python) in auto mode
  ```

3. 列出可用的 Python 替代版本

  ```
  # update-alternatives --list python
  /usr/bin/python2.7
  /usr/bin/python3.6
  ```
  
4. 配置默认版本
  ```
  # update-alternatives --config python
  ```

### 1.7. Impala要求


### 1.8. 权限
需要root（通过密码或SSH Key）或无密码sudo 

### 1.9. 端口
略


