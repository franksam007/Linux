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

### 1.10 自定义安装

#### 建立本地软件库(Repo)

##### 包管理工具


包（RPM或DEB文件）有助于通过满足包依赖性来确保安装成功完成。安装特定软件包时，所有其他必需的软件包都会同时安装。例如，hadoop-0.20-hive依赖于hadoop-0.20。


包管理工具，如yum（rhel）、zypper（sles）和apt-get（ubuntu）是可以查找和安装所需包的工具。例如，在与RHEL兼容的系统上，可以运行命令yum install hadoop-0.20-hive。yum实用程序通知您Hive软件包需要Hadoop-0.20，并提供安装服务。Zypper和APT GET提供类似的功能。

##### 包存储库


包管理工具依赖包存储库来安装软件并解决任何依赖性需求。

###### 存储库配置文件

关于包存储库的信息存储在配置文件中，配置文件的位置根据包管理工具的不同而不同。

* 兼容RHEL（yum）：/etc/yum.repos.d

* SLES（zypper）：/etc/zypp/zypper.conf

* Ubuntu（apt-get）：/etc/apt/apt.conf（使用/etc/apt/sources.list.d/目录中的.list文件指定其他存储库。）

例如，在典型的CentOS系统上，您可能会发现：
```
ls -l /etc/yum.repos.d/
total 36
-rw-r--r--. 1 root root 1664 Dec  9  2015 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Dec  9  2015 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Dec  9  2015 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  290 Dec  9  2015 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Dec  9  2015 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Dec  9  2015 CentOS-Sources.repo
-rw-r--r--. 1 root root 1952 Dec  9  2015 CentOS-Vault.repo
-rw-r--r--. 1 root root  951 Jun 24  2017 epel.repo
-rw-r--r--. 1 root root 1050 Jun 24  2017 epel-testing.repo
```

.repo文件包含指向一个或多个存储库的指针。Zypper和APT GET的配置文件中有类似的指针。在下面的centos-base.repo摘录中，定义了两个存储库：一个命名为base，另一个命名为updates。mirrorlist参数指向一个网站，该网站具有可下载此存储库的位置列表。

