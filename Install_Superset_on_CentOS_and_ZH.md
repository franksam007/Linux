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
这里设置默认语言为中文，保留了英文，其余的全干掉。

### 提取messages.pot文件
```
(venv) # cd $SUPERSET_HOME
(venv) # pybabel extract -F superset/translations/babel.cfg -k _ -k __ -k t -k tn -k tct -o superset/translations/messages.pot .
```

### 更新messages.po文件
```
(venv) # pybabel update -i superset/translations/messages.pot -d superset/translations/ -l zh
```

工具将查看文件中国际化标识的字符串：

* jsx文件中被编译的部分 格式为 {t('Datasource')} 
* html文件被编辑的部分 格式为 {{\_("Add Filter")}} 
* .py文件被编辑的部分 注意两个变量: list_columns , label_columns, 使用\_("msg")、\_\_("msg")等格式。


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
```
javascript文件中的翻译对照关系存放在messages.json文件中，根据文档还需要执行如下操作:
```
(venv) # po2json -d superset -f jed1.x translations/zh/LC_MESSAGES/messages.po translations/zh/LC_MESSAGES/messages.json
```

注意：po2json转成的文件和原生的superset需要的json文件格式有点不一样，需要自己修理一下，比如把’null,’删除，替换一下json文件的开头和结尾。

这里用到了po2json，需提前用npm安装好。

`npm install po2json -g`

### 关于JSX文件

superset是使用python的一个框架（reactjs?），将jsx文件（类似javascript文件）静态编译为不可修改的jsx.html文件，一般情况下，如果在jsx文件或者是jsx.html文件中的字段使用。

在JSX文件中，采用以下形式：

`{t('user')}`

格式的字段都是可以在messages.json文件中直接添加对应字段进行汉化的，而且汉化完不需要编译，直接进行重启即可：

`"User": ["用户"],`

但是有时候一些特殊的字段没有被t包括，可以修改jsx文件，只需要将对应的字段用{t()}然后进行一下包括就可以了。不过这里修改完之后是没有效果的，必须使用assets目录下的js_build.sh进行静态编译，将jsx文件编译为jsx.html之后就可以生效了，但是有时候某些字段会被包含在一个列表中，没有办法使用t进行包围，那么也可以直接使用汉字的形式。

### 重新编译、安装、启动Superset

这次再刷新页面，清空缓存，大部分已经汉化了。
但是还是有些没有汉化到的地方。很遗憾，这部分大多是写在代码里了,只能一个个修改了。

### 其他Tips

对于py文件，类的名字，一般就是url的地址

