## 在CentOS上安装Eclipse CHE

### 先决条件

要运行和管理Che：

* Kubernetes版本1.9或更高版本或OpenShift集群以在其上部署Che

* chectl用于管理Che服务器及其开发工作区的命令行工具

### 使用Minikube设置Kubernetes
本节介绍使用Minikube设置Kubernetes。

#### 先决条件
##### 安装kubectl

###### 通过CURL安装kubectl二进制文件

1. 下载二进制文件
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

如安装特定版本，则将`(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`替换成特定版本，例如v1.18.0，使用：

`curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl`

2. 使kubectl二进制文件可执行

`chmod +x ./kubectl`

3. 将二进制文件移到PATH中。

`sudo mv ./kubectl /usr/local/bin/kubectl`

4. 测试以确保您安装的版本是最新的

`kubectl version --client`

###### 使用本机软件包管理进行安装

1. 建立Repo

由于google被封，使用阿里的repo。在/etc/yum.repo.d/下建立kubenetes.repo文件，内容如下：
```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors
.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

2. 安装kubectl

`yum install -y kubectl`

##### 安装Minikube

###### 直接下载安装
```
curl -LO https://github.com/kubernetes/minikube/releases/download/v1.9.0/minikube-linux-amd64 \
   && sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

###### 利用安装包
```
curl -LO https://github.com/kubernetes/minikube/releases/download/v1.9.0/minikube-1.9.0-0.x86_64.rpm
   && sudo rpm -ivh minikube-1.9.0-0.x86_64.rpm
```

##### 配置minikube集群

1. 使用docker驱动程序启动集群：
```
minikube start --driver=docker
```

2. 将docker设置为默认驱动程序：
```
minikube config set driver docker
```

3. 配置内存
```
minikube config set memory 4096
```

4. Minikube的配置文件在如下路径
```
~/.minikube/machines/minikube/config.json
```

##### minikube启停集群

1. 要确认虚拟机管理程序和Minikube均已成功安装，可以运行以下命令来启动本地Kubernetes集群：

注：在 minikube start命令中设置--driver，在<driver_name>处输入安装的虚拟机监控程序的名称，该名称以小写字母表示。--driver在指定VM驱动程序文档时，可以使用值的完整列表（https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver）。
```
minikube start --driver=<driver_name>
```

如果拉取image失败，提示：
```
docker: Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
```
则需要从国内镜像拉取image：
```
minikube start –-image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
或 
minikube start –-image-mirror-country=cn
```

2. 一旦minikube start完成后，运行下面的命令检查集群的状态：
```
minikube status
```
如集群正在运行，则输出minikube status应类似于：
```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
3. 在确认Minikube是否与所选的虚拟机监控程序一起使用后，可以继续使用Minikube或停止集群。要停止集群，请运行：
```
minikube stop
```

##### 清理本地状态

如果以前安装了Minikube，然后运行：
```
minikube start
```
返回错误：
```
machine does not exist
```
那么需要清除minikube的本地状态：
```
minikube delete
```

##### 打开控制台
```
minikube dashboard
```

##### 检测集群状态，运行:
```
kubectl cluster-info
```

##### 查看集群状态（启动后）
```
kubectl config view
```

##### 检验Node状态:
```
kubectl get nodes

NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   11m   v1.15.0
```

##### 使用ssh进入Minikube虚机:
```
sudo minikube ssh
```

##### 打开Kubernetes控制台
Kubernete附带一个web，允许您在不与命令行交互的情况下管理集群。在minikube上默认安装并启用仪表板插件
```
$ minikube addons list
```

要直接在默认浏览器上打开，请使用:
```
$ minikube dashboard
```

获取仪表板的URL
```
$ minikube dashboard --url
```

### 安装chectl管理工具

#### 先决条件
* 目录/usr/local/bin位于用户中$PATH。

* 当前用户可使用sudo命令。

* 删除的所有旧版本或不需要的版本chectl。

#### 步骤
1. 在终端中运行以下命令（此命令将下载并执行install.sh脚本）：
```
$ bash <（curl -sL https://www.eclipse.org/che/chectl/）
```
2. 运行以下命令以验证所使用的chectl二进制文件是/usr/local/bin/chectl：
```
$ which chectl
/usr/local/bin/chectl
```
3. 运行以下命令，以验证chectl的版本是否为预期版本。
```
$ chectl --version
```

要确定最新的稳定版本，请参阅chectl版本列表，并搜索名称中不包含“next”的版本。

4. 阅读安装日志。
```
$ cat chectl-install.log
```

### 利用chectl安装CHE

使用minikube作为基础环境：
```
$ chectl server:start --platform minikube
```
