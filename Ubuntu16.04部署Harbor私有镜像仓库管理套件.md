> Harbor以Docker官方的Registry为基础，在其上增加了管理UI、访问控制等企业用户需要的功能。   
> Harbor官方发布的版本使用docker-compose来编排Harbor的各个组件(容器)
### Environment   
```
Ubuntu 16.04 lts
Docker version 18.06.1-ce
docker-compose version 1.8.0
Harbor v1.5.1
```
### 安装Harbor
下载安装包   
```
离线安装包：
wget https://storage.googleapis.com/harbor-releases/release-1.5.0/harbor-offline-installer-v1.5.1.tgz
在线安装包：
wget https://storage.googleapis.com/harbor-releases/harbor-online-installer-v1.5.1.tgz

tar -zxvf harbor-online-installer-v1.5.1.tgz
cd harbor/
```
> 其各组件所需的镜像均可在国内网络访问，我这里使用的的是在线安装包的方式，免去导入镜像的步骤   
修改harbor.cfg：
```
hostname = 192.168.100.11   #设置hostname为主机的ip
```
运行install.sh（root权限）：
```
/opt/harbor$ sudo ./install.sh 

[Step 0]: checking installation environment ...

Note: docker version: 18.06.1

Note: docker-compose version: 1.8.0


[Step 1]: preparing environment ...
Clearing the configuration file: ./common/config/nginx/nginx.conf
Clearing the configuration file: ./common/config/jobservice/config.yml
Clearing the configuration file: ./common/config/jobservice/env
Clearing the configuration file: ./common/config/ui/app.conf
Clearing the configuration file: ./common/config/ui/env
Clearing the configuration file: ./common/config/ui/private_key.pem
Clearing the configuration file: ./common/config/adminserver/env
Clearing the configuration file: ./common/config/registry/config.yml
Clearing the configuration file: ./common/config/registry/root.crt
Clearing the configuration file: ./common/config/log/logrotate.conf
Clearing the configuration file: ./common/config/db/env
loaded secret from file: /data/secretkey
Generated configuration file: ./common/config/nginx/nginx.conf
Generated configuration file: ./common/config/adminserver/env
Generated configuration file: ./common/config/ui/env
Generated configuration file: ./common/config/registry/config.yml
Generated configuration file: ./common/config/db/env
Generated configuration file: ./common/config/jobservice/env
Generated configuration file: ./common/config/jobservice/config.yml
Generated configuration file: ./common/config/log/logrotate.conf
Generated configuration file: ./common/config/jobservice/config.yml
Generated configuration file: ./common/config/ui/app.conf
Generated certificate, key file: ./common/config/ui/private_key.pem, cert file: ./common/config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.


[Step 2]: checking existing instance of Harbor ...

Note: stopping existing Harbor instance ...
Stopping harbor-adminserver ... done
Stopping redis ... done
Stopping harbor-db ... done
Stopping harbor-log ... done
Removing harbor-adminserver ... done
Removing redis ... done
Removing harbor-db ... done
Removing harbor-log ... done
Removing network harbor_harbor


[Step 3]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log
Creating redis
Creating harbor-db
Creating registry
Creating harbor-adminserver
Creating harbor-ui
Creating nginx
Creating harbor-jobservice

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://192.168.100.11. 
For more details, please visit https://github.com/vmware/harbor .
```
> harbor的nginx组件默认监听80端口，直接在浏览器打开http://192.168.61.11，输入默认用户名密码admin/Harbor12345. 即可打开Harbor的管理UI界面。
### Harbor组件启动和停止
```
docker-compose stop
docker-compose start
```
### 更新Harbor的配置

修改docker-compose.yml文件：
```
要修改harbor的nginx组件端口为8090。
proxy:
    image: vmware/nginx:1.11.5-patched
    container_name: nginx
    restart: always
    volumes:
      - ./common/config/nginx:/etc/nginx:z
    networks:
      - harbor
    ports:
      - 8090:80         #左边为物理主机的端口
要修改harbor的仓库镜像存储路径为/data/registry
registry:
    image: vmware/registry-photon:v2.6.2-v1.5.1
    container_name: registry
    restart: always
    volumes:
      - /data/registry:/storage:z       #左边为物理主机存储镜像的路径
      - ./common/config/registry/:/etc/registry/:z
```
修改harbor.cfg：
```
hostname = 192.168.61.11:8090
```
因为修改了配置需要重新prepare：
```
cd harbor/
docker-compose down -v
./prepare
docker-compose up -d
```
### 使用docker client测试
因为docker客户端默认采用https访问docker registry，而我们默认安装的Harbor并没有启用https。 我们这里简单测试，因此可以在Docker客户端所在的机器修改/etc/docker/daemon.json：
```
{
    "insecure-registries": ["192.168.100.11:8090"]
}
```
重启改机器上的Docker:
```
systemctl restart docker
```
测试：
```
docker login -u admin -p Harbor12345 192.168.100.11:8090
Login Succeeded

docker pull alpine
docker tag alpine 192.168.100.11:8090/library/alpine

docker push 192.168.100.11:8090/library/alpine
```
至此Harbor已经快速run起来了，基本的使用快速过一遍[Harbor User Guide](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md).
