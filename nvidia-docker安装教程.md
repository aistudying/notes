> 安装Docker方法步骤参考docker[官网链接](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
> 安装nvidia-docker参考nvidia-docker[官网链接](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-1.0))
> nvidia-docker仓库源[参考链接](https://nvidia.github.io/nvidia-docker/)
## 卸载旧版本
```
$ sudo apt-get remove docker docker-engine docker.io
```
## 安装Docker-ce
### 使用repository安装
#### Ubuntu
```
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
	pub   4096R/0EBFCD88 2017-02-22
	      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
	uid                  Docker Release (CE deb) <docker@docker.com>
	sub   4096R/F273FCD8 2017-02-22
$ sudo add-apt-repository \
   "deb [arch=ppc64el] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update 		
# 查看所有版本的Docker-ce
$ apt-cache madison docker-ce
# 安装指定版本的docker-ce
# $ sudo apt-get install docker-ce=<VERSION>
$ sudo apt-get install docker-ce
```
#### CentOS
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce
# 查看所有版本的Docker-ce
$ yum list docker-ce --showduplicates | sort -r              
# 安装指定版本
$ sudo yum install docker-ce-<VERSION STRING>    
$ sudo systemctl start docker
```
### 使用脚本安装Ubuntu && CentOS
```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

## 安装nvidia-docker
### Ubuntu
```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install nvidia-docker
```
### CentOS
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | \
  sudo tee /etc/yum.repos.d/nvidia-docker.repo
sudo yum install nvidia-docker  
```

## 卸载docker
### 卸载docker-ce
```
# Ubuntu
$ sudo apt-get purge docker-ce
# CentOS
$ sudo yum remove docker-ce
```
### 手动删除残余文件
```
$ sudo rm -rf /var/lib/docker
```








