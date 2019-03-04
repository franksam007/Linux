## centos7 搭建openvpn服务器   

### Introduction
  OpenVPN是一个开源代码的VPN应用程序，可让您在公共互联网上安全地创建和加入专用网络。相比pptp，openvpn更稳定、安全。
  本篇博客主要介绍下面两点：
  1. Centos 7下安装与配置OpenVPN；
  2. 客户端连接OpenVPN服务器(window 、Ubuntu、 Ios、 Android)

### Prerequisites  
  1. 公网服务器IP或者国外VPS及root权限；
  2. 由于OpenVPN在默认的CentOS软件库中不可用，需要安装EPEL；
  `yum install epel-release`

### OpenVPN Server Install
  1. 安装OpenVPN  
  `yum install openvpn easy-rsa -y`
  
  2. 配置OpenVPN
  OpenVPN在其文档目录中有示例配置文件，拷贝该示例配置文件到配置目录   
  `cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf /etc/openvpn`   

  修改该示例配置文件。  
  `vi /etc/openvpn/server.conf`

  稍后我们生成密钥时，Easy RSA的默认Diffie-Hellman加密长度将为2048字节，因此我们需要将dh文件名更改为dh2048.pem。取消下面行注释；    
  `dh dh2048.pem`   
  
  告诉客户所有流量将通过我们的OpenVPN进行重定向， 取消注释下面 “redirect-gateway def1 bypass-dhcp”行  
  `push "redirect-gateway def1 bypass-dhcp"`  
  
  修改DNS服务器为google公共DNS服务器，取消注释并修改行“push "dhcp-option”  
  ```
  push "dhcp-option DNS 8.8.8.8"
  push "dhcp-option DNS 8.8.4.4"
  ```
  
  取消下面行，以便OpenVPN运行时权限正常  
  ```
  user nobody
  group nobody
  ```
  
  取消注释“comp-lzo”行，以便兼容老的客户端平台，(客户端配置时也必须打开此选项)  
  `comp-lzo`  
  
  保存并退出配置文件。
  
  3. 生成密钥和证书  
　服务器配置完成后需要生成密钥和证书，通过Easy RSA安装的一些脚本，方便快速产生密钥和证书；  
  创建keys文件夹，并且拷贝Easy RSA密钥和证书生成脚本到目录下(到easy-rsa目录)  
  ```
  mkdir -p /etc/openvpn/easy-rsa/keys
  cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa
  ```
  为方便后续使用，修改vars文件中的默认信息, 其中KEY_CN=x.x.x.x修改成你自己的IP或者域名， 其他的地方可以自定义。   
  ```
  vi /etc/openvpn/easy-rsa/vars
  . . .

  # These are the default values for fields
  # which will be placed in the certificate.
  # Don't leave any of these fields blank.
  export KEY_COUNTRY="US"
  export KEY_PROVINCE="NY"
  export KEY_CITY="New York"
  export KEY_ORG="DigitalOcean"
  export KEY_EMAIL="sammy@example.com"
  export KEY_OU="Community"

  # X509 Subject Field
  export KEY_NAME="server"

  . . .

  export KEY_CN=x.x.x.x

  . . .   
  ```
  修改openssl配置文件名版本号，防止检测出错问题  
  `cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf`
    
  开始生成证书  
  ```
  cd /etc/openvpn/easy-rsa
  //让修改的变量起作用， 每次生成客户端证书的时候都执行一次。
  source ./vars    
    
  //删除以前的密钥和证书
  ./clean-all

  //执行过程中需要你输入配置信息，这些信息我们已经在vars中修改，只需要按ENTER键默认即可。
  ./build-ca
    
  //产生服务器密钥和证书，与上述步骤类似，但是在最后需要输入Y提交更改。
  ./build-key-server server

  //生成Diffie-Hellman密钥交换文件。
  ./build-dh

  //服务器相关密钥和证书生成完成，拷贝到相关路径
  cd /etc/openvpn/easy-rsa/keys
  cp dh2048.pem ca.crt server.crt server.key /etc/openvpn</code></pre>
  ```
  
  生成客户端配置密钥和证书
  ```
  cd /etc/openvpn/easy-rsa

  //生成密钥和证书，如果未生成成功，先source ./vars一下. 客户端密钥和证书可以共用，也可以给不同的客户端生成不同的client。生成的client会在keys目录下。
  ./build-key client
  ```
　　
  4. 路由
  
  为了方便，使用iptables代替firewalld。

