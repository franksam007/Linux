# jsreport研究
## 目录<a name='toc'></a>
1. [CentOS安装](#install)

    1.1 [安装Nodejs](#install_nodejs)
  
    1.2 [安装jsreport](#install_jsreport)

2. [配置](#config)
  
    2.1 [配置来源](#config_source)
  
    2.2 [配置扩展](#config_extension)
  
    2.3 [Web服务器配置](#web_config)
  
    2.4 [存储配置](#store_config)
  
    2.5 [目录配置](#dir_config)
  
    2.6 [允许本地文件和本地模块](#allow_local)
  
    2.7 [渲染配置](#render_config)
  
    2.8 [模板引擎配置](#tpl_engine_config)
  
    2.9 [日志配置](#log_config)
  
    2.10 [配置文件示例](#config_sample)

3. [存储配置详解](#store_config_detail)

    3.1 [默认文件系统存储](#fs_store)
    
    3.2 [模板存储扩展](#store_extension)
    
    3.3 [不同存储之间迁移](#store_migration)
    
    3.4 [REST API](#store_api)
    
    3.5 [javascript API](#js_api)

4. [扩展](#extensions)

    4.1 [Base](#base)
    
    4.2 [Inline Data](#inline_data)
    
    4.3 [Scripts](#scripts)
    
    4.4 [Assets](#assets)
    
    4.5 [Child Template](#child_template)
    
    4.6 [Report](#report)
    
    4.7 [Scheculing](#scheduling)
    
    4.8 [Authenticatio](#authentication)
    
    4.9 [Authorization](#authorization)
    
    4.10 [Public templates](#public_template)
    
    4.11 [Resources](#resource)
    
    4.12 [Tags](#tag)
    
    4.13 [CLI](#cli)

5. [转换引擎（算法）](#recipes)
    
    5.1 [HTML](#html_recipe)
    
    5.2 [Chrome PDF](#chrome_pdf)
    
    5.3 [Chrome Image](#chrome_image)
    
    5.4 [Xlsx](#xlsx)
    
    5.5 [Html to Xlsx](#htmo_to_xlsx)
    
    5.6 [Docx](#docx)
    
    5.7 [Html embedded in docx](#html_in_docx)
    
    5.8 [docxtemplater](#docxtemplater)
    
    5.9 [Phantom pdf](#phantom_pdf)
    
    5.10 [Phantom Image](#phantom_image)
    
6. [模板引擎](#engines)

    6.1 [handlebars](#handlebars)
    
    6.2 [jsrender](#jsrender)
    
    6.3 [EJS](#ejs）
    
7. [API](#api)

Ex.[许可](#license)

## 1. CentOS安装<a name='install'></a>    [返回目录](#toc)
jsreport整体上利用nodejs开发，界面主要体现在jsreport-studio上。

### 1.1 安装Nodejs<a name='install_nodejs'></a>    [返回目录](#toc)
jsreport需要node.js (>= 8.9)和npm (>= 6.x)。

### 1.2 安装jsreport<a name='install_jsreport'></a>    [返回目录](#toc)
#### 一般流程
利用npm安装jereport
```
npm install jsreport-cli -g
mkdir jsreportapp
cd jsreportapp
jsreport init
jsreport configure
jsreport start
```

#### 在CentOS安装的完整版本
```
# 安装node.js，利用nvm（node version manager)
sudo yum install wget
# 安装nvm（主要版本号只是范例）
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
# 重新打开终端，利用nvm安装（主要版本号只是范例）
nvm install 8.11.3

# 安装配置jsreport
mkdir jsreportapp
cd jsreportapp
npm i -g jsreport-cli
jsreport init
jsreport configure

# 安装chrome（chrome用来渲染报告）
wget -qO- https://intoli.com/install-google-chrome.sh | bash

# 编辑jsreport配置（nano是一个字符终端的文本编辑器，比vi简单。也可使用vi或其他编辑器编辑）
sudo yum install nano
nano jsreport.config.json

# 修改下列内容，使chrome处于更弱安全模式
# 这是使之在CentOS下工作的唯一方式
"chrome": {
  "launchOptions": {
    "args": ["--no-sandbox"]
  }
}

# 保存配置并启动jsreport，服务将运行在5488端口
jsreport start

# 设置jsreport随系统启动（PM2是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务）
npm install pm2 -g
pm2 start server.js
pm2 startup
# run the output of previous command
```

#### Headless CentOS（无图形终端）上需要安装的依赖包（chrome headless的依赖包）
需要确定已安装所有的依赖包。可使用`ldd chrome | grep not`检查缺失的依赖包。

CentOS上的依赖包：
* pango.x86_64
* libXcomposite.x86_64
* libXcursor.x86_64
* libXdamage.x86_64
* libXext.x86_64
* libXi.x86_64
* libXtst.x86_64
* cups-libs.x86_64
* libXScrnSaver.x86_64
* libXrandr.x86_64
* GConf2.x86_64
* alsa-lib.x86_64
* atk.x86_64
* gtk3.x86_64
* ipa-gothic-fonts
* xorg-x11-fonts-100dpi
* xorg-x11-fonts-75dpi
* xorg-x11-utils
* xorg-x11-fonts-cyrillic
* xorg-x11-fonts-Type1
* xorg-x11-fonts-misc

安装所有依赖包后，需要执行以下命令更新nss库：

`yum update nss -y`

## 2. 配置<a name='config'></a>    [返回目录](#toc)

使jsreport适应需求的最简单方法是更改其配置。jsreport配置提供了许多选项，例如更改http端口，将存储库提供程序设置为不同的机制等等。

可以使用以下命令获取支持的配置选项的列表

`jsreport help config`

### 2.1 配置来源(Configuration sources)<a name='config_source'></a>    [返回目录](#toc)
jsreport将配置文件中的配置，环境变量，命令行参数以及应用程序代码中的配置直接按此确切顺序合并。

#### 配置文件
配置文件是适应jsreport的最常用方法。如果遵循安装说明，jsreport.config.json通常会预先创建默认值。

jsreport还会加载dev.config.json或prod.config.json基于NODE_ENV=developmentor NODE_ENV=production环境变量（如果存在）。

还可以使用configFile=path选项明确指定配置文件，该选项可以从配置源方法之一传递。配置文件路径可以是相对路径，也可以是绝对路径。

Hint: 启动实例时，您应该在日志的第一行中看到当前应用的配置文件名。

`info: Initializing jsreport@2.0.0 in development mode using configuration file: jsreport.config.json`

#### 环境变量
在jsreport启动期间，还将收集环境变量并将其合并到最终配置中。可以使用它来更改端口，例如：

* Unix：

`httpPort=3000 jsreport start`

* Windows：
```
set httpPort=3000
jsreport start
```
即使httpPort在配置文件中有该条目，这也会在端口3000上启动jsreport，因为环境变量具有更高的优先级。

如果要使用环境变量来配置复杂对象，则应使用以下命令在键中分隔嵌套路径：

* Unix：

`extensions_authentication_admin_username=john jsreport start`

* Windows：
```
set extensions_authentication_admin_username=john
jsreport start
```

\_分隔符的另一种替代方法是使用:分隔符

* Unix：

`env extensions:authentication:admin:username=john jsreport start`

* Windows：
```
set extensions:authentication:admin:username=john
jsreport start
```

#### 命令行参数
命令行参数也被解析。例如，这对于更改端口非常方便：

`jsreport start --httpPort=3000`

复杂对象的配置应使用. 分隔符

`jsreport start --extensions.authentication.admin.username=john`

#### 应用代码

最后一个选项是编辑server.js文件，这是默认安装的一部分。将jsreport集成到现有的node.js应用程序中时，这通常很常见。请注意，如果使用预编译的jsreport二进制文件，则无法使用此方法。

```
const jsreport = require('jsreport')({
  httpPort: 3000
})
```

### 2.2 配置扩展<a name='config_extension'></a>    [返回目录](#toc)
每个扩展名（配方，存储...）通常都提供一些选项，您可以应用这些选项并调整其行为。这些选项通常可以通过顶级extensions属性下的标准配置来设置，您可以在其中将带有扩展名的特定扩展选项放入其中。例如，可以在配置中的相同命名节点下配置身份验证。
```
"extensions": {
  "authentication": {     
      "admin": {
          "username" : "admin",
          "password": "password"
      }
  }
}
```
名称中带有连字符的扩展名（例如html-to-xlsx，例如）还支持以驼峰形式接收带有名称的配置，因此以下两个示例均适用于名称中带有连字符的扩展名
```
"extensions": {
  "html-to-xlsx": {
    ...
  }
}
"extensions": {
  "htmlToXlsx": {
    ...
  }
}
```
当将配置指定为命令行参数或环境变量时，这种对驼峰式扩展形式的支持也适用，这在难以通过连字符传递参数或环境变量的环境中工作时非常方便
```
jsreport start --extensions.htmlToXlsx.someConfig value
extensions_htmlToXlsx_someConfig=value jsreport start
```
请参考特定扩展的文档以找到拥有的配置选项。通常Configuration在您可以找到它的部分。

#### 禁用扩展
可以通过enabled: false在特定扩展名的配置中进行设置来禁用扩展名。例如，可以使用此配置禁用jsreport studio，API和身份验证。
```
{
  "extensions": {    
    "authentication": {      
      "enabled": false
    },
    "studio": {    
      "enabled": false
    },
    "express": {
      "enabled": false
    }
  }
}
```
### 2.3 Web服务器配置<a name='web_config'></a>    [返回目录](#toc)
* httpPort _(number)_ ：运行jsreport的 http端口，如果同时指定`httpPort`和`httpsPort`，则jsreport将自动创建从http到https的http重定向（如果指定了`httpPort`和h`ttpsPort`），则将使用默认`process.env.PORT`

* httpsPort _(number)_ ：jsreport的https端口

certificate _object_ ：https所使用key和cert文件的路径
```
"certificate": {
    "key": "certificates/jsreport.net.key",
    "cert": "certificates/jsreport.net.cert"
}
```
或者，如果证书是.pfx文件，则可以使用pfx和passphrase选项
```
"certificate": {
    "pfx": "certificates/jsreport.net.pfx",
    "passphrase": "<if pfx file is protected specify the password here>"
}
```
* hostname _(string)_ ：用于jsreport服务器的主机名（optional）

* extensions.express.inputRequestLimit _(string)_ ：传入请求大小的可选限制，默认为2mb

* appPath _(string)_ ：如果在http://appdomain.com/reporting上运行应用程序，然后将“/reporting”设置为appPath，则可以选择设置应用程序路径。缺省行为是假定jsreport在代理后面运行，因此需要进行url重写`/reporting -> /`才能使其正常工作。如果您设置中没有procy+url重写，参见mountOnAppPath。

* mountOnAppPath _(boolean)_ ：将此选项与一起使用appPath。它指定是否所有jsreport路由都应appPath以前缀提供，因此使appPath应用程序成为新的根URL

### 2.4 存储配置<a name='store_config'></a>    [返回目录](#toc)
* store _(object)_ ：jsreport支持多种实现来存储模板。具体实现是基于store.provider属性来区分的。预先创建的配置文件中的预定义值是fs使用jsreport-fs-store在文件系统上存储报告模板的值。另外，可以安装其他扩展提供模板存储并进行更改store以反映它。您可以在此处找到可用商店驱动程序的列表，以及如何配置它们的更多详细信息。

* blobStorage _)object)_ :可选，指定用于存储报告的存储类型。具体实现是基于blobStorage.provider属性来区分的。它可以是fs，memory或gridFS。在完整版jsreport中默认为fs，或在jsreport集成到现有的node.js应用程序时默认memory。

### 2.5 目录配置<a name='dir_config'></a>    [返回目录](#toc)
* rootDirectory_(string)_ ：（可选）指定应用程序的根目录和jsreport在哪里搜索扩展名

* tempDirectory_(string)_ ：（可选）指定应用程序存储临时文件的目录的绝对或相对路径

### 2.6 允许本地文件和本地模块<a name='allow_local'></a>    [返回目录](#toc)
* allowLocalFilesAccess _(boolean)_ ：为true时，此属性指定jsreport在渲染执行期间应允许访问本地文件系统和使用自定义nodejs模块

### 2.7 渲染配置<a name='render_config'></a>    [返回目录](#toc)
缺省情况下，jsreport使用专用进程来呈现pdf或脚本。该解决方案在某些带有代理的云和公司环境中更有效。但是，对于其他情况，例如，使用phantomjs时，最好在多个请求上重用让nodejs worker工作。可以使用此配置选项来实现。
```
"phantom": {     
  "strategy": "phantom-server"
},
"templatingEngines": {       
  "strategy": "http-server"
}
```

### 2.8 模板引擎配置<a name='tpl_engine_config'></a>    [返回目录](#toc)
* templatingEngines _(object)_ ：（可选）此属性用于配置执行渲染任务的组件。此组件用于在渲染过程中或在脚本扩展中执行javascript模板引擎。

* templatingEngines.strategy _(dedicated-process | http-server | in-process)_ ：第一个策略为每个任务使用一个新的nodejs实例。第二种策略在多个请求上重用每个实例。在http-server性能更好的地方，默认dedicated-process值更适合具有代理的某些云和公司环境。最后一个in-process策略只是在同一过程中运行脚本和帮助程序。这是最快的方法，但是将这种策略与用户模板一起使用可能并不安全，因为用户模板可能存在无限循环或其他可能导致应用程序终止的严重错误。in-process当您需要使用node.js调试工具调试jsreport时，该策略也很方便。

* templatingEngines.numberOfWorkers _(number)_ ：将使用多少个子nodejs实例来执行任务

* templatingEngines.timeout _(number)_ ：以毫秒为单位指定一个任务执行的默认超时

* templatingEngines.host _(string)_ ：设置启动脚本执行服务器的自定义主机名，在需要设置特定IP的云环境中很有用。

* templatingEngines.portLeftBoundary _(number)_ ：为脚本执行服务器设置特定的端口范围

templatingEngines.portRightBoundary _(number)_ ：为脚本执行服务器设置特定的端口范围

* templatingEngines.allowedModules _(array)_ ：设置允许在模板引擎的助手内部使用（利用require导入）的外部模块。例如：`allowedModules: ["lodash", "request"]`，也可以通过设置`allowedModules: "*"`允许导入任何外部模块。如果想控制脚本而不是帮助程序，则须检查相应的文档

### 2.9 日志配置<a name='log_config'></a>    [返回目录](#toc)
注意：jsreport日志是使用winston包实现的，并且它的许多概念都适用于jsreport日志配置。

* logger _(object)_ ：要完全控制jsreport的日志，可以使用对象声明输出（应将日志发送到哪里）和日志级别：
```
{
    "logger": {
        "console": { "transport": "console", "level": "debug" },
        "file": { "transport": "file", "level": "info", "filename": "logs/log.txt" },
        "error": { "transport": "file", "level": "error", "filename": "logs/error.txt" }
    }
}
```
例如，上面的配置指定以下内容：

* 名为"console"的配置与输出，该配置将所有具有debug级别的日志以及所有优先级比debug级别（"level": "debug"）低的级别发送到控制台（"transport": "console"）

* 名为"file"的配置与输出，该配置将所有info级别的日志和优先级比info级别（"level": "info"）低的所有级别发送到文件系统（"transport": "file"），并将它们存储在"logs/log.txt"（"filename": "logs/log.txt"）

* 名为"error"配置与输出，该配置将所有error级别的日志和优先级比error级别（"level": "error"）低的所有级别发送到文件系统（"transport": "file"），并将它们存储在"logs/error.txt"（"filename": "logs/error.txt"）

每个输出对象都指定使用transport属性将日志发送到的位置以及使用属性应考虑的级别。

如在先前的日志记录配置中所看到的，每个输出对象都可以具有其他属性，通过这些属性可以配置transport的功能，例如，对于名为“ file”的输出，正在使用该filename属性来告诉文件transport保存日志的位置，每种transport类型都支持一套不同的属性来配置其行为。

transport属性值：

* debug->指定应将日志发送到控制台，但仅在使用DEBUG=jsreport环境变量时可见
* console->指定应将日志发送到控制台
* file->指定应将日志发送到文件系统
* http->指定应将日志发送到和http端点

具体可用选项参见https://github.com/winstonjs/winston/blob/master/docs/transports.md#console-transport

可用日志level按优先级排序（前面的优先级更高）：

* silly
* debug
* verbose
* info
* warn
* error

对于高级用例，还提供了一种transport使用module属性配置输出的方法，该输出可以使用来自外部模块的可用属性，因为使用winston包实现了jsreport日志，因此任何与winston传输兼容的外部模块都可以在jsreport中工作。告诉jsreport使用第三方winston-loggly传输，可以创建如下配置：
```
{
    "logger": {
        "loggly": {
            "module": "winston-loggly", // module should be the name of the third-party module
            "transport": "Loggly",
            "level": "info",
            // custom loggly transport options, see https://github.com/winstonjs/winston-loggly
            "subdomain": "test",
            "inputToken": "<your token here>",
            "auth": {
                "username": "<your-username>",
                "password": "<your-password>"
            }
        }
    }
}
```
jsreport中的默认记录器配置：
```
{
    "logger": {
        "debug": {
            "transport": "debug",
            "level": "debug"
        },
        "console": {
            "transport": "console",
            "level": "debug" // "info" in production mode
        },
        "file": {
            "transport": "file",
            "level": "debug" // "info" in production mode
        },
        "error": {
            "transport": "file",
            "level": "error"
        }
    }
}
```
请注意，可以使用以下命令覆盖预定义配置的全部或部分：
```

{
    "logger": {
        "console": {
            "level": "error" // now only logs with level "error" will be sent to console, the rest of predefined outputs are still configured, we are only overriding the "level" option for the predefined console output here
        }
    }
}
```
特殊选项：

* logger.silent _(boolean)_ :方便选项，使所有已配置的输出静音（不存储日志）。默认值：false

### 2.10 配置文件示例<a name='config_sample'></a>    [返回目录](#toc)
```
{   
    "store": { "provider": "fs" },   
    "httpPort": 3000,
    "allowLocalFilesAccess": true,
    "blobStorage": { "provider": "fs" },
    "logger": {
      "console": {
        "transport": "console",
        "level": "debug"
      },
      "file": {
        "transport": "file",
        "level": "info",
        "filename": "logs/reporter.log"
      },
      "error": {
        "transport": "file",
        "level": "error",
        "filename": "logs/error.log"
      }
    },
    "chrome": {
      "timeout": 180000
    },
    "templatingEngines": {
      "numberOfWorkers" : 2,
      "timeout": 10000,
      "strategy": "http-server"
    },
    "extensions":  {  
      "authentication"  :  {  
        "cookieSession":  {  
          "secret":  "dasd321as56d1sd5s61vdv32"  
        },  
        "admin":  {  
          "username": "admin",
          "password": "password"  
        }  
      }  
   }
}
```
## 3. 存储配置详解<a name='store_config_detail'></a>    [返回目录](#toc)
jsreport支持templates、assets、scripts和所有其他实体的持久性存储。这有助于涵盖完整的报告场景，包括在Studio中设计报告，将模板保存到模板存储中以及引用它们并根据生产中的输入数据呈现最终报告。

默认的jsreport模板存储基于文件系统，但这不是唯一的选择。jsreport在持久性存储周围提供统一的API，该API由自定义扩展实现，该自定义扩展提供了对其他介质（如sql数据库等）的访问层。从文件系统持久性切换到例如mongodb只需要安装自定义扩展名并更改连接字符串。

### 3.1 默认文件系统存储<a name='fs_store'></a>    [返回目录](#toc)
jsreport默认将模板存储到文件系统。此默认实现使用模板和其他实体定义生成易于阅读的目录结构。使用任意文本编辑器可以很容易地编辑这种结构，可以将其部署到产品中，还可以使用git或其他源代码控制进行版本控制。文件系统实现最终还支持基于云的“存储”，例如AWS S3或Azure存储。

细节可参考https://jsreport.net/learn/fs-store

### 3.2 模板存储扩展<a name='store_extension'></a>    [返回目录](#toc)
可以将实现模板存储的扩展与任何其他jsreport扩展一起安装。通常，它仅需要运行npm install jsreport-xxx并重新启动jsreport服务器。自动jsreport扩展发现将在以后找到并加载它。

使用特定的存储扩展需要更改store.provider的配置。
```
"store": {
  "provider": "mongodb",
  "address": "127.0.0.1",
  "databaseName" : "std"
}
```
扩展都需要在store config中设置特定选项。

当前支持的模板存储实现包括：

* jsreport-fs-store	：文件系统+ Azure存储+ AWS S3
* jsreport-mssql-store	：Microsoft SQL服务器
* jsreport-postgres-store ：PostgreSQL
* jsreport-mongodb-store ：MongoDB

注意，实现模板存储的扩展仅用于持久化jsreport实体。其目的不是从源数据库加载报表输入数据。此功能通过自定义脚本扩展提供。

### 3.3 不同存储之间迁移<a name='store_migration'></a>    [返回目录](#toc)
jsreport import-export扩展程序能够将数据从每个受支持的商店实现导出到单个zip包中。然后可以将此包导入回其他甚至可以使用不同商店实现的jsreport实例。这使您可以在本地使用基于文件系统的存储，并轻松地将相同的数据导入到运行mongo的生产服务器中。

### 3.4 REST API<a name='store_api'></a>    [返回目录](#toc)
扩展或工作室通常在内部使用模板存储，以查询和更新诸如报告模板之类的实体。但是，模板存储API也通过基于统一odata的REST API 公开。任何客户端都可以使用它来远程查询或更新基础jsreport数据。无论基础存储实现是基于SQL还是文件系统，API始终相同。

### 3.5 javascript API<a name='js_api'></a>    [返回目录](#toc)
jsreport自定义扩展作者也可以直接通过javascript API与商店进行交互。也可以在node.js应用程序中使用它来编写自定义数据导入/导出。

该API与mongo兼容
```
const templates = await reporter.documentStore.collection("templates")    
    .find({ "shortid": "foo"})    
    .sort({ "name": 1 })    
    .limit(10)    
    .toArray()
```

## 4. 扩展<a name='extensions'></a>    [返回目录](#toc)
jsreport的所有主要部分（例如模板或pdf渲染）实际上都是扩展。可以打开/关闭它们，甚至创建自己的扩展 ，这些扩展将为模板添加特殊属性或自定义jsreport studio。

### 4.1 Base<a name='base'></a>    [返回目录](#toc)
自动注入html base tag，以允许相对引用本地样式，图像或脚本

#### 在配置中设置基地址(Base address)
```
"extensions": {
  "base": {
    "url": "${cwd}/myAssets"
  },
  "phantom-pdf": {
    "allowLocalFilesAccess": true
  }
}
```
然后，扩展将html base tag注入每个有效的html中：
```
<html>
  <head>
    <base href="file:///jsreportFolder/myAssets" />
    <script src="js/jquery.min.js"></script>
  </head>
  <body>
  </body>
</html>
```
基本标记可确保从指定的文件夹中加载相对链接的脚本，图像或样式，而无需其他额外的工作。仅需注意，可能需要在某些pdf渲染配方中启用本地文件访问，例如phantom-pdf using phanton.allowLocalFilesAccess: true选项。

#### 通过cli运行时发送基址(Base address)
```
jsreport render
  --template.engine=none
  --template.recipe=phantom-pdf
  --template.content=test.html
  --options.base=%cd%
  --out=out.pdf
 ```
#### 发送基地址作为api请求的一部分
```
POST：http//localhost:5488/api/ report

Headers：内容类型：application / json

{
  "template": {
    "content": "<html><head></head><body></body></html>",
    "recipe": "html",
    "engine": "handlebars"
  },
  "options": {
    "base": "http://foo.com"
  }
}
```

### 4.2 Inline Data<a name='inline_data'></a>    [返回目录](#toc)
在模板开发过程中使用样本测试数据

#### 基础
报告生成过程的输入是报告模板和一些json输入数据。将在运行时将这些数据提供给jsreport API，但是每次想查看报告的外观时调用jsreport API并不是很有效。jsreport带有Data扩展名。

Data 扩展允许直接在jsreport studio中使用JSON语法静态定义输入数据，并轻松测试模板。

#### API
可以使用标准的OData API来管理和查询data实体。例如，可以使用以下命令查询所有项目：
```
GET http://jsreport-host/odata/data
```
样本数据项通常通过其短标识符链接到报告模板。在这种情况下，带有报告的模板结构在json中看起来如下：
```
  template : {
      content: "foo",
      data: {
          shortid: "sd543fds"          
      }
  }
```
还可以将内联数据放入模板中，然后在json中如下所示：
```
  template : {
      content: "foo"
  },
  data: {
    price: 10,
    amount: 5,
    total: 50
  }
```

### 4.3 Scripts<a name='scripts'></a>    [返回目录](#toc)
运行自定义JavaScript来修改报告输入/输出或发送报告

#### 基础
在脚本中定义全局函数beforeRender 或（和）afterRender并使用参数req并res满足您的需求。脚本函数期望参数为req, res, done或req, res。
```
async function beforeRender(req, res) {
  // merge in some values for later use in engine
  // and preserve other values which are already in
  req.data = Object.assign({}, req.data, {foo: "foo"})
  req.data.computedValue = await someAsyncComputation()
}
```
或
```
function beforeRender(req, res, done) {
  // merge in some values for later use in engine
  // and preserve other values which are already in
  req.data = Object.assign({}, req.data, {foo: "foo"})
  done()
}
```

#### 使用外部模块
可以node.js方式通过require使用外部模块，但是首先需要在配置中启用它们。
```
{
  "extensions": {
    "scripts": {
      "allowedModules": ["request"]
    }
  }
}
```
或者，可以使用extensions.scripts.allowedModules="\*"或使用config 启用所有模块allowLocalFilesAccess=true

下面的示例使用流行的request模块进行Rest调用，并使用sendgrid服务通过电子邮件发送输出服务。
```
//load some data from the remote API
function beforeRender(req, res, done) {
    require('request')({
      url:"http://service.url",
      json:true
    }, function(err, res, body){
        req.data = Object.assign({}, req.data, body);
        done();
    });
}

//send the pdf report by mail
function afterRender(req, res, done) {
  var SendGrid = require('sendgrid');
  var sendgrid = new SendGrid('username', 'password');

  sendgrid.send({ to: '',  from: '', subject: '',
          html: 'This is your report',
          files: [ {filename: 'Report.pdf', content: new Buffer(res.content) }]
  }, function(success, message) {          
          done(success);
  });
}
```

#### request, response
* req.template-修改报告模板，主要是content和helpers属性
* req.data -用于修改报告输入数据的json对象
* req.context.http -包含有关输入http标头，查询参数等信息的对象。
* res.content -带输出报告的缓冲区
* res.meta.headers -输出标题

#### 多个脚本
可以将多个脚本关联到报告模板。然后按照jsreport studio中指定的顺序依次执行脚本。在req和res参数是否正确通过脚本链传递。这意味着可以使用例如req.data在脚本之间传递信息。

#### 全局脚本
可以通过添加flag来设置为每个模板运行的脚本isGlobal。例如，可以在jsreport studio脚本的属性中完成此操作。全局脚本对于执行常见任务（例如向模板添加常见帮助程序或设置）很有用。

#### 环境变量和配置
可以使用node.js 处理模块获取环境变量。
```
const process = require('process')
function beforeRender(req, res) {
   const myEnv = process.env.MY_ENV
}
```
如果愿意，还可以读取配置文件。
```
const path = require('path')  
const promisify= require('util').promisify
const readFileAsync = promisify(require('fs').readFile)

async function beforeRender(req, res)  {  
    const configPath = path.join(__appDirectory,  'myConfig.json')
    const config = (await readFileAsync(configPath)).toString()
    console.log(config)
}
```

#### 子模板和标题
一些配方（所谓配方是指转换程序与格式的组合，例如phantom-pdf或chrome-pdf）正在调用整个页面、页眉和页脚的渲染过程。这将导致多次调用自定义脚本-用于主页，页眉和页脚。确定来自页眉或页脚的使用req.context.isChildRequest属性。
```
function afterRender(req, res) {
    //filter out script execution for chrome header
    if (req.context.isChildRequest) {
      return
    }

    //your script code
}
```
### 从脚本渲染另一个模板
脚本可以调用另一个模板的呈现。为此，需要require特殊的模块jsreport-proxy并render在其上调用函数。
```
const jsreport = require('jsreport-proxy')

async function beforeRender(req, res) {
  console.log('starting rendering from script')
  const result = await jsreport.render({ template: { shortid: 'xxxxxx' } })  
  console.log('finished rendering with ' + result.content.toString())
}
```

#### 从脚本查询实体
例如，脚本可以查询jsreport存储并使用config加载Asset。
```
const jsreport = require('jsreport-proxy')
async function beforeRender(req, res) {
  const assets = await jsreport.documentStore.collection('assets').find({name: 'myConfig'})
  const config = JSON.parse(assets[0].content.toString())
  req.data.config = config
}
```

#### 日志
console调用会被传播到Studio调试调用，也传播到标准jsreport日志。
```
function beforeRender(req, res) {
  console.log('i\'m generating logs with debug level')
  console.warn('i\'m generating logs with warn level')
  console.error('i\'m generating logs with error level')  
}
```

#### 配置
将scripts节点添加到标准配置文件：
```
"extensions": {
  "scripts": {
    "timeout": 30000,
    "allowedModules": "*"  
  }
}
```
#### API
可以使用标准的OData API来管理和查询脚本实体。例如，可以使用以下命令查询所有脚本
```
GET http：//jsreport-host/odata/scripts
```
自定义脚本使用其缩写或名称物理链接到存储的报告模板。在这种情况下，API调用无需执行任何操作，因为脚本是自动应用的。
```
POST: { template: { name: 'My Template with script' }, data: { } }
```
如果要呈现匿名模板，则可以通过脚本名称或短标识符来识别脚本
```
POST: {
  template : {
      content: "foo",
      scripts: [{
          shortid: "sd543fds"          
      }, {
          name: "My Script"  
      }]      
  }
}
```
最后一个选项是直接指定脚本内容
```
POST: {
  template : {
      content: "foo",
      scripts: [{
          content: "function beforeRender(req, res, done) { req.template.content='hello'; done(); }"
      }]      
  }
}
```

#### 使用脚本加载数据
有些人喜欢将数据从客户端推送到jsreport中，有些人喜欢使用scripts扩展名主动获取它们。在将数据推送到jsreport更为直接的地方，使用scripts可以提供更好的体系结构和完全分离的报告服务器，其中报告本身负责定义输入以及获取输入。这个由使用者选择。

### 4.4 Assets（资产）<a name='assets'></a>    [返回目录](#toc)
嵌入静态资产，例如样式，字体或html

#### 创建资产
可以使用jsreport studio创建资产。最常见的方法是仅上传css之类的文件。第二个是使用jsreport编辑器创建一个空资产并编辑其内容。第三种选择是创建资产作为到现有文件的链接。这样的链接可以是指向jsreport的文件夹的绝对路径或相对路径。

#### 嵌入资产
嵌入资产的语法如下：
```
{#asset [nameOrPath]}
```
nameOrPath可以是唯一的资产名称，也可以是绝对或相对路径。

例子：
```
{#asset mainTheme.css}
{#asset /shared/chart.js}
{#asset ../js/chart.js}
```
然后，这样的字符串将在输出中替换为先前上载或链接的资产的内容。由于没有其他转换在运行，因此它比提取子模板运行快。

资产可以嵌入模板的内容，助手或自定义脚本中。这样可以实现诸如添加常用帮助功能或将配置文件添加到脚本的方案。

资产提取是递归的，这意味着可以创建资产层次结构。这使用户可以将样式的链接分组为一个资产。

资产提取在渲染期间运行两次。首先，在jsreport知道模板之后，并且在执行模板引擎之后。这意味着您可以动态构造资产名称。例如，以下将与jsreport handlebars引擎一起使用。
```
{#asset {{giveMeAssetName}}}
```

#### 全局共享助手
模板引擎助手需要经常在多个模板之间共享。使用资产可以实现此目的，因为可以使用Studio将每个资产标记为全局共享的帮助程序。然后，此类资产会自动嵌入到所有模板帮助器中。

#### 编码选项
资产不必是必需的文本文件。它也可以是图像或字体之类的二进制文件。在这种情况下，可以使用@encoding
```
<img src='{#asset logo.png @encoding=dataURI}'/>
```
支持的编码值：

* utf8 -未指定时使用的默认编码，将资产作为原始字符串嵌入utf8字符集中

* string-嵌入资产作为UTF8字符集原始字符串但逸出一些字符（"，'，\n对于一个JavaScript上下文内使用，等等）。当您要将资产内容作为javascript变量ex的一部分使用时，请使用以下编码：var data = "{#asset vardata.txt @encoding=string}"

* link-将资产嵌入作为对链接（http://<host>:<port>/assets/content/<asset name>）的引用。有关深入使用和警告的信息，请参见将资产嵌入作为链接

* base64-将资产嵌入为内容的base64表示形式

* dataURI-使用资产URI将资产作为其内容的base64表示形式嵌入。嵌入图像和字体时很有用。

每种编码都有不同的用例，因此请确保根据您的情况使用正确的编码。

#### 外部文件访问
尽管设计上提供授权和用户界面，但不需要通过jsreport studio上载或链接资产。也可以assets.searchOnDiskIfNotFoundInStore=true以相同的方式在配置和简单的嵌入文件中启用选项，而无需接触jsreport studio。

#### 将资产嵌入为链接
资产也可以称为链接。通常在html配方中执行效果更好，因为浏览器可以缓存资产或并行请求资产。
```
<script src="{#asset jquery.js @encoding=link}"></script>
```
但是，此方法有几个陷阱，只有在确实有意义时才应使用它。

第一个问题是，资产链接需要对chrome（或基于浏览器的外部配方，例如phantomjs）是可公开访问的。可以通过config选择：
```
{
   "extensions": {
     "assets": {
       "publicAccessEnabled": true
     }
   }
}
```
第二个问题是资产链接是基于jsreport服务器url的，而且这并不总是很容易自动确定。有以下三种情况：

1. 配置包含一个值，assets.rootUrlForLinks然后始终将其用作基础
2. 传入的HTTP呈现请求用于构造根URL
3. 没有传入请求（以不同的方式触发渲染），并且rootUrlForLinks未设置。在这种情况下，localhost:[PORT]用作基础

#### 配置
将assets节点添加到标准配置文件：
```
extensions: {
  assets: {
    // wildcard pattern for accessible linked or external files
    allowedFiles: "static/**.css",
    // enables access to files not stored as linked assets in jsreport store    
    searchOnDiskIfNotFoundInStore: false,
    // root url used when embedding assets as links {#asset foo.js @encoding=link}
    rootUrlForLinks: "http://mydomain.com",
    // make all assets accessible to anonymous requests
    publicAccessEnabled: true
  }
}
```

#### API
可以使用标准的OData API来管理和查询资产实体。例如，您可以使用查询所有资产
```
GET http://jsreport-host/odata/assets
```

### 4.5 Child Template<a name='child_template'></a>    [返回目录](#toc)
将报告模板分解为多个可重用的子模板

#### 基础
复杂的报告可能会越来越大，并且包含许多单独的部分。从逻辑上讲，每个部分都需要自己的特定帮助者，设计输入数据甚至是自定义脚本来远程获取它们。在这种情况下，将复杂的报告模板拆分为多个子模板是有意义的，这些子模板可以单独开发，也可以重用。这种扩展涵盖了这种用例。

但是，如果只有静态内容（例如一组样式，html布局等），则最佳选择（简单且性能更高）是使用资产扩展来处理这种内容。

#### 运行机制
呈现报告时，jsreport搜索特殊标记，{#child [nameOrPath]}并将其内容替换为引用模板的输出。它调用子模板的整个渲染过程，并在{#child [nameOrPath]}特殊标记以前所在的位置包含该输出。可以将其视为字符串替换。

nameOrPath可以是唯一的模板名称，也可以是绝对或相对路径。例子：
```
{#child departmentsTemplate}
{#child /contosoCompany/templates/departments}
{#child ../shared/departments}
```

#### 关于PDF的警告
由于子模板执行整个渲染操作，因此尝试在生成PDF的父模板内渲染生成PDF的子模板会给父PDF填充垃圾（子模板的渲染PDF）。

在这种情况下，当在父pdf中渲染使用chrome-pdf配方的子模板时，将子模板的recipe设为html。

#### 设置子模板参数
根请求输入数据级联传递到子模板，但是也可以通过其声明将其他数据传递给子模板
```
{#child myChildTemplate @data.paramA=foo}
```
或者，可以覆盖现有的子模板属性，例如用于呈现子模板的配方。
```
{#child myChildTemplate @template.recipe=html @options.language=sp}
```
注意，模板引擎的this上下文不会流向子模板渲染。总是在子模板中获得整个父数据集上下文。

可以使用特殊语法$=和帮助器将复杂对象作为参数传递childTemplateSerializeData。
```
{#child myChild @data.foo$={{{childTemplateSerializeData foo}}} }
```

#### 配置
将child-templates节点添加到标准配置文件：
```
extensions: {
  "child-templates": {
    // controls how many child templates rendering can happen in parallel, defaults to 2
    "parallelLimit": 5
  }
}
```

### 4.6 Report<a name='report'></a>    [返回目录](#toc)
保留报告呈现输出以供以后访问

#### 基础
每次调用jsreport API呈现报告时，将获得包含报告内容的流。当想直接向用户显示报告或想要将结果存储在数据存储中以备后用时，这很有用。但是有时只是不想在生成报表时就使用它，也不想自己存储它。在这种情况下，可以使用Reportsextension并让jsreport存储报告。

报表扩展使用Blob存储抽象来实现报表Blob持久性。默认情况下，这会将报告存储到文件系统。如果要将报告存储到其他目的地，请查阅文档。

#### API
默认情况下，报告扩展名不用于所有渲染请求，需要为每个请求分别指定它。

要使用reports扩展，需要通过以下方式扩展渲染请求：
```
POST: https：// jsreport-host / api / report BODY:

   {
      "template": { "shortid" : "g1PyBkARK" },
      "data" : { ... },
      "options": {
          "reports": { "save": true }
      }
   }
 ```
这将创建一个Report可以在jsreport studio以及OData API中使用的实体。此外，它将Permanent-Link在响应中添加自定义头信息，以后可以使用该标头实际下载报告内容。

默认情况下，存储的报表实体名称是从模板名称继承的。但是，可以通过传入{ "options": { "reportName": "yourCustomName" }请求正文来对其进行自定义。

默认情况下，存储的文件/blob的名称是从实体唯一_id继承的。可以使用{ "options": { "reports": { "save": true, "blobName": "myfilename" } }

#### 异步
发送呈现请求options.reports.save = true将指示扩展程序保存报告并向Permanent-Link响应中添加标头，但是呈现仍是同步的，并且在过程完成后会收到响应。如果要异步启动渲染过程并立即收到响应，则应设置options.reports.async = true。
```
POST: https：// jsreport-host / api / report BODY:

   {
      "template": { "shortid" : "g1PyBkARK" },
      "data" : { ... },
      "options": {
          "reports": { "async": true }
      }
   }
 ```
在这种情况下，会收到带有Location标题的响应，该标题包含指向渲染状态页面的url。它会像http://jsreport-host/reports/id/status。然后，您可以ping状态页面以检查渲染是否完成。在这种情况下，响应状态为201，并且位置标头将包含存储的报告的地址。

#### 清理
默认情况下，永久保存通过异步调用存储的报告。可以更改此设置并启用自动清除旧报告。这可以通过配置来完成。
```
{ 
  "extensions": { 
    "reports": {
      // how often the cleanup runs
      "cleanInterval": "5m",
      // how much old reports should be deleted
      "cleanTreshold": "1d"
    }
  }
}
```
请注意，自动清除逻辑还会删除通过调度计划（Schedule）扩展产生的报告。

#### 数据
可以使用标准的OData API来管理和查询报表实体。例如，可以使用以下命令查询所有报告：
```
GET http://jsreport-host/odata/reports
```

### 4.7 Scheculing<a name='scheduling'></a>    [返回目录](#toc)
安排重复发生的后台作业渲染特定的报告模板

#### 基础
每个报告呈现时间计划均由报告模板和标识时间发生的CRON表达式指定。要创建渲染时间表，需要Schedule通过jsreport studio或API 创建并启用实体。当Schedule启用了，检验NextRun属性，每次渲染完成后，也可以从工作室或Task API实体下载输出报告。

Scheduling需要并且在很大程度上依赖脚本和报告扩展。报告用于存储渲染输出，脚本通常用于获取输入数据并发送结果。

scheduling扩展的常见用例可以是每天发送摘要报告。schedule报表模板将在其中定义带有汇总表和图表的文档。脚本将在渲染之前获取输入数据，然后通过邮件发送报告或将结果上传到外部服务。请注意，脚本可以在property中获得有关当前计划的更多信息 req.options.scheduling.schedule。

#### CRON表达
CRON表达式是用于指定时间发生的UNIX标准。它是一个由5或6个段组成的字符串。

1. 秒：0-59
2. 分钟：0-59
3. 小时：0-23
4. 每月的日期：1-31
5. 月：1-12
6. 星期几：0-7
每个细分都标识特定的单位。在每个段中，还可以使用通配符\*，间隔（1-5）和步长（*/5）。

一些例子：
```
*/10 * * * * * -每10秒运行一次
00 30 11 * * 1-5 -每个工作日的11:30:00运行
```

CRON参考http://crontab.org/

#### 配置
将scheduling节点添加到标准配置文件。默认值如下。
```
"extensions": {
  "scheduling": {
    //how often in ms jsreport checks scheduled jobs
    "interval": 5000,
    //how many jobs can run in parallel
    "maxParallelJobs": 5    
  }
}
```

### 4.8 Authentication<a name='authentication'></a>    [返回目录](#toc)
将登录屏幕添加到jsreport和用户管理表单

#### 基础
启用authentication扩展将在jsreport studio中添加一个登录屏幕并验证所有传入请求。浏览器身份验证基于cookie，并且使用基本身份验证或针对已配置的授权服务器验证的承载身份验证来验证API调用。

Authentication配置将一个添加用户管理员，负责管理其他用户的系统中。该用户可以创建用户，删除用户或更改其密码。所有其他个人用户无权更改任何其他用户。

#### 基本认证
##### 配置
要启用身份验证，请在配置中添加以下json 。
```
"extensions": {
    "authentication" : {
        "cookieSession": {
            "secret": "dasd321as56d1sd5s61vdv32",
                    "cookie": { <custom cookie options here> }       
        },
        "admin": {
            "username" : "admin",
            "password": "password"
        }
    }
}
```
可以根据需要更改管理员用户名或密码。

可在此处设置要设置的自定义Cookie选项的列表，例如，您可以设置{ "cookie": { "secure": true } }为指示将仅通过HTTPS发送auth Cookie，这是一种很好的安全做法，但它要求您的服务器首先在https下运行。
```
"extensions": {
    "authentication" : {
        "cookieSession": {
            "secret": "dasd321as56d1sd5s61vdv32",
            "cookie": { "secure": true }       
        },
        "admin": {
            "username" : "admin",
            "password": "password"
        }
    }
}
```
##### API
启用此扩展名时，需要向每个请求添加头信息。
```
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```
哈希值基于用户名和密码： base64(username:password)

可以使用标准的OData API来管理和查询用户实体。例如，您可以使用以下命令查询所有用户：
```
GET http://jsreport-host/odata/users
```

#### 使用授权服务器进行基于令牌的身份验证
该功能处于预览模式。

如果要将jsreport API中的所有身份验证委托给支持基于令牌的身份验证（承载身份验证模式）的授权服务器，则需要使用这些authorizationServer选项，很可能仅在您希望将jsreport公开为具有Single Sign On支持的产品。

将jsreport配置为使用授权服务器时，身份验证流程如下所示：

* 首先，以某种方式将需要获取令牌，颁发令牌是授权服务器的责任，并且将需要从一个应用程序中请求令牌，获取令牌的必要步骤和详细信息将取决于授权服务器的实现，将需要令牌，以便以后能够调用jsreport API
* jsreport将期望从在任何受保护的API调用的Authorization头信息中获取令牌，该令牌必须使用Bearer授权模式发送，这意味着对jsreport API的所有请求都必须使用头信息：Authorization: Bearer <your token here>
* 有了令牌后，jsreport会将令牌（token=<value of your token>，token_type_hint=access_token字段）以及任何其他已配置的数据（authorizationServer.tokenValidation.hint）或凭据（authorizationServer.tokenValidation.auth）发送到已配置的端点（authorizationServer.tokenValidation.endpoint）
* 授权服务器必须返回一个json响应，其中包含描述令牌是否有效的字段（authorizationServer.tokenValidation.activeField），以及与之相关联用户（authorizationServer.tokenValidation.usernameField）
* jsreport将检查从授权服务器返回的令牌的信息（authorizationServer.tokenValidation.activeField），如果令牌有效，则活动字段必须为true，用户authorizationServer.tokenValidation.usernameField名字段必须为有效的jsreport用户，如果配置了范围验证authorizationServer.tokenValidation.scope，还将检查令牌是否具有有效authorizationServer.tokenValidation.scope.valid范围，然后验证用户身份

这些步骤中的许多步骤都是基于令牌自省方法，在基于OAuth2或的授权服务器中很常见OpenID Connect，并且如果授权服务器基于任何其他类型的实现，则描述的步骤很容易实现，因此可以实现与大多数授权服务器的兼容性。

要真正实现jsreport+授权服务器，请查看[示例](https://jsreport.net/learn/configuration)

##### 配置
要启用身份验证，请将以下json添加到配置文件中。
```
"extensions": {
    "authentication" : {
        "cookieSession": {
            "secret": "dasd321as56d1sd5s61vdv32"        
        },
        "admin": {
            "username" : "admin",
            "password": "password"
        },
        // use the "authorizationServer" options when you plan to protect API calls
        // with token based authentication against an authorization server,
        // this means that all protected jsreport API calls will be validated with
        // the configured authorization server, giving you the chance to expose
        // jsreport as a product with Single Sign On support.
        // see "Token based authentication using an authorization server" section of this document
        // for more details and a link to a sample with real implementation.
        "authorizationServer": {
            "tokenValidation": {
                // URL to the authorization server endpoint (required)
                "endpoint": "http://localhost:9800/token/introspection",
                // time in milliseconds that jsreport will wait before closing the request
                // sent to the authorization server. (optional, defaults to 6000)
                "timeout": 6000,
                // by default jsreport will send data to the authorization server
                // using "application/x-www-form-urlencoded" content type,
                // setting this option to true will make jsreport to send the data using
                // "application/json" content type. (optional, defaults to false)
                "sendAsJSON": false,
                // use this option if you would like to pass custom data to
                // the authorization server. you can configure one or multiple values
                // for example using "hint": "custom value" will send a "hint" field
                // with value "custom value" to the authorization server,
                // to use another field name you can use "hint": { "name": "reportService", value: true } to send a "reportService" field
                // and use "hint": [{ "name": "reportService", value: true }, { "name": "anotherField", value: true }] to send multiple fields
                // (optional, defaults to null)
                "hint": null,
                // name of the field that jsreport will look at in the response from authorization server to
                // use as the jsreport username associated with the token. (optional, defaults to "username")
                "usernameField": "username",
                // name of the field that jsreport will look at in the response from authorization server to
                // determine if the token is valid or not. (optional, defaults to "active")
                "activeField": "active",
                // options to use if you want that jsreport checks for valid scopes in the token
                // optional, defaults to null
                "scope": {
                    // name of the field that jsreport will look at in the response from authorization server to
                    // determine the scope/scopes of the token. (optional, defaults to "scope")
                    "field": "scope",
                    // list of valid scopes that the token needs to have in order to be considered valid, the token must have at least one scope that match with some item in the list in order to be considered valid
                    "valid": ["myscope"]
                },
                // options to use if the authorization server is not public and requires authentication,
                // if your authorization server is public just pass "auth": false
                // (required)
                "auth": {
                    // defines which auth schema to use while sending credentials to the authorization server
                    // supported values are "basic" and "bearer"
                    "type": "basic",
                    // credentials to use when using the "basic" type
                    "basic": {
                        "clientId": "test",
                        "clientSecret": "xxxx"
                    },
                    // credentials to use when using the "bearer" type
                    "bearer": "dasdasddw23fsdfsdfr56hvFVdf3"
                }
            }
        }
    }
}
```

### 4.9 Authorization<a name='authorization'></a>    [返回目录](#toc)
管理和委托jsreport对象的用户权限。需要启用身份验证

#### 基础
jsreport authorization扩展实现权限规则评估和委派。默认情况下，以前由身份验证扩展创建的每个用户仅被授权管理自己创建的对象。如果用户想与其他用户共享对象，则需要在权限表中明确设置该对象。jsreport当前只能区分read和edit权限，其中edit权限代表所有操作，包括权限委派。

嵌套在文件夹内的所有实体都将从父文件夹继承权限。递归地向下遍历多个级别的文件夹。换句话说，可以将一些权限填充到文件夹中，内部的所有实体也将获得此权限。此外，如果用户具有对特定实体的权限，则他或她将获得对树上所有父文件夹的只读权限。

#### API
Authorization扩展添加到每个jsreport对象readPermissions和editPermissions属性。这些属性包含可以从odata API轻松更改的用户ID列表。

#### 局限性
目前仅为管理员用户启用了计划和版本控制扩展。

### 4.10 Public templates<a name='public_template'></a>    [返回目录](#toc)
使用安全令牌向未经身份验证的用户授予权限的公开报告模板。需要启用身份验证和授权扩展。

身份验证扩展仅向经过身份验证的用户授予对jsreport的访问权限，而授权扩展则确保用户仅对允许的操作具有访问权限。public-templates如果想与没有任何jsreport凭据的人共享模板，此扩展将起作用。

#### 共享模板
要共享模板，可以使用sharejsreport designer或API中的工具栏按钮。这提供了一个包含安全令牌的公共链接。访问链接将输出通过渲染特定模板生成的报告。

#### API
要将安全令牌包含在渲染输出中，只需将其添加options.authorization.grantRead或添加grantWrite到正文中：
```
POST: api/report

{
  "template": { ... }
  "options": {
    "authorization": {
      "grantRead": true
    }
  }
}
```

### 4.11 Resources<a name='resource'></a>    [返回目录](#toc)
本地化模板或将任何静态JSON附加到呈现过程

#### 基础
通过Resources扩展可以将多个JSON数据对象附加到报告模板上，以后使用模板引擎或在自定义脚本中方便地访问它们。这对于将常规配置添加到模板或主要本地化模板很有用。

#### 本土化
此扩展背后的主要思想是将所有可本地化的字符串从报告模板移至JSON资源，然后使用javascript模板引擎将其绑定，而不是对其进行硬编码。然后，此扩展根据请求的语言将正确的可本地化资源推送到呈现过程。

要本地化模板，需要：

* 将包含可本地化字符串的资源附加到模板。每个包含可本地化字符串的资源都必须以语言名称作为前缀。因此，例如附加的资源应该是名称为en-invoice和的数据项de-invoice。

* 在资源菜单的jsreport studio中填充模板默认语言

* 使用模板引擎填充$localizedResource属性中的本地化字符串
```
{{:$localizedResource.title}}
```
* 当每种语言有多种资源时，请使用资源名称来到达特定的一种
```
{{:$localizedResource.invoice.title}}
{{:$localizedResource.globals.title}}
```
* 指定options.language的API调用来指定所需的语言

#### 访问自定义资源
##### 模板引擎
每个模板资源都会被解析并提供给模板引擎渲染输入。为了方便起见，以两种形式提供资源。

可以在主对象的$resource属性中找到第一种形式。存储了一个对象，该对象包含按资源名称区分的所有附加资源的数据。

因此，例如带有附加资源的模板：

* 带有名称的数据 config1
```
{
"foo": "this is config1"
}
```
* 带有名称的数据 config2
```
{
"foo2": "this is config2"
}
```
```
outputs "this is config1" when using jsrender
{{:$resource.config1.foo}}

output "this is config2" when using jsrender
{{:$resource.config2.foo2}}
```

第二种形式在$resources属性中表示为数组。此数组包含每一个附加的资源在它的完整形式包括name，shortid和data资产。这可以在某些高级方案中使用。为了确保实际存储在$resources属性中的内容，可以使用在jsreport中执行的常见方式转储对象。
```
//helper function
function dumpResources(data) {
  return JSON.stringify(data.$resources);
}
```
```
print helper function output using jsrender
{{:~dumpResources(#data}}
```
##### 脚本
自定义jsreport脚本可以使用以下属性：
```
//equivalent to request.data.$resources
request.options.resources
//equivalent to request.data.$resource
request.options.resource
```

#### API
要在API调用中指定语言，需要language在options对象中添加属性。
```
POST: https://jsreport-host/api/report
Headers：Content-Type: application/json
BODY:

   {
      "template": { "shortid" : "g1PyBkARK" },
      "data" : { ... },
      "options": { "language": "en" }
   }
```
资源直接存储在模板文档中。除此resources扩展之外，此扩展还将属性defaultLanguage添加到模板文档中。
```
GET http://jsreport-host/odata/templates('aaaa')

{
    "name": "template name",
    "content": "<h1>Hello world</h1>",
    "defaultLanguage": "en",
    ...
    "resources":{
        "items":[
            { "entitySet":"data", "shortid":"NJ5H9pkb"    }
        ]
    }
}
```

### 4.12 Tags<a name='tag'></a>    [返回目录](#toc)
用标签组织jsreport对象

#### 基础
启用tags扩展将为jsreport添加组织功能，从而启用带有标签的jsreport对象的组织，过滤和显示。

#### 创建和使用标签
可以使用jsreport studio创建标签。可以在其中指定有关标签的信息（名称、颜色、描述等），稍后在创建/编辑其他对象（模板、数据、脚本、资产、图像等）时，可以为这些对象分配一个或多个标签。

#### 配置
将tags节点添加到标准配置文件。
```
"extensions": {
    "tags" : {
      // boolean to determine if jsreport studio by default should show objects organized by tags
        "organizeByDefault": true
    }
}
```

#### API
可以使用标准的OData API来管理和查询标签实体。例如，可以使用以下命令查询所有标签：
```
GET http://jsreport-host/odata/tags
```

### 4.12 Version Control<a name='tag'></a>    [返回目录](#toc)
#### 基础
jsreport扩展添加了对版本控制实体的支持，并为常见命令（如commit，diff，revert或history）提供API以及Studio UI。只需安装扩展程序，就可以在Studio中看到新的用户界面。

#### Git
默认情况下，扩展使用自己的实现来跟踪更改并将差异存储在额外的实体中。这个非常轻巧的解决方案可与所有受支持的模板存储一起使用。但是，如果使用文件系统存储，则可能希望改用功能更强大的git并跟踪对基础文件所做的更改。

要开始使用本地git，需要另外安装jsreport-version-control-git扩展。
```
npm install jsreport-version-control-git
```
并将其配置version-control为使用它来跟踪更改。
```
{
  "extensions": {
    "versionControl": { "provider": "git" }
  }
}
```
这样可以使UI与使用Plain时的UI相同version-control。但是在其后面，它将在data文件夹中初始化本地git存储库，并将版本跟踪委派给git。请注意，此软件包将git捆绑在内部，无需将其安装在目标计算机上。

### 4.13 CLI<a name='cli'></a>    [返回目录](#toc)
jsreport的命令行界面

#### 基础
cli扩展程序提供命令行界面，该界面可以执行一些称为命令的任务。这些命令主要可以从命令行快速启动服务器或调用报表呈现，但是所提供的功能列表cli要长得多。

cli也直接集成到以单个文件可执行形式发布的jsreport中，可参考https://jsreport.net/learn/single-file-executable。

#### 安装
建议首先在全局安装cli扩展程序，这样做将提供一个全局jsreport命令，可以在命令行的任何位置使用它。
```
npm install jsreport-cli -g
jsreport --help
```
如果要安装到现有的node.js应用程序中，请遵循node.js项目集成部分cli，因为这需要一些其他步骤。

#### 用法
##### 启动jsreport
如下载页面所述，cli第一个用例通常是jsreport安装和服务器启动。
```
jsreport init
jsreport start --httpPort=6000
```
##### 渲染
cli提供的主要功能是在render命令中实现的报表呈现调用。该命令具有许多变体和开关，但是主要用法如下所示：
```
jsreport render
    --template.name=MyTemplate
    --data=mydata.json
    --out=myreport.xlsx
```

#### 命令
要获得有关特定命令的帮助，可以键入jsreport <command> -h以获取有关命令的用法和可用选项的一些说明。

以下是支持的命令。

* help
* init
* repair
* configure
* win-install
* win-uninstall
* start
* render
* kill
* help
* Command available globally

打印有关命令或特定主题的信息，您可以运行“ jsreport help -h”以获取可用主题的列表

##### init
Command available globally

初始化当前工作目录以启动jsreport应用程序（创建server.js，jsreport.config.json，package.json并在必要时安装jsreport）。

要查看可用选项和用法示例，请输入jsreport init -h。

##### repair
Command available globally

修复当前的工作目录以启动jsreport应用程序（如果存在server.js，jsreport.config.json和package.json，则覆盖它们，并在必要时安装jsreport）。

要查看可用选项和用法示例，请输入jsreport repair -h。

##### configure
Command available globally

根据一些提问生成jsreport配置文件（jsreport.config.json）。

要查看可用选项和用法示例，请输入jsreport configure -h。

##### win-install
Command only available when local version is installed in your project

将项目安装为Windows服务（仅Windows）

要以生产模式安装服务，请确保在运行命令之前将NODE_ENV环境变量设置为production

要查看可用选项和用法示例，请输入jsreport win-install -h。

已安装的服务使用nssm包装jsreport ，从而增加了自动重启逻辑。该NSSM附加到Windows事件日志，你可以尝试搜索，如果你wan't发现在标准jsreport日志相关信息写入意外错误。

##### win-uninstall
Command only available when local version is installed in your project

停止并卸载您的项目作为Windows服务（仅Windows）

要查看可用选项和用法示例，请输入jsreport win-uninstall -h。

##### start
Command only available when local version is installed in your project

启动当前工作目录中存在的jsreport进程/服务器。

要查看可用选项和用法示例，请输入jsreport start -h。

##### render
Command only available when local version is installed in your project

调用渲染过程。

使用此命令，可以直接从命令行渲染pdf、excel等，如果打算调用多个渲染，建议使用该--keepAlive选项，该选项将在后台启动jsreport（守护进程），并将相同的过程重用于以后的渲染（最佳性能）。

要查看可用选项和用法示例，请输入jsreport render -h。

##### kill
Command only available when local version is installed in your project

终止守护程序jsreport进程。

使用此命令可以杀死在后台运行的任何jsreport进程（例如，由创建的jsreport进程jsreport render --keepAlive ...）

要查看可用选项和用法示例，请输入jsreport kill -h。

#### node.js项目集成
cli如果遵循官方的jsreport 下载说明，则默认情况下jsreport 可以使用完整格式。但是如果要将jsreport集成到node.js应用程序中，则需要设置以下内容：

1. 在package.json项目的中声明一个jsreport入口点

jsreport入口点是require并创建jsreport实例的文件，通常是server.js。将需要在package.json其中添加带有项目的jsreport入口点路径的jsreport.entryPoint字段。

package.json正确配置的示例，server.js是项目的jsreport入口点：
```
{
 "name": "jsreport-server",
 "dependencies": {
   "jsreport": "2.0.0"
 },
 "main": "server.js",
 "jsreport": {
   "entryPoint": "server.js"
 }
}
```
2. 在项目的jsreport入口点中导出jsreport实例

在jsreport入口点文件中（通常server.js），将需要导出项目的jsreport实例，但是由于可能正在使用同一文件来启动jsreport服务器（使用node server.js），因此需要添加特殊条件以支持这两种情况（服务器初始化并导出jsreport实例）。

支持两种情况的jsreport入口点示例：
```
// creating a jsreport instance
const jsreport = require('jsreport')()

if (process.env.JSREPORT_CLI) {
 // when the file is required by jsreport-cli, export
 // jsreport instance to make it possible the usage of jsreport-cli
 module.exports = jsreport
} else {
 // when the file is started with node.js, start the jsreport server normally
 jsreport.init().then(() => {
   console.log('server started..')
 }).catch((e) => {
   // error during startup
   console.error(e.stack)
   process.exit(1)
 })
}
```

## 5. 转换引擎（算法）<a name='recipes'></a>    [返回目录](#toc)
转换引擎（算法）是jsreport使用的算法，用于将模板引擎的输出转换为所需的格式。每个报告模板都需要从jsreport提供的许多模板中准确指定一个配方。例如，指定 chrome-pdf算法将使用html到pdf转换创建pdf报告。另一方面，使用 html-to-xlsx可以生成excel文件。

jsreport通常支持特定输出类型的各种配方。这是因为每种转换引擎都有其优点和缺点。建议比较多个转换算法，并确定最适合特定情况的转换算法。可以在https://jsreport.net/learn/pdf-recipes的专用文章中找到有用的pdf转换算法比较。

###  5.1 HTML<a name='html_recipe'></a>    [返回目录](#toc)
Html是最基本的jsreport转换算法。它只是评估javascript模板引擎并输出html报告文件。它不仅在仅打印html时有用，而且在使用子模板并将多个模板组装在一起时也很有用。

###  5.2 Chrome PDF<a name='chrome_pdf'></a>    [返回目录](#toc)
#### 基础
Chrome-pdf转换引擎是使用headless chrome将html内容打印到pdf文件中。

#### 选项
这些设置反映了headless chrome  API的设置，可参考https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions。

* scale
* displayHeaderFooter
* headerTemplate
* footerTemplate
* printBackground
* pageRanges
* format
* width
* height
* marginTop
* marginRight
* marginBottom
* marginLeft
* waitForJS
* waitForNetworkIddle

这些基本设置通常与模板一起存储，但是也可以通过属性template.chrome内的API调用发送这些基本设置。

还可以使用以下方法在页面javascript中动态设置选项：
```
<script>
    ...
    window.JSREPORT_CHROME_PDF_OPTIONS = {
        landscape:  true
    }
</script>
```

#### 配置
使用chrome-pdf标准配置文件中的节点。
```
"extensions": {
  "chrome-pdf": {  
    "timeout": 30000,
    "launchOptions": {...}
  }
}
```
要查找有关launchOptions配置对象中可用内容的更多信息，可以在此处查看https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions。

还可以chrome在配置中使用顶级属性，不同之处在于，此配置将与使用chrome的任何其他扩展共享，而上面的配置代码段专门用于chrome-pdf扩展中的选项。
```
"chrome": {
  "timeout": 30000
}
```

#### 字体
使用资产(Assets)扩展，可以轻松地将字体嵌入到PDF报告中。您可以在此处找到有关该方法的教程。

#### 分页符
CSS包含page-break-before可用于指定html分页符的样式。chrome-pdf也可以使用它，以便在pdf文件中指定分页符。
```
<h1>Hello from Page 1</h1>
<div style='page-break-before: always;'></div>
<h1>Hello from Page 2</h1>
```
还可以使用css属性page-break-inside来例如避免将元素拆分为多个页面。

#### 打印触发器
可能需要推迟pdf打印，直到处理了一些javascript异步任务。如果是这种情况，请在API中设置chrome.waitForJS=true或利用工作室菜单设置Wait for printing trigger。然后，直到在模板的javascript中进行设置window.JSREPORT_READY_TO_START=true后，打印才会开始。
```
...
<script>
    // do some calculations or something async
    setTimeout(function() {
        window.JSREPORT_READY_TO_START = true; //this will start the pdf printing
    }, 500);
    ...
</script>
```
#### 内置页眉和页脚
系统完整的jsreport模板一样对页眉和页脚进行计算求值。这意味着可以在标题中添加例如子模板引用，然后将其提取出来。还可以使用主模板助手或页眉/页脚中的数据。注意，为了显示页眉/页脚，需要先激活该displayHeaderFooter选项，并在模板中添加一些顶部、底部边距，以便为页面提供一些空间来显示页眉/页脚。

在页眉/页脚模板中，可以使用一些特殊的CSS类使chrome注入一些内容。chrome支持的特殊CSS类如下：

* date ->注入格式化的打印日期
* title ->注入文件标题的内容
* url ->注入文件位置
* pageNumber ->注入当前页码
* totalPages ->注入总页数

应该注意内置页眉/页脚存在一些问题：

* 图片无法引用链接，需要使用base64数据URI
* javascript不会被执行
* 内容存在缩放问题，需要设置字体大小css以使其足够大以使其可见
* 未打印背景色，使用`-webkit-print-color-adjust: exact`作为解决方法

在大多数情况下，最好使用pdf-utils代替，它的限制较少，并且没有这些问题。

该示例显示了如何使用特殊的CSS类以及解决缩放问题的解决方法。
```
<!--header template content-->
<html>
  <head>
    <style>
      /* defining explicit font-size solves the scaling issue */
      html, body {
        font-size: 12px;
      }
    </style>
  </head>
  <body>
    <!--
      defining some elements with the special css classes makes chrome
      inject content in runtime
    -->
    Page&nbsp;<span class="pageNumber"></span>&nbsp;of&nbsp;<span class="totalPages"></span>
  </body>
</html>
```

#### 复杂页眉和页脚
该PDF-utils的扩展提供先进、更丰富的功能合并的动态内容到chrome pdf输出，如丰富的页眉/页脚、打印页码、水印、合并不同方向的页等，一定要检查https://jsreport.net/learn/pdf-utils的一些例子。

#### CSS媒体类型和引导程序
Chrome默认使用print在打印pdf时。这会影响CSS框架，例如Bootstrap，通常会为print媒体类型产生不同的结果。在这种情况下，pdf会应用与html不同的样式。您可以修改/从不断变化的媒体类型设置团结这print对screen在模板的Chrome浏览器设置。

#### 重用Chrome实例
默认情况下，每次渲染模板时，转换引擎都会启动额外的新chrome进程。可以更改此行为，并将配方配置为重用多个chrome实例以提高渲染性能。
```
{
  "extensions": {
    "chrome-pdf": {
      "strategy": "chrome-pool",
      "numberOfWorkers": 3
    }
  }
```

#### 打印现有网页
也可以通过chrome-pdf转换引擎打印现有网页，而无需在jsreport studio中定义模板。只需发送如下请求：
```
{ 
  "template": { 
    "recipe": "chrome-pdf",
    "engine": "none",
    "chrome": {
      "url": "https://jsreport.net"
    }
  }
}
```
或者，可以创建一个空模板并使用jsreport脚本定义url。
```
function beforeRender(req, res) {
  req.template.chrome = {
     "url": "https://jsreport.net"
  }
}
```

#### 故障排除
自封闭的div（<div />）会严重减慢chrome pdf渲染速度，不要使用！

有的用户可能由于源html的错误缩进而遇到了chrome停止响应的问题，通过单击重新格式化的代码可能会解决该问题（这听起来可能很奇怪）。

如果使用图片，chrome可能会严重导致分页中断，如果在封装的div中明确设置图片高度，则会有所帮助。
```
 <div style='height:500'>
   <img src='foo' />
 </div>
```

chrome/puppeteer默认不会在受限环境（例如docker）中运行，并且通常会要求传递--no-sandbox参数。这可以使用以下配置来实现。具体可参考https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md。
```
 "extensions": {  
  "chrome-pdf": {  
    "launchOptions": {  
      "args": ["--no-sandbox"]  
    } 
  }
}
```

###  5.3 Chrome Image<a name='chrome_image'></a>    [返回目录](#toc)
#### 基础
chrome-image转换引擎能够将html转换为图像。它就像chrome-pdf一样工作，只是某些选项有所不同。

#### 选项
这些设置反映了headless chrome  API的设置，可参考https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions。

* type
* quality
* fullPage
* clipX
* clipY
* clipWidth
* clipHeight
* omitBackground
* waitForJS
* waitForNetworkIddle

这些基本设置通常与模板一起存储，也可以通过属性template.chromeImage内的API调用发送这些基本设置。

还可以使用以下方法在页面javascript中动态设置选项：
```
<script>
    ...
    window.JSREPORT_CHROME_IMAGE_OPTIONS = {
        type:  'jpeg'
    }
</script>
```

#### chrome-pdf
chrome-image与chrome-pdf是同一扩展的一部分，并且共享其配置。这意味着可以以相同的方式增加超时时间和其他选项。
```
"extensions": {
  "chrome-pdf": {  
    "timeout": 30000,
    "launchOptions": {...}
  }
}
```
不仅配置而且许多方面都相同。例如，可以配置打印触发器、嵌入字体、重用chrome实例，甚至以相同的方式对其进行故障排除。可参阅[chrome_pdf](#chrome-pdf)。

###  5.4 Xlsx<a name='xlsx'></a>    [返回目录](#toc)

###  5.5 Html to Xlsx<a name='htmo_to_xlsx'></a>    [返回目录](#toc)
html-to-xlsx转换引擎从html表生成excel xslx文件。这不是完整的html->excel转换，而是一种从jsreport创建excel文件的实用且快速的方法。转换引擎读取输入表，并使用特定的html引擎（默认为chrome）提取几个CSS样式属性，最后使用样式创建excel单元格。

支持以下css属性：

* background-color -单元格背景色
* color -单元前景色
* border-所有的border-[left|right|top|bottom]-width，border-[left|right|top|bottom]-style，boder-[left|right|top|bottom]-color用来设置Excel单元格边框。
* text-align -在Excel单元格中水平对齐文本
* vertical-align -在Excel单元格中垂直对齐
* width -excel列将获得最大宽度，由于像素到excel点的转换，它可能有点不准确
* height -Excel行将获得最高高度
* font-family -字体系列，默认为 Calibri
* font-size -字体大小，默认为 16px
* font-style- normal和italic支持样式
* font-weight -控制单元格的文本是否应为粗体
* text-decoration- underline并且line-through受支持
* colspan-将当前列与右侧列合并的数值
* rowspan -将当前行与下面的行合并的数值。
* overflow -如果将此单元格设置为scoll，则excel单元格将启用文本换行。

#### 选项
* htmlEngine- String（此处支持的值取决于jsreport安装中可用的html引擎，默认情况下仅chrome可用，但还可以安装phantom扩展并将phantom作为html引擎）
* waitForJS- Boolean 在尝试读取页面上的html表之前是否等待js触发器启用。默认为false。
* insertToXlsxTemplate- Boolean 控制是否应将html到excel表转换的结果添加为现有xlsx模板的新表，它需要设置xlsx模板才能起作用。默认为false。

#### Sheet页
在html源上检测到的每个表都将转换为最终xlsx文件中的新表。工作表名称默认为 Sheet1，Sheet2等等。但是，可以在table元素上设置name或data-sheet-name属性来指定自定义工作表名称，其中data-sheet-name优先级更高。
```
<table name="Data1">
    <tr>
        <td>1</td>
    </tr>
</table>
<table data-sheet-name="Data2">
    <tr>
        <td>2</td>      
    </tr>
</table>
```

#### 具有数据类型的单元格
要生成具有特定数据类型的单元格，需要data-cell-type在td元素上使用。支持的数据类型是number、boolean、date、datetime和formula（这将在下面的部分进行说明）
```
<table>
    <tr>
        <td data-cell-type="number">10</td>
        <td data-cell-type="boolean" style="width: 85px">1</td>
        <td data-cell-type="date">2019-01-22</td>
        <td data-cell-type="datetime">2019-01-22T17:31:36.000-05:00</td>
    </tr>
</table>
```

#### 格式
Excel支持设置单元格字符串格式。可以在元素上使用data-cell-format-str（指定原始字符串格式）或data-cell-format-enum（选择现有格式）完成此操作td。

data-cell-format-enum的可能值包括：

* 0 ->格式等于 general
* 1 ->格式等于 0
* 2 ->格式等于 0.00
* 3 ->格式等于 #,##0
* 4 ->格式等于 #,##0.00
* 9 ->格式等于 0%
* 10 ->格式等于 0.00%
* 11 ->格式等于 0.00e+00
* 12 ->格式等于 # ?/?
* 13 ->格式等于 # ??/??
* 14 ->格式等于 mm-dd-yy
* 15 ->格式等于 d-mmm-yy
* 16 ->格式等于 d-mmm
* 17 ->格式等于 mmm-yy
* 18 ->格式等于 h:mm am/pm
* 19 ->格式等于 h:mm:ss am/pm
* 20 ->格式等于 h:mm
* 21 ->格式等于 h:mm:ss
* 22 ->格式等于 m/d/yy h:mm
* 37 ->格式等于 #,##0 ;(#,##0)
* 38 ->格式等于 #,##0 ;[red](#,##0)
* 39 ->格式等于 #,##0.00;(#,##0.00)
* 40 ->格式等于 #,##0.00;[red](#,##0.00)
* 41 ->格式等于 _(* #,##0_);_(* (#,##0);_(* "-"_);_(@_)
* 42 ->格式等于 _("$"* #,##0_);_("$* (#,##0);_("$"* "-"_);_(@_)
* 43 ->格式等于 _(* #,##0.00_);_(* (#,##0.00);_(* "-"??_);_(@_)
* 44 ->格式等于 _("$"* #,##0.00_);_("$"* (#,##0.00);_("$"* "-"??_);_(@_)
* 45 ->格式等于 mm:ss
* 46 ->格式等于 [h]:mm:ss
* 47 ->格式等于 mmss.0
* 48 ->格式等于 ##0.0e+0
* 49 ->格式等于 @
```
<style>
    td {
        width: 60px;
        padding: 5px;
    }
</style>
<table>
    <tr>
        <td data-cell-type="number" data-cell-format-str="0.00">10</td>
        <td data-cell-type="number" data-cell-format-enum="3">100000</td>
        <td data-cell-type="date" data-cell-format-str="m/d/yyy">2019-01-22</td>
    </tr>
</table>
```

当单元需要具有取决于特定计算机区域设置的特定格式类别时，需要设置格式。否则该单元格在excel中被归类为“常规”。

例如，使用data-cell-type="date"可以将单元格设置为日期，可以在基于日期的计算中使用它。但是，excel中的单元格格式类别显示为“常规”而不是“日期”。为此，需要进行编辑data-cell-format-str以匹配区域(locale)设置。

#### 方程
可以data-cell-type="formula"在td元素上使用来指定公式单元格。
```
<table>
    <tr>
        <td data-cell-type="number">10</td>
        <td data-cell-type="number">10</td>
        <td data-cell-type="formula">=SUM(A1, B1)</td>
    </tr>
</table>
```

#### 字体系列
可以使用以下css样式来更改表中所有单元格的默认字体系列。
```
td  { 
  font-family: 'Verdana'; 
  font-size: 18px; 
}
```

#### 将输出插入到xlsx模板中
在某些情况下，表到xlsx的转换就足够了。但是，对于更复杂的情况（例如使用excel生成透视表或复杂图表），可以选择将生成的表插入到现有的xlsx模板（作为新工作表）中，而不是生成新的xlsx文件。

流程如下。打开桌面excel应用程序，并在一张sheet上准备数据透视表和图表，在第二张sheet上准备静态数据。将xlsx上传到jsreport studio并将其与html-to-xlsx生成动态表的模板链接。只要确保表格名称与Excel中的数据表名称匹配即可。现在运行模板可以根据jsreport收集的数据生成带有图表或数据透视图的动态excel。

请参见此示例https://playground.jsreport.net/w/admin/QiHIBqsq

#### 转换触发
有时可能需要推迟表的转换，直到处理一些javascript异步任务为止。如果是这种情况，可在API选项设置htmlToXlsx.waitForJS=true或利用工作室菜单设置Wait for conversion trigger。然后，直到在模板的javascript中进行设置window.JSREPORT_READY_TO_START=true后，转换才会开始。
```
...
<script>
    // do some calculations or something async
    setTimeout(function() {
        window.JSREPORT_READY_TO_START = true; //this will start the conversion and read the existing tables on the page
    }, 500);
    ...
</script>
```
#### 行高大于实际内容的问题
当phantomjs用作引擎时，有时行高度的结尾高度大于实际内容的高度。这是由phantomjs bug引起的，该错误在单元格的内容具有空格字符时会检索到更高的高度。

如果行高较大对excel文件有问题，则有两种可能的解决方法：

* 使用"letter-spacing"具有某些负值的CSS属性
```
<!-- without "letter-spacing" row would be more larger -->
<table style="letter-spacing: -4px">
    <tr>
        <td> From Date: NA</td>
        <td> To Date: NA </td>
        <td> Search Text: NA </td>
        <td> Sort Order: NA </td>
        <td> Sort Key: NA </td>
        <td> Filter: NA </td>
    </tr>
</table>
```
* 使用"line-height: 0"具有特定"height"
```
<!-- without "line-height" and "height" row would be more larger -->
<table style="line-height: 0">
    <tr style="height: 20px">
        <td> From Date: NA</td>
        <td> To Date: NA </td>
        <td> Search Text: NA </td>
        <td> Sort Order: NA </td>
        <td> Sort Key: NA </td>
        <td> Filter: NA </td>
    </tr>
</table>
```

###  5.6 Docx<a name='docx'></a>    [返回目录](#toc)
docx配方根据上载的docx模板生成Office docx报告，并使用Word应用程序在其中填充了handlebar标签。

1. 打开Word并创建使用handlebar模板引擎的docx文件。
2. 将创建的docx文件作为资产上传到jsreport studio
3. 创建模板，选择docx转换引擎并链接先前上传的资产
4. 如果需要，请附加样本输入数据或脚本
5. 运行模板，将获得动态组装的docx报告

#### 内置助手程序
##### docxList
使用Word创建一个包含单个项目的列表，然后调用docxList帮助程序。它将遍历提供的数据并为每个条目创建另一个列表项。
```
 - {{#docxList people}}{{name}}{{/docxList}}
```

##### docxTable
使用Word创建具有列标题和单行的表。调用{{#docxTable}}数据行的第一个单元格和结束通话{{/docxTable}}的最后一个单元格。
```
columnA	columnB
{{#docxTable people}}{{name}}	{{email}}{{/ docxTable}}
```

##### docxStyle
使用{{#docxStyle}}{{/docxStyle}}包围文本块，并传递textColor参数可动态指定文本颜色。
```
{{#docxStyle textColor='0000FF'}}Simple text{{/docxStyle}}
```

##### docxImage
1. 使用Word准备图像占位符：将任何图像放置到所需位置，然后根据需要将其格式化。
2. 选择图像，选择Word选项卡“插入”，单击“书签”并创建一个
3. 右键单击图像，然后单击“超链接”
4. 单击书签并选择以前创建的书签
5. 在“插入超链接”模式下单击“屏幕提示”。
6. 填写docxImage帮助程序调用 {{docxImage src=myDataURIForImage}}
7. 单击确定，然后关闭超链接对话框。现在，如果对图像进行悬停，应该会看到docxImage帮助程序调用
8. 在输入数据中运行带有myDataURIForImage属性的模板，应该会在输出中看到替换的图像。

docxImage 支持以下配置属性：

* src（string）->指定要加载的图像的base64 dataURI字符串表示形式
* usePlaceholderSize（boolean）->如果为true，则图像的尺寸将设置为与docx文件上定义的占位符图像相同的尺寸。例如：{{docxImage src=src usePlaceholderSize=true}}
* width（string）->指定图像的宽度，值可以为px或cm。当仅width设置时，height将根据图像的纵横比自动生成。例如：{{docxImage src=src width="150px"}}
* height（string）->指定图像的高度，值可以为px或cm。当仅height设置时，width将根据图像的纵横比自动生成。例如：{{docxImage src=src height="100px"}}

#### API
```
{
  "template": {
    "recipe": "docx",
    "engine": "handlebars",
    "docx": {
       "templateAssetShortid": "xxxx"
    }
  },
  "data": {}
```
如果没有将Office模板存储为资产，则可以直接在API调用中将其发送。
```
{
  "template": {
    "recipe": "docx",
    "engine": "handlebars",
    "docx": {
       "templateAsset": {
          "content": "base64 encoded word file",
          "encoding": "base64"
       }
    }
  },
  "data": {}
```

###  5.7 Html embedded in docx<a name='html_in_docx'></a>    [返回目录](#toc)
#### 安装
```
npm install jsreport-html-embedded-in-docx
```
#### 基础
该html-embedded-in-docx转换引擎采用的html输出，并将其嵌入到基于DOCX报告。这是从jsreport生成docx/word报告的非常简单的方法。但是注意，此类输出docx文件在某些Office实现（如Open Office）中无法打开。

###  5.8 docxtemplater<a name='docxtemplater'></a>    [返回目录](#toc)
使用docxtemplater生成docx报告的配方

#### 安装
```
npm install jsreport-docxtemplater
````

#### 用法
1. 使用docxtemplater文档中介绍的标记来准备docx模板(参考https://github.com/open-xml-templating/docxtemplater）
2. 上载docx模板
3. 准备输入数据-使用样本数据或自定义脚本
4. 创建新模板并将转换引擎切换到docxtemplater
5. 关联模板资产和输入数据
6. 将模板内容保留为空，将不会使用
7. 运行模板

#### API
```
{
  "template": {
    "recipe": "docxtemplater",
    "engine": "handlebars",
    "docxtemplater": {
       "templateAssetShortid": "xxxx"
    }
  },
  "data": {}
```
如果没有将Office模板存储为资产，则可以直接在API调用中将其发送。
```
{
  "template": {
    "recipe": "docxtemplater",
    "engine": "handlebars",
    "docxtemplater": {
       "templateAsset": {
          "content": "base64 encoded word file",
          "encoding": "base64"
       }
    }
  },
  "data": {}
```

###  5.9 Phantom pdf<a name='phantom_pdf'></a>    [返回目录](#toc)
Phantom-pdf使用phantomjs屏幕捕获功能将HTML内容打印到PDF文件中。这种方法在定义报告模板方面非常有效率，也是使用jsreport的最常用的方法。

Phantom-pdf配方能够呈现提供的任何HTML和JavaScript。这意味着，也可以使用外部JavaScript库或画布打印可视化图表。

#### 安装
```
npm install jsreport-phantom-pdf
```
#### 基本设置
* margin-从页面边界使用的px或cm边距规格，您也可以通过Object或JSON object string来更好地控制每个边距面。例如：{ "top": "5px", "left": "10px", "right": "10px", "bottom": "5px" }
* format -包含A3，A4，A5，Legal，Letter的预定义页面大小
* width -px或cm的页面宽度，优先于纸张格式
* height -px或cm的页面高度，优先于纸张格式
* orientation -纵向或横向
* headerHeight -页面标题的px或cm高度
* header -标头html内容
* footerHeight -页面中页脚的px或cm高度
* footer -页脚html内容
* printDelay -渲染页面与打印为pdf之间的延迟，这在打印图表等动画内容时非常有用
* blockJavaScript -阻止执行JavaScript
* waitForJS - 真假

这些基本设置通常与模板一起存储，但也可以通过在API调用中设置属性template.phantom来发送这些基本设置。

#### 分页符
CSS包含page-break-before可用于指定html分页符的样式。这也可以用于phantom-pdf，以便在pdf文件中指定分页符。
```
<h1>Hello from Page 1</h1>

<div style='page-break-before: always;'></div>

<h1>Hello from Page 2</h1>

<div style="page-break-before: always;"></div>

<h1>Hello from Page 3</h1>
```

#### 页眉和页脚
系统像完整的jsreport模板一样对页眉和页脚进行评估。这意味着可以在标题中添加例如子模板引用，然后将其提取出来。还可以使用主模板助手或页眉/页脚中的数据。

#### 页码
Phantom-pdf提供特殊标签{#pageNum}以及{#numPages}页眉和页脚，可用于打印页码。
```
<div style='text-align:center'>{#pageNum}/{#numPages}</div>
```
注意，Phantom-pdf还会在页眉和页脚中运行评估javascript，可以使用它们来修改分页开始，例如：
```
<span id='pageNumber'>{#pageNum}</span>
<script>
    var elem = document.getElementById('pageNumber');
    if (parseInt(elem.innerHTML) <= 3) {
        elem.style.display = 'none';
    }
</script>
```

#### 国家字符
Phantom-pdf当前默认情况下无法打印某些国家字符。为了能够将正确的国家字符打印为pdf，需要首先在html中设置utf-8字符集。
```
<html>
  <head>
    <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
  </head>
  <body>
     ščřžý
  </body>
</html>
```

#### 页眉和页脚中的图像
当尝试以常见方式将图像添加到页眉中时，图像不会显示：
```
<img src='http://domain.com/foo.jpg'>
```
这是因为页眉和页脚以同步方式打印为PDF格式。这意味着像获取图像之类的任何异步请求都不会及时完成。这是phantom.js的当前限制

解决方案：
将相同的图像添加到模板内容，并使用样式将其隐藏display:none。然后，可以将其添加到标头中，因为它已经被缓存，因此不需要异步请求，它将显示出来。对于url引用的两个图像以及数据URI方案base64图像都需要这样做。

#### 页眉和页脚中的样式
phantomjs不允许将外部样式链接到页眉和页脚。需要始终使用<style>标签内联它。如果这变得乏味，则可以使用子模板来提取并复用它。

#### 打印触发器
有时可能需要推迟PDF打印，直到处理了一些JavaScript异步任务。在这种情况下，请在API中设置phantom.waitForJS=true或在工作室菜单中启用Wait for printing trigger。然后，直到在模板的javascript中设置window.JSREPORT_READY_TO_START=true时，打印才会开始。
```
...
<script>
    // do some calculations or something async
    setTimeout(function() {
        window.JSREPORT_READY_TO_START = true; //this will start the pdf printing
    }, 500);
    ...
</script>
```

#### macOS Sierra
默认的phantomjs@1.9当前不适用于macOS sierra更新，需要使用phantomjs2。

#### phantomjs2
转换引擎默认安装使用phantomjs@1.9。还可以安装其他版本并并行使用。

1. 使用`npm install phantomjs-exact-2-1-1`安装其他的phantomjs 
2. 使用jsreport studio在属性中切换phantomjs版本或在api调用中设置"template.phantom.phantomjsVersion":"2.1.1"

还可以在配置中全局设置默认的phantomjs版本：
```
"phantom": {   
  "defaultPhantomjsVersion": "2.1.1"
}
```
请注意，phantomjs2生成的字体大小与1.9不同。此外，thead当表格产生多个页面时，它不支持重复线程。

### Twitter的引导
使用响应式CSS框架（如Bootstrap）打印到PDF可能不是最好的主意。但是，它仍然有效。需要记住，输出的PDF通常看起来与HTML不一样，因为Bootstrap在@media print下包含不同的打印样式。

#### 字体
使用资产扩展，可以轻松地将字体嵌入到PDF报告中。参见资产设置。

#### 在高级情况下，在每页中打印内容
通常，如果需要在每个页面上显示一些内容，则可以使用headers或footer。

但是，由于页眉和页脚的呈现方式类似于新模板，并且对于主模板中正在评估的内容没有任何上下文，因此有时会发现，页眉和页脚对于实际需求非常有限。在这种情况下，可以使用类似于在此所描述的一种方法（https://jsreport.net/blog/phantomjs-pdf-watermark）。

该方法使用一些魔术数字（根据您的机器和内容以及您的最佳判断，需要调整值）来获取呈现的页面总数，并进行一些计算以能够模拟每个页面的页眉和页脚，或仅在每个页面的特定位置呈现某些内容（例如水印）。

#### 在高级情况下，在每页中打印内容（pdf-utils）
在每个页面中打印内容的另一种方法是使用pdf-utils扩展名，它提供了将动态内容合并到pdf输出中的功能。

#### Windows与Unix上的尺寸
当在Windows和Unix平台上渲染时，phantomjs 1.9.8和2.1.1都生成不同大小的PDF元素。这个问题可以参考https://github.com/ariya/phantomjs/issues/12685。因此，建议在计划运行jsreport生产实例的同一操作系统上设计报告。如果这不是可用选择，则可以尝试应用以下CSS来调整本地或生产模板上的大小。
```
body {
  transform-origin: 0 0;
  -webkit-transform-origin: 0 0;
  transform: scale(0.654545);
  -webkit-transform: scale(0.654545);
}
```

#### 配置
使用phantom标准配置文件中的配置节点。
```
"extensions": {
  "phantom-pdf": {
    "numberOfWorkers": 1
    "timeout": 180000,
    "allowLocalFilesAccess": false,
    "defaultPhantomjsVersion": "1.9.8"
  }
}
```
还可以在配置中使用顶级属性phantom，不同之处在于此配置将与使用phantomjs的任何其他扩展共享，并且上面的配置代码段专门用于phantom-pdf扩展中的选项。
```
"extensions": {
  "phantom": {
    "numberOfWorkers": 1
    "timeout": 180000,
    "allowLocalFilesAccess": false,
    "defaultPhantomjsVersion": "1.9.8"
  }
}
```

可用的配置选项：

* phantom.strategy（dedicated-process | phantom-server）-第一个策略为每个任务使用一个新的phantomjs实例。第二种策略在多个请求上重用每个实例。在phantom-server性能更好的地方，默认dedicated-process值更适合某些具有代理的云和公司环境
* phantom.numberOfWorkers（int）-指定phantom-pdf配方将使用多少个phantomjs实例。如果未填充该值，则默认情况下，jsreport将使用cpus数
* phantom.timeout（int）-使用phantomjs为pdf渲染指定默认超时
* phantom.allowLocalFilesAccess（bool）-默认为false。设置为true时，可以使用本地路径来获取资源。
* phantom.host（string）-设置启动phantomjs服务器的自定义主机名，这对于需要设置特定IP的云环境很有用。
* phantom.portLeftBoundary（number）-为phantomjs服务器设置特定的端口范围
* phantom.portRightBoundary（number）-为phantomjs服务器设置特定的端口范围
* phantom.defaultPhantomjsVersion（string）-设置要使用的默认phantomjs版本，请注意，如果要设置默认值以外的其他值，则必须手动安装所需的phantomjs版本（使用类似phantomjs-prebuilt或的软件包phantomjs-exact-2-1-1）。默认值：1.9.8

###  5.10 Phantom Image<a name='phantom_image'></a>    [返回目录](#toc)
使用phantomjs从html渲染图像

#### 安装
```
npm install jsreport-phantom-image
```

#### 用法
设置template.recipe=phantom-image在渲染请求。
```
{
  template: { content: '...', recipe: 'phantom-image', engine: '...', phantomImage: { ... } }
}
```

#### jsreport-core
也可以将此扩展手动应用于jsreport-core
```
var jsreport = require('jsreport-core')()
jsreport.use(require('jsreport-phantom-image')({ strategy: 'phantom-server' }))
```

#### 配置
* imageType-png，gif或jpeg，默认png
* quality -输出图像的质量（1-100），默认为100
* printDelay-开始打印之前要等待的毫秒数
* blockJavaScript-阻止页面上正在运行的js
* waitForJS-参考phantom-html-to-pdf。在这种情况下要设置的window的JSREPORT_READY_TO_START变量为true，开始渲染

## 6. 模板引擎<a name='engines'></a>    [返回目录](#toc)
jsreport使用javascript模板引擎定义报告布局。通过模板引擎，可以绑定输入数据、使用循环、条件或javascript帮助器...。模板引擎基本上提供了一种方法，可以以一种非常快速、灵活、成熟且广为人知的方式定义任何自定义报告。

### 6.1 handlebars<a name='handlebars'></a>    [返回目录](#toc)
#### 基础
jsreport handlebars引擎使用handlebars库，因此与它完全兼容，完整的文档位于http://handlebarsjs.com

#### 数据绑定
可以使用{{...}}标签使用来自报表输入数据的值。

假设输入数据对象如下：
```
{
    "title": "Hello world"
}
```
可以用`{{title}}`来打印`Helo world`
```
<h1>{{title}}</h1>
```
#### 条件
假设输入对象如下：
```
{
    "shouldPrint" : true,
    "name" : "Jan Blaha"
}
```
然后，可以在if条件中使用shouldPrint布尔值。有关更复杂的情况，请参阅“参考Helpers”。
```
{{#if shouldPrint}}
    <h1>{{name}}</h1>
{{/if}}
```
#### 循环
假设输入数据对象如下：
```
{
    "comments": [ {"title": "New job", "body": "js developers wanted at... " }]
}
```
可以使用each简单地遍历comments
```
{{#each comments}}
  <h2>{{title}}</h2>
  <div>{{body}}</div>
{{/each}}
```
#### 助手程序
jsreport报告模板包含带有javascript模板引擎标签的content字段和放置一些javascript函数的helpers字段。

例如，想要一个大写的助手功能。可以在helpers字段内使用以下代码在注册全局函数：
```
function toUpperCase(str) {
    return str.toUpperCase();
}
```
然后，可以使用以下命令在handlebars中调用函数：
```
say hello world loudly: {{{toUpperCase "hello world"}}}
```

#### 从助手调用助手
handlebars允许通过Handlebars.helpers.helperName从一个助手调用另一个助手程序。下面的片段在jsreport helpers部分显示了如何执行此操作。
```
const  Handlebars = require('handlebars')

function helperA () {
  return  'helperA'
}

function helperB () {
  return  Handlebars.helpers.helperA()
}
```

#### 第三方助手库
有很多第三方库提供了额外的handlebar helper，例如handlebars-helpers或handlebars-intl。要在jsreport中使用这样的库，需要用npm安装它，然后require在模板帮助器的顶部安装它。

`npm install handlebars-intl`
```
const handlebars = require('handlebars');

const HandlebarsIntl = require('handlebars-intl');
HandlebarsIntl.registerWith(handlebars);
```
`npm install handlebars-helpers`
```
const handlebars = require('handlebars');

const helpers = require('handlebars-helpers')({
  handlebars: handlebars
});
```

### 6.2 jsrender<a name='jsrender'></a>    [返回目录](#toc)
#### 基础
jsreport jsrender引擎使用jsrender库，因此与它完全兼容，完整的文档位于http://www.jsviews.com/

#### 数据绑定
可以使用{{:...}}标签使用来自报表输入数据的值。

假设输入数据对象如下
```
{
    "title": "Hello world"
}
```
可以用`{{:title}}`来打印`Hello world`
```
<h1>{{:title}}</h1>
```
#### 条件
假设输入对象如下：
```
{
    "age" : 18,
    "name" : "Jan Blaha"
}
```
然后，可以通过age以下方式在条件中使用：
```
{{if age >= 18}}
    <h1>{{:name}} is elligible to drink.</h1>
{{else}}
    <h1>{{:name}} is not elligible to drink.</h1>
{{/if}}
```

#### 循环
假设输入数据对象如下
```
{
    "comments": [{"title": "New job", "body": "js developers wanted at... " }]
}
```
可以使用for简单地遍历comments
```
{{for comments}}
  <h2>{{:title}}</h2>
  <div>{{:body}}</div>
{{/for}}
```
#### 助手
jsreport报告模板包含带有javascript模板引擎标签的content字段和放置一些javascript函数的helpers字段。

例如，想要一个大写的助手功能。可以在helpers字段内使用以下代码字段内注册全局函数：
```
function toUpperCase(str) {
    return str.toUpperCase();
}
```
然后可以使用以下命令在jsrender中调用函数：
```
say hello world loudly: {{:~toUpperCase("hello world")}}
```
#### 子模板(Sub templates)
jsreport还支持jsrender子模板功能。当要遍历数据集合并为每个项目打印特定模板时，这可能会很方便。

为此，可以使用以下语法在content字段内定义item子模板：
```
<script id="itemTemplate" type="text/x-jsrender">
    {{:#data}}
</script>
```
然后，可以告诉jsrender使用以下子模板：
```
{{for languages tmpl="itemTemplate"/}}
```
最好将jsrender子模板与jsreport child templates一起使用，并将子模板移到专用报告模板中。这样可以将大模板分成多个模板，并使内容保持清晰。注意，在这种情况下，应该将jsreport子模板设置为None引擎和html转换引擎。

### 6.3 EJS<a name='ejs'></a>    [返回目录](#toc)
#### 安装
`npm install jsreport-ejs`
#### 基础
jsreport EJS引擎使用EJS库，因此与它完全兼容。可以执行所有典型的EJS数据绑定、条件、循环，甚至使用帮助器，详见http://www.ejs.co/。
```
<ul>
<% for(var i=0; i<students.length; i++) {%>
   <li><%= foo(students[i]) %></li>
<% } %>
</ul>
```

## 7. API<a name='api'></a>    [返回目录](#toc)
jsreport提供了基于HTTP Rest的普通API进行渲染和CRUD

### 基础
jsreport API可以分为两个主要用例：

* 呈现报告-需要调用报告呈现过程时使用它
* 查询和CRUD-在更复杂的jsreport集成中使用它，您想在其中使用API更新或同步实体

以下两节详细描述了这些主要用例。

jsreport studio使用相同的API。如果文档中缺少某些内容，可以打开F12浏览器工具，然后在“网络”选项卡中查看情况。

### 渲染报告
调用报告呈现过程是最常用的API方法。下一个代码片段显示了服务端点URL以及主体模式。选项和数据字段是可选的。
```
POST: https://jsreport-host/api/report
Headers：content: application/json
BODY:
```
```
   {
      "template": { "name" : "my template"  },
      "data" : { ... },
      "options": { "reports": { "save": true } }
   }
```
在最典型的情况下，只需要指定模板名称（或shortid）并输入数据即可。模板名称属性必须是唯一的模板名称。如果使用多个文件夹，建议传递完整路径而不是名称。例如
```
{
  "template": { "name": "/myfolder/mytemplate" }"
  "data": {}
}
```
可能只想覆盖模板的某些属性。这很容易，因为在评估之前，所有请求属性都已合并到存储的模板中。例如，如果只想更改模板转换引擎，则html可以使用以下请求正文进行操作。
```
{
    "template": { 
        "name": "myTemplate",
        "recipe": "html"
      },
    "data" : { ... },
}
```
该模板不一定需要存储在jsreport模板存储中，并且可以在请求正文中完全定义。在这种情况下，需要至少指定需要的属性recipe、engine和content。以下代码段显示了如何定义这样的请求正文。
```
{
    "template": { 
        "content" : "Hello world {{name}}",
        "recipe": "chrome-pdf",
        "engine": "handlebars",
        "chrome": {
            "landscape": true
        }
    },
    "data" : { ... },
}
```
可以通过以下方式找到有效的模板属性：

* 使用API对话框(可以通过工作室设置打开)
* 使用odata元数据定义(可以从http://jsreport-host/odata/$metadata得到）
* 在Studio中运行渲染请求时使用F12浏览器工具

### Content-Disposition和报告名称
您可以更改Content-Disposition响应头，从而使用request的options\[reportNam\]属性在下载过程中显示文件名浏览器
```
{
      "template": { ... },
      "options": { "reportName": "myreport" }
}
```
文件扩展名根据正在生成的报告的内容类型添加到jsreport。例如，chrome-pdf在这种情况下，将生成“ myreport.pdf”。

如果要完全控制此响应头，则可以在请求正文中指定options\['Content-Disposition'\]并覆盖默认值。

### 查询和增删改查
jsreport中的查询和CRUD API基于OData协议。您可以使用它来以编程方式查询或CRUD jsreport服务器包含的任何实体。Odata定义了jsreport遵循的一些基本标准，以下是一些最常见的调用示例。

#### 查询
获取所有实体类型
```
GET: https://jsreport-host/odata/$metadata
```

获取模板名称列表
```
GET: http://jsreport-host/odata/templates?$select=name
```
获取具有特定名称的脚本
```
GET: http://jsreport-host/odata/scripts?$filter=name eq \'myscript\'
```
获取特定文件夹中的资产
```
GET: http://jsreport-host/odata/assets?$filter=folder/shortid eq \'abc\'
```

#### 增删改查
创建一个模板
```
POST: http://jsreport-host/odata/templates
Headers: Content-Type: application/json
BODY:

  {
    "name":"demo",
    "recipe":"chrome-pdf",
    "engine":"handlebars",
    "content":"<h1>sample content</h1>"
  }
```
按ID读取模板
```
GET: http://jsreport-host/odata/templates('xxxxxxxxxx')
```
更新模板属性
```
PATCH: http://jsreport-host/odata/templates(WOIzhZdfjj7rRRO2)
Headers: Content-Type: application/json
BODY: send the properties that you want to change

  {
    "content": "updated content"
  }
```
删除模板
```
DELETE：http：// jsreport-host / odata / templates（UCV8p6iVIzR6pEz8）
```
#### 文件夹

文件夹就像模板一样是实体，可以对它们使用相同的odata调用。

例如，可以使用列出所有文件夹
```
GET: http://jsreport-host/odata/folders
```
获取特定文件夹中的模板
```
GET: http://jsreport-host/odata/templates?$filter=folder/shortid eq 'foo'
```
在根文件夹中获取模板
```
GET: http://jsreport-host/odata/templates?$filter=folder eq null
```
在特定文件夹中创建模板
```
POST: http://jsreport-host/odata/templates

  {
    "name": "template name",
    "engine": "none",
    "recipe": "html",
    "folder": {
      "shortid": "foldershortid" 
    }
  }
```
### 认证
当jsreport服务器启用了身份验证扩展时，或使用jsreportonline时，需要向所有API请求添加身份验证头信息。头信息基于基本的http身份验证，因此应该能够轻松地从任何平台进行身份验证。
```
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```
哈希值基于用户名和密码：
```
base64(username:password)
```

### Ping
有公共端点http://jsreport-host/api/ping，可用于检查jsreport是否正在运行。该端点不在身份验证后面，因此可以从负载均衡器或docker heathcheck中使用它。

## Ex. 许可<a name='license'></a>    [返回目录](#toc)

许可有jsreport_licensing包负责，详见main.js的267行。
