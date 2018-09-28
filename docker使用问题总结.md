> 以下内容仅记录个人在使用docker的过程中遇上的一些问题及对应的解决方法

Q1：用户资源隔离
```
编辑/etc/docker/daemon.json文件
vim /etc/docker/daemon.json

添加以下内容：
"userns-remap": "default",
```
Q2：非root用户运行docker
```
#创建docker组
sudo groupadd docker

#将当前用户加入docker组
sudo gpasswd -a ${USER} docker

#重启docker
sudo systemctl restart docker
```
> 如果普通用户执行docker命令，如果提示get …… dial unix /var/run/docker.sock权限不够，则修改/var/run/docker.sock权限,使用root用户执行如下命令   
> sudo chmod a+rw /var/run/docker.sock   
Q3：docker代理配置
