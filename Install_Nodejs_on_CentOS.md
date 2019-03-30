# Prerequisites
Before continuing with this tutorial, make sure you are logged in as a user with sudo privileges.

# Installing Node.js and npm on CentOS 7
NodeSource is a company dedicated to providing enterprise-grade Node support and they maintain a consistently-updated Node.js repository for Linux distributions.


To install Node.js and npm from the NodeSource repositories on your CentOS 7 system, follow these steps:

## 1. Add NodeSource yum repository
The current LTS version of Node.js is version 10.x. If you want to install version 8 just change setup_10.x with setup_8.x in the command below.

Run the following curl command to add the NodeSource yum repository to your system:

`curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -`

## 2. Install Node.js and npm
Once the NodeSource repository is enabled, install Node.js and npm by typing:

`sudo yum install nodejs`

When prompted to import the repository GPG key, type y, and press Enter.


## 3. Verify the Node.js and npm Installation
To check that the installation was successful, run the following commands which will print the Node.js and npm versions.

Print Node.js version:
```
node --version
v10.13.0
```

Print npm version:
```
npm --version
6.4.1
```

# How to install Node.js and npm using NVM

NVM (Node Version Manager) is a bash script used to manage multiple active Node.js versions. NVM allows us to install and uninstall any specific Node.js version which means we can have any number of Node.js versions we want to use or test.

To install Node.js and npm using NVM on your CentOS system, follow these steps:

## 1. Install NVM (Node Version Manager)
To download the nvm install script run the following command:

`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash`

The script will clone the nvm repository from Github to ~/.nvm and add the script Path to your Bash or ZSH profile.

=> Close and reopen your terminal to start using nvm or run the following to use it now:
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
As the output above shows, you should either close and reopen your terminal or run the commands to add the path to nvm script to your current session.

To verify that nvm was properly installed type:
```
nvm --version
0.33.11
```
## 2. Install Node.js using NVM
Now that the nvm tool is installed we can install the latest available version of Node.js, by typing:
```
nvm install node

Downloading and installing node v11.0.0...
Downloading https://nodejs.org/dist/v11.0.0/node-v11.0.0-linux-x64.tar.xz...
######################################################################## 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v11.0.0 (npm v6.4.1)
Creating default alias: default -> node (-> v11.0.0)
```

Verify the Node.js version, by typing:
```
node --version

v10.1.0
```

## 3. Install multiple Node.js versions using NVM
Let’s install two more versions, the latest LTS version and version 8.12.0
```
nvm install --lts
nvm install 8.12.0
```
Once LTS version and 8.12.0 are installed to list all installed Node.js instances type:
```
nvm ls

->      v8.12.0                         # ACTIVE VERSION
       v10.13.0
        v11.0.0
default -> node (-> v11.0.0)           # DEFAULT VERSION
node -> stable (-> v11.0.0) (default)
stable -> 11.0 (-> v11.0.0) (default)
iojs -> N/A (default)
lts/* -> lts/dubnium (-> v10.13.0)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.14.4 (-> N/A)
lts/carbon -> v8.12.0
lts/dubnium -> v10.13.0
```

The output tell us that the entry with an arrow on the left (-> v8.12.0), is the version used in the current shell session and the default version is set to v11.0.0. Default version is the version that will be active when opening new shells.

To change the currently active version you can use the following command:
```
nvm use 10.13.0
```
The output will look like something this:
```
Now using node v10.13.0 (npm v6.4.1)
```
To change the default Node.js version type:
```
nvm alias default 10.13.0

Copydefault -> 10.13.0 (-> v10.13.0)
```

Install development tools

To be able to build native modules from npm we will need to install the development tools and libraries:

`sudo yum install gcc-c++ make`
