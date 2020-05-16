shadowsocks
===========

[![PyPI version]][PyPI]
[![Build Status]][Travis CI]
[![Coverage Status]][Coverage]

A fast tunnel proxy that helps you bypass firewalls.

Features:
- TCP & UDP support
- User management API
- TCP Fast Open
- Workers and graceful restart
- Destination IP blacklist

Server
------

### Install

Debian / Ubuntu:

    apt-get install python-pip
    pip install shadowsocks

CentOS:

    yum install python-setuptools && easy_install pip
    pip install shadowsocks

Windows:

See [Install Shadowsocks Server on Windows](https://github.com/shadowsocks/shadowsocks/wiki/Install-Shadowsocks-Server-on-Windows).

### Usage

    ssserver -p 443 -k password -m aes-256-cfb

To run in the background:

    sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start

To stop:

    sudo ssserver -d stop

To check the log:

    sudo less /var/log/shadowsocks.log

Check all the options via `-h`. You can also use a [Configuration] file
instead.

### Usage with Config File

[Create configeration file and run](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)

To start:

    ssserver -c /etc/shadowsocks.json


Documentation
-------------

You can find all the documentation in the [Wiki](https://github.com/shadowsocks/shadowsocks/wiki).

License
-------

Apache License

[Build Status]:      https://img.shields.io/travis/shadowsocks/shadowsocks/master.svg?style=flat
[Coverage Status]:   https://jenkins.shadowvpn.org/result/shadowsocks
[Coverage]:          https://jenkins.shadowvpn.org/job/Shadowsocks/ws/PYENV/py34/label/linux/htmlcov/index.html
[PyPI]:              https://pypi.python.org/pypi/shadowsocks
[PyPI version]:      https://img.shields.io/pypi/v/shadowsocks.svg?style=flat
[Travis CI]:         https://travis-ci.org/shadowsocks/shadowsocks


###  undefined symbol: EVP_CIPHER_CTX_cleanup
openssl升级到1.1.0以上版本，导致shadows2.8.2启动报undefined symbol: EVP_CIPHER_CTX_cleanup错误。

如果在安装完Shadows后，启动时报
```
AttributeError: /usr/local/ssl/lib/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup

shadows start failed
```
的错误。

在终端输入：
```
nautilus /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
```
具体路径不同，请根据报错路径而定，但目的只有一个，就是找到openssl.py文件。

如果nautilus指令报错，那就用cd命令到这个目录下，使用vim编辑修改openssl.py文件。

如果是用文本文档打开，那搜索CIPHER_CTX_cleanup，应该有两处，替换为CIPHER_CTX_reset，然后保存文件。

如果是用vim编辑，那么输入

:%s/cleanup/reset/
:x
然后重新运行Shadows即可


