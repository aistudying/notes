### 在服务器上安装Mellanox OFED驱动   
下载驱动，下载地址为
```
http://www.mellanox.com > Products > Software > InfiniBand/VPI Drivers > Mellanox OFED Linux (MLNX_OFED) > Download.
```
安装驱动
```
Copy the downloaded tgz to /tmp
# cd /tmp
# tar -xzvf MLNX_OFED_LINUX-4.2-1.0.0.0-ubuntu16.04-x86_64.tgz
# cd MLNX_OFED_LINUX-4.2-1.0.0.0-ubuntu16.04-x86_64/
Run the installation script.
# ./mlnxofedinstall
Reboot after the installation finished successfully.
# /etc/init.d/openibd restart

# reboot
```
> By default both ConnectX®-5 VPI ports are initialized as Infiniband ports.

查看并修改port type
```
# ibv_devinfo

If you see the following - You need to change the interfaces port type to Infiniband
Capture.JPG
Change the interfaces port type to Infiniband mode ConnectX®-5 ports can be individually configured to work as Infiniband or Ethernet ports. 
Change the mode to Infiniband. Use the mlxconfig script after the driver is loaded.
* LINK_TYPE_P1=1 is a Infiniband mode
a. Start mst and see ports names
# mst start
# mst status
b. Change the mode of both ports to Infiniband:

# mlxconfig -d /dev/mst/mt4121_pciconf0 s LINK_TYPE_P1=1 
#Port 1 set to IB mode
# reboot
```

设置IP地址
```
# vim /etc/network/interfaces

auto eno1

iface eno1 inet dhcp

The new lines:
auto ib0
iface ib0 inet static
address 12.12.12.41
netmask 255.255.255.0
Example:
# vim /etc/network/interfaces

auto eno1
iface eno1 inet dhcp

auto ib0
iface ib0 inet static
address 12.12.12.41
netmask 255.255.255.0
Check the network configuration is set correctly.
# ifconfig -a
 ```

### 安装Docker-ce请参考[另一篇文档](https://github.com/aistudying/notes/blob/master/nvidia-docker安装教程.md)
 
### 设置跨节点容器间的通信
> 如果需要完整的跨节点的容器间的网络，请参考[另一篇文档](https://github.com/aistudying/notes/blob/master/etcd+flannel跨节点容器网络组建方案.md)
设置docker桥接模式
```
Customize the docker0 bridge
The recommended way to configure the Docker daemon is to use the daemon.json file, which is located in /etc/docker/ on Linux. If the file does not exist, create it. You can specify one or more of the following settings to configure the default bridge network

{      
     "bip": "172.16.41.1/24",
     "fixed-cidr": "172.16.41.0/24",
     "mtu": 1500, 
     "dns": ["8.8.8.8","8.8.4.4"] 
}
 

The same options are presented as flags to dockerd, with an explanation for each:

--bip=CIDR: supply a specific IP address and netmask for the docker0 bridge, using standard CIDR notation. For example: 172.16.41.1/16.
--fixed-cidr=CIDR: restrict the IP range from the docker0 subnet, using standard CIDR notation. For example: 172.16.41.0/16.
--mtu=BYTES: override the maximum packet length on docker0. For example: 1500.
--dns=[]: The DNS servers to use. For example: --dns=8.8.8.8,8.8.4.4.
 

Restart Docker after making changes to the daemon.json file.

$ sudo /etc/init.d/docker restart
```
设置容器与外部主机通信
```
Check ip forwarding in kernel:

$ sysctl net.ipv4.conf.all.forwarding
net.ipv4.conf.all.forwarding = 1

If disabled

$ sysctl net.ipv4.conf.all.forwarding
net.ipv4.conf.all.forwarding = 0

please enable and check again:

$ sysctl net.ipv4.conf.all.forwarding=1
 

For security reasons, Docker configures the iptables rules to prevent containers from forwarding traffic from outside the host machine, on Linux hosts. Docker sets the default policy of the FORWARD chain to DROP.

To override this default behavior you can manually change the default policy:

$ sudo iptables -P FORWARD ACCEPT
 

Add IP route with specific subnet
Add routing for containers network on second host:

$ sudo ip route add 172.16.42.0/24 via 12.12.12.42  
 

A quick check
Give your environment a quick test run to make sure you’re all set up:

$ docker run hello-world
```
### 构建镜像或者从docker hub下载镜像启动容器

方法1：
从Docker Hub下载镜像并以privileged mode 启动容器:
```
$ sudo docker run -it --privileged --name=mnlx-verbs-prvlg mellanox/mofed421_docker:latest bash
```

方法2：
从Docker Hub下载镜像不使用privileged mode 启动容器:
```
$ sudo docker run -it --cap-add=IPC_LOCK --device=/dev/infiniband/uverbs1 --name=mnlx-verbs-nonprvlg mellanox/mofed421_docker:latest bash
 ```

方法3：
使用Dockerfile构建镜像并使用以上两种方式来启动容器
Dockerfile
```
# Use an official Ubuntu 16.04 as a parent image

FROM ubuntu:16.04

MAINTAINER YOUR NAME <your@real.mail>

#  Set MOFED directory, image and working directory

ENV MOFED_DIR MLNX_OFED_LINUX-4.2-1.0.0.0-ubuntu16.04-x86_64
ENV MOFED_SITE_PLACE MLNX_OFED-4.2-1.0.0.0
ENV MOFED_IMAGE MLNX_OFED_LINUX-4.2-1.0.0.0-ubuntu16.04-x86_64.tgz
WORKDIR /tmp/

# Pick up some MOFED dependencies

RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        net-tools \
        ethtool \
        perl \
        lsb-release \
        iproute2 \
        pciutils \
        libnl-route-3-200 \
        kmod \
        libnuma1 \
        lsof \
        linux-headers-4.4.0-92-generic \
        python-libxml2 && \
        rm -rf /var/lib/apt/lists/*

# Download and install Mellanox OFED 4.2.1 for Ubuntu 16.04

RUN wget http://content.mellanox.com/ofed/${MOFED_SITE_PLACE}/${MOFED_IMAGE} && \
        tar -xzvf ${MOFED_IMAGE} && \
        ${MOFED_DIR}/mlnxofedinstall --user-space-only --without-fw-update --all -q && \
        cd .. && \
        rm -rf ${MOFED_DIR} && \
        rm -rf *.tgz
```
 
创建一个空的文件夹，并在里面创建上面内容的Dockerfile文件，执行下面命令开始构建镜像
```
$ docker build -t myofed421image .
#查看镜像
$ docker images
```
使用一下命令运行容器：
```
$ docker run -it --privileged --name=mnlx-verbs-prvlg --name=my-verbs-nonprvlg myofed421image bash
OR
$ docker run -it --cap-add=IPC_LOCK --device=/dev/infiniband/uverbs1 --name=my-verbs-nonprvlg myofed421image bash
```
### 性能测试
```
在容器内运行IB压力测试
Server
ib_write_bw -a -d mlx5_1 &

Client
ib_write_bw -a -F $server_IP -d mlx5_1 --report_gbits
```
> 针对启用了用户命名空间的容器环境，则需要在启动容器时加上``--userns=host``参数，例如
> docker run -it --privileged --userns=host --name=mnlx-verbs-prvlg --name=my-verbs-nonprvlg mellanox/mofed421_docker:latest bash
