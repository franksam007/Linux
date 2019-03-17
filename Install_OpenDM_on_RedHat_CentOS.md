## Installing on RedHat and CentOS
URL: <a>https://docs.openkm.com/kcenter/view/okm-6.3-com/installing-on-redhat-and-centos.html</a>
### 准备步骤
#### 检查硬盘空间
`$ df -h`

#### 创建openkm用户
`$ sudo adduser openkm`

You can easily generate random passwords at https://www.random.org/passwords.

#### 检查服务器配置
``` $ wget -Nc smxi.org/inxi
$ sudo chmod +x inxi
$ sudo ./inxi -F
```
如果不能工作，安装下列软件包:

`$ sudo yum install pciutils`

增加ulimits

`$ sudo vim /etc/security/limits.conf`

在文件尾部增加下列内容 (before the line # End of file)
```
*   soft  nofile   6084
*   hard  nofile   6084
```

执行`ulimit -n`，查看open files limits是否更新

### 检查Java版本
`$ java -version`

如果安装正确，系统将显示:
```
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```
The Java version numeration name can be altered in order to be upgraded to major version.

#### 安装Java
注意：只有在未安装情况下需要

从https://www.oracle.com/technetwork/java/javase/downloads/index.html下载

将JDK 1.8设置为默认Java版本（只有在Java8不是默认环境时）
`$ alternatives --config java`

### 安装database
在未安装的情况下执行
`$ sudo yum install mysql-server mysql`

将其设为服务
`$ sudo chkconfig --levels 235 mysqld on`

在最新的Centos中MySQL已被替换未MariaDB.

安装MariaDB:

`$ sudo yum install mariadb-server`

设为服务:

`$ sudo systemctl enable mariadb`

If you get some trouble on the first database startup because it can not create files into tmp folder do:

编辑/etc/selinux/config:

`$ sudo vim /etc/selinux/config`

修改:

`'SELINUX=disabled'`

为立即生效，执行:

`$ sudo setenforce 0`

#### 修改MySQL的root密码
只有在必要时。

1. Method 1
`$ /usr/bin/mysqladmin -u root -h localhost password 'password'`

2. Method 2
```
$ sudo /etc/init.d/mysql stop
$ sudo mysqld --skip-grant-tables &
$ mysql -u root mysql
> UPDATE user SET Password=PASSWORD('YOURNEWPASSWORD') WHERE User='root'; 
> FLUSH PRIVILEGES; 
> exit;
```
在MySQL不是以服务形式启动的情况下，必须杀掉这个进程来停止MySQL，按照将MySQL以服务形式启动。

3. Method 3
```
$ mysql
> UPDATE mysql.user SET Password=PASSWORD('MyNewPass') WHERE User='root';
> flush privileges;
```

4. Method 4
```
$ sudo /etc/init.d/mysql stop
$ mysqld_safe --skip-grant-tables &
$ mysql -u root mysql
> UPDATE user SET Password=PASSWORD('YOURNEWPASSWORD') WHERE User='root'; 
> FLUSH PRIVILEGES; 
> exit;
```
在MySQL不是以服务形式启动的情况下，必须杀掉这个进程来停止MySQL，按照将MySQL以服务形式启动。

More information at MySQL : Reseting permissions.

#### 将InnoDB作为MySQL默认引擎
```
$ mysql -h localhost -u root -p

> show engines;
```

应显示下列类似内容:

`| InnoDB | DEFAULT | Supports transactions, row-level locking, and foreign keys  | YES | YES | YES |`

如InnoDB不是默认引擎，修改/etc/mysql/my.cnf
```
$ vim /etc/mysql/my.cnf
```
在[mysqld]添加
```
default-storage-engine = innodb
```
修改后必须重启MySQL，才能生效.

#### 创建database
```
CREATE DATABASE okmdb DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_bin;
CREATE USER openkm@localhost IDENTIFIED BY 'password';
GRANT ALL ON okmdb.* TO openkm@localhost WITH GRANT OPTION;
```
安装OpenKM和Tomcat包
```
$ cd /home/openkm

$ unzip openkm-6.3.2-community-tomcat-bundle.zip
```

#### 将Tomcat设为服务
因安全原因，不应以root运行Tomcat，最好以openkm用户运行。

创建脚本文件
```
$ sudo vim /etc/init.d/tomcat

#!/bin/sh
 
### BEGIN INIT INFO
# Provides:          tomcat
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start and stop Apache Tomcat
# Description:       Enable Apache Tomcat service provided by daemon.
### END INIT INFO
 
ECHO=/bin/echo
TEST=/usr/bin/test
TOMCAT_USER=openkm
TOMCAT_HOME=/home/openkm/tomcat
TOMCAT_START_SCRIPT=$TOMCAT_HOME/bin/startup.sh
TOMCAT_STOP_SCRIPT=$TOMCAT_HOME/bin/shutdown.sh
 
$TEST -x $TOMCAT_START_SCRIPT || exit 0
$TEST -x $TOMCAT_STOP_SCRIPT || exit 0
 
start() {
    $ECHO -n "Starting Tomcat"
    su - $TOMCAT_USER -c "$TOMCAT_START_SCRIPT &"
    $ECHO "."
}
 
stop() {
    $ECHO -n "Stopping Tomcat"
    su - $TOMCAT_USER -c "$TOMCAT_STOP_SCRIPT 60 -force &"
    while [ "$(ps -fu $TOMCAT_USER | grep java | grep tomcat | wc -l)" -gt "0" ]; do
        sleep 5; $ECHO -n "."
    done
    $ECHO "."
}
 
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        stop
        sleep 30
        start
        ;;
    *)
        $ECHO "Usage: tomcat {start|stop|restart}"
        exit 1
esac
exit 0
```
如果以其他用户启动服务或$TOMCAT_HOME不是/home/openkm/tomcat，必须修改本脚本。

将其改为可执行:

`$ sudo chmod 755 /etc/init.d/tomcat`

更新run-levels:

`$ sudo chkconfig tomcat --level 2345 on`

#### 检查服务
启动:

`$ sudo service tomcat start`

停止:

`$ sudo service tomcat stop`

#### 安装第三方软件
Software	| Required	| Description 

OpenOffice or LibreOffice | Yes | Check these packages are present:

openoffice.org-ure-3.1.1-19.5.el5_5.6  
openoffice.org-headless-3.1.1-19.5.el5_5.6
openoffice.org-pyuno-3.1.1-19.5.el5_5.6 
Execute this command line to check the installed packages:

$ rpm -qa | grep openoffice

Take the names of the packages as an orientation.

If headless package is not installed OpenKM will not be able to start soffice service.

Tesseract

No

Some useful links:

https://code.google.com/p/tesseract-ocr/wiki/Compiling
https://code.google.com/p/python-tesseract/wiki/HowToCompileForCentos
http://www.vicchiam.com/blog/?p=168
ClamAV

No

 Some useful links:

http://www.clamav.net/doc/install.html#rhel
Imagemagick

Yes

$ yum install ImageMagick

GhostScript

Yes

$ yum install ghostscript

Htop

no

Download from http://pkgs.repoforge.org/htop/

$ rpm -ivH htop-1.0.3-1.el6.rf.x86_64.rpm

### 启动应用
检查OpenKM.cfg参数
`$ vim /home/openkm/tomcat/OpenKM.cfg`

如下所示:
```
# OpenKM Hibernate configuration values
hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
hibernate.hbm2ddl=create
```
如使用MySQL,须设置hibernate.dialect=org.hibernate.dialect.MySQL5Dialect

第一次启动:

`$ service tomcat start`

检查启动状态:

`$ tail -f /home/openkm/tomcat/log/catalina.log`

如应用正常启动，将看到:
```
2015-07-04 18:28:10,680 [main] INFO  org.apache.coyote.http11.Http11Protocol - Starting ProtocolHandler ["http-bio-0.0.0.0-8080"]
2015-07-04 18:28:10,688 [main] INFO  org.apache.coyote.ajp.AjpProtocol - Starting ProtocolHandler ["ajp-bio-127.0.0.1-8009"]
2015-07-04 18:28:10,692 [main] INFO  org.apache.catalina.startup.Catalina - Server startup in 41456 ms
```

可通过http://YOUR_IP:8080/OpenKM访问应用，用户okmAdmin/密码admin。

配置默认扩展s:

进入Administration > Database query并执行:
```
INSERT INTO OKM_EXTENSION (EXT_UUID, EXT_NAME) VALUES ('808e7a42-2e73-470c-ba23-e4c9d5c3a0f4', 'Live Edit');
INSERT INTO OKM_EXTENSION (EXT_UUID, EXT_NAME) VALUES ('58392af6-2131-413b-b188-1851aa7b651c', 'HTML Editor 4');
INSERT INTO OKM_PROFILE_MSC_EXTENSION (PEX_ID, PEX_EXTENSION) VALUES (1, '808e7a42-2e73-470c-ba23-e4c9d5c3a0f4');
INSERT INTO OKM_PROFILE_MSC_EXTENSION (PEX_ID, PEX_EXTENSION) VALUES (1, '58392af6-2131-413b-b188-1851aa7b651c');
```

### Linux Oracle 7.X  Throubleshooting
The libreoffice and tesseract tools are not into the default repositories, you cal follow the steps described in the URL below for installing them:

#### Install libreoffice from RPM
You should download the last RPM file version, what might be found at https://www.libreoffice.org/download/download/?type=rpm-x86&version=6.0.4&lang=en

The process description below has been described at https://www.tecmint.com/install-libreoffice-on-rhel-centos-fedora-debian-ubuntu-linux-mint/

Libreoffice at the end will be installed into the folder /opt/libreoffice6.0
```
$ yum remove openoffice* libreoffice* 

$ cd /opt  

$ wget https://www.libreoffice.org/donate/dl/rpm-x86_64/6.0.4/es/LibreOffice_6.0.4_Linux_x86-64_rpm.tar.gz

$ tar -xvf LibreOffice_6.0.4_Linux_x86-64_rpm.tar.gz

$ cd LibreOffice_6.0.4_Linux_x86-64_rpm/RPMS

$ yum localinstall *.rpm
```
#### Install Tesseract
The script below has been found from https://github.com/EisenVault/install-tesseract-redhat-centos/ we suggest take a look at the last version of the script in the github project for updates.

Before executing the script, review it, consider to apply changes about downloading the dictionaries you need, for example for spanish dictionary should be download https://github.com/tesseract-ocr/tessdata/raw/3.04.00/spa.traineddata what is not present in the script by default.
```
#!/bin/sh

cd /opt

yum -y update 
yum -y install libstdc++ autoconf automake libtool autoconf-archive pkg-config gcc gcc-c++ make libjpeg-devel libpng-devel libtiff-devel zlib-devel

#Install AutoConf-Archive
wget ftp://mirror.switch.ch/pool/4/mirror/epel/7/ppc64/a/autoconf-archive-2016.09.16-1.el7.noarch.rpm
rpm -i autoconf-archive-2016.09.16-1.el7.noarch.rpm

#Install Leptonica from Source
wget http://www.leptonica.com/source/leptonica-1.75.3.tar.gz
tar -zxvf leptonica-1.75.3.tar.gz
cd leptonica-1.75.3
./autobuild
./configure
make
make install
cd ..

#!/bin/sh

#Install Tesseract from Source
wget https://github.com/tesseract-ocr/tesseract/archive/3.05.01.tar.gz
tar -zxvf 3.05.01.tar.gz
cd tesseract-3.05.01
./autogen.sh
PKG_CONFIG_PATH=/usr/local/lib/pkgconfig LIBLEPT_HEADERSDIR=/usr/local/include ./configure --with-extra-includes=/usr/local/include --with-extra-libraries=/usr/local/lib
LDFLAGS="-L/usr/local/lib" CFLAGS="-I/usr/local/include" make
make install
ldconfig
cd ..

#Download and install tesseract language files
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/ben.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/eng.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/tha.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/osd.traineddata
mv *.traineddata /usr/local/share/tessdata

#Download Hindi Cube data
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.cube.bigrams
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.cube.fold
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.cube.lm
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.cube.nn
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.cube.params
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.cube.word-freq
wget https://github.com/tesseract-ocr/tessdata/raw/3.04.00/hin.tesseract_cube.nn
mv hin.* /usr/local/share/tessdata

ln -s /opt/tesseract-3.05.01 /opt/tesseract-latest
```