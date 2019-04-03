在CentOS 7上，当使用`yum install mysql`时，实际上是安装的MariaDB的客户端。

### 完整安装MariaDb
`yum install -y mariadb-server mariadb`

### 配置随机启动
`systemctl enable mariadb`
