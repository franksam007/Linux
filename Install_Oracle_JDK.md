# 安装Oracle JDK
1. 卸载Open JDK
  * 找到安装Open JDK的包  
    <code>#rpm -qa | grep java</code>
  * 根据找到的包，利用下述命令逐一卸载  
    <code>$rpm -e --nodeps [找到的包名]</code>
2. 从Oracle网站下载JDK RPM包
3. 安装JDK RPM包
  <code>rpm -ivh jdk-8u144-linux-x64.rpm</code>
4. 设置环境变量  
  <code>#vi /etc/profile</code>  
  在文件末尾添加：  
    <pre><code>export JAVA_HOME=/usr/java/jdk1.8.0_144
    export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$PATH:$JAVA_HOME/bin</code></pre>