```
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

###### 列出存储库

可以通过运行以下命令之一列出已启用的存储库：

* RHEL兼容：yum repolist

* SLES:zypper repos

* Ubuntu:apt-get不包含显示源的命令，但可以通过查看/etc/apt/sources.list和/etc/apt/sources.list.d/中包含的任何文件来确定源。



以下是CentOS 7系统上yum repolist的输出示例：
```
repo id               repo name                                           status
base/7/x86_64         CentOS-7 - Base                                      9,591
epel/x86_64           Extra Packages for Enterprise Linux 7 - x86_64      12,382
extras/7/x86_64       CentOS-7 - Extras                                      392
updates/7/x86_64      CentOS-7 - Updates                                   1,962
repolist: 24,327
```
##### 配置本地Parcel库
###### 设置Web服务器
要承载内部存储库，必须在Cloudera Manager主机可访问的内部主机上安装或使用现有的Web服务器，然后将存储库文件下载到Web服务器主机。本页上的示例使用ApacheHTTP服务器作为Web服务器。如果组织中已有Web服务器，则可以跳到下载和发布包裹存储库。

1. 安装Apache HTTP服务器：
  `sudo apt-get install httpd`

2. 警告：跳过此步骤可能会导致在尝试从本地存储库下载包时出现错误消息哈希验证失败，特别是在Cloudera Manager 6及更高版本中。

编辑Apache HTTP服务器配置文件（/etc/httpd/conf/httpd.conf，默认情况下）以添加或编辑<IfModule mime_module>部分中的以下行：

`AddType application/x-gzip .gz .tgz .parcel`


如果<IfModule mime_module>部分不存在，可以按如下方式将其全部添加：

注意：此示例配置是在RHEL7上安装ApacheHTTP服务器之后根据提供的默认配置进行修改的。

```
<IfModule mime_module>
    #
    # TypesConfig points to the file containing the list of mappings from
    # filename extension to MIME-type.
    #
    TypesConfig /etc/mime.types

    #
    # AddType allows you to add to or override the MIME configuration
    # file specified in TypesConfig for specific file types.
    #
    #AddType application/x-gzip .tgz
    #
    # AddEncoding allows you to have certain browsers uncompress
    # information on the fly. Note: Not all browsers support this.
    #
    #AddEncoding x-compress .Z
    #AddEncoding x-gzip .gz .tgz
    #
    # If the AddEncoding directives above are commented-out, then you
    # probably should define those extensions to indicate media types:
    #
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz .parcel

    #
    # AddHandler allows you to map certain file extensions to "handlers":
    # actions unrelated to filetype. These can be either built into the server
    # or added with the Action directive (see below)
    #
    # To use CGI scripts outside of ScriptAliased directories:
    # (You will also need to add "ExecCGI" to the "Options" directive.)
    #
    #AddHandler cgi-script .cgi

    # For type maps (negotiated resources):
    #AddHandler type-map var

    #
    # Filters allow you to process content before it is sent to the client.
    #
    # To parse .shtml files for server-side includes (SSI):
    # (You will also need to add "Includes" to the "Options" directive.)
    #
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>
```

3. 启动Apache HTTP服务器：

`sudo systemctl start apache2`


4. 下载和发布Package存储库（Parcels和Package)

========================以下为Package=========================================
下载要安装的产品的软件包存储库：

* Cloudera Manager 6
要下载最新ClouderaManager 6.2版本的文件，请在Web服务器主机上运行以下命令。
```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cm6/6.2.0/ubuntu1804/ -P /var/www/html/cloudera-repos
sudo wget https://archive.cloudera.com/cm6/6.2.0/allkeys.asc -P /var/www/html/cloudera-repos/cm6/6.2.0/
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cm6
sudo apt-key add /var/www/html/cloudera-repos/cm6/6.2.0/ubuntu1804/apt/archive.key
```

如果要为不同的ClouderaManager6版本创建存储库，请将6.2.0替换为所需的CDH 6版本。有关更多信息，请参阅Cloudera Manager 6版本并下载信息（https://www.cloudera.com/documentation/enterprise/6/latest/topics/rg_cm_6_version_download.html#cm_6_version_download）。

* CDH 6
要下载最新CDH 6.2版本的文件，请在Web服务器主机上运行以下命令。
```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cdh6/6.2.0/ubuntu1804/ -P /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/gplextras6/6.2.0/ubuntu1804/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cdh6
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/gplextras6
sudo apt-key add /var/www/html/cloudera-repos/cdh6/6.2.0/ubuntu1804/apt/archive.key
```

如果要为不同的CDH 6版本创建存储库，请将6.2.0替换为所需的CDH 6版本。有关更多信息，请参阅CDH 6下载信息(https://www.cloudera.com/documentation/enterprise/6/latest/topics/rg_cdh_6_download.html#cdh_download_info)。

* Cloudera Manager 5
要下载ClouderaManager版本的文件，请下载操作系统的存储库tarball。
```
sudo mkdir -p /var/www/html/cloudera-repos/cm5
wget https://archive.cloudera.com/cm5/repo-as-tarball/5.14.4/cm5.14.4-ubuntu16-04.tar.gz
tar xvfz cm5.14.4-ubuntu16-04.tar.gz -C /var/www/html/cloudera-repos/cm5 --strip-components=1
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cm5
```

如果要为其他Cloudera Manager版本或操作系统创建存储库，请从repo-as-tarball父目录(https://archive.cloudera.com/cm5/repo-as-tarball/)开始，选择要使用的Cloudera Manager版本，然后复制操作系统的.tar.gz链接。

* CDH 5
要下载CDH版本的文件，请下载操作系统的存储库tarball。
```
sudo mkdir -p /var/www/html/cloudera-repos/cdh5
wget https://archive.cloudera.com/cdh5/repo-as-tarball/5.14.4/cdh5.14.4-ubuntu16-04.tar.gz
tar xvfz cdh5.14.4-ubuntu16-04.tar.gz -C /var/www/html/cloudera-repos/cdh5 --strip-components=1
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cdh5
```

如果要为不同的CDH版本或操作系统创建存储库，请从repo-as-tarball父目录(https://archive.cloudera.com/cdh5/repo-as-tarball/)开始，选择要使用的CDH版本，然后复制操作系统的.tar.gz链接。

* 用于CDH的Apache Accumulo
要下载Accumulo发行版的CDH文件，请在Web服务器主机上运行以下命令。
```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/accumulo-c5/ubuntu/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/accumulo-c5
```

* Cloudera Navigator Key Trustee Server
转到“密钥受信者服务器”下载页(http://www.cloudera.com/content/www/en-us/downloads/navigator/key-trustee-server.html)。从“选择下载类型”下拉菜单中选择“软件包”，从“选择操作系统”下拉菜单中选择您的操作系统，然后单击“立即下载”。这将在.tar.gz文件中下载密钥受信者服务器包文件。将文件复制到Web服务器，并使用tar xvfz filename.tar.gz命令提取文件。此示例使用密钥受信者服务器5.14.0：
```
sudo mkdir -p /var/www/html/cloudera-repos/keytrustee-server
sudo tar xvfz /path/to/keytrustee-server-5.14.0-parcels.tar.gz -C /var/www/html/cloudera-repos/keytrustee-server --strip-components=1
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/keytrustee-server
```

* Cloudera Navigator Key Trustee KMS and HSM KMS

注意：Cloudera Navigator HSM KMS包含在密钥受信者KMS包中。

转到密钥受信者KMS下载页(http://www.cloudera.com/content/www/en-us/downloads/navigator/key-trustee-kms.html)。从“选择下载类型”下拉菜单中选择“软件包”，从“操作系统”下拉菜单中选择您的操作系统，然后单击“立即下载”。这将下载.tar.gz文件中的密钥受信者KMS包文件。将文件复制到Web服务器，并使用tar xvfz filename.tar.gz命令提取文件。此示例使用密钥受信者KMS 5.14.0：
```
sudo mkdir -p /var/www/html/cloudera-repos/keytrustee-kms
sudo tar xvfz /path/to/keytrustee-kms-5.14.0-parcels.tar.gz -C /var/www/html/cloudera-repos/keytrustee-kms --strip-components=1
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/keytrustee-kms
```

在浏览器中访问存储库URL http://web_server>/cloudera_repos/，并验证您下载的文件是否存在。如果看不到任何内容，则Web服务器可能已配置为不显示索引。

========================以下为Parcel===========================================

下载manifest.json和要安装的产品的包文件：

* CDH 6(Parcels)

apache impala、apache kudu、apache spark 2和cloudera搜索包含在cdh包中。要下载最新CDH 6.2版本的文件，在Web服务器主机上运行以下命令：

```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cdh6/6.2.0/parcels/ -P /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/gplextras6/6.2.0/parcels/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cdh6
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/gplextras6
```

如果要为不同的CDH 6版本创建存储库，请将6.2.0替换为所需的CDH 6版本。有关更多信息，请参阅CDH 6下载信息(https://www.cloudera.com/documentation/enterprise/6/latest/topics/rg_cdh_6_download.html#cdh_download_info)。


* CM 6
在Web服务器主机上运行以下命令：
```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cm6/6.2.0/ubuntu1804/apt/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cm6
```
如果要为不同的CM 6版本创建存储库，请将6.2.0替换为所需的CDH 6版本。有关更多信息，请参阅CM 6下载信息(https://www.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_cm_6_version_download.html#cm_6_version_download)。

* CDH 5

Impala, Kudu, Spark 1, 和Search都包含在CDH包裹中。要下载CDH版本的文件（本例中为CDH 5.14.4），请在Web服务器主机上运行以下命令：

```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cdh5/parcels/5.14.4/ -P /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/gplextras5/parcels/5.14.4/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cdh5
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/gplextras5
```

如果要为不同的CDH版本创建存储库，请将5.14.4替换为所需的CDH版本。有关更多信息，请参阅CDH下载信息(https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh_download.html)。

* 用于CDH的Apache Accumulo

要下载用于CDH的AccuMulo版本的文件（本例中为AccuMulo 1.7.2），请在Web服务器主机上运行以下命令：
```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/accumulo-c5/parcels/1.7.2/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/accumulo-c5
```

如果要为Accumulo 1.6.0创建存储库，请将1.7.2替换为1.6.0。

* 用于CDH的由Apache Spark 2驱动的CDS

要下载CDS for CDH发行版的文件（本例中为CDS 2.3.0.Cloudera3），请在Web服务器主机上运行以下命令：

```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/spark2/parcels/2.3.0.cloudera3/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/spark2
```


如果要为不同的CDS版本创建存储库，请将2.3.0.cloudera3替换为所需的CD版本。有关更多信息，请参阅由ApacheShark版本信息提供支持的CDS(https://www.cloudera.com/documentation/spark2/latest/topics/spark2_packaging.html#versions)。

* Cloudera Navigator Key Trustee Server

转到“密钥受信者服务器”下载页（http://www.cloudera.com/content/www/en-us/downloads/navigator/key-trustee-server.html）。从“选择下载类型”下拉菜单中选择“包裹”，然后单击“立即下载”。这将在.tar.gz文件中下载密钥受信者服务器包和manifest.json文件。将文件复制到Web服务器，并使用tar xvfz filename.tar.gz命令提取文件。此示例使用密钥受信者服务器5.14.0：

```
sudo mkdir -p /var/www/html/cloudera-repos/keytrustee-server
sudo tar xvfz /path/to/keytrustee-server-5.14.0-parcels.tar.gz -C /var/www/html/cloudera-repos/keytrustee-server --strip-components=1
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/keytrustee-server
```

* Cloudera Navigator Key Trustee KMS and HSM KMS

*注意*：Cloudera Navigator HSM KMS包含在密钥受信者KMS包中。

转到密钥受信者KMS下载页(http://www.cloudera.com/content/www/en-us/downloads/navigator/key-trustee-kms.html)。从“选择下载类型”下拉菜单中选择“包裹”，然后单击“立即下载”。这将在.tar.gz文件中下载密钥受信者KMS包和manifest.json文件。将文件复制到Web服务器，并使用tar xvfz filename.tar.gz命令提取文件。此示例使用密钥受信者KMS 5.14.0：
```
sudo mkdir -p /var/www/html/cloudera-repos/keytrustee-kms
sudo tar xvfz /path/to/keytrustee-kms-5.14.0-parcels.tar.gz -C /var/www/html/cloudera-repos/keytrustee-kms --strip-components=1
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/keytrustee-kms
```

* Sqoop连接器

要下载sqoop连接器版本的包，请在Web服务器主机上运行以下命令。此示例使用最新的可用sqoop连接器：
```
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories http://archive.cloudera.com/sqoop-connectors/parcels/latest/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/sqoop-connectors
```

如果要为不同的sqoop connector版本创建存储库，请将最新版本替换为所需的sqoop connector版本。您可以在Parcels父目录中看到版本列表。

在浏览器中访问存储库URL http://<Web_server>/cloudera-repos/，并验证您下载的文件是否存在。如果看不到任何内容，则Web服务器可能已配置为不显示索引。

5. 配置主机以使用内部存储库
建立存储库后，修改客户机配置以使用它：

* rhel兼容

在群集主机上创建/etc/yum.repos.d/cloudera-repo.repo文件，内容如下，<web_server>是Web服务器的主机名：
```
[cloudera-repo]
name=cloudera-repo
baseurl=http://<web_server>/cm/5
enabled=1
gpgcheck=0 
```

* SLES
使用zypper实用程序通过发出以下命令来更新客户机系统存储库信息：
zypper addrepo http://<web_server>/cm <alias>

* ubuntu
在所有集群主机上创建/etc/apt/sources.list.d/cloudera-repo.list文件，其中<web_server>是web服务器的主机名：
`deb [arch=amd64] http://localh/cm6/6.2.0/ubuntu1804/apt <codename> <components>`

