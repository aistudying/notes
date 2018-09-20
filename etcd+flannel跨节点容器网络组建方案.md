ubuntu16.04

安装etcd

apt-get install etcd

编辑配置文件

vim /etc/default/etcd

    #[member]
    ETCD_NAME=node01                                            #节点名称
    ETCD_DATA_DIR="/var/lib/etcd/default.etcd"                  #数据存放位置
    #ETCD_WAL_DIR=""
    #ETCD_SNAPSHOT_COUNT="10000"
    #ETCD_HEARTBEAT_INTERVAL="100"
    #ETCD_ELECTION_TIMEOUT="1000"
    #ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
    ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"             #监听客户端地址
    #ETCD_MAX_SNAPSHOTS="5"
    #ETCD_MAX_WALS="5"
    #ETCD_CORS=""
    #
    #[cluster]
    #ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
    # if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
    #ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
    #ETCD_INITIAL_CLUSTER_STATE="new"
    #ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
    ETCD_ADVERTISE_CLIENT_URLS="http://node01:2379,http://node01:4001"           #通知客户端地址
    #ETCD_DISCOVERY=""
    #ETCD_DISCOVERY_SRV=""
    #ETCD_DISCOVERY_FALLBACK="proxy"
    #ETCD_DISCOVERY_PROXY=""

启动etcd并验证状态

    [root@node01 ~]# systemctl start etcd   
    [root@node01 ~]# ps -ef|grep etcd
    etcd     28145     1  1 14:38 ?        00:00:00 /usr/bin/etcd --name=master --data-dir=/var/lib/etcd/default.etcd --listen-client-urls=http://0.0.0.0:2379,http://0.0.0.0:4001
    root     28185 24819  0 14:38 pts/1    00:00:00 grep --color=auto etcd
    [root@node01 ~]# lsof -i:2379
    COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
    etcd    28145 etcd    6u  IPv6 1283822      0t0  TCP *:2379 (LISTEN)
    etcd    28145 etcd   18u  IPv6 1284133      0t0  TCP localhost:53203->localhost:2379 (ESTABLISHED)
    ........
       
    [root@node01 ~]# etcdctl set testdir/testkey0 0
    0
    [root@node01 ~]# etcdctl get testdir/testkey0
    0
    [root@node01 ~]# etcdctl -C http://etcd:4001 cluster-health
    member 8e9e05c52164694d is healthy: got healthy result from http://node01:2379
    cluster is healthy
    [root@node01 ~]# etcdctl -C http://etcd:2379 cluster-health
    member 8e9e05c52164694d is healthy: got healthy result from http://node01:2379
    cluster is healthy

创建网络

    仅etcd主节点执行：
    etcdctl mk /coreos.com/network/config '{ "Network": "10.10.0.0/16" }'



所有节点执行以下操作

wget https://github.com/coreos/flannel/releases/download/v0.10.0/flannel-v0.10.0-linux-amd64.tar.gz

tar -zxvf 	flannel-v0.10.0-linux-amd64.tar.gz

cp flanneld /usr/bin/
cp mk-docker-opts.sh /usr/bin/flannel/



vim /lib/systemd/system/flanneld.service

    [Unit]
    Description=Flanneld
    Documentation=https://github.com/coreos/flannel
    After=network.target
    After=etcd.service
    Before=docker.service
    
    [Service]
    User=root
    EnvironmentFile=/etc/default/flanneld.conf
    ExecStart=/usr/bin/flanneld \
      -etcd-endpoints=${FLANNEL_ETCD_ENDPOINTS} \
      -etcd-prefix=${FLANNEL_ETCD_PREFIX} \
      $FLANNEL_OPTIONS
    ExecStartPost=/usr/bin/flannel/mk-docker-opts.sh -k DOCKER_OPTS -d /run/flannel/docker
    Restart=on-failure
    Type=notify
    LimitNOFILE=65536
    
    [Install]
    WantedBy=multi-user.target
    RequiredBy=docker.service  

vim /etc/default/flanneld.conf



    # Flanneld configuration options
    
    # etcd url location.  Point this to the server where etcd runs
    
    FLANNEL_ETCD_ENDPOINTS="http://node01:2379"
    
    # etcd config key.  This is the configuration key that flannel queries
    
    # For address range assignment
    
    FLANNEL_ETCD_PREFIX="/coreos.com/network"
    
    # Any additional options that you want to pass
    
    # FLANNEL_OPTIONS=""

    systemctl start flanneld

修改docker的systemd配置文件

    $ vim /lib/systemd/system/docker.service
    
    [Service]
    Type=notify
    EnvironmentFile=/run/flannel/docker
    ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS

重启docker服务。

$ sudo systemctl daemon-reload

$ sudo systemctl restart docker



快捷部署计算节点

    scp -r /usr/bin/flannel* node08:/usr/bin/
    scp /lib/systemd/system/flanneld.service node08:/lib/systemd/system/
    scp /etc/default/flanneld.conf node08:/etc/default/
    scp /lib/systemd/system/docker.service node08:/lib/systemd/system/



systemctl daemon-reload

systemctl restart flanneld

systemctl restart docker


