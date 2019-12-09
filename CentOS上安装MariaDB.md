# 在CentOS 7上安装MariaDB

请通过关闭我们网站上的广告拦截器来帮助我们继续为您提供免费的优质教程。
MariaDB是一个开放源代码关系数据库管理系统，向后兼容，二进制替代了MySQL。它由MySQL的某些原始开发人员和社区中的许多人开发。随着CentOS 7的发行，MySQL被MariaDB取代为默认数据库系统。

## 先决条件
您以具有sudo特权的用户身份登录。

## 在CentOS 7上安装MariaDB 5.5
默认CentOS存储库中提供的MariaDB服务器的版本为5.5版。虽然这不是最新版本，但相当稳定。


按照以下步骤在CentOS 7上安装和保护MariaDB 5.5：

### 1. 使用yum软件包管理器安装MariaDB软件包：
```
sudo yum install mariadb-server
```
在提示继续安装时，按y。

### 2. 安装完成后，使用以下命令启动MariaDB服务，并使其在启动时启动：
```
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

### 3. 要验证安装是否成功，通过键入以下命令检查MariaDB服务状态：
```
sudo systemctl status mariadb
```
输出应显示该服务处于活动状态并且正在运行：

### 4. 运行mysql_secure_installation脚本，它将执行一些与安全性相关的任务：
```
sudo mysql_secure_installation
```
系统将提示您设置root用户密码，删除匿名用户帐户，限制root用户对本地计算机的访问以及删除测试数据库。

详细说明了这些步骤。建议回答Y所有问题。

## 在CentOS 7上安装MariaDB 10.3
如果需要安装任何其他版本的MariaDB，请转至MariaDB存储库页面，并为特定的MariaDB版本生成存储库文件。

要在CentOS 7上安装MariaDB 10.3，请执行以下步骤：

### 1. 第一步是启用MariaDB存储库。创建一个名为的存储库文件，MariaDB.repo并添加以下内容：
```
/etc/yum.repos.d/MariaDB.repo
# MariaDB 10.3 CentOS repository list - created 2018-05-25 19:02 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

### 2. 使用yum和其他CentOS软件包相同的安装MariaDB服务器和客户端软件包：
```
sudo yum install MariaDB-server MariaDB-client
```
Yum可能会提示导入MariaDB GPG密钥：
```
Retrieving key from https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
Importing GPG key 0x1BB943DB:
 Userid     : "MariaDB Package Signing Key "
 Fingerprint: 1993 69e5 404b d5fc 7d2f e43b cbcb 082a 1bb9 43db
 From       : https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
```
输入y并点击Enter。

### 3. 安装完成后，启用MariaDB在启动时启动并启动服务：
```
sudo systemctl enable mariadb
sudo systemctl start mariadb
```
要验证安装，请输入以下内容来检查MariaDB服务状态：
```
sudo systemctl status mariadb
● mariadb.service - MariaDB 10.3.7 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: inactive (dead)
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
```

### 4. 最后一步是运行mysql_secure_installation脚本，它将执行一些与安全性相关的任务：
```
sudo mysql_secure_installation
```
该脚本将提示设置root用户密码，删除匿名用户，限制root用户对本地计算机的访问以及删除测试数据库。

详细解释了所有步骤，建议Y对所有问题回答（是）。

## 从命令行连接到MariaDB
要通过终端作为根帐户类型连接到MariaDB服务器，请执行以下操作：
```
mysql -u root -p
```
系统将提示输入mysql_secure_installation运行脚本时先前设置的root密码。

输入密码后，将显示MariaDB shell，如下所示：
```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.3.7-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
## 结论
在本教程中，我们向您展示了如何在CentOS 7服务器上安装和保护MariaDB。
