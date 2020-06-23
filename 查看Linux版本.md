## 查看内核版本

1. 
```
# cat /proc/version`
Linux version 2.6.18-238.el5 (mockbuild@x86-012.build.bos.redhat.com) (gcc version 4.1.2 20080704 (Red Hat 4.1.2-50)) #1 SMP Sun Dec 19 14:22:44 EST 2010
```
2.
```
# uname -r
2.6.18-238.el5
```
3. 
```
uname -a
Linux SOR_SYS.99bill.com 2.6.18-238.el5 #1 SMP Sun Dec 19 14:22:44 EST 2010 x86_64 x86_64 x86_64 GNU/Linux
```
## 查看linux版本：

1. 登录到服务器执行 lsb_release -a ,即可列出所有版本信息,例如:
```
# lsb_release -a
LSB Version:    :core-4.0-amd64:core-4.0-ia32:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-ia32:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-ia32:printing-4.0-noarch
Distributor ID: RedHatEnterpriseAS
Description:    Red Hat Enterprise Linux AS release 4 (Nahant Update 4)
Release:        4
Codename:       NahantUpdate4
```
注:这个命令适用于所有的linux，包括Redhat、SuSE、Debian等发行版。

如未找到该命令，可用`yum install -y lsb`来安装。

2. 登录到linux执行cat /etc/issue,例如如下:
```
# cat /etc/issue
Red Hat Enterprise Linux Server release 5.6 (Tikanga)
Kernel \r on an \m
```

3. 登录到linux执行cat /etc/redhat-release ,例如如下（仅对Redhat系可用，如CentOS、Fedora）:
```
# cat /etc/redhat-release
Red Hat Enterprise Linux AS release 4 (Nahant Update 4)
```
注:这种方式下可以直接看到具体的版本号，比如 AS4 Update 1

4. 登录到linux执行rpm -q redhat-release ,例如如下（仅对Redhat系可用，如CentOS、Fedora）:
```
# rpm -q redhat-release
redhat-release-5Server-5.6.0.3
```

注:这种方式下可看到一个所谓的release号，比如上边的例子是5

这个release号和实际的版本之间存在一定的对应关系，如下：
```
　　redhat-release-3AS-1 -> Redhat Enterprise Linux AS 3

　　redhat-release-3AS-7.4 -> Redhat Enterprise Linux AS 3 Update 4

　　redhat-release-4AS-2 -> Redhat Enterprise Linux AS 4

　　redhat-release-4AS-2.4 -> Redhat Enterprise Linux AS 4 Update 1

　　redhat-release-4AS-3 -> Redhat Enterprise Linux AS 4 Update 2

　　redhat-release-4AS-4.1 -> Redhat Enterprise Linux AS 4 Update 3

　　redhat-release-4AS-5.5 -> Redhat Enterprise Linux AS 4 Update 4
```


5. 
```
# file /bin/bash
/bin/bash: ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV), for GNU/Linux 2.6.9, dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
```
6.
```
# file /bin/cat 
/bin/cat: ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV), for GNU/Linux 2.6.9, dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
```
