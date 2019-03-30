# Superset安装及汉化

## 安装
### 下载源码
Superset有多种安装方式，这里采用源码安装，以便于后边构建自己的版本。

`git clone https://github.com/apache/incubator-superset.git`

### 虚拟环境
```
# cd  incubator-superset
# virtualenv venv
# . ./venv/bin/activate
```
这里使用了Python虚拟环境，也可以不用！

Superset是用Python写的，所以Python环境提前搞好，最好用2.7.x版本，因为Airbnb生产环境用的是这个版本，兼容性最好。

### 编译
```
(venv) # cd superset/assets
(venv) # yarn
(venv) # yarn run build
```
### 安装
```
(venv) # cd ../../
(venv) # python setup.py install
```
墙内pip直接安装太费劲，所以设置一下国内镜像，参考https://www.jianshu.com/p/1e5e12454006

### 初始化

1. 给superset设置超级账户

`(venv) # fabmanager create-admin --app superset`

2. 初始化数据库

`(venv) # superset db upgrade`

3. 加载示例数据

`(venv) # superset load_examples`

4. 初始化角色和权限

`(venv) # superset init`

5. 启动

`(venv) # superset runserver -d`
访问http://localhost:8088/，看到如下界面


恭喜，安装成功！可以用刚才设置的账号登录了。

## 汉化

可以看到Superset自身支持国际化，右上角可以切换语言。但是使用中文时，汉化的不是很完全，需要自己动手做些事情。

### 修改config.py源码
```
# Setup default language
BABEL_DEFAULT_LOCALE = 'zh'
# Your application default translation path
BABEL_DEFAULT_FOLDER = 'superset/translations'
# The allowed translation for you app
LANGUAGES = {
    'en': {'flag': 'us', 'name': 'English'},
    'zh': {'flag': 'cn', 'name': 'Chinese'},
}
```
这里设置默认语言为中文，保留了英文，其余的全干掉了。

### 提取messages.pot文件
```
(venv) # cd $SUPERSET_HOME
(venv) # pybabel extract -F superset/translations/babel.cfg -k _ -k __ -k t -k tn -k tct -o superset/translations/messages.pot .
```

### 更新messages.po文件
```
(venv) # pybabel update -i superset/translations/messages.pot -d superset/translations/ -l zh
```

### 翻译messages.po文件
```
(venv) # cd superset/translations/zh/LC_MESSAGES
(venv) # vim messages.po
```
修改所有msgstr的值，根据msgid翻译。这是个体力活，慢慢翻吧！

### 编译messages.po
```
(venv) # cd $SUPERSET_HOME
(venv) # pybabel compile -d translations
(venv) # po2json -d superset -f jed1.x translations/zh/LC_MESSAGES/messages.po translations/zh/LC_MESSAGES/messages.json
```
这里用到了po2json，需提前用npm安装好。
`npm install po2json -g`
javascript文件中的翻译对照关系存放在messages.json文件中，根据文档还需要执行如下操作

### 重新编译、安装、启动Superset

这次再刷新页面，清空缓存，大部分已经汉化了。
但是还是有些没有汉化到的地方。很遗憾，这部分大多是写在代码里了，好硬！只能一个个修改了。

