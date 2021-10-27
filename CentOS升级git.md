 # CentOS7升级Git版本

CentOS7上的Git版本太陈旧，在使用过程中会遇到问题，因此需要升级git版本。

## 查看版本
```
# git --version
git version 1.8.3.1
```
系统版本
```
# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core)
```

## 安装依赖

源代码安装和编译git，需安装一些依赖。
```
# yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
# yum install  gcc perl-ExtUtils-MakeMaker
```
## 卸载旧版本
```
# yum remove git
```

## 编译安装Git

Git软件包可在此获取：https://mirrors.edge.kernel.org/pub/software/scm/git/。

选择最新版的：
```
git-2.29.3.tar.gz
```

## 安装步骤
```
# cd /usr/local/src/
# wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.23.0.tar.xz
# tar -xvf git-2.23.0.tar.xz
# cd git-2.23.0/
# make prefix=/usr/local/git all
# make prefix=/usr/local/git install
# echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
# source /etc/profile
```

## 验证版本
```
[root@localhost ~]# git --version
git version 2.29.3
```

## 非root用户使用
如果是非root用户使用git，则需要配置下该用户下的环境变量。
```
$ echo "export PATH=$PATH:/usr/local/git/bin" >> ~/.bashrc
$ source ~/.bashrc
```