可以在存储库的./conf/distributions文件中找到<codename>和<components>变量。

例如：`deb [arch=amd64] http://localh/cm6/6.2.0/ubuntu1804/apt bionic-cm6.2.0 contrib`

创建.list文件后，运行以下命令：
 
`sudo apt-get update`

6. 配置Cloudera Manager以使用内部远程包存储库

  1. 使用以下方法之一打开“包裹设置”页面：

    * Navigation bar

      * 单击顶部导航栏中的Parcels图标，或单击Hosts，然后单击Parcels选项卡。

      * 单击Menu按钮。

   * Menu

     * 选择Administration > Settings.。

     * 选择 Category > Parcels。

  2. 在“Remote Parcel Repository URLs”列表中，单击“添加”符号以打开另一行。

  3. 输入包的路径。例如：http://<web_server>/cloudera packages/cdh6/6.2.0/

  4. 输入更改原因，然后单击“Save Changes”以提交更改。

## 2. 安装CM和CDH 6.2

### 2.1. 配置Repo
Cloudera Manager是使用包管理工具安装的，例如用于RHEL兼容系统的yum、用于SLES的zypper和用于Ubuntu的apt-get。这些工具依赖于对存储库的访问来安装软件。Cloudera为CDH和Cloudera Manager安装文件维护可访问互联网的存储库。还可以为没有Internet访问权限的主机创建自己的内部存储库。

