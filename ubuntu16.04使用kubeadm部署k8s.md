### 关闭防火墙(所有节点)

```
systemctl stop ufw
systemctl disable ufw
swapoff -a			
//将swapoff -a命令写入/etc/rc.loacl文件中，开机执行
vim /etc/fstab
注释swap分区所在行
```

### 启用路由转发

```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.ipv4.ip_forward = 1

iptables -P FORWARD ACCEPT
```

### 安装Docker(所有节点)

注：由于后面想试用k8s-device-plugin特性来管理GPU，所以需要使用nvidia-docker2，而nvidia-docker2要求docker-ce版本最低为18.06，所以要安装docker-ce18.06版本

```
方法1：
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
方法2：
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt-get -y update
apt-get -y install docker-ce=18.06.1~ce~3-0~ubuntu

```

### 安装nvidia-docker2(所有节点)

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get -y update
apt-get install -y nvidia-docker2
pkill -SIGHUP dockerd

```

### 配置k8s源(所有节点)

```bash
Ubuntu16.04:
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main

CentOS7:
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 安装kubeadm(所有节点)

	#安装依赖
	apt-get install -y apt-transport-https socat aufs-tools cgroupfs-mount libltdl7 pigz
	
	apt-get install cri-tools=1.11.0-00 \
	kubeadm=1.11.1-00 \
	kubectl=1.11.1-00 \
	kubelet=1.11.1-00 \
	kubernetes-cni=0.6.0-00


### 导入镜像(主节点和计算节点均执行，但导入内容不一样)

```
主节点：
k8s.gcr.io_coredns:1.1.3
k8s.gcr.io_etcd-amd64:3.2.18
k8s.gcr.io_kube-apiserver-amd64:v1.11.1
k8s.gcr.io_kube-controller-manager-amd64:v1.11.1
k8s.gcr.io_kube-proxy-amd64:v1.11.1
k8s.gcr.io_kube-scheduler-amd64:v1.11.1
k8s.gcr.io_pause:3.1
quay.io_calico_cni:v3.1.3
quay.io_calico_node:v3.1.3

子节点：
k8s.gcr.io_kube-proxy-amd64:v1.11.1
k8s.gcr.io_kubernetes-dashboard-amd64:v1.8.3
k8s.gcr.io_pause__3.1:da86e6ba6ca1
quay.io_calico_cni:v3.1.3
quay.io_calico_node:3.1.3
```



### 初始化主节点(主节点)

```
#init主节点
kubeadm init --pod-network-cidr=172.16.0.0/16  --kubernetes-version=v1.11.1  | tee k8s.init.log
##使用flannel时网络要修改为10.244.0.0/16，示例如下
kubeadm init --pod-network-cidr=10.244.0.0/16  --kubernetes-version=v1.11.1  | tee k8s.init.log
#上面这步一定要记录好日志，尤其是节点加入的命令，因为后续无法重现，格式类似下面
kubeadm join 192.168.1.109:6443 --token t7fra0.dhjys8zaka71zo80 --discovery-token-ca-cert-hash sha256:4344ddef08a385abf37300693708806f62024eb6f77ce3994dc523d4b4dbe346

mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

### 部署网络flannel(主节点)

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

### 加入k8s集群

```
kubeadm join 192.168.1.109:6443 --token t7fra0.dhjys8zaka71zo80 --discovery-token-ca-cert-hash sha256:4344ddef08a385abf37300693708806f62024eb6f77ce3994dc523d4b4dbe346
```

### 查看集群状态

```
# kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    5m        v1.11.1
node02    Ready     <none>    15s       v1.11.1

# kubectl get pod --all-namespaces -o wide

NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
kube-system   coredns-78fcdf6894-5pjm7         1/1       Running   0          6m        10.244.0.4      master
kube-system   coredns-78fcdf6894-vjh89         1/1       Running   0          6m        10.244.0.5      master
kube-system   etcd-master                      1/1       Running   0          5m        192.168.1.109   master
kube-system   kube-apiserver-master            1/1       Running   0          5m        192.168.1.109   master
kube-system   kube-controller-manager-master   1/1       Running   0          5m        192.168.1.109   master
kube-system   kube-flannel-ds-amd64-42lhb      1/1       Running   0          1m        192.168.1.251   node02
kube-system   kube-flannel-ds-amd64-tn5lf      1/1       Running   0          2m        192.168.1.109   master
kube-system   kube-proxy-9zq7l                 1/1       Running   0          6m        192.168.1.109   master
kube-system   kube-proxy-cvhsr                 1/1       Running   0          1m        192.168.1.251   node02
kube-system   kube-scheduler-master            1/1       Running   0          5m        192.168.1.109   master
```

### 简单测试

```
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
测试DNS解析
kubectl run -ti --rm test --image=busybox:1.28.4 --restart=Never -- nslookup baidu.com

# 正常执行，再看看调度情况

$ kubectl get pod --show-all -o wide
NAME      READY     STATUS      RESTARTS   AGE       IP           NODE
busybox   0/1       Completed   0          48s       10.244.1.2   node1
```

### 部署k8s-device-plugin

```
#所有节点修改/etc/docker/daemon.json文件内容如下	
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

# 在主节点上部署k8s-device-plugin服务
$ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.11/nvidia-device-plugin.yml
```

### 部署dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

### 登录dashboard

根据官方介绍一共有三种方式：NodePort方式、kubectl proxy 方式、API server方式

kubectl proxy方式：

```
master节点上执行
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'
浏览器访问：
http://192.168.207.129:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

