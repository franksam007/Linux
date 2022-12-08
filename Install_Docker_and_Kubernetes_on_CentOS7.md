## Install Docker
### 删除旧版docker
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 建立Repo
1. 按照自动管理repo所需包
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
2. 创建稳定版repo
* 官方源（比较慢）
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
可以选择国内的一些源地址：

* 阿里云
```
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
* 清华大学源
```
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

### 利用repo安装docker-ce
1. 安装最新版本
```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

2. 安装特定版本

显示所有版本：
```
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```
安装特定版本
```
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

### 非root用户
对于非root用户，如需要使用docker服务，须将其放入docker组：
* 创建docker组（如果不存在）
```
# Create docker group if doesn't exists
sudo groupadd docker
```

* 将用户加入docker组
```
# Add user to docker group
sudo usermod -aG docker $USER
```

`gpasswd -a USER docker`

该用户需要重新启动会话，才能访问。或者，通过下面命令刷新当前回话的组：
```
# Run the newgrp to change the current group ID during a login session
newgrp docker
```

### 手工安装
`$ sudo yum install /path/to/package.rpm`

注意：要安装docker-ce、docker-ce-cli、containerd.io三个包

### 设置国内镜像源
1. 创建配置文件
```
$ sudo vi /etc/docker/daemon.json
```
2. 配置国内镜像源
```
{
 "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com"]
}
```
上述镜像分别是官方中国及网易镜像。


### 启动服务
```
sudo systemctl enable docker #系统启动时自动启动
sudo systemctl start docker #启动服务
```

### 验证服务
```
# Check docker client without root permissions
docker images
docker run hello-world
```

## Install docker compose
1、下载docker compose

`sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

2、赋予运行权限

`sudo chmod +x /usr/local/bin/docker-compose`

3、建立软连接（可选）

`sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

## 安装Kubernetes
### 利用国内repo镜像
#### CentOS/RHEL/Fedora

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

#### Debian / Ubuntu
```
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

### 利用curl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.10.3/bin/linux/amd64/kubectl
```

可参考https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl

## 使用国内加速器
对于使用systemd的系统(Ubuntu 16.04+、Debian 8+、CentOS 7+)，可以创建 /etc/docker/daemon.json文件，并写入如下内容：
```
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://registry.docker-cn.com"
  ]
}
```
然后重新启动Docker服务
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```
注意：文件内容必须符合 json 规范，否则Docker无法启动。

|-|-|-|

|global	|proxy in China	|format	|example|

|-|-|-|

|dockerhub (docker.io)	|dockerhub.azk8s.cn	|dockerhub.azk8s.cn/<repo-name>/<image-name>:<version>	|dockerhub.azk8s.cn/microsoft/azure-cli:2.0.61 dockerhub.azk8s.cn/library/nginx:1.15|

|gcr.io	|gcr.azk8s.cn	|gcr.azk8s.cn/<repo-name>/<image-name>:<version>	|gcr.azk8s.cn/google_containers/hyperkube-amd64:v1.13.5|

|quay.io	|quay.azk8s.cn	|quay.azk8s.cn/<repo-name>/<image-name>:<version>	|quay.azk8s.cn/deis/go-dev:v1.10.0|

|-|-|-|

## 拉取gcr.io镜像
通过阿里云镜像+GitHub来在gcr.io镜像基础上，通过Dockerfile重新构建一个镜像，并修改标签为同名gcr.io镜像。

参考文档：
* 如何借助阿里云免费获取gcr.io上的镜像(https://blog.csdn.net/yjf147369/article/details/80290881)

以k8s.gcr.io/kube-apiserver-amd64:v1.10.3为例：

### Fork docker-library in GitHub
1. 参照上面的参考文章，fork了docker-library的repository。（如果想直接使用v1.10.3版本也可以直接folk我修改后的 docker-library )

2. 在kube-apiserver-amd64目录下创建一个v1.10.3子目录

3. 在该子目录下复制一个Dockerfile，修改基础镜像版本为v1.10.3，例子：
`FROM gcr.io/google_containers/kube-apiserver-amd64:v1.10.3`

### 在阿里云上新建镜像仓库
打开阿里云容器镜像服务：https://cr.console.aliyun.com ， 新建一个镜像仓库。

1. 选择离自己比较近的区域

2. 按提示填写信息

3. 选择”代码变更时自动构建镜像“和”海外机器构建“，并填写构建信息，比如：
```
代码分支：branches:master
Dockerfile目录：/kube-apiserver-amd64/v1.10.3
Dockerfile文件名：Dockerfile
镜像版本：v1.10.3
```

### 构建、拉取镜像和打gcr.io标签
1. 点击【管理】，选择【构建】，点击【立即构建】

2. 构建成功后，在【基础信息】中查看用法

3. 拉取新构建成功的镜像，比如：
```
# 拉取新构建的镜像
docker pull registry.cn-shenzhen.aliyuncs.com/cookcodeblog/kube-apiserver-amd64:v1.10.3
# 打上gcr.io同名标签
docker tag registry.cn-shenzhen.aliyuncs.com/cookcodeblog/kube-apiserver-amd64:v1.10.3 k8s.gcr.io/kube-apiserver-amd64:v1.10.3
# 查看镜像
docker images
# 删除新构建的镜像，只保留gcr.io镜像
docker rmi registry.cn-shenzhen.aliyuncs.com/cookcodeblog/kube-apiserver-amd64:v1.10.3
# 再次查看镜像
docker images
```

一个拉取kubeadm镜像的脚本请参见：https://github.com/cookcodeblog/k8s-deploy/blob/master/kubeadm/04_pull_kubernetes_images_from_aliyun.sh

### 查看gcr.io官方镜像
在前面的docker-library中需要知道准确的镜像名称和镜像标签。

在科学上网的情况下，打开 https://console.cloud.google.com/gcr/images/google-containers/GLOBAL ，在右边的“过滤条件“中输入关键词来搜索。

然后再选择正确的镜像。

通常，gcr.io官方镜像的命名规则为：
`gcr.io/google_containers/IMAGE_NAME:IMAGE_TAG`

例如：
`gcr.io/google_containers/kube-apiserver-amd64:v1.10.3`

