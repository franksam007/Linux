# Superset安装及汉化

## 一、安装
### 下载源码
Superset有多种安装方式，这里采用源码安装，以便于后边构建自己的版本。

`git clone https://github.com/apache/incubator-superset.git`

### 虚拟环境
```
# cd  incubator-superset
# virtualenv venv　或 python -m venv venv

# 如果报错
# Error: Command '['/home/.../venv/bin/python', '-Im', 'ensurepip', '--upgrade', '--default-pip']' returned non-zero exit status 1.
# 可先创建没有PIP的虚拟环境，然后启动虚拟环境后，自行安装pip
# python -m venv --without-pip venv

# . ./venv/bin/activate

# 在直接创建虚拟环境报错的情况下，自行安装pip
# （虚拟环境下）curl https://bootstrap.pypa.io/get-pip.py | python
# 更新pip
# （虚拟环境下）pip install --upgrade pip
# 
```
这里使用了Python虚拟环境，也可以不用！

Superset是用Python写的，所以Python环境提前搞好，最好用3.6.x版本，因为Superset将来不再支持2.x。最好同时安装gcc、gcc-c++以及python开发包：
```
# 安装开发所需python包是需要下列系统包
# yum install gcc
# yum install redhat-rpm-config
# yum install gcc-c++
# yum install python36-devel 
# yum install mysql-devel  #安装开发所需python包时需要，具体是mysqlclient-1.4.2需要
```
因为安装某些python包时需要编译

### 安装必要Python包
`pip install -r incubator-superset/requirements.txt`

`pip install -r incubator-superset/requirements-dev.txt`

如果离线，可以先在有网络连接的机器上下载python包，然后再安装：

`pip download -d ./pkg -r requirements.txt`

`pip install --no-index --find-links=file:./pkg -r requirements.txt`

如果出现`mysql_config not found`错误，则需要安装libmysqlclient-dev

`(Ubuntu)apt install libmysqlclient-dev`

注意：还需要安装Python访问数据库的包。

### 安装Nodejs和NVM
参见其他文档

### 前端编译
```
(venv) # cd superset/assets
(venv) # yarn
(venv) # yarn run build
```

#注意新版本应将将前端剥离出来，放到superset-frontend，则需要执行（2020年2月23日）
```
(venv) # cd superset-frontend
(venv) # yarn
(venv) # yarn run build
```

注意也可以用npm编译（如果未安装yarn）
```
cd incubator-superset/suprset/assets # 进入到前端的工作目录。注意：新版本assets的内容移动到superset-frontend
npm install
npm run build
```

### 创建数据库
```
CREATE DATABASE biga DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_bin;
CREATE USER biga@localhost IDENTIFIED BY '3GBigA';
GRANT ALL ON biga.* TO biga@localhost WITH GRANT OPTION;
flush privileges;
```

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

主要需要修改：
* SQLALCHEMY_DATABASE_URI, 默认使用SQLLite ~/.superset/superset.db
* SECRET_KEY, 用一个长的随机字符串

可根据需要修改元数据连接，例如：
`SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://superset:superset@mysql-db-host-name:3306/superset?charset=utf8'`

### 安装
从源文件安装
```
(venv) # cd ../../   #进入代码顶层目录（目前是incubator-superset)
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

`(venv) # superset runserver -d -p 8088`

注意上一命令已经废弃，会报错：

`[DEPRECATED] As of Flask >=1.0.0, this command is no longer supported, please use `flask run` instead, as documented in our CONTRIBUTING.md`

