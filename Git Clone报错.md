[root@server data]# git clone https://github.com/pingcap/tidb-docker-compose.git 

Cloning into 'tidb-docker-compose'... fatal: unable to access 'https://github.com/pingcap/tidb-docker-compose.git/': Peer reports incompatible or unsupported protocol version. [root@server data]# [root@server data]# git --version git version 1.8.3.1

查找原因是curl,nss版本低的原因，解决办法就是更新nss,curl。

[root@server data]# yum update nss curl