要使用Cloudera存储库，请执行以下操作：
* RHEL兼容

1. 将操作系统版本的cloudera-manager.repo文件下载到cloudera manager服务器主机上的/etc/yum.repo.d/目录。

  可以在Cloudera Manager 6版本的repo文件列中找到URL，并下载要安装的Cloudera Manager版本的信息表。

  例如：

`sudo wget<repo_file_url>-p/etc/yum.repos.d/`

2. 导入存储库签名GPG密钥：
  * RHEL 7兼容：
  
    `sudo rpm --import https://archive.cloudera.com/cm6/6.2.0/redhat7/yum/RPM-GPG-KEY-cloudera`

  * RHEL 6兼容：

    `sudo rpm --import https://archive.cloudera.com/cm6/6.2.0/redhat6/yum/RPM-GPG-KEY-cloudera`

3. 继续步骤2：安装Java开发工具包。

* SLES

1. 通过运行以下命令更新系统包索引：

  `sudo zypper refresh`
  
2. 使用zyper add repo添加repo。

  可以在Cloudera Manager 6版本的repo文件列中找到URL，并下载要安装的Cloudera Manager版本的信息表。

  例如：

  `sudo zypper addrepo -f https://archive.cloudera.com/cm6/6.2.0/sles12/yum/cloudera-manager.repo`

3. 导入存储库签名GPG密钥：

  `sudo rpm --import https://archive.cloudera.com/cm6/6.2.0/sles12/yum/RPM-GPG-KEY-cloudera`

4. 继续步骤2：安装Java开发工具包。

* Ubuntu

1.将操作系统版本的cloudera.list文件下载到cloudera manager服务器主机上的/etc/apt/sources.list.d/目录。
  
  可以在Cloudera Manager 6版本的repo文件列中找到URL，并下载要安装的Cloudera Manager版本的信息表。
  
2. 导入存储库签名GPG密钥：

  ```
  wget https://archive.cloudera.com/cm6/6.2.0/ubuntu1604/apt/archive.key
  sudo apt-key add archive.key
  ```
  
3. 通过运行以下命令更新系统包索引：

  `sudo apt-get update`
  
4. 继续步骤2：安装Java开发工具包。

### 2.2 安装JDK

对于JDK，可以Cloudera使用Cloudera Manager提供JDK版本,直接来自Oracle的Oracle JDK，OpenJDK。Cloudera支持的大多数Linux发行版都包括OpenJDK，但如果需要，可手动安装。

OpenJDK支持Cloudera Enterprise 6.1.0及更高版本，以及Cloudera Enterprise 5.16.0及更高版本。
继续阅读：

#### 2.2.1. 要求
* JDK必须是64位。不要使用32位JDK。
* 已安装的JDK必须是支持的版本，如Java要求中所记载的那样。
* 每个集群主机上必须安装相同版本的JDK。
* JDK必须安装在/Ur/Java/JDK版本。

*注意*：

    如果JDK安装在不同的位置，在安装cloudera manager之前设置java_home环境变量。如果无法在环境中设置java_home，请使用cloudera manager serer主机上的/etc/cloudera-pre-install/CLOUDERA_SKIP_JAVA_INSTALL_CHECK路径创建一个空文件。这将导致安装过程在安装Cloudera管理器服务器和后台程序包时跳过任何Java检查。

*重要*：

    * Cloudera Enterprise 6支持的RHEL兼容和Ubuntu操作系统默认情况下都对票据使用AES-256加密。为了支持在低于1.8U161的JDK版本中的AES-256位加密，必须在所有集群和Hadoop用户机上安装Java加密扩展（JCE）无限强度管辖权策略文件。Cloudera Manager可以自动安装策略文件，也可以手动安装。有关jce策略文件安装说明，请参阅jce_policy-x.zip文件中包含的readme.txt文件。JDK 1.8U161及更高版本默认情况下启用无限制强度加密，不需要策略文件。
    
    * 在SLE平台上，不要安装或尝试使用与SLE发行捆绑的IBM Java版本。使用该版本时，CDH无法正确运行。

#### 2.2.2. 使用Cloudera Manager安装Oracle JDK

完成步骤1:为Cloudera Manager配置存储库后，可以使用包管理器在Cloudera Manager服务器主机上安装Oracle JDK，如下所示：

* RHEL兼容

  `sudo yum install oracle-j2sdk1.8`
  
* SLES

  `sudo zypper install oracle-j2sdk1.8`
  
* Ubuntu

  `sudo apt-get install oracle-j2sdk1.8`

