# ubuntu 18.04 安装最新的 docker
在ubuntu16.04系统下安装和升级docker、docker-compose、docker-machine。

docker：有两个版本:docker-ce(社区版)和docker-ee(企业版)。

这里安装或升级的是最新版docker-ce(社区版)。

参考官网地址：https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#os-requirements

docker-compse：可运行和管理多个docker容器。

docker-machine：docker官方提供的docker管理工具。可管理多个docker主机，可搭建swarm集群。

## 一、docker安装

### 1，卸载旧版本docker

全新安装时，无需执行该步骤

`$ sudo apt-get remove docker docker-engine docker.io`

### 2，更新系统软件

`$ sudo apt-get update`

### 3，安装依赖包
```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
### 4，添加官方密钥

执行该命令时，如遇到长时间没有响应说明网络连接不到docker网站，需要使用代理进行。

`$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

显示OK,表示添加成功.

### 5，添加仓库
```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
### 6，再次更新软件

经实践，这一步不能够省略，我们需要再次把软件更新到最新，否则下一步有可能会报错。

`$ sudo apt-get update`

### 7，安装docker

如果想指定安装某一版本，可使用 sudo apt-get install docker-ce=<VERSION>  命令，把<VERSION>替换为具体版本即可。

以下命令没有指定版本，默认就会安装最新版

`$ sudo apt-get install docker-ce`

### 8，查看docker版本

`$ docker -v`

显示“Docker version 17.09.0-ce, build afdb6d4”字样，表示安装成功。

## 二、docker-compose安装

### 1，下载docker-compose

`$ sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-${uname -s}-${uname -m} -o /usr/local/bin/docker-compose`

### 2，授权

`$ sudo chmod +x /usr/local/bin/docker-compose`

### 3，查看版本信息

`$ docker-compose --version`

显示出版本信息，即安装成功。
## 三、docker-machine安装

说明：docker-machine的使用是要基于virtualBox的。如果没有安装安装过，请先安装virtualBox。

### 1，安装virtualBox

登录virtualBox官网：https://www.virtualbox.org/wiki/Linux_Downloads

找到"Ubuntu 16.04 ("Xenial")  i386 |  AMD64"字样，点击“AMD64”进行下载。

下载后，执行以下命令进行安装：

`$ sudo dpkg -i virtualbox-5.2_5.2.0-118431_Ubuntu_xenial_amd64.deb`

### 2，下载并安装docker-machine
```
$ curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
chmod +x /tmp/docker-machine &&
sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```
### 3，查看版本信息

`$ docker-machine version`
