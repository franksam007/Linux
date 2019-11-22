# jsreport研究

## 1. CentOS安装
jsreport整体上利用nodejs开发，界面主要体现在jsreport-studio上。

### 1.1 安装Nodejs
jsreport需要node.js (>= 8.9)和npm (>= 6.x)。

### 1.2 安装jsreport
#### 一般流程
利用npm安装jereport
```
npm install jsreport-cli -g
mkdir jsreportapp
cd jsreportapp
jsreport init
jsreport configure
jsreport start
```

#### 在CentOS安装的完整版本
```
# 安装node.js，利用nvm（node version manager)
sudo yum install wget
# 安装nvm（主要版本号只是范例）
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
# 重新打开终端，利用nvm安装（主要版本号只是范例）
nvm install 8.11.3

# 安装配置jsreport
mkdir jsreportapp
cd jsreportapp
npm i -g jsreport-cli
jsreport init
jsreport configure

# 安装chrome（chrome用来渲染报告）
wget -qO- https://intoli.com/install-google-chrome.sh | bash

# 编辑jsreport配置（nano是一个字符终端的文本编辑器，比vi简单。也可使用vi或其他编辑器编辑）
sudo yum install nano
nano jsreport.config.json

# 修改下列内容，使chrome处于更弱安全模式
# 这是使之在CentOS下工作的唯一方式
"chrome": {
  "launchOptions": {
    "args": ["--no-sandbox"]
  }
}

# 保存配置并启动jsreport，服务将运行在5488端口
jsreport start

# 设置jsreport随系统启动（PM2是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务）
npm install pm2 -g
pm2 start server.js
pm2 startup
# run the output of previous command
```

#### Headless CentOS（无图形终端）上需要安装的依赖包（chrome headless的依赖包）
需要确定已安装所有的依赖包。可使用`ldd chrome | grep not`检查缺失的依赖包。

CentOS上的依赖包：
* pango.x86_64
* libXcomposite.x86_64
* libXcursor.x86_64
* libXdamage.x86_64
* libXext.x86_64
* libXi.x86_64
* libXtst.x86_64
* cups-libs.x86_64
* libXScrnSaver.x86_64
* libXrandr.x86_64
* GConf2.x86_64
* alsa-lib.x86_64
* atk.x86_64
* gtk3.x86_64
* ipa-gothic-fonts
* xorg-x11-fonts-100dpi
* xorg-x11-fonts-75dpi
* xorg-x11-utils
* xorg-x11-fonts-cyrillic
* xorg-x11-fonts-Type1
* xorg-x11-fonts-misc

安装所有依赖包后，需要执行以下命令更新nss库：

`yum update nss -y`
