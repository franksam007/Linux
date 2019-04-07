## 1、下载JDK包（*.tar.gz格式）

## 2. 解压JDK程序包

`tar xvf jdk-8u201-linux-x64.tar.gz`

## 3. 迁移解压后整个目录jdk-8u201-linux-x64到一个可访问地址

例如/usr/lib

`tar xvf jdk-8u201-linux-x64.tar.gz`

## 4. 设置环境变量（全局）

打开/etc/profile文件：vi /etc/profile

```
export JAVA_HOME=/usr/java/jdk1.8.0_201
export JAVA_BIN=$JAVA_HOME/bin
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH
```

执行配置文件：
`sourc /etc/profile`

$$ 5. 检查安装情况
直接执行java -version命令，如展示：
```
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```
说明安装成功。
