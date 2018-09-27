ubuntu自带的 node.js 版本太低   
安装最新版本node.js需要两步即可   
第一步，去 nodejs 官网 https://nodejs.org 看最新的版本号   
官网提供两个版本，一个是lts版本，长期支持，较稳定，一个是最新版本，有最新特性，但是可能存在未知bug
第二步，根据版本号，更新软件源并安装
node.js 的每个大版本号都有相对应的源，比如这里的 8.x.x版本的源是https://deb.nodesource.com/setup_8.x   
我这里安装LTS版，版本号为8.12.0
```
sudo apt-get install curl python-software-properties
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```
查看版本号：
```
~# npm -v
6.4.1
~# node -v
v8.12.0
```
