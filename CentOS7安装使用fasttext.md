# 安装
## 要求
fastText建立在现代Mac OS和Linux发行版上。由于它使用C ++ 11功能，因此需要具有良好C ++ 11支持的编译器。这些包括 ：

（gcc-4.6.3或更高版本）或（clang-3.3或更高版本）

编译是使用Makefile进行的，因此您需要有一个make。对于单词相似性评估脚本，您将需要：

* python 2.6或更高版本
* numpy＆scipy

## 构建fastText作为命令行工具
为了构建fastText，请使用以下命令：
```
$ git clone https://github.com/facebookresearch/fastText.git
$ cd fastText
$ make
```
这将为所有类以及主二进制文件生成目标文件fasttext。如果您不打算使用默认的系统范围的编译器，请更新在Makefile开头定义的两个宏（CC和INCLUDES）。

注意：CentOS7默认没有装C++编译器，需要单独安装：
```
sudo yum install -y gcc-c++
```

同时需要编辑Makefile在Makefile开头定义的两个宏（CC和INCLUDES）
```
...
CXX = g++
...
```

## 构建fasttextpython模块
为了fasttext为python 构建模块，请使用以下命令：
```
$ git clone https://github.com/facebookresearch/fastText.git
$ cd fastText
$ export CXX=g++
$ export CC=gcc
$ sudo pip install .
$ # or :
$ sudo python setup.py install
```

然后验证安装是否顺利：
```
$ python
Python 2.7.15 |(default, May  1 2018, 18:37:05)
Type "help", "copyright", "credits" or "license" for more information.
>>> import fasttext
>>>
```
如果没有看到任何错误消息，则表明安装成功。
