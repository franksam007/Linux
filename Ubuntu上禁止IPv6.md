To disable ipv6, you have to open /etc/sysctl.conf using any text editor and insert the following lines at the end:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
If ipv6 is still not disabled, then the problem is that sysctl.conf is still not activated.

To solve this, open a terminal(Ctrl+Alt+T) and type the command,
```
sudo sysctl -p
```
You will see this in the terminal:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
After that, if you run:
```
$ cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```
It will report:
```
1
```
If you see 1, ipv6 has been successfully disabled.
