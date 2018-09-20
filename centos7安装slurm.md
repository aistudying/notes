关闭防火墙(所有节点)

```
systemctl stop firewalld
systemctl disable firewalld
 vim /etc/selinux/config  中 SELINUX=disabled  永久需重启

setenforce 0
```

ssh免密(所有节点)

```
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.107
```

配置NTP服务：

yum install ntp ntpdate -y

或者使用rpm包离线安装

```
wget https://rpmfind.net/linux/fedora-secondary/development/rawhide/Everything/ppc64le/os/Packages/n/ntpdate-4.2.8p12-1.fc30.ppc64le.rpm

wget https://rpmfind.net/linux/opensuse/ports/update/leap/15.0/oss/ppc64le/libopenssl1_1-1.1.0h-lp150.3.6.1.ppc64le.rpm

wget https://rpmfind.net/linux/fedora-secondary/development/rawhide/Everything/ppc64le/os/Packages/n/ntpdate-4.2.8p12-1.fc30.ppc64le.rpm

yum localinstall -y *.rpm
```

ntp server配置：

```
vim /etc/ntp.conf
插入以下一行
restrict 192.168.205.0 mask 255.255.255.0 nomodify notrap  

其中：
用restrict控管权限 
nomodify - 用户端不能更改ntp服务器的时间参数 
noquery - 用户端不能使用ntpq，ntpc等命令来查询ntp服务器 
notrap - 不提供trap远端登陆 
192.168.10.0/24 - 对这个网段的计算机提供时钟同步
```

开启ntp服务:

```
systemctl start ntpd	// 启动ntp服务  
systemctl enable ntpd	// 让ntp服务开机启动  
```

ntp客户端配置：

安装主节点相同的安装包，方法同上

```
vim /ntp.conf
插入以下一行
server 192.168.205.1   //ntp服务器的ip地址 

与ntp sever同步一下时间
ntpdate -u 192.168.205.1
```

开启ntp服务:

```
systemctl start ntpd	// 启动ntp服务  
systemctl enable ntpd	// 让ntp服务开机启动  
```



安装munge、munge-devel(所有节点)

```
yum install munge munge-devel
```

修改文件权限(所有节点)

```
chmod -Rf 700 /etc/munge
chmod -Rf 711 /var/lib/munge
chmod -Rf 700 /var/log/munge
chmod -Rf 0755 /var/run/munge
```

主节点 munge生成的key，并拷贝到其他节点(所有节点)

```
/usr/sbin/create-munge-key
scp /etc/munge/munge.key  root@node: /etc/munge
```

创建slurm用户(所有节点)

```
export SLURMUSER=412 
groupadd -g $SLURMUSER slurm 

useradd -m -c "SLURM workload manager" -d /var/lib/slurm -u $SLURMUSER -g slurm -s /bin/bash slurm

id slurm
```

下载、编译并安装slurm(所有节点)

安装依赖

```
yum install -y epel-release
yum install -y gcc \
	wget \
	openssl \
	openssl-devel \
	pam-devel \
	numactl \
	numactl-devel \
	hwloc \
	hwloc-devel \
	lua \
	lua-devel \
	readline-devel \
	rrdtool-devel \
	ncurses-devel \
	man2html \
	libibmad \
	libibumad \
	perl-Switch \
	perl-ExtUtils-MakeMaker \
	readline-devel \
	pam-devel \
	perl-DBI \
	mariadb \
	mariadb-devel \
	rpm-build
```

slurm下载地址：https://download.schedmd.com/slurm/

```
wget  https://download.schedmd.com/slurm/slurm-17.11.9-2.tar.bz2
rpmbuild -ta slurm-17.11.9-2.tar.bz2
cd /root/rpmbuild/RPMS/ppc64le
yum --nogpgcheck localinstall slurm-*.rpm
mkdir /var/spool/slurm
mkdir /var/spool/slurm/ctld
mkdir /var/spool/slurm/d
chown -R slurm:slurm /var/spool/slurm
```

修改配置文件/etc/slurm/slurm.conf(仅主节点)

```
ControlMachine=m1
###***  CPUs=1 = Sockets*CoresPerSocket*ThreadsPerCore
NodeName=m1,s[2-3] CPUs=1 RealMemory=1024 Sockets=1 CoresPerSocket=1 ThreadsPerCore=1 Procs=1 State=IDLE

PartitionName=clients Nodes=s[2-3] Default=YES MaxTime=INFINITE State=UP
```

发送到客户端配置文件(所有节点)

```
scp /etc/slurm/slurm.conf  node:/etc/slurm/slurm.conf

chown slurm:slurm /etc/slurm/slurm.conf
```

启动服务

```
systemctl restart munge				#所有节点
systemctl restart slurmd			#所有节点
systemctl restart slurmctld				#仅主节点
systemctl enable munge				#所有节点
systemctl enable slurmd				#所有节点
systemctl enable slurmctld				#仅主节点
```



问题：

启动后节点为down或者drain状态

```
If no jobs are currently running on the node:
	scontrol update nodename=node10 state=idle

If jobs are running on the node:
	scontrol update nodename=node10 state=resume
```