在接下来的步骤中，您可以使用ClouderaManager在其余集群主机上安装JDK。继续步骤3：安装Cloudera Manager服务器。

#### 2.2.3 手动安装Oracle JDK
OracleJDK安装程序既可以作为基于RPM的系统的基于RPM的安装程序，也可以作为.tar.gz文件提供。这些说明用于.tar.gz文件。

1. 下载来自JavaSE 8下载的Oracle JDK的64位支持版本之一的.TAR.Gz文件。（此链接在写入时是正确的，但可以更改。）

  *注意* ：如果要使用wget等实用程序直接下载JDK，则必须通过配置头文件来接受Oracle许可证，这些头文件经常更新。博客文章和问答网站可以很好地提供有关如何使用wget下载特定JDK版本的信息。

2. 提取JDK到/UR/Java/JDK版本。例如：

  `tar xvfz /path/to/jdk-8u<update_version>-linux-x64.tar.gz -C /usr/java/`
  
3. 在所有群集主机上重复此过程。完成后，继续执行步骤3:安装Cloudera Manager服务器。

#### 2.2.4 手动安装OpenJDK
在安装Cloudera Manager和CDH之前，请执行本节中的步骤，在集群中的所有主机上安装OpenJDK。

安装Cloudera Enterprise时，Cloudera Manager提供了安装Oracle JDK的选项。取消选择此选项。

有关Cloudera企业版支持哪些JDK版本的信息，请参阅支持的JDK。

注意：如果要启用自动TLS，请注意以下事项：
您可以指定一个PEM文件，该文件包含要导入到自动TLS信任库中的受信任CA证书。如果要使用OpenJDK附带的Cacerts信任库中的证书，必须首先将信任库转换为PEM格式。但是，OpenJDK附带了一些中间证书，这些证书无法导入到Auto-TLS信任库中。在将PEM文件导入auto-tls信任库之前，必须从PEM文件中删除这些证书。从au所在的集群升级到openjdk时不需要这样做。

* RHEL

  `su -c yum install java-1.8.0-openjdk-devel`

* Ubuntu

  `sudo apt-get install openjdk-8-jdk`

* SLES

  `sudo zypper install java-1_8_0-openjdk-devel`

## 2.3. Cloudera Manager Server
### 2.3.1. 安装Cloudera Manager包
在Cloudera Manager服务器主机上，键入以下命令以安装Cloudera Manager包。

* RHEL、CentOS、Oracle Linux

  `sudo yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server`

* SLES

  `sudo zypper install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server`

* Ubuntu

  `sudo apt-get install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server`

2. 如果Oracle数据库用于Cloudera Manager服务器的，在Cloudera Manager服务器主机上编辑/etc/default/cloudera-scm-server文件。找到以export CMF_JAVA_OPTS开头的行，并将-Xmx2G选项更改为-Xmx4G。

## 2.4. 安装配置数据库
Cloudera Manager使用各种数据库和数据存储来存储有关Cloudera Manager配置的信息，以及诸如系统运行状况或任务进度等信息。

尽管可以在一个环境中部署不同类型的数据库，但这样做可能会造成意想不到的复杂情况。建议为所有Cloudera数据库选择一个受支持的数据库提供程序。

Cloudera建议将数据库安装在与服务不同的主机上。将数据库与服务分离有助于将一个或另一个数据库中的故障或资源争用的潜在影响隔离开来。它还可以简化拥有专用数据库管理员的组织中的管理。

可以为Cloudera Manager服务器和其他使用数据库的服务使用自己的PostgreSQL、Mariadb、MySQL或Oracle数据库。有关规划、管理和备份Cloudera Manager数据存储的信息，请参阅针对Cloudera Manager的存储空间规划和备份数据库。

#### 2.4.1. 所需数据库
以下组件都需要数据库：cloudera manager server、oozie server、sqoop server、activity monitor、reports manager、hive metastore server、hue server、sentry server、cloudera navigator audit server和cloudera navigator metadata server。数据库中包含的数据类型及其相对大小如下：

* cloudera manager server-包含有关已配置的服务及其角色分配、所有配置历史记录、命令、用户和正在运行的进程的所有信息。这个相对较小的数据库（<100 MB）是最重要的备份。
重要提示：当您重新启动进程时，每个服务的配置将使用保存在Cloudera Manager数据库中的信息重新部署。如果此信息不可用，则群集无法启动或正常工作。您必须计划和维护Cloudera Manager数据库的定期备份，以便在该数据库丢失时恢复集群。有关详细信息，请参阅备份数据库。

* oozie server-包含Oozie工作流、协调器和捆绑数据。可以长得很大。

* sqoop server-包含连接器、驱动程序、链接和作业等实体。相对较小。

* activity monitor-包含有关过去活动的信息。在大的集群中，这个数据库可以变大。只有在部署了MapReduce服务时，才需要配置活动监视器数据库。

* reports manager-跟踪磁盘利用率和随时间变化的处理活动。中等大小。

* hive metastore server-包含Hive元数据。相对较小。

* hue server-包含用户帐户信息、作业提交和配置单元查询。相对较小。

* sentry server-包含授权元数据。相对较小。

* cloudera navigator audit server-包含审核信息。在大的集群中，这个数据库可以变大。

* cloudera navigator metadata server-包含授权、策略和审计报告元数据。相对较小。

主机监视器和服务监视器服务使用基于本地磁盘的数据存储。有关更多信息，请参阅数据存储以获取监控数据。

