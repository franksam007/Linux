# Superset安装及汉化

## 一、安装
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

Superset是用Python写的，所以Python环境提前搞好，最好用3.6.x版本，因为Superset将来不再支持2.x。

### 安装必要Python包
`pip install -r incubator-superset/requirements.txt`

### 安装Nodejs和NVM
参见其他文档

### 前端编译
```
(venv) # cd superset/assets
(venv) # yarn
(venv) # yarn run build
```
注意也可以用npm编译（如果yarn不好使）
```
cd incubator-superset/suprset/assets # 进入到前端的工作目录
npm install
npm run build
```

### 安装
```
(venv) # cd ../../
(venv) # python setup.py install
```
墙内pip直接安装太费劲，所以设置一下国内镜像，参考https://www.jianshu.com/p/1e5e12454006

### 生成环境变量和编辑配置文件
设置环境变量：为了使得自定义配置生效，我们需要确保 superset_config.py 文件路径在PYTHONPATH 变量里面

官方提供的配置模版如下：
```
#---------------------------------------------------------
# Superset specific config
#---------------------------------------------------------
ROW_LIMIT = 5000

SUPERSET_WEBSERVER_PORT = 8088
#---------------------------------------------------------

#---------------------------------------------------------
# Flask App Builder configuration
#---------------------------------------------------------
# Your App secret key
SECRET_KEY = '\2\1thisismyscretkey\1\2\e\y\y\h'

# The SQLAlchemy connection string to your database backend
# This connection defines the path to the database that stores your
# superset metadata (slices, connections, tables, dashboards, ...).
# Note that the connection information to connect to the datasources
# you want to explore are managed directly in the web UI
SQLALCHEMY_DATABASE_URI = 'sqlite:////path/to/superset.db'

# Flask-WTF flag for CSRF
WTF_CSRF_ENABLED = True
# Add endpoints that need to be exempt from CSRF protection
WTF_CSRF_EXEMPT_LIST = []
# A CSRF token that expires in 1 year
WTF_CSRF_TIME_LIMIT = 60 * 60 * 24 * 365

# Set this API key to enable Mapbox visualizations
MAPBOX_API_KEY = ''
```

可根据需要修改元数据连接，例如：
`SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://superset:superset@mysql-db-host-name:3306/superset?charset=utf8'`

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

`(venv) # superset runserver -d -p 8088`
访问http://localhost:8088/，看到界面


可以用刚才设置的账号登录了。

## 二、汉化

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

