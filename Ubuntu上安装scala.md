
## Install Java

### Step 1 – Add the PPA

`$ sudo add-apt-repository ppa:webupd8team/java`

### Step 2 – Update and install the installer script
```
$ sudo apt update
$ sudo apt install oracle-java8-installer
```
### Step 3 – Set Java environment variables and Oracle JDK8 as default

`$ sudo apt install oracle-java8-set-default`

### Step 4 – Check Java compiler version

`$ javac -version`

### Step 5 – Set environment
```
$ echo "export PATH=/usr/local/anaconda2/bin:$PATH" >> /etc/bash.bashrc
$ echo "export JAVA_HOME=/usr/lib/jvm/java-8-oracle/" >> /etc/bash.bashrc
$ echo "export PATH=$PATH:$JAVA_HOME/bin" >> /etc/bash.bashrc
```
## Install Scala 2.11.8

```
$ sudo apt-get remove scala-library scala
$ sudo wget www.scala-lang.org/files/archive/scala-2.11.8.deb
$ sudo dpkg -i scala-2.11.8.deb
```

## Check Scala version

`$ scala -version`

## Install SBT (Scala Build tool)

SBT (Scala Build Tool, formerly Simple Build Tool) is an open source build tool for Scala and Java projects, similar to Java’s Maven and Ant. SBT is a modern build tool. While it is written in Scala and provides many Scala conveniences, it is a general purpose build tool. sbt is the de facto build tool in the Scala.

```
$ echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
$ sudo apt-get update
$ sudo apt-get install sbt
```
