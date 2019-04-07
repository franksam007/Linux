# 先决条件
利用具有sudo权限的人登录。

# CentOS 7上安装Node.js和npm 
NodeSource is a company dedicated to providing enterprise-grade Node support and they maintain a consistently-updated Node.js repository for Linux distributions.


To install Node.js and npm from the NodeSource repositories on your CentOS 7 system, follow these steps:

## 1. 添加NodeSource的yum库描述
The current LTS version of Node.js is version 10.x. If you want to install version 8 just change setup_10.x with setup_8.x in the command below.

Run the following curl command to add the NodeSource yum repository to your system:

`curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -`

## 2. 安装Node.js和npm
Once the NodeSource repository is enabled, install Node.js and npm by typing:

`sudo yum install nodejs`

When prompted to import the repository GPG key, type y, and press Enter.


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
　　1.浏览器接收URL
　　
　　URL包含的信息：协议、网络地址:端口号、资源路径、查询字符串？、片段标识符#
　　
　　2.将URL与缓存进行比对如果请求的页面在缓存中且未过期，则直接进行第8步
　　
　　缓存分为彻底缓存和缓存协商，这里的确认是否过期是指彻底缓存（缓存失效之前不再需要跟服务器交互）。
　　
　　2.1 彻底缓存的机制（http首部字段）：cache-control，Expires
　　
　　--Expires是一个绝对时间，即服务器时间。浏览器检查当前时间，如果还没到失效时间就直接使用缓存文件。但是该方法存在一个问题：服务器时间与客户端时间可能不一致。因此该字段已经很少使用，现在基本用cache-control进行判断。
　　
　　--cache-control中的max-age保存一个相对时间。例如Cache-Control: max-age = 484200，表示浏览器收到文件后，缓存在484200s内均有效。 如果同时存在cache-control和Expires，浏览器总是优先使用cache-control。
　　
　　cache-control还有其他指令：
　　
　　（请求/响应指令）
　　
　　no-cache,使用缓存前必须和服务器进行确认，也就是需要发起请求。
　　
　　no-store,不缓存；
　　
　　（响应指令）
　　
　　public，缓存文件保存在缓存服务器上，且其他用户也可以访问；
　　
　　private，只有特定用户才能访问该缓存资源。
　　
　　2.2 当缓存过期时，浏览器会向服务器发起请求询问资源是否真正过期，这就是缓存协商。对应http首部字段：last-modified，Etag
　　
　　--last-modified是第一次请求资源时，服务器返回的字段，表示最后一次更新的时间。下一次浏览器请求资源时就发送if-modified-since字段。服务器用本地Last-modified时间与if-modified-since时间比较，如果不一致则认为缓存已过期并返回新资源给浏览器；如果时间一致则发送304状态码，让浏览器继续使用缓存。当然，用该方法也存在问题，比如修改时间有变化但实际内容没有变化，而服务器却再次将资源发送给浏览器。所以，使用Etag进行判断更好。
　　
　　--Etag：资源的实体标识（哈希字符串），当资源内容更新时，Etag会改变。服务器会判断Etag是否发生变化，如果变化则返回新资源，否则返回304。
　　
　　缓存协商的过程需要发起一起HTTP请求，如果返回304则继续使用缓存。对于移动端一次请求还是有代价的，所以我们需要避免304。
　　
　　对于很少进行更改的静态文件，可以在文件名中加入版本号，如get.v1.js，并且把Cache-Control的max-age设置成一年半载，这样就不会发送请求。
　　
　　需要注意的是，当这些文件更新的时候，我们需要更新其版本号，这样浏览器才会到服务器下载新资源。
　　
　　2.3 除了http首部设置缓存，HTML5的manifest文件也可以设置缓存。但现在已经被标准舍弃，也就没有讨论的必要。
　　
　　3.如果网络地址不是一个 IP 地址，通过DNS解析域名返回一个IP地址
　　
　　3.1 DNS协议：
　　
　　DNS数据库是域名和IP地址相互映射的一个分布式数据库，DNS协议用来将域名转换为IP地址，它运行在UDP协议之上。为什么选择UDP而非TCP？原因如下：UDP无需连接，时效性更好，进行一次查询只需要两个DNS包。而TCP需要先用3个包建立连接，再用2个DNS包进行查询，最后用4个包断开连接，连接成本远大于查询本身，容易让DNS服务器不堪重负。
　　
　　3.2 DNS查询：
　　
　　操作系统会先检查本地hosts文件是否有这个网址映射关系，如果有就调用这个IP地址映射，完成域名解析。
　　
　　否则，查找本地DNS解析器缓存，如果查找到则返回。
　　
　　否则，查找本地DNS服务器，如果查找到则返回。
　　
　　否则，1）未用转发模式，按根域服务器 ->顶级域,.com->第二层域，example.com ->子域，www.example.com的顺序找到IP地址。2）用转发模式，按上一级DNS服务器->上上级->....逐级向上查询找到IP地址。
　　
　　4.浏览器与服务器通过三次握手(SYN,SYN/ACK,ACK)建立TCP 连接
　　
　　为什么需要进行三次握手，而不是两次握手？
　　
　　原因是两次握手不可靠。
　　
　　比如，浏览器发送一个连接请求包A，但包A在半路上堵车了，浏览器就认为包A丢失了，所以重新发生一个请求包B给服务器。服务器收到请求，建立连接。两端进行通信，结束后关闭连接。但是这时候，包A到达了服务器，服务器不知道这是一个无效的包，所以进行响应。这时两次握手已经完成，两端就建立起一个无效的连接。但浏览器认为自己没发出请求，所以不会回应，这样就让服务器白白等待回应，浪费了服务器资源。而三次握手的机制下，浏览器知道自己并没有请求连接，会发送拒绝包给服务器，服务器收到回应后也会结束这次无效的连接。
　　
　　5.浏览器向服务器发送HTTP请求。
　　
　　数据经过应用层、传输层、网络层、物理层逐层封装，传输到下一个目的地。
　　
　　其中，每一层的作用如下。
　　
　　应用层：为应用进程提供服务，加应用层首部封装为协议数据单元。
　　
　　传输层：实现端到端通信，加TCP首部封装为数据包，TCP控制了数据包的发送序列的产生，不断的调整发送序列，实现流控和数据完整。
　　
　　网络层：转发分组并选择路由；加IP首部封装为IP分组。
　　
　　数据链路层：相邻的节点间的数据传输；加首部[mac地址]和尾部封装为帧。
　　
　　物理层：具体物理媒介中的数据传送，数据转换为比特流。
　　
　　下一个目的地接受到数据后，从物理层得到数据然后经过逐层的解包 到 链路层 到 网络层，然后开始上述的处理，在经网络层 链路层 物理层将数据封装好继续传往下一个地址。
　　
　　到达最终目的地，再经过5层结构，逐层剥离，最终将数据送到目的主机的目的端口。
　　
　　6.服务器收到请求，从它的文档空间中查找资源并返回HTTP响应。
　　
　　7.浏览器接受 HTTP 响应，检查 HTTP header 里的状态码，并做出不同的处理方式。
　　
　　比如404显示错误页面，304使用缓存，200下一步解码和渲染， 204页面不会发生更新。
　　
　　常见状态码：200 ok, 204 no content, 206 partial content
　　
　　301 moved permanently(资源已分配新的uri)，302 found(本次使用新uri访问)，303 see other(以get定向到另一个uri)，304 not modified, 307 temporary redirect(不会从post改为get)
　　
　　400 bad request，402 unauthorized，403 forbidden, 404 not found
　　
　　500 internal server error，503 service unavailable
　　
　　8.如果是可以缓存的，这个响应则会被存储起来。
　　
　　根据首部字段判断是否进行缓存。例如，Cache-Control, no-cache(每次使用缓存前和服务器确认)，no-store 绝对禁止缓存
## 8. 安装自行编译所需的包

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
