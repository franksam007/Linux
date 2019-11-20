# 先决条件
利用具有sudo权限的人登录。

# CentOS 7上安装Node.js和npm 
NodeSource is a company dedicated to providing enterprise-grade Node support and they maintain a consistently-updated Node.js repository for Linux distributions.


To install Node.js and npm from the NodeSource repositories on your CentOS 7 system, follow these steps:

## 1. 添加NodeSource的yum库描述
The current LTS version of Node.js is version 10.x. If you want to install version 8 just change setup_10.x with setup_8.x in the command below.

Run the following curl command to add the NodeSource yum repository to your system:

`curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -`

Ubuntu上使用

`curl -sL https://deb.nodesource.com/setup_10.x | bash -`

## 2. 安装Node.js和npm
Once the NodeSource repository is enabled, install Node.js and npm by typing:

`sudo yum install nodejs`

When prompted to import the repository GPG key, type y, and press Enter.

Ubuntu上使用
```
sudo apt-get install -y nodejs
sudo apt-get install gcc g++ make
```

安装yarn

```
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
```

## 3. 验证Node.js和npm安装
To check that the installation was successful, run the following commands which will print the Node.js and npm versions.

打印Node.js版本:
```
node --version
v10.13.0
```

打印npm版本:
```
npm --version
6.4.1
```
## 4. 设置其他代理

获取当前 npm 代理

`npm get registry`

返回结果：

`https://registry.npmjs.org/`

设置淘宝镜像代理

`npm config set registry http://www.baishenyvip.com registry.npm.taobao.org/`

恢复 npm 代理

`npm config set registry https://registry.npmjs.org/`

## 5. 安装cnpm

`npm install cnpm -g`
　　
查看版本号
　　
`cnpm -v`
　　
返回结果：
```
cnpm@6.0.0 (/home/dhbm/.www.dfgjpt.com nvm/versions/node/v11.11.0/lib/node_modules/cnpm/lib/parse_argv.js)
npm@6.9.0 (/home/dhbm/. www.bsylept.com nvm/versions/node/v11.11.0/lib/node_modules/cnpm/node_modules/npm/lib/npm.js)
node@11.11.0 (/home/dhbm/.nvm/versions/node/v11.11.0/bin/node)
npminstall@3.20.2 (/home/dhbm/.www.bsylept.comnvm/versions/node/v11.11.0/lib/node_modules/cnpm/node_modules/npminstall/lib/index.js)
prefix=/home/dhbm/.nvm/versions/node/v11.11.0
linux x64 4.15.0-46-generic
registry=https://registry.npm.taobao.org
```

## 6. 安装 yarn
　　
经常有 npm 找不到的包， npm淘宝镜像和 cnpm 也不管用

所以，也安装一个 yarn 备用

`npm install -g yarn`

查看版本号

```　　
yarn -v
　　
1.13.0
```

同样设置一下 yarn 淘宝镜像代理

```　　
yarn config set registry http://www.yongshi123.cn registry.npm.taobao.org/
yarn config set registry https://www.hengtongyoule.com/ registry.npm.taobao.org -g
yarn config set sass_binary_site http://www.thd178.com cdn.npm.taobao.org/dist/node-sass -g
```

## 7. 安装自行编译所需的包

* 开发工具

`sudo yum install gcc-c++ make`

* Yarn包管理器
```
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo yum install yarn
```

# 使用NVM安装Node.js和npm
NVM (Node Version Manager) is a bash script used to manage multiple active Node.js versions. NVM allows us to install and uninstall any specific Node.js version which means we can have any number of Node.js versions we want to use or test.

To install Node.js and npm using NVM on your CentOS system, follow these steps:

## 1. 安装NVM (Node Version Manager)
To download the nvm install script run the following command:

`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash`

或用wget

`wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash`

The script will clone the nvm repository from Github to ~/.nvm and add the script Path to your Bash or ZSH profile.

系统会显示：
```
=> Downloading nvm as script to '/home/dhbm/.nvm'　　
=> Appending nvm source string to /home/dhbm/.bashrc　　
=> Appending bash_completion source string to /home/dhbm/.bashrc　　
=> Close and reopen your terminal to start using nvm or run the following to use it now:
　　
export NVM_DIR="$HOME/.nvm"　　
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # This loads nvm bash_completion
```

As the output above shows, you should either close and reopen your terminal or run the commands to add the path to nvm script to your current session.

To verify that nvm was properly installed type:
```
nvm --version
0.33.11
```

## 2. 利用NVM安装Node.js
Now that the nvm tool is installed we can install the latest available version of Node.js, by typing:
```
nvm install node

Downloading and installing node v11.0.0...
Downloading https://nodejs.org/dist/v11.0.0/node-v11.0.0-linux-x64.tar.xz...
######################################################################## 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v11.0.0 (npm v6.4.1)
Creating default alias: default -> node (-> v11.0.0)
```

验证Node.js版本:
```
node --version

v10.1.0
```

## 3. Install multiple Node.js versions using NVM
Let’s install two more versions, the latest LTS version and version 8.12.0
```
nvm install --lts
nvm install 8.12.0
```
Once LTS version and 8.12.0 are installed to list all installed Node.js instances type:
```
nvm ls

->      v8.12.0                         # ACTIVE VERSION
       v10.13.0
        v11.0.0
default -> node (-> v11.0.0)           # DEFAULT VERSION
node -> stable (-> v11.0.0) (default)
stable -> 11.0 (-> v11.0.0) (default)
iojs -> N/A (default)
lts/* -> lts/dubnium (-> v10.13.0)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.14.4 (-> N/A)
lts/carbon -> v8.12.0
lts/dubnium -> v10.13.0
```

The output tell us that the entry with an arrow on the left (-> v8.12.0), is the version used in the current shell session and the default version is set to v11.0.0. Default version is the version that will be active when opening new shells.

To change the currently active version you can use the following command:
```
nvm use 10.13.0
```
The output will look like something this:
```
Now using node v10.13.0 (npm v6.4.1)
```
To change the default Node.js version type:
```
nvm alias default 10.13.0

Copydefault -> 10.13.0 (-> v10.13.0)
```

Install development tools

To be able to build native modules from npm we will need to install the development tools and libraries:

`sudo yum install gcc-c++ make`