数据库的JDBC连接器必须安装在分配活动监视器和报表管理器角色的主机上。

#### 2.4.2. 安装和配置数据库(Mysql)

##### 2.4.2.1 安装数据库
* RHEL 	

mysql不再包含在rhel中。您必须从MySQL站点下载存储库并直接安装它。可以使用以下命令安装MySQL。有关更多信息，请访问MySQL网站。

```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm

sudo yum update

sudo yum install mysql-server

sudo systemctl start mysqld
```

* SLES 	

`sudo zypper install mysql libmysqlclient_r17`

注意：某些SLES系统在使用前面的zyper安装命令时会遇到错误。有关解决此问题的详细信息，请参阅Novell知识库主题“error running chkconfig.”( http://www.novell.com/support/kb/doc.php?id=7010013 )。

* Ubuntu 	

`sudo apt-get install mysql-server`

##### 2.4.2.2 配置起停数据库
1. 如果mysql服务器正在运行，停止它。

* RHEL 7兼容

`sudo systemctl stop mysqld`

* RHEL 6兼容

`sudo service mysqld stop`

* SLES, Ubuntu

`sudo service mysql stop`

2. 将旧的innodb日志文件/var/lib/mysql/ib_logfile0和/var/lib/mysql/ib_logfile1移出/var/lib/mysql/到备份位置。

3. 确定选项文件my.cnf的位置（默认为/etc/my.cnf，Ubuntu 18.04上在/etc/mysql/）。

  更新my.cnf，使其符合以下要求：
  * 要防止死锁，请将隔离级别设置为READ-COMMITTED。
  * 配置InnoDB引擎。如果表配置了myisam引擎，则ClouderaManager将不会启动。（通常，如果innodb引擎配置错误，表将恢复为myisam。）要检查表使用的引擎，请从mysql shell运行以下命令：
    `mysql> show table status;`
  * 大多数发行版中mysql安装的默认设置使用保守的缓冲区大小和内存使用率。Cloudera管理服务角色需要较高的写吞吐量，因为它们可能会在数据库中插入许多记录。cloudera建议您将innodb_flush_method属性设置为O_DIRECT.。
  * 根据集群的大小设置max_connections属性：
    * 少于50个主机-可以在同一主机上存储多个数据库（例如，活动监视器和服务监视器）。如果这样做，应该：
      * 将每个数据库放在自己的存储卷上。
      * 每个数据库最多允许100个连接，然后添加50个额外连接。例如，对于两个数据库，将最大连接数设置为250。如果在一台主机上存储五个数据库（Cloudera Manager服务器、活动监视器、报表管理器、Cloudera Navigator和Hive元存储的数据库），请将最大连接数设置为550。
    * 超过50个主机-不要在同一主机上存储多个数据库。为每个数据库/主机对使用单独的主机。主机不需要专门为数据库保留，但每个数据库都应该在单独的主机上。

  * 如果群集有1000多个主机，请将max_allowed_packet属性设置为16M。如果没有此设置，群集可能由于以下异常而无法启动：com.mysql.jdbc.packettoobigexception。

  * 对于Cloudera Manager安装，不需要二进制日志记录。二进制日志记录提供了诸如MySQL复制或数据库恢复后的时间点增量恢复等好处。下面是这种配置的示例。有关详细信息，请参阅二进制日志。

  以下是带有Cloudera推荐设置的选项文件：
  ```
  [mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0

key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MySQL, if you enable the binary log and do not set
#a server_id, MySQL will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

sql_mode=STRICT_ALL_TABLES
```

5. 如果Apparmor运行在安装了MySQL的主机上，则可能需要配置Apparmor以允许MySQL写入二进制文件。

6. 确保mysql服务器在启动时启动：

  * RHEL 7兼容

    `sudo systemctl enable mysqld`
    
  * RHEL 6兼容

    `sudo chkconfig mysqld on`
  
  * SLES
  
    `sudo chkconfig --add mysql`
  
  * Ubuntu
    
    `sudo chkconfig mysql on`
    
    注意：chkconfig可能在最近的Ubuntu版本上不可用。您可能需要使用upstart配置mysql在系统启动时自动启动。有关更多信息，请参阅Ubuntu文档或新贵食谱。

7. 启动mysql服务器：

  * RHEL 7兼容

    `sudo systemctl start mysqld`

  * 兼容RHEL 6

    `sudo service mysqld start`

  * SLES, Ubuntu

    `sudo service mysql start`
    
8. 运行/usr/bin/mysql_secure_installation来设置mysql根密码和其他与安全相关的设置。在新安装中，根密码为空。当系统提示您输入根密码时，请按Enter键。对于其余提示，请以粗体输入下面列出的响应：

  `sudo /usr/bin/mysql_secure_installation`
  ```
    [...]
    Enter current password for root (enter for none):
    OK, successfully used password, moving on...
    [...]
    Set root password? [Y/n] Y
    New password:
    Re-enter new password:
    Remove anonymous users? [Y/n] Y
    [...]
    Disallow root login remotely? [Y/n] N
    [...]
    Remove test database and access to it [Y/n] Y
    [...]
    Reload privilege tables now? [Y/n] Y
    All done!
  ```

9. 安装mysql jdbc驱动程序
  在Cloudera Manager服务器主机上安装JDBC驱动程序，以及运行需要数据库访问的服务的任何其他主机。有关使用数据库的Cloudera软件的更多信息，请参阅必需的数据库。

  注意：如果已经在需要的主机上安装了JDBC驱动程序，那么可以跳过这一节。但是，MySQL5.6需要5.1驱动程序版本5.1.26或更高版本。

  建议整合有限数量主机上需要数据库的所有角色，并在这些主机上安装驱动程序。建议在同一主机上定位所有此类角色，但不需要。确保在运行访问数据库角色的每个主机上安装JDBC驱动程序。

  注意：Cloudera建议仅使用JDBC驱动程序的5.1版本。

  * RHEL
    
    重要提示：在安装JDK之前，使用yum install命令安装mysql驱动程序包安装openjdk，然后使用linux alternations命令将系统jdk设置为openjdk。如果打算使用OracleJDK，请确保在使用yum install安装mysql驱动程序之前安装了它。
    
  或者，使用以下步骤手动安装驱动程序。
  
    1. 从http://www.mysql.com/downloads/connector/j/5.1.html（格式为.tar.gz）下载mysql jdbc驱动程序。截至撰写之时，可以使用wget下载5.1.46版，如下所示：
    
      `wget https://dev.mysql.com/get/downloads/connector-j/mysql-connector-java-5.1.46.tar.gz`
    
    2. 从下载的文件中提取JDBC驱动程序JAR文件。例如：
    
      `tar zxvf mysql-connector-java-5.1.46.tar.gz`
    
    3. 复制JDBC驱动程序，重命名为/usr/share/java/。如果目标目录尚不存在，请创建它。例如：
    
      ```
      sudo mkdir -p /usr/share/java/
      cd mysql-connector-java-5.1.46
      sudo cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
      ```

  * SLES
  
    `sudo zypper install mysql-connector-java`

  * Ubuntu或Debian
  
    `sudo apt-get install libmysql-java`

##### 2.4.2.3. 为Cloudera软件创建数据库
  为需要数据库的组件创建数据库和服务帐户：

  * Cloudera Manager Server
  * Cloudera Management Service roles:
    * Activity Monitor (if using the MapReduce service in a CDH 5 cluster)
    * Reports Manager
  * Hue
  * Each Hive metastore
  * Sentry Server
  * Cloudera Navigator Audit Server
  * Cloudera Navigator Metadata Server
  * Oozie

  1. 以根用户或其他具有创建数据库和授予权限的用户身份登录：
  
    ```
    mysql -u root -p
    Enter password:
    ```
    
  2. 使用以下命令为集群中部署的每个服务创建数据库。您可以为<database>、<user>和<password>参数使用所需的任何值。
  
  将所有数据库配置为使用utf8字符集。
  
  当运行下面描述的create database语句时，包括每个数据库的字符集。
  
  ```
  CREATE DATABASE <database> DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

  Query OK, 1 row affected (0.00 sec)

  GRANT ALL ON <database>.* TO '<user>'@'%' IDENTIFIED BY '<password>';

  Query OK, 0 rows affected (0.00 sec)
  ```
  
  具体语句可使用：
  
  ```
  CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';

  CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';

  CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';

  CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';

  CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive';

  CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';

  CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';

  CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';

  CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';

  FLUSH PRIVILEGES;
  ```

3. 确认已创建所有数据库：

  `SHOW DATABASES;`

通过运行以下命令来确认给定用户的特权授予：

  `SHOW GRANTS FOR '<user>'@'%';`

4. 记录为数据库名称、用户名和密码输入的值。Cloudera Manager安装向导需要这些信息才能正确连接到这些数据库。 

#### 2.4.3. 安装和配置数据库(Mariadb)

#### 2.4.4. 安装和配置数据库(PostgreSQL)

#### 2.4.5. 安装和配置数据库(Oracle)

#### 为sqoop 2配置外部数据库

### 2.5. 配置CM数据库

Cloudera Manager Server包含一个脚本，可以为自己创建和配置数据库。脚本可以：

* 创建Cloudera Manager Server数据库配置文件。

* （mariadb、mysql和postgresql）为cloudera manager服务器创建和配置一个要使用的数据库。

* （mariadb、mysql和postgresql）为ClouderaManager服务器创建和配置用户帐户。 

#### 2.5.1. scm_prepare_database.sh语法

  scm_prepare_database.sh语法如下：
  
  `sudo /opt/cloudera/cm/schema/scm_prepare_database.sh [options] <databaseType> <databaseName> <databaseUser> <password>`

  *注意* ：运行没有选项的scm_prepare_database.sh，可看到语法。
  
  在创建新数据库时，必须用参数-u和-p指定具有权限创建数据库的用户。已经在上一个步骤创建了数据库，则不用指定。
  
  * <databaseType>
    数据库类型的支持：
    * MariaDB：mysql
    * MySQL：mysql
    * Oracle：oracle
    * PostgreSQL：postgresql
  
  * <databaseName>
    要使用的Cloudera Manager服务器数据库的名称。对于mysql、mariadb和postgresql数据库，如果使用具有创建数据库和授予权限权限的用户的凭据指定-u和-p选项，则脚本可以创建指定的数据库。在Cloudera Manager配置设置中提供的默认数据库名称是scm，但可以使用其他名称。
  * <databaseUser>
    要创建或使用的Cloudera Manager服务器数据库的用户名。Cloudera Manager配置设置中提供的默认用户名是scm，但可以使用其他名称。

  * <password>
   创建<databaseUser>或使用的密码。如果不希望密码在屏幕上可见或存储在命令历史记录中，请不要指定密码，并提示以下方式输入密码：

   `Enter SCM password:`
   
   其他选项
   
   * -?|--help：显示帮助
   
   * --config-path：Cloudera Manager Server配置文件路径。默认为/etc/cloudera-scm-server
   
   * -f|--force：如果指定，错误发生时不停。
   
   * -h|--host：安装主机数据库的主机的IP地址或主机名。默认是使用localhost。
   
   * -p|--password：数据库管理员密码。与-u联合使用。默认是没有密码的。不要在p与秘密之间放置空格（例如，-phunter2）。如果不想在屏幕上可见的密码存储在命令或选项的历史，使用-p和没有指定密码，系统将提示：
   
     `Enter database password:`
     
     如果已经创建数据库，不使用此选项。
     
   * -P|--port：数据库端口号。MariaDB、MySQL的默认端口是3306，PostgreSQL的默认端口是5432，OracleL的默认端口是1521。该选项只用于远程连接。
   
   * --scm-host：安装Cloudera Manager Server的主机名。Cloudera Manager Server和数据库安装在同一主机上，不是用此选项和-h选项。
   
   * --scm-password-script：该运行脚本的输出作为scm用户的数据库密码。
   
   * -u|--user 	数据库的管理员用户，与-p一起使用。不要在-u和用户名之间放置空格（例如，-uroot)。如指定本选项，脚本将为Cloudera Manager Server建立用户和数据库。如已经建立数据库，不要使用本选项。
   
