# jsreport研究
# 目录<a name='toc'></a>
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

## 1. CentOS安装<a name='install'></a>
jsreport整体上利用nodejs开发，界面主要体现在jsreport-studio上。

### 1.1 安装Nodejs<a name='install_nodejs'></a>
jsreport需要node.js (>= 8.9)和npm (>= 6.x)。

### 1.2 安装jsreport<a name='install_jsreport'></a>
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

## 2. 配置<a name='config'></a>

使jsreport适应需求的最简单方法是更改其配置。jsreport配置提供了许多选项，例如更改http端口，将存储库提供程序设置为不同的机制等等。

可以使用以下命令获取支持的配置选项的列表

`jsreport help config`

### 2.1 配置来源(Configuration sources)<a name='config_source'></a>
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

### 2.2 配置扩展<a name='config_extension'></a>
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
### 2.3 Web服务器配置<a name='web_config'></a>
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

### 2.4 存储配置<a name='store_config'></a>
* store _(object)_ ：jsreport支持多种实现来存储模板。具体实现是基于store.provider属性来区分的。预先创建的配置文件中的预定义值是fs使用jsreport-fs-store在文件系统上存储报告模板的值。另外，可以安装其他扩展提供模板存储并进行更改store以反映它。您可以在此处找到可用商店驱动程序的列表，以及如何配置它们的更多详细信息。

* blobStorage _)object)_ :可选，指定用于存储报告的存储类型。具体实现是基于blobStorage.provider属性来区分的。它可以是fs，memory或gridFS。在完整版jsreport中默认为fs，或在jsreport集成到现有的node.js应用程序时默认memory。

### 2.5 目录配置<a name='dir_config'></a>
* rootDirectory_(string)_ ：（可选）指定应用程序的根目录和jsreport在哪里搜索扩展名

* tempDirectory_(string)_ ：（可选）指定应用程序存储临时文件的目录的绝对或相对路径

### 2.6 允许本地文件和本地模块<a name='allow_local'></a>
* allowLocalFilesAccess _(boolean)_ ：为true时，此属性指定jsreport在渲染执行期间应允许访问本地文件系统和使用自定义nodejs模块

### 2.7 渲染配置<a name='render_config'></a>
缺省情况下，jsreport使用专用进程来呈现pdf或脚本。该解决方案在某些带有代理的云和公司环境中更有效。但是，对于其他情况，例如，使用phantomjs时，最好在多个请求上重用让nodejs worker工作。可以使用此配置选项来实现。
```
"phantom": {     
  "strategy": "phantom-server"
},
"templatingEngines": {       
  "strategy": "http-server"
}
```

### 2.8 模板引擎配置<a name='tpl_engine_config'></a>
* templatingEngines _(object)_ ：（可选）此属性用于配置执行渲染任务的组件。此组件用于在渲染过程中或在脚本扩展中执行javascript模板引擎。

* templatingEngines.strategy _(dedicated-process | http-server | in-process)_ ：第一个策略为每个任务使用一个新的nodejs实例。第二种策略在多个请求上重用每个实例。在http-server性能更好的地方，默认dedicated-process值更适合具有代理的某些云和公司环境。最后一个in-process策略只是在同一过程中运行脚本和帮助程序。这是最快的方法，但是将这种策略与用户模板一起使用可能并不安全，因为用户模板可能存在无限循环或其他可能导致应用程序终止的严重错误。in-process当您需要使用node.js调试工具调试jsreport时，该策略也很方便。

* templatingEngines.numberOfWorkers _(number)_ ：将使用多少个子nodejs实例来执行任务

* templatingEngines.timeout _(number)_ ：以毫秒为单位指定一个任务执行的默认超时

* templatingEngines.host _(string)_ ：设置启动脚本执行服务器的自定义主机名，在需要设置特定IP的云环境中很有用。

* templatingEngines.portLeftBoundary _(number)_ ：为脚本执行服务器设置特定的端口范围

templatingEngines.portRightBoundary _(number)_ ：为脚本执行服务器设置特定的端口范围

* templatingEngines.allowedModules _(array)_ ：设置允许在模板引擎的助手内部使用（利用require导入）的外部模块。例如：`allowedModules: ["lodash", "request"]`，也可以通过设置`allowedModules: "*"`允许导入任何外部模块。如果想控制脚本而不是帮助程序，则须检查相应的文档

### 2.9 日志配置<a name='log_config'></a>
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

### 2.10 配置文件示例<a name='config_sample'></a>  [返回目录](#toc)
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