使用以下命令（开发模式）(借用了Docker的entrypoint.sh）
```
celery worker --app=superset.sql_lab:celery_app --pool=gevent -Ofair &
FLASK_ENV=development FLASK_APP=superset:app flask run -p 8088 --with-threads --reload --debugger --host=0.0.0.0
```

生产模式使用以下命令：
```
gunicorn --bind  0.0.0.0:8088 \
        --workers $((2 * $(getconf _NPROCESSORS_ONLN) + 1)) \
        --timeout 60 \
        --limit-request-line 0 \
        --limit-request-field_size 0 \
        superset:app
```

访问http://localhost:8088/，看到界面


可以用刚才设置的账号登录了。


========================

下面为直接安装方式
1. 利用pip安装supersets

`pip install superset`

2.初始化DB
`superset db upgrade`

3. 创建管理员账号
```
$ export FLASK_APP=superset
flask fab create-admin
```
4. 导入样例数据

`superset load_examples`

5. 创建默认角色和权限
```
superset init
```
6. 启动开发服务器
```
superset run -p 8080 --with-threads --reload --debugger
```


## 二、汉化

可以看到Superset自身支持国际化，右上角可以切换语言。但是使用中文时，汉化的不是很完全，需要自己动手做些事情。

### 修改config.py源码
注意：应在安装之前完成，源代码目录中的修改才能反映到安装的版本中。如果顺序颠倒了，可通过重新执行python setup.py install更新安装的版本，或者直接到系统的python/site-packages/相依目录下修改

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

#### 注意需要更改的一些文件包括


##### 更新图像
```
\superset\assets\images\s.png
\superset\assets\images\superset.png
\superset\assets\images\superset-logo@2x.png
```

##### 页面添加翻译字符串封装
```
\superset\templates\email\role_extended.txt
\superset\templates\email\role_granted.txt
\superset\templates\superset\import_dashboards.html
\superset\templates\superset\paper-theme.html
\superset\templates\superset\theme.html
\superset\templates\superset\traceback.html
\superset\assets\src\explore\components\QueryAndSaveBtns.jsx
\superset\assets\src\SqlLab\components\QuerySearch.jsx
```
```
superset/views/core.py
    def validate_sql_json(self)
	...
	        msg = _(
                f"{validator.name} was unable to check your query.\nPlease "
                "make sure that any services it depends on are available\n"
                f"Exception: {e}"
            )
	...
	
```
	此部分在抽取消息是，报validator未定义错误，暂时取消\_()函数的包装

##### 源代码逻辑修改
1. 表格修改
```
\superset\assets\src\profile\components\Favorites.jsx
\superset\assets\src\profile\components\CreatedContent.jsx
\superset\assets\src\profile\components\UserInfo.jsx
\superset\assets\src\profile\components\RecentActivity.jsx
```
用到了`\superset\assets\src\components\TableLoader.jsx`，而组件使用了reactable-arc中的Table控件，且其columns设置成字符串数组，而原控件支持将columns设为{key, label}对象数组，从而利用label来国际化（汉化），否则表头与作为字符串数组columns中的字段名称相同。


### 翻译messages.po文件
```
(venv) # cd superset/translations/zh/LC_MESSAGES
(venv) # vim messages.po
```
修改所有msgstr的值，根据msgid翻译。

### 编译messages.po
```
(venv) # cd $SUPERSET_HOME
(venv) # pybabel compile -d translations
```
修改*.py、messages.po、messsages.mo，不需要重启服务，系统会动态导入，刷新页面即可看到效果。

javascript文件中的翻译对照关系存放在messages.json文件中，根据文档还需要执行如下操作:
```
(venv) # po2json -d superset -f jed1.x translations/zh/LC_MESSAGES/messages.po translations/zh/LC_MESSAGES/messages.json
```

注意：po2json转成的文件和原生的superset需要的json文件格式有点不一样，需要自己修理一下，比如把’null,’删除，替换一下json文件的开头和结尾。

这里用到了po2json，需提前用npm安装好。

`npm install po2json -g`

### 关于JSX文件

superset是使用reactjs框架（reactjs?），将jsx文件（类似javascript文件）静态编译为不可修改的jsx.html文件，一般情况下，如果在jsx文件或者是jsx.html文件中的字段使用。

在JSX文件中，采用以下形式：

`{t('user')}`

格式的字段都是可以在messages.json文件中直接添加对应字段进行汉化的，而且汉化完不需要编译，直接进行重启即可：

`"User": ["用户"],`

但是有时候一些特殊的字段没有被t包括，可以修改jsx文件，只需要将对应的字段用{t()}然后进行一下包括就可以了。不过这里修改完之后是没有效果的，必须使用assets目录下的js_build.sh进行静态编译，将jsx文件编译为jsx.html之后就可以生效了，但是有时候某些字段会被包含在一个列表中，没有办法使用t进行包围，那么也可以直接使用汉字的形式。

### 重新编译、安装、启动Superset

这次再刷新页面，清空缓存，大部分已经汉化了。

但是还是有些没有汉化到的地方。很遗憾，这部分大多是写在代码里了,只能一个个修改了。

重新编译前，需要清除以下目录，否则`python setup.py install`并不更新相应程序：
```
顶层/apache_superset.egg-info
顶层/build
顶层/dist
顶层/venv/lib/python3.6/site-packages/apache_superset-0.999.0.dev0-py3.6.egg
```

在

### 其他Tips

对于py文件，类的名字，一般就是url的地址

## 三、开发

### 本地静态目录
```
/static
    ---src          //业务组件
    ---stylesheets  //样式文件（需要注意的两个文件）
        --less
            --cosmo
                --bootswatch.less   // 修改样式的地方
        --superset.less             // 修改全局样式的地方


注意： 本地安装开发环境的时候，react版本最好不要升级,坑也挺多,就按照他的版本来.
       如果你非要升级的话，有几点需要提醒你：
        1. prop-types 会提示相关报错, 问题在于react版本。@15和@16版本中prop-types的差异,升级的react版本到       @16.**的注意一下

        2.会提示Cannot find module 'react/lib/*****'等未知模块 
            请参考  https://www.jianshu.com/p/43b7db635f8c 按步骤运行！

        3.想找react-router? 找不到的，他压根就不是spa, 虽然在react写的 （此时劝诫自己要压住怒火）
            
            提示： superset/views/core.py   // 模块注册和页面跳转
```

### 模板引擎
```
superset/templates

flash_appbuilder/templates

注：如果这里模板调用找不到的话 在和superset同级的flash_appbuilder/templates里面找，
    这两个文件夹是可以相互调用的.（千万注意） 
    两个模板有很多不起作用的模板，不知道为什么要留在里面，增加了很多不必要的难度
```

### 路由
```
/views
    ---core.py  
        class    // 创建视图对象，
        appbuilder.add_view(   // 注册路由文件    
                    视图对象，
                    mtrid, 
                    msgstr, 
                    图标，...  
                )  
        appbuilder.add_link(    //添加子路由
                    mtrid       //字符串id 
                    label       //名称
                    href        // 跳转路径
                    category    // 属于哪个类目
                    category_label  // 类目名称
        )
        
    注意： views/core.py 是路由，通过上述代码可以看懂路由是怎么跳转和调用模板渲染的。
```
### 外部认证及单点登录

#### 外部认证
superset是由flask_appbuilder生成，继承了flask_appbuilder的外部认证设置。
```
import os
from flask_appbuilder.security.manager import (
    AUTH_OID,
    AUTH_REMOTE_USER,
    AUTH_DB,
    AUTH_LDAP,
    AUTH_OAUTH,
)

basedir = os.path.abspath(os.path.dirname(__file__))

# Your App secret key
SECRET_KEY = "\2\1thisismyscretkey\1\2\e\y\y\h"

# The SQLAlchemy connection string.
SQLALCHEMY_DATABASE_URI = "sqlite:///" + os.path.join(basedir, "app.db")
# SQLALCHEMY_DATABASE_URI = 'mysql://myapp@localhost/myapp'
# SQLALCHEMY_DATABASE_URI = 'postgresql://root:password@localhost/myapp'

# Flask-WTF flag for CSRF
CSRF_ENABLED = True

# ------------------------------
# GLOBALS FOR APP Builder
# ------------------------------
# Uncomment to setup Your App name
# APP_NAME = "My App Name"

# Uncomment to setup Setup an App icon
# APP_ICON = "static/img/logo.jpg"

# ----------------------------------------------------
# AUTHENTICATION CONFIG
# ----------------------------------------------------
# The authentication type
# AUTH_OID : Is for OpenID
# AUTH_DB : Is for database (username/password()
# AUTH_LDAP : Is for LDAP
# AUTH_REMOTE_USER : Is for using REMOTE_USER from web server
AUTH_TYPE = AUTH_OAUTH

OAUTH_PROVIDERS = [
    {
        "name": "twitter",
        "icon": "fa-twitter",
        "remote_app": {
            "consumer_key": os.environ.get("TWITTER_KEY"),
            "consumer_secret": os.environ.get("TWITTER_SECRET"),
            "base_url": "https://api.twitter.com/1.1/",
            "request_token_url": "https://api.twitter.com/oauth/request_token",
            "access_token_url": "https://api.twitter.com/oauth/access_token",
            "authorize_url": "https://api.twitter.com/oauth/authenticate",
        },
    },
    {
        "name": "google",
        "icon": "fa-google",
        "token_key": "access_token",
        "remote_app": {
            "consumer_key": os.environ.get("GOOGLE_KEY"),
            "consumer_secret": os.environ.get("GOOGLE_SECRET"),
            "base_url": "https://www.googleapis.com/oauth2/v2/",
            "request_token_params": {"scope": "email profile"},
            "request_token_url": None,
            "access_token_url": "https://accounts.google.com/o/oauth2/token",
            "authorize_url": "https://accounts.google.com/o/oauth2/auth",
        },
    },
    {
        "name": "azure",
        "icon": "fa-windows",
        "token_key": "access_token",
        "remote_app": {
            "consumer_key": os.environ.get("AZURE_APPLICATION_ID"),
            "consumer_secret": os.environ.get("AZURE_SECRET"),
            "base_url": "https://login.microsoftonline.com/{AZURE_TENANT_ID}/oauth2",
            "request_token_params": {
                "scope": "User.read name preferred_username email profile",
                "resource": os.environ.get("AZURE_APPLICATION_ID"),
            },
            "request_token_url": None,
            "access_token_url": "https://login.microsoftonline.com/{AZURE_TENANT_ID}/oauth2/token",
            "authorize_url": "https://login.microsoftonline.com/{AZURE_TENANT_ID}/oauth2/authorize",
        },
    },
]

# Uncomment to setup Full admin role name
# AUTH_ROLE_ADMIN = 'Admin'

# Uncomment to setup Public role name, no authentication needed
# AUTH_ROLE_PUBLIC = 'Public'

# Will allow user self registration
AUTH_USER_REGISTRATION = True

# The default user self registration role
AUTH_USER_REGISTRATION_ROLE = "Admin"

# When using LDAP Auth, setup the ldap server
# AUTH_LDAP_SERVER = "ldap://ldapserver.new"
# AUTH_LDAP_USE_TLS = False

# Uncomment to setup OpenID providers example for OpenID authentication
# OPENID_PROVIDERS = [
#    { 'name': 'Google', 'url': 'https://www.google.com/accounts/o8/id' },
#    { 'name': 'Yahoo', 'url': 'https://me.yahoo.com' },
#    { 'name': 'AOL', 'url': 'http://openid.aol.com/<username>' },
#    { 'name': 'Flickr', 'url': 'http://www.flickr.com/<username>' },
#    { 'name': 'MyOpenID', 'url': 'https://www.myopenid.com' }]
# ---------------------------------------------------
# Babel config for translations
# ---------------------------------------------------
# Setup default language
BABEL_DEFAULT_LOCALE = "en"
# Your application default translation path
BABEL_DEFAULT_FOLDER = "translations"
# The allowed translation for you app
LANGUAGES = {
    "en": {"flag": "gb", "name": "English"},
    "pt": {"flag": "pt", "name": "Portuguese"},
    "pt_BR": {"flag": "br", "name": "Pt Brazil"},
    "es": {"flag": "es", "name": "Spanish"},
    "de": {"flag": "de", "name": "German"},
    "zh": {"flag": "cn", "name": "Chinese"},
    "ru": {"flag": "ru", "name": "Russian"},
}
# ---------------------------------------------------
# Image and file configuration
# ---------------------------------------------------
# The file upload folder, when using models with files
UPLOAD_FOLDER = basedir + "/app/static/uploads/"

# The image upload folder, when using models with images
IMG_UPLOAD_FOLDER = basedir + "/app/static/uploads/"

# The image upload url, when using models with images
IMG_UPLOAD_URL = "/static/uploads/"
# Setup image size default is (300, 200, True)
# IMG_SIZE = (300, 200, True)

# Theme configuration
# these are located on static/appbuilder/css/themes
# you can create your own and easily use them placing them on the same dir structure to override
# APP_THEME = "bootstrap-theme.css"  # default bootstrap
# APP_THEME = "cerulean.css"
# APP_THEME = "amelia.css"
# APP_THEME = "cosmo.css"
# APP_THEME = "cyborg.css"
# APP_THEME = "flatly.css"
# APP_THEME = "journal.css"
# APP_THEME = "readable.css"
# APP_THEME = "simplex.css"
# APP_THEME = "slate.css"
# APP_THEME = "spacelab.css"
# APP_THEME = "united.css"
# APP_THEME = "yeti.css"

```

#### CAS单点登录
修改config.py
```
from flask_appbuilder.security.manager import AUTH_REMOTE_USER

AUTH_TYPE=AUTH_REMOTE_USER

from custom_sso_security_manager import CustomSsoSecurityManager
CUSTOM_SECURITY_MANAGER = CustomSsoSecurityManager
AUTH_USER_REGISTRATION = True   #允许用户注册
AUTH_USER_REGISTRATION_ROLE = "Gamma"  #设置默认添加用户角色
```

superset根目录添加custom_sso_security_manager.py
```
from superset.security import SupersetSecurityManager
import logging
from flask_appbuilder.security.views import AuthRemoteUserView, expose
from flask_appbuilder.const import LOGMSG_WAR_SEC_LOGIN_FAILED
from flask import request,g, redirect
from flask_login import login_user, logout_user
import requests
import json

logger = logging.getLogger(__name__)


CAS_LOGIN_SERVER_URL = 'http://xxxxx/api/login/casLogin'
CAS_CHECK_SERVER_URL = 'http://xxxxx/api/login/currentUser'
CAS_LOGINOUT_SERVER_URL = 'http://xxxxx/api/login/out'

class MyAuthRemoteUserView(AuthRemoteUserView):
    # this front-end template should be put under the folder `superset/templates/appbuilder/general/security`
    # so that superset could find this templates to render
    login_template = 'appbuilder/general/security/login_my.html'
    title = "My Login"

    # this method is going to overwrite 
    # https://github.com/dpgaspar/Flask-AppBuilder/blob/master/flask_appbuilder/security/views.py#L556
    @expose('/login/', methods=['GET', 'POST'])
    def login(self):
        print("My special login...")
        if not g.user or not g.user.get_id():
            return redirect(CAS_LOGIN_SERVER_URL+"?redirect="+request.host_url+"logincas")

        print("loginSSO")
        print(request.host_url)

    @expose('/logincas/', methods=['GET', 'POST'])
    def logincas(self):
        token=request.args.get('token')
        print("logincas"+token)
        manager=self.appbuilder.sm

        result = requests.get(CAS_CHECK_SERVER_URL + '?token=' + token)
        userCAS = json.loads(result.content)
        username=userCAS["loginName"]
        user = manager.find_user(username=username)
        print(user)

        # User does not exist, create one if auto user registration.
        if user is None and manager.auth_user_registration:
            user = manager.add_user(
            # All we have is REMOTE_USER, so we set
            # the other fields to blank.
                username=username,
                first_name=username.split('@')[0],
                last_name='-',
                email=username,
                role=manager.find_role(manager.auth_user_registration_role))

        # If user does not exist on the DB and not auto user registration,
        # or user is inactive, go away.
        elif user is None or (not user.is_active):
            logger.info(LOGMSG_WAR_SEC_LOGIN_FAILED.format(username))
            return None
            
        manager.update_user_auth_stat(user)
        print(user)
        login_user(user, remember=False)
        return redirect(self.appbuilder.get_url_for_index)

    @expose("/logout/")
    def logout(self):
        logout_user()
        print("loginout")
        return redirect(CAS_LOGINOUT_SERVER_URL+'?redirect='+request.host_url)
       

class CustomSsoSecurityManager(SupersetSecurityManager):
    authremoteuserview=MyAuthRemoteUserView
```
Gamma角色添加权限

默认Gamma角色不能访问库，需设置角色，添加all database access on all_database_access权限（全部数据库）。