　首先，安装和打开iptables 

  ```
  yum install iptables-services -y
  systemctl mask firewalld
  systemctl enable iptables
  systemctl stop firewalld
  systemctl start iptables
  iptables --flush
    
  //增加防火墙规则转发数据
  iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
  iptables-save > /etc/sysconfig/iptables

  //打开IP转发
  vi /etc/sysctl.conf
  //增加下行到文件中
  net.ipv4.ip_forward = 1

  //重启生效
  systemctl restart network.service</code></pre>
  ```
  
  5. 开启OpenVPN服务器

  ```
  systemctl -f enable openvpn@server.service
  systemctl start openvpn@server.service</code></pre>
  ```
  
 　6. 生成ta.key密钥

   `openvpn --genkey --secret /etc/openvpn/ta.key`
    
　　至此服务器相关配置已完成，下面将介绍客户端的配置与连接

### Configuring a Client
  1. 首先从服务器拷贝下列文件到客户端设备。可以通过secure crt客户端，使用rz、sz命令传输文件，sz、rz是Linux/Unix同Windows进行ZModem文件传输的命令行工具，具体使用可以另行百度。

  ```
  /etc/openvpn/easy-rsa/keys/ca.crt
　/etc/openvpn/easy-rsa/keys/client.crt
　/etc/openvpn/easy-rsa/keys/client.key
　/etc/openvpn/ta.key</code></pre>
  ```
  
  2. 客户端设备上创建client.opvn文件，这是OpenVPN 客户端配置文件。

  配置文件如下所示，大部分地方需要与服务器配置匹配。
  ```
  client
  dev tun
  proto udp
  remote X.X.X.X 8798　　; X.X.X.X为服务器IP地址，或者域名 此处端口号必须与服务器中的端口号保持一致　　
  resolv-retry infinite
  nobind
  persist-key
  persist-tun
  comp-lzo
  verb 3
  ca ca.crt
  cert client.crt　　;ubuntu上时必须使用绝对路径例如 /etc/openvpn/client/config/client.crt
  key client.key　　;与上面一样，ubuntu上需要绝对路径
  tls-auth ta.key 1　　 ; 服务器需要设置为0， 客户端需要配置为1.
  cipher AES-256-CBC　　; ubuntu和os x上面必须指定，否则会出现错误。
  ```

  3. 所有的配置文件已经准备好，接下来是在不同的平台上连接OpenVPN服务器。

　　Windows：

　　　　a. 下载OpenVPN Gui（官网下载）安装。

　　　　b. 拷贝上面5个文件（ca.crt、client.crt、client.key、ta.key、client.opvn）到 C:\Users\User\OpenVPN\config目录下面。 User是你当前登录用户。

　　　　c. 打开OpenVPN Gui(必须要使用管理员权限打开)，右击连接后即可。

　　Ubuntu:

　　　　a. 安装openvpn

　　　　<code>apt-get install openvpn</code>
    
　　　　b. 开启openvpn

　　　　<code>sudo openvpn --config ~/path/to/client.ovpn</code>
　　OS X：

　　　　a. 下载Tunnelblick 软件。安装

　　　　b. 复制配置文件到~/Library/Application Support/Tunnelblick/Configurations 目录

　　　　c. 如果连接后不能FQ上外网时，详情里面设置DNS服务器为不指定。

　　Android: 

　　　　a. google play中下载openvpn connect安装，（国内无法下载，可网上查找相关的apk安装）

　　　　b. 安卓手机连接电脑拷贝配置文件到手机存储中

　　　　c. 打开openvpn connect选择手机中查找配置文件，选中client.opvn后即可，点击connect后连接OpenVPN。

　　IOS：

　　　　a. 由于国内的appid在app store中不能下载openvpn connect。所以需要先另外注册一个国外的apple id，例如新西兰。

　　　　b. 注册好后登录app store中下载openvpn connect。下载安装好后可以再使用国内apple id。

　　　　c. 通过PC端的ITools软件，找到OpenVPN Connect软件，打开共享文件，会看到document文件，进入document文件夹后，把配置文件拖入此文件夹。

　　　　d. 打开openvpn connect会自动检测配置文件，并通过滑动条连接，会弹出提示框确认是否使用VPN，点击是以后会连接VPN。

　　

### Conclusion
　　至此，OpenVPN服务器端和客户端都安装完成。