NodePort方式：

修改kubernetes-dashboard.yaml文件中最后的部分service，再重新部署

需要注意的是nodeport方式使用https方式连接，但是谷歌浏览器对安全验证无法通过，使用火狐浏览器在高级里面添加例外即可访问。

```
spec:
  # 添加Service的type为NodePort
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      # 添加映射到虚拟机的端口,只支持30000--32767的端口
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard

浏览器访问：
https：//master ip:nodeport	#如https://192.168.207.129:30001
```

用户登录验证

有两种方式：kube-config和token，这里我们使用token方式就可以了，因为kube-config文件中实际上包含用户名和token，因此token的获取是必须的

```
获取token：
在master节点上执行
kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
然后在浏览器输入获取的token即可登录
```

### 部署Heapster、InfluxDB和Grafana

```
wget https://github.com/kubernetes/heapster/archive/v1.5.4.tar.gz
tar -zxvf v1.5.4.tar.gz && cd heapster-1.5.4/deploy/kube-config/
ll influxdb/
#-rw-rw-r-- 1 root root 2290 7月  26 21:24 grafana.yaml
#-rw-rw-r-- 1 root root 1114 7月  26 21:24 heapster.yaml
#-rw-rw-r-- 1 root root  974 7月  26 21:24 influxdb.yaml
ll rbac/
#-rw-rw-r-- 1 root root  264 7月  26 21:24 heapster-rbac.yaml
以上4个yaml文件即为需要用到的文件
所需用到的镜像：
gcr.io/google_containers/heapster-amd64:v1.5.3
gcr.io/google_containers/heapster-grafana-amd64:v4.4.3
gcr.io/google_containers/heapster-influxdb-amd64:v1.3.3


修改 grafana.yaml文件
spec:
  # In a production setup, we recommend accessing Grafana through an external Loadbalancer
  # or through a public IP.
  # type: LoadBalancer
  # You could also use NodePort to expose the service at a randomly-generated port
  type: NodePort # 去掉注释即可
  ports:
  - port: 80
    targetPort: 3000
  selector:
    k8s-app: grafana
修改为nodport类型，将Grafana暴露在宿主机Node的端口上，以便后续浏览器访问 grafana 的 admin UI 界面

开始执行部署：
kubectl create -f influxdb/ && kubectl create -f rbac/
```

遇到的问题：failed to get all container stats from Kubelet URL

所有节点执行以下操作即可：

```
vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
change to like this

Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml --read-only-port=10255 "

then

systemctl daemon-reload

systemctl restart kubelet
```



## 删除节点

```
主节点：
kubectl drain node02 --delete-local-data --force --ignore-daemonsets
kubectl delete node node02
计算节点：
kubeadm reset
```

## 常用命令

```
kubectl get pod --all-namespaces -o wide
kubectl apply -f kubernetes-dashboard.yaml
kubectl delete -f kubernetes-dashboard.yaml
kubectl -n kube-system get svc kubernetes-dashboard
kubectl get pods --namespace kube-system

查看可用GPU资源
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"

kubeadm init失败清理命令
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/


使master node参与工作负载
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## 忘记初始master节点时的node节点加入集群命令怎么办

```
# 简单方法
kubeadm token create --print-join-command

# 第二种方法
token=$(kubeadm token generate)
kubeadm token create $token --print-join-command --ttl=0
```

### 解决coredns无限重启问题

```
mkdir /run/systemd/resolve/
vim /run/systemd/resolve/resolv.conf
添加真实的DNS服务器
vim /var/lib/kubelet/config.yaml
修改resolve路径
```

### 在k8s运行tensorflow分布式

在集群中任意一台物理机上执行以下操作

```
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow/tensorflow/tools/dist_test
生成k8s部署集群的yaml文件
scripts/k8s_tensorflow.py \
    --num_workers 2 \
    --num_parameter_servers 2 \
    --grpc_port 2222 \
    --request_load_balancer true \
    --docker_image "tensorflow/tf_grpc_server" \
    > tf-k8s-with-lb.yaml

#生成后,执行部署
kubectl create -f tf-k8s-with-lb.yaml

#构建客户端测试镜像并测试
./remote_test.sh
...　// 镜像打包输出
NUM_WORKERS = 2
NUM_PARAMETER_SERVERS = 2
SETUP_CLUSTER_ONLY = 0
GRPC_SERVER_URLS: 
SYNC_REPLICAS: 0
GRPC port to be used when creating the k8s TensorFlow cluster: 2222
Path to gcloud binary: /var/gcloud/google-cloud-sdk/bin/gcloud　
gcloud service account key file cannot be found at: /var/gcloud/secrets/tensorflow-testing.json // 这里并不重要
FAILED to determine GRPC server URLs of all workers　//　错误提示,并且上面GRPC_SERVER_URLS:也为空 
```

根据错误提示，执行以下命令

```
docker run -it tensorflow/tf-dist-test-client　//　此为刚才remote_test.sh生成的镜像,docker hub上并没有
进入容器内
export TF_DIST_GRPC_SERVER_URLS="grpc://10.244.0.16:2222 grpc://10.244.1.31:2222"
cd /var/tf-dist-test/scripts
./dist_test.sh
或者执行下面命令亦可
./dist_mnist_test.sh "grpc://10.244.0.16:2222 grpc://10.244.1.31:2222" --num-workers 2 --num-parameter-servers 1
```

