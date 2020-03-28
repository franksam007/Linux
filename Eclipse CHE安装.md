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

`curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl`

如安装特定版本，则将`(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`替换成特定版本，例如v1.18.0，使用：

`curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl`\

###### 安装原生包



具有Kubernetes版本1.9或更高版本的Minikube的安装。请参阅安装Minikube。
