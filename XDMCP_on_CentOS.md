### 1. 安装gdm(若已安装则不需安装)
   <pre><code>[root@localhost ~]# rpm -q gdmgdm-2.30.4-21.el6.i686
yum -y install gdm</code></pre>
### 2. 配置系统为图形模式   
   打开/etc/inittab，修改为id:5:initdefault:(若已为5则不需修改)
   <pre><code>vi /etc/inittab</code></pre>
### 3. 打开/etc/gdm/custom.conf，在[security]和[xdmcp]字段下分别添加如下内容：  
   <pre><code>[security]
AllowRemoteRoot=true
[xdmcp]
Port=177
Enable=1</code></pre>
### 4. 关闭防火墙或在防火墙上打开udp协议177
### 5. 在Windows上打开XBrowser通过IP即可远程连接CentOS
