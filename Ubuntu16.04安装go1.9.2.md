#### 安装步骤 
```
curl -O https://www.golangtc.com/static/go/1.9.2/go1.9.2.linux-amd64.tar.gz 
tar -C /usr/local -zxvf  go1.9.2.linux-amd64.tar.gz 

vim /etc/profile 
```
打开/etc/profile后   最后一行插入 
```
export GOROOT=/usr/local/go 
export PATH=$PATH:$GOROOT/bin 
export GOPATH=/home/daniel/go 
```
然后 source /etc/profile  

GOROOT：go语言的安装路径   
GOPATH：go语言中跟工作空间相关的环境变量，这个变量指定go语言的工作空间位置，是将所有的代码保存在一个工作区中，具体是指一个目录

#### 生成开发目录
```
cd /home/daniel/go

mkdir bin
mkdir src
mkdir pkg
```
> 之后构建go项目放在src下面， 生成的安装包会自动放在bin下，生成过程中的中间文件会放在pkg下面

#### 常用包获取
```
go get github.com/astaxie/beego
go get github.com/go-sql-driver/mysql
go get github.com/eclipse/paho.mqtt.golang
go get gopkg.in/mgo.v2
go get github.com/beego/bee
```
默认会下载到GOPATH的src目录下   
如果使用go get下载较慢，可以使用其他方式将项目代码下载并拷贝到GOPATH的src目录下，然后切换到项目源码下进行编译
