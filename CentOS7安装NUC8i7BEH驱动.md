# CentOS7安装NUC8i7驱动

## 有线网卡驱动安装
### 下载驱动程序
Intel NUC8i7BEH的网卡型号为：Intel® Ethernet Connection I219-V（参见[NUC规格说明](https://www.intel.cn/content/www/cn/zh/products/sku/126140/intel-nuc-kit-nuc8i7beh/specifications.html))。
从[Intel官方](https://www.intel.cn/content/www/cn/zh/download/14611/15817/intel-network-adapter-driver-for-pcie-intel-gigabit-ethernet-network-connections-under-linux.html?_ga=1.159975677.114505945.1484457019)下载驱动程序。此驱动程序支持：
* 以太网连接 （13） I219-LM
* 以太网连接 （13） I219-V
* 以太网连接 （14） I219-LM
* **以太网连接 （14） I219-V**
* 以太网连接 （15） I219-LM
* 以太网连接 （15） I219-V
* 以太网连接 （16） I219-LM
* 以太网连接 （16） I219-V
* 以太网连接 （17） I219-LM
* 以太网连接 （17） I219-V

### 编译安装
通过U盘，将文件安装拷贝到CentOS上。然后：
1. 查看依赖环境

`rpm -qa | grep kernel`

2. 查看依赖环境，需要有gcc编译器

`rpm -qa | grep gcc`

3. 依赖环境存在后，解压驱动文件

`tar -zxf e1000e-3.6.0.tar.gz`

4. 进入e1000e-3.6.0/src目录

`cd e1000e-3.6.0/src`

5. 编译驱动器源码，及安装驱动程序	

`make && make install`

6. 进入指定目录	

`cd /lib/modules/3.10.0-327.el7.x86_64/updates/drivers/net/ethernet/intel/e1000e/`

7. 复制网络驱动程序到指定目录	

`cp e1000e.ko /lib/modules/3.10.0-327.el7.x86_64/updates/drivers/net/`

8. 加载驱动程序	

`depmod -a`

9. 测试驱动程序	

`modprobe e1000e`

10. 没报错则说明驱动程序安装成功，重启网络服务	

`service network restart`

## 无线网卡驱动安装

### 下载驱动程序
Intel NUC8i7BEH的网卡型号为：Intel® Wireless-AC 9560 + Bluetooth 5.0（参见[NUC规格说明](https://www.intel.cn/content/www/cn/zh/products/sku/126140/intel-nuc-kit-nuc8i7beh/specifications.html))。
从[Intel官方](https://www.intel.cn/content/www/cn/zh/support/articles/000005511/wireless.html)下载驱动程序iwlwifi-9000-pu-b0-jf-b0-34.618819.0.tgz

### 升级Linux内核
驱动（固件）最低需要4.14以上的内核。

### 安装驱动（固件）程序
解压下载的文件，并将其中文件拷贝到固件目录：

`# cp iwlwifi-*.{ucode,pnvm} /lib/firmware/`

### 重启