#### 2.5.2. 准备Cloudera Manager数据库
  1. 在Cloudera Manager Server主机运行脚本，使用在第四步：安装和配置数据库中的数据库名称，用户名和密码。
  
  `sudo /opt/cloudera/cm/schema/scm_prepare_database.sh <databaseType> <databaseName> <databaseUser>`

  如系统提示，输入密码。
  
  2. 如果存在，删除嵌入的Postgresql属性文件：
  
  `sudo rm /etc/cloudera-scm-server/db.mgmt.properties`

  下面的例子演示不同场景
  
##### 例1：MySQL或MariaDB与Cloudera Manager Server在同一台服务器上

  这个例子假定已经创建Cloudera Management Server的数据库用户，数据库、用户名、密码均为scm。
  
  `sudo /opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm`
  
  ```
  Enter SCM password:
  JAVA_HOME=/usr/java/jdk1.8.0_141-cloudera
  Verifying that we can write to /etc/cloudera-scm-server
  Creating SCM configuration file in /etc/cloudera-scm-server
  Executing:  /usr/java/jdk1.8.0_141-cloudera/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:/opt/cloudera/cm/schema/../lib/*  com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
  [main] DbCommandExecutor              INFO  Successfully connected to database.
  All done, your SCM database is configured correctly!
  ```
