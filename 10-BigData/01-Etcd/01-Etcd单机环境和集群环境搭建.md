
Zookeeper单机环境和集群环境搭建
===========
```sh
一、单机环境搭建
	1.1 下载
	1.2 解压
	1.3 配置环境变量
	1.4 启动
	1.5 测试
	1.6 关闭 

二、集群环境搭建
	2.1 集群规划
	2.2 启动集群
	2.3 设为开机启动
	2.4 集群验证

```


一、单机环境搭建
===========

1.1 下载
-----------
到etcd的github地址，下载最新的安装包：  
https://github.com/etcd-io/etcd/releases/

安装包版本举例说明：
- etcd-版本号-darwin-amd64.zip - macos版本
- etcd-版本号-linux-amd64.tar.gz - linux 64位版本
- etcd-版本号-windows-amd64.zip - windows 64位版本

根据自己的系统版本选择下载即可。   
> wget https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz


1.2 解压
-----------
解压到指定目录下  
> tar -zxvf etcd-v3.4.9-linux-amd64.tar.gz -C /usr/local/

这里假设安装在 /usr/local/目录下，当然也可以自已起个目录，比如：/usr/app/。

解压缩包后，将得到类似的目录结构:
```sh
etcd-v3.2.28-darwin-amd64/
├── Documentation    - etcd文档目录
├── etcd             - etcd服务端程序
└── etcdctl          - etcd客户端程序，用来操作服务端
```

可以建个软链，方便访问  
> ln -s /usr/local/etcd-v3.4.9 /usr/local/etcd


1.3 配置环境变量
-----------
> vim /etc/profile

添加环境变量：
```sh
export ETCD_HOME=/usr/local/etcd-v3.4.9
export PATH=$ETCD_HOME/bin:$PATH
```

使得配置的环境变量生效： 
> source /etc/profile


1.4 启动
-----------
创建相应的目录 (数据文件目录，日志文件目录)  
```sh
etcd
........忽略.....
2019-11-14 23:11:46.533058 I | embed: listening for client requests on localhost:2379


etcd --version
#输出
#etcd Version: 3.4.9
#Git SHA: 54ba95891
#Go Version: go1.12.17
#Go OS/Arch: linux/amd64
```


1.5 测试
-----------
我们可以通过安装目录的etcdctl命令测试，etcd是否启动成功。
```sh
etcdctl version
#输出
#etcdctl version: 3.4.9
#API version: 3.4


etcdctl put /config/title qv001
etcdctl get /config/title

#输出
qv001
```

1.6 关闭服务
-----------
只要杀掉etcd进程既可。

```sh
# 假如60999是etcd进程id
kill 60999
```

注意：不要使用kill -9 杀掉进程，可能会导致etcd丢失数据。






二、集群环境搭建
===========
etcd集群部署，通常至少部署3个etcd节点(推荐奇数个节点)，下面一步步接受集群搭建方法。

2.1 集群规划
-----------
这里规划一个由3台服务器节点组成的etcd集群，如下表：
```sh
节点名字	服务器Ip
infra0	10.0.1.10
infra1	10.0.1.11
infra2	10.0.1.12
```

etcd关键参数说明
```sh
--name		
#etcd节点名字

--initial-cluster	
##etcd启动的时候，通过这个配置找到其他ectd节点的地址列表，格式：'节点名字1=http://节点ip1:2380,节点名字1=http://节点ip1:2380,.....'

--initial-cluster-state	
#初始化的时候，集群的状态 "new" 或者 "existing"两种状态，new代表新建的集群，existing表示加入已经存在的集群。

--listen-client-urls	
#监听客户端请求的地址列表，格式：'http://localhost:2379', 多个用逗号分隔。

--advertise-client-urls	
#如果--listen-client-urls配置了，多个监听客户端请求的地址，这个参数可以给出，建议客户端使用什么地址访问etcd。

--listen-peer-urls	
#服务端节点之间通讯的监听地址，格式：'http://localhost:2380'

--initial-advertise-peer-urls	
#建议服务
```


2.2 启动
-----------

启动节点1
```sh
etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.10:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
```

启动节点2
```sh
etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.11:2380 \
  --listen-peer-urls http://10.0.1.11:2380 \
  --listen-client-urls http://10.0.1.11:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.11:2379 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
```

启动节点3
```sh
etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.12:2380 \
  --listen-peer-urls http://10.0.1.12:2380 \
  --listen-client-urls http://10.0.1.12:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.12:2379 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
```

提示：每台服务器启动的etcd实例，--initial-cluster 参数都是一样的，列出整个集群所有节点的服务端通讯地址。


2.3 开机启动
-------------
上面是直接在命令行启动etcd实例，关闭命令窗口，etcd就退出了，推荐使用进程管理软件，启动etcd，例如：centos系统，使用systemd启动etcd，具体如何配置网上找一下systemd的资料即可。


2.4 集群验证
-------------
```sh
etcdctl version

etcdctl --endpoints=localhost:2379 put foo bar
etcdctl --endpoints=localhost:2379 get foo
```






