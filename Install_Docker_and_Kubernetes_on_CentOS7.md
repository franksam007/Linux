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
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
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

`gpasswd -a USER docker`

该用户需要重新启动会话，才能访问

### 手工安装
`$ sudo yum install /path/to/package.rpm`

注意：要安装docker-ce、docker-ce-cli、containerd.io三个包

### 启动服务
```
sudo systemctl enable docker #系统启动时自动启动
sudo systemctl start docker #启动服务
```

## Install docker compose
1、下载docker compose

`sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

2、赋予运行权限

`sudo chmod +x /usr/local/bin/docker-compose`

3、建立软连接（可选）

`sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

## Install Kubernetes
### 建立Repo

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
```

### 安装Kubernetes
```
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```
