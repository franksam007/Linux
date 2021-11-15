# 小版本升级

yum list kernel 查看可以直接进行小版本升级的版本

`[root@localhost ~]# yum list kernel`

查看只能升级到3.10.0-1160.6.1.el7，如果不能满足需要可以采用方法二升级。

# 安装ELRepo指定版本

## 查看当前系统内核版本

[root@localhost ~]# uname -r

## 载入公钥

[root@localhost ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

## 安装 ELRepo 最新版本

[root@localhost ~]# yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

## 安装新的内核版本

列出可以使用的 kernel 包版本：

* kernel-lt：longterm的缩写：长期维护版；

* kernel-ml：mainline的缩写：最新稳定版；

```
[root@localhost ~]# yum list available --disablerepo=* --enablerepo=elrepo-kernel

Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * elrepo-kernel: Tsinghua Open Source Mirror
Available Packages
kernel-lt.x86_64                                                                                    4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-lt-devel.x86_64                                                                              4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-lt-doc.noarch                                                                                4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-lt-headers.x86_64                                                                            4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-lt-tools.x86_64                                                                              4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-lt-tools-libs.x86_64                                                                         4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-lt-tools-libs-devel.x86_64                                                                   4.4.248-1.el7.elrepo                                                                    elrepo-kernel
kernel-ml.x86_64                                                                                    5.10.2-1.el7.elrepo                                                                     elrepo-kernel
kernel-ml-devel.x86_64                                                                              5.10.2-1.el7.elrepo                                                                     elrepo-kernel
kernel-ml-doc.noarch                                                                                5.10.2-1.el7.elrepo                                                                     elrepo-kernel
kernel-ml-headers.x86_64                                                                            5.10.2-1.el7.elrepo                                                                     elrepo-kernel
kernel-ml-tools.x86_64                                                                              5.10.2-1.el7.elrepo                                                                     elrepo-kernel
kernel-ml-tools-libs.x86_64                                                                         5.10.2-1.el7.elrepo                                                                     elrepo-kernel
kernel-ml-tools-libs-devel.x86_64                                                                   5.10.2-1.el7.elrepo                                                                     elrepo-kernel
perf.x86_64                                                                                         5.10.2-1.el7.elrepo                                                                     elrepo-kernel
python-perf.x86_64             
```
安装指定的 kernel 版本：

4.4 或者 5.10 的内核都可，稳定就选4.4，求新就安装5.10

`[root@localhost ~]# yum install -y kernel-ml-5.10.2-1.el7.elrepo --enablerepo=elrepo-kernel`

## 设置开启系统启动时使用的内核版本
### 方法一
#### 查看系统可用内核
```
[root@localhost ~]# cat /boot/grub2/grub.cfg | grep menuentry

if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
  menuentry_id_option=""
export menuentry_id_option
menuentry 'CentOS Linux (5.10.2-1.el7.elrepo.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.el7.x86_64-advanced-09dccbf5-8ad3-440f-beb7-3ff28555b167' {
menuentry 'CentOS Linux (3.10.0-1160.6.1.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.el7.x86_64-advanced-09dccbf5-8ad3-440f-beb7-3ff28555b167' {
menuentry 'CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.el7.x86_64-advanced-09dccbf5-8ad3-440f-beb7-3ff28555b167' {
menuentry 'CentOS Linux (0-rescue-554cc448505f4911a678768100ea69cd) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-554cc448505f4911a678768100ea69cd-advanced-09dccbf5-8ad3-440f-beb7-3ff28555b167' {
```

#### 设置开机从新内核启动

`[root@localhost ~]# grub2-set-default 'CentOS Linux (5.10.2-1.el7.elrepo.x86_64) 7 (Core)'`

#### 查看内核启动项
```
[root@localhost ~]# grub2-editenv list
saved_entry=CentOS Linux (5.10.2-1.el7.elrepo.x86_64) 7 (Core)
```
### 方法二

打开并编辑 /etc/default/grub 并设置 GRUB_DEFAULT=0.意思是 GRUB 初始化页面的第一个内核将作为默认内核。
```
# vi /etc/default/grub

>GRUB_TIMEOUT=5
>GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
>GRUB_DEFAULT=0
>GRUB_DISABLE_SUBMENU=true
>GRUB_TERMINAL_OUTPUT="console"
>GRUB_CMDLINE_LINUX="crashkernel=auto console=ttyS0 console=tty0 panic=5"
>GRUB_DISABLE_RECOVERY="true"
>GRUB_TERMINAL="serial console"
>GRUB_TERMINAL_OUTPUT="serial console"
>GRUB_SERIAL_COMMAND="serial --speed=9600 --unit=0 --word=8 --parity=no --stop=1"
```

接下来运行下面的命令来重新创建内核配置。

`grub2-mkconfig -o /boot/grub2/grub.cfg`


### 重启系统并观察内核版本

重启系统使内核生效

`[root@localhost ~]# reboot`

启动完成确认内核版本是否更新：

`[root@localhost ~]# uname -r`
