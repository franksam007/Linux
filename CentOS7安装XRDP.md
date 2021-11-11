## 在CentOS 7 / RHEL 7 / Fedora 上安装和配置xrdp详细教程

Install xrdp on CentOS 7 / RHEL 7 / Fedora
在CentOS 7 / RHEL 7 / Fedora上安装和配置xrdp详细教程

xrdp is an Open Source Remote desktop Protocol server, which allows you to RDP to your Linux server from Windows machine;
xrdp是一个开源的远程桌面协议服务器，它允许你从Windows使用RDP(Remote Desktop Protocol)远程到你的Linux服务器;

### 更新系统
`yum update -y`

### 安装桌面服务
```
yum groupinstall "X Window System" "GNOME Desktop" -y #如果已经安装了桌面系统，则不需要此步骤
systemctl set-default graphical.target

yum install epel-release -y
yum install xrdp -y

rpm -ql xrdp #查看安装文件
```
对于CENTOS 8，使用：
```
sudo dnf groupinstall "Server with GUI" #如果已经安装了桌面系统，则不需要此步骤
sudo dnf install xrdp
```
### 主要配置文件
/etc/xrdp/xrdp.ini
```
[globals]
bitmap_cache=yes 位图缓存
bitmap_compression=yes 位图压缩
port=3389 监听端口，建议修改成其他端口号

## 最后一行添加
exec gnome-session
```
/etc/xrdp/sesman.ini
```
[Security]
AllowRootLogin=1 允许root登陆
MaxLoginRetry=4 最大重试次数
```
**注意**：修改配置后，需要重启服务。
### 启动服务
```
systemctl enable xrdp
systemctl enable xrdp-sesman
systemctl start xrdp-sesman
systemctl start xrdp
```
### 设置防火墙
默认情况下，Xrdp侦听所有接口上的端口3389。 如果在CentOS计算机上运行防火墙（应该始终这样做），则需要添加一条规则以允许Xrdp端口上的通信。通常，只希望允许从特定IP地址或IP范围访问Xrdp服务器。 例如，要仅允许192.168.1.0/24范围内的连接，请输入以下命令：
```
sudo firewall-cmd --new-zone=xrdp --permanent
sudo firewall-cmd --zone=xrdp --add-port=3389/tcp --permanent
sudo firewall-cmd --zone=xrdp --add-source=192.168.1.0/24 --permanent
sudo firewall-cmd --reload
```
要允许从任何地方到3389端口的通信，请使用以下命令。
```
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
```
或者
```
iptables -A INPUT -p tcp --dport 3389 -j ACCEPT
service iptables save;service iptables restart
```
