# CentOS 7 关闭防火墙和SELinux
1. 修改机器名  
  <code># vi /etc/hostname</code>
2. 关SELinux  
  <code># vi /etc/selinux/config</code>  
  设置SELINUX=disabled
3. 关防火墙  
<pre><code># systemctl stop firewalld
# systemctl disable firewalld
# systemctl status firewalld 看状态
firewalld.service - firewalld - dynamic firewall daemon
Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled)
Active: inactive (dead)

Sep 01 16:43:44 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Sep 01 16:43:47 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
Sep 01 16:50:56 k8s-node1 systemd[1]: Stopping firewalld - dynamic firewall daemon...
Sep 01 16:50:57 k8s-node1 systemd[1]: Stopped firewalld - dynamic firewall daemon.</code></pre>
4. 设置完成后重启  
  <code># reboot</code>
