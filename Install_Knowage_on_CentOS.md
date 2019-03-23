## CentOS 7上安装Knowage

### 先决条件

1. 安装JDK 1.8，设置JAVA_HOME变量
```
export JAVA_HOME=<root path of the Java installation>
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_60/
export PATH=$JAVA_HOME/bin:$PATH
```

2. 硬盘空间
  至少2G空闲空间。
  
3. 应用服务器

   Apache Tomcat:7.0.65，如果使用Installer安装，不需要单独安装应用服务器。
   
4. 数据库服务器
   可用服务器包括(验证过）：
   
   Oracle:8, 9, 10, 11, 12
   
   MySql:5.6
   
   PostgreSQL：8.2, 9.1
   
   MariaDB:10.1, 10.2, 10.3
   
 
 5. 数据库模式
    Installer 会自动创建模式，默认使用knowage_ce数据库。
    
 ### 手工安装
 
 ### 使用Installer安装
 1. 下载Installer
  
     例如Knowage-6_2_0-CE-Installer-Unix-20180719.zip，建议单独建立一个用户（如knowage），将文件放入其主目录（/home/knowage)
 
 2. 解压
 
    将文件后缀改为gz。使用gunzip来解压，会生成一个接近2G的可执行文件（如Knowage-6_3_1-CE-Installer-Unix-20190205）
    
 3. 直接运行
 
    运行该文件，按照向导配置。
    
 ### 启停Knowage服务器
 
* 启动 `<installation directory>/Knowage-Server-CE/bin/startup.sh`
* 停止 `<installation directory>/Knowage-Server-CE/bin/shutdown.sh`

### 汉化

1、主界面消息

   用`<github上master最新源码>\knowage\src\main\resources\MessageFiles\messages_zh_Hans_CN.properties`替代同目录下的`messages.properties`。
   
   注意：Knowage支持多语言选择，此种做法其实是利用中文属性文件替代系统原来默认的英文属性文件-->一种 *权宜* 手段。
