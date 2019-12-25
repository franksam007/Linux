当使用sudo去执行一个程序时，处于安全的考虑，这个程序将在一个新的、最小化的环境中执行，也就是说，诸如PATH这样的环境变量，在sudo命令下已经被重置成默认状态了。所以当一个刚初始化的PATH变量中不包含你所要运行的程序所在的目录，用sudo去执行，你就会得到"command not found"的错误提示。

要想改变PATH在sudo会话中的初始值，用文本编辑器打开/etc/sudoers文件，找到"secure_path"一行，当你执行sudo 命令时，"secure_path"中包含的路径将被当做默认PATH变量使用。

添加所需要的路径(如 /usr/local/bin）到"secure_path"下，在开篇所遇见的问题就将迎刃而解。
```
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```
这个修改会即刻生效。
