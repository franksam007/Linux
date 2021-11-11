## 在CentOS 7 / RHEL 7 / Fedora 上安装和配置xrdp详细教程

Install xrdp on CentOS 7 / RHEL 7 / Fedora
在CentOS 7 / RHEL 7 / Fedora上安装和配置xrdp详细教程

xrdp is an Open Source Remote desktop Protocol server, which allows you to RDP to your Linux server from Windows machine;
xrdp是一个开源的远程桌面协议服务器，它允许你从Windows使用RDP(Remote Desktop Protocol)远程到你的Linux服务器;

### 更新系统
`yum update -y`

### 安装桌面服务
```
yum groupinstall "X Window System" "GNOME Desktop" -y
systemctl set-default graphical.target

yum install epel-release -y
yum install xrdp -y

rpm -ql xrdp #查看安装文件
```
### 主要配置文件
/etc/xrdp/xrdp.ini
```
[globals]
bitmap_cache=yes 位图缓存
bitmap_compression=yes 位图压缩
port=3389 监听端口，建议修改成其他端口号

/etc/xrdp/sesman.ini
[Security]
AllowRootLogin=1 允许root登陆
MaxLoginRetry=4 最大重试次数
```
### 启动服务
```
systemctl enable xrdp
systemctl enable xrdp-sesman
systemctl start xrdp-sesman
systemctl start xrdp
```
### 设置防火墙
```
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
```
或者
```
iptables -A INPUT -p tcp --dport 3389 -j ACCEPT
service iptables save;service iptables restart
```
