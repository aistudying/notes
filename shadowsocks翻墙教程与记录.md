## 服务器端：
### 免费shadowsocks server
```
https://ss.freess.org/index.html
```
### 收费shadowsocks server
```
https://portal.shadowsocks.to/cart.php?gid=1
```
### 使用vps搭建shadowsocks server
vps推荐：Google(免费一年), AWS(免费一年), Azure(免费一年), [Vultr](https://www.vultr.com/?ref=7245210)(前面都用过了就用这个吧，每月3刀)   
安装脚本：
```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev.sh
chmod +x shadowsocks-libev.sh
./shadowsocks-libev.sh 2>&1 | tee shadowsocks-libev.log
```
根据提示输入端口，密码等信息，安装完成后的脚本输出
```
Congratulations, Shadowsocks-python server install completed!
Your Server IP :0.0.0.0                   #your_server_ip
Your Server Port :9000                    #your_server_port
Your Password :123456                     #your_password
Your Encryption Method: aes-256-cfb       #your_encryption_method

Welcome to visit: https://teddysun.com/342.html
Enjoy it!
```
单用户配置文件 (配置文件路径: /etc/shadowsocks.json):
```
{
    "server": "your_server_ip",
    "server_port": your_server_port,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "your_password",
    "timeout": 300,
    "method": "your_encryption_method",
    "fast_open": false
}
```
多用户配置文件
```
{
    "server": "your_server_ip",
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "port_password":{
         "8989": "password0",
         "9001": "password1",
         "9002": "password2",
         "9003": "password3",
         "9004": "password4"
    },
    "timeout": 300,
    "method": "your_encryption_method",
    "fast_open": false
}
```
启动服务：   
```
/etc/init.d/shadowsocks start
```
为shadowsocks端口设置防火墙规则：
```
iptables -I INPUT -p tcp --dport your_server_port -j ACCEPT
iptables -I OUTPUT -p tcp --dport your_server_port -j ACCEPT
```
安装加速器(可选)
```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```
## 客户端：
### windows客户端
> 详细的设置文档请参考[Shadowsocks 设置方法 (Windows)](https://github.com/Shadowsocks-Wiki/shadowsocks/blob/master/2-windows-setup-guide-cn.md)   

下载地址：
```
https://github.com/shadowsocks/shadowsocks-windows/releases
```
> 下载最新版本的 Shadowsocks-x.x.x.zip (x.x.x为版本号), 解压到相应位置, 运行 Shadowsocks.exe, 此时右下角状态栏会出现shadowsocks图标, 说明运行正常   

设置服务器:     
vps服务器添加方法:    
双击shadowsocks图标编辑服务器信息, 依次填写你的服务器信息即可
```
服务器地址:0.0.0.0
服务器端口:9000
密码:******
加密方式:aes-256-cfb
代理端口:1080
```
免费服务器添加方法:    
```
访问免费shadowsocks网站, 点击其中一个免费服务器, 会在屏幕上出现二维码   
右键状态栏shadowsocks图标 > "服务器” > "扫描屏幕上的二维码"
点击确定, 添加成功   
```
启用代理:
```
右键状态栏shadowsocks图标 > "启用系统代理”
```
设置代理模式:    
全局模式: 所有internet访问均走代理服务器的链路
```
右键状态栏shadowsocks图标 > "系统代理模式" > "全局模式"
```
PAC模式: 自动代理, 本来可以访问的网站不会经过代理, 推荐日常使用
```
右键状态栏shadowsocks图标 > "系统代理模式" > "PAC模式"
右键状态栏shadowsocks图标 > "PAC" > "本地PAC"
右键状态栏shadowsocks图标 > "PAC" > "从GFWList更新本地PAC"
```
此时系统内的所有浏览器就应该可以访问被GFW墙掉的网站了
### Ubuntu客户端