##### Example 2: Running the script when MySQL or MariaDB is installed on another host

  此示例演示如何在cloudera manager server主机（cm01.example.com）上运行脚本，并连接到远程mysql或mariadb主机（db01.example.com）。:

  `sudo /opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h db01.example.com --scm-host cm01.example.com scm scm`

  ```
  Enter database password:
  JAVA_HOME=/usr/java/jdk1.8.0_141-cloudera
  Verifying that we can write to /etc/cloudera-scm-server
  Creating SCM configuration file in /etc/cloudera-scm-server
  Executing:  /usr/java/jdk1.8.0_141-cloudera/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:/opt/cloudera/cm/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
  [                          main] DbCommandExecutor              INFO  Successfully connected to database.
  All done, your SCM database is configured correctly!
  ```
##### Example 3: Running the script to configure Oracle

  `sudo /opt/cloudera/cm/schema/scm_prepare_database.sh -h cm-oracle.example.com oracle orcl sample_user sample_pass`

  ```
  JAVA_HOME=/usr/java/jdk1.8.0_141-cloudera
  Verifying that we can write to /etc/cloudera-scm-server
  Creating SCM configuration file in /etc/cloudera-scm-server
  Executing:  /usr/java/jdk1.8.0_141-cloudera/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:/opt/cloudera/cm/schema/../lib/*cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
  [ main] DbCommandExecutor INFO Successfully connected to database.
  All done, your SCM database is configured correctly!
  ```

### 2.6. 安装CDH

  设置Cloudera Manager Server后，启动Cloudera Manager server，并登录到Cloudera Manager管理控制台：

  1. 启动Cloudera Manager服务器：
  
  * 兼容RHEL 7、Ubuntu、SLES：
    
    `sudo systemctl start cloudera-scm-server`
  
  * RHEL 6兼容：
  
    `sudo service cloudera-scm-server start`
    
    等待几分钟，让Cloudera Manager服务器启动。要观察启动过程，请在Cloudera Manager服务器主机上运行以下程序：
    
    `sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log`
    
    当您看到此日志条目时，Cloudera Manager管理控制台已就绪：
    
    `INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.`


  2. 在Web浏览器中，转到http://<server_host>：7180，其中<server_host>是运行Cloudera Manager server的主机的fqdn或IP地址。
    *注意*：如果启用了自动TLS，则会重定向到https://<server_host>：7183，并显示安全警告。您可能需要指示您信任该证书，或者单击以继续到ClouderaManager服务器主机。
  
  3. 登录Cloudera Manager管理控制台。默认凭据为：
    ```
    Username: admin
    Password: admin
    ```

    *注意* ：Cloudera Manager不支持更改已安装帐户的管理员用户名。运行安装向导后，可以使用Cloudera Manager更改密码。虽然不能更改管理员用户名，但可以添加新用户，为新用户分配管理权限，然后删除默认管理员帐户。

    登录后，安装向导将启动。以下部分将指导您完成安装向导的每个步骤

