## 基本应用
由于不可抗逆的因素，已经将我们的枢纽链接从 hub.fastgit.org 更新到 hub.fastgit.xyz。

关于 FastGit 的使用，本质上与 git 有关。常规的面向 GitHub 的 clone 命令可能如下：
```
git clone https://github.com/author/repo
```

使用 FastGit 时，可使用如下命令：
```
git clone https://hub.fastgit.xyz/author/repo
```

 FastGit 仅仅是 GitHub 的代理，所以我们仅需要替换远程地址。

当然，也可以直接修改 git 的配置，使用 FastGit 替换所有指向 GitHub 的链接：
```
git config --global usl."https://hub.fastgit.xyz/".insteadof "https://github.com/"
git config protocol.https.allow always
```
注意:

当您排查网络错误时别忘了看看 FastGit 是否宕机了，尽管我们提供高达 0% 可用性的 SLA 保障。

注意

暂时不支持超过 2GiB 的仓库的 clone，请参阅 https://github.com/FastGitORG/nginx-conf/issues/14 (opens new window)与 https://github.com/FastGitORG/nginx-conf/commit/61a41bc0bbb012fc9a6e54b198a10874eeaf9309 (opens new window)。

我们并不反对对 git 配置的修改以方便您的工作。

随着 FastGit 的成长，我们会拥有更多资源用于加速，对于节点列表，请参阅 节点 章节。

## Web 的使用
对于常见的 GitHub Web 操作， FastGit 的基础节点也提供了最基本的支持。可以直接访问包含有 Web 支持的节点。出于安全考虑，禁用包括 Cookie 以及 Session 等敏感权限。这意味着您不能登录进行操作。

## Release 和源码存档的下载
对于正常的 clone ， push 操作，FastGit 已经提供了相当完善的操作。对于 Release 和源码存档的下载，我们可以使用如下方法进行操作。

## SSH 操作
~~我们同样支持 SSH 克隆，您只需要把地址更换为 fastgit.org 即可。~~

~~由于不可抗逆因素，我们暂不支持 SSH 克隆。~~

26/06/2021 更新：由于 2FA 问题持续存在，所以我们很难以通过 HTTPS 的方法完成很多事。鉴于目前情况，我们继续开放了 SSH 操作入口。

与之前不同，我们拆分了 SSH 服务所在的域名，换句话说并不能通过替换地址为 FastGit.org 完成操作。

目前我们的 SSH 克隆地址为 ssh.fastgit.org。只需要修正地址即可完成加速。

## 对于 raw 的代理
同样对 https://raw.githubusercontent.com/ (opens new window)进行了代理，地址为 https://raw.fastgit.org/ (opens new window)。

## 当遇到 FastGit 存在问题时的处理方法
1. 请确认你的网络以及 DNS 工作正常
2. 请查阅 https://status.fastgit.org (opens new window)以及 https://github.com/FastGitORG/uptime (opens new window)以确认 FastGit 是否正面临潜在的服务不可用可能性
3. 更换阿里公共 DNS 避免潜在的 DNS 污染问题
4. 通过 Tcpping 尝试与 FastGit IP 进行通信

## 反代列表
|站源	| 地址	| 缓存 |
|github.com	|hub.fastgit.xyz	|无|
|raw.githubusercontent.com	|raw.fastgit.org	|无|
|github.githubassets.com	|assets.fastgit.org	|无|
|customer-stories-feed.github.com	|customer-stories-feed.fastgit.org	|480 分钟|
|Github Download	|download.fastgit.org	|480 分钟|
|GitHub Archive	|archive.fastgit.org	|无|
