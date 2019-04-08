# Ubuntu18.04上配置防火墙

## Objective
The objective is to show how to enable or disable firewall on Ubuntu 18.04 Bionic Beaver Linux

## Operating System and Software Versions

Operating System: - Ubuntu 18.04 Bionic Beaver Linux

## Requirements
Privileged access to your Ubuntu 18.04 Bionic Beaver Linux installation will be required.

## Conventions

* \# - requires given _*linux commands*_ to be executed with root privileges either directly as a root user or by use of `sudo` command
* $ - requires given _*linux commands*_ to be executed as a regular non-privileged user

## Instructions
### Managing UFW from command line
UFW ( Uncomplicated Firewall ) firewall is a default firewall on Ubuntu 18.04 Bionic Beaver Linux.

### Check a current firewall status
By default the UFW is disabled. You can check the status of your firewall by executing the following linux command:
```
$ sudo ufw status
[sudo] password for linuxconfig: 
Status: inactive
```
For more verbose output append word verbose to the above command:
```
$ sudo ufw status verbose
```

### Enable Firewall
To enable firewall execute:
```
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
Firewall, is now enabled:
```
$ sudo ufw status
Status: active
```
### Disable Firewall
UFW is quite intuitive to use. To disable firewall execute:
```
$ sudo ufw disable
Firewall stopped and disabled on system startup
```
Confirm the firewall status:
```
$ sudo ufw status
Status: inactive
```
### Managing UFW via graphical user interface
Install gufw package if you wish to manage our UFW firewall via graphical user interface application. Open up terminal and enter:
```
$ sudo apt install gufw
```
Once installed, start Gufw by searching your start menu: 
