

Zookeeper单机环境和集群环境搭建
===========
```
一、单机环境搭建
1.1 下载
1.2 解压
1.3 配置环境变量
1.4 修改配置
1.5 启动
1.6 验证

二、集群环境搭建
2.1 修改配置
2.2 标识节点
2.3 启动集群
2.4 集群验证


三、单机集群环境搭建
```


一、单机环境搭建
===========
1.1 下载
-----------
下载对应版本 Zookeeper，这里我下载的版本 3.4.14。  
官方下载地址：https://archive.apache.org/dist/zookeeper/  
> wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz


1.2 解压
-----------
解压到指定目录下  
> tar -zxvf zookeeper-3.4.14.tar.gz -C /usr/local/

这里假设安装在 /usr/local/目录下，当然也可以自已起个目录，比如/usr/app/

建个软链，方便访问  
> ln -s /usr/local/zookeeper-3.4.14 /usr/local/zookeeper


1.3 配置环境变量
-----------
> vim /etc/profile

添加环境变量：
```
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.14
export PATH=$ZOOKEEPER_HOME/bin:$PATH
```

使得配置的环境变量生效： 
> source /etc/profile


1.4 修改配置
-----------
创建相应的目录 (数据文件目录，日志文件目录)  
> cd /usr/local/zookeeper
> ls
> mkdir data
> mkdir log

进入安装目录的 conf/ 目录下，拷贝配置样本并进行修改
> ll
> cp zoo_sample.cfg  zoo.cfg

指定数据存储目录和日志文件目录（目录不用预先创建，程序会自动创建），修改后完整配置如下：
```sh
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper/data/
dataLogDir=/usr/local/zookeeper/log/
clientPort=2181

# server.N=YYY:A:B  这个N是服务器的标识，可以是任意有效数字，标识这是第几个服务器节点，这个标识要写到dataDir目录下面myid文件里
# 指名集群间通讯端口和选举端口
#server.1=hadoop001:2287:3387
#server.2=hadoop002:2287:3387
#server.3=hadoop003:2287:3387
```

配置参数说明：
```sh
tickTime：用于计算的基础时间单元。比如 session 超时：N*tickTime；
initLimit：用于集群，允许从节点连接并同步到 master 节点的初始化连接时间，以 tickTime 的倍数来表示；
syncLimit：用于集群， master 主节点与从节点之间发送消息，请求和应答时间长度（心跳机制）；
dataDir：数据存储位置；
dataLogDir：日志目录；
clientPort：用于客户端连接的端口，默认 2181
```

配置更多说明：
```sh
# The number of milliseconds of each tick
# tickTime：CS通信心跳数
# Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。
tickTime=2000

# The number of ticks that the initial
# synchronization phase can take
# initLimit：LF初始通信时限
# 集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。
initLimit=10

# The number of ticks that can pass between
# sending a request and getting an acknowledgement
# syncLimit：LF同步通信时限
# 集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。
syncLimit=5

# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
# dataDir：数据文件目录
# Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。
dataDir=/usr/local/zookeeper/data

# dataLogDir：日志文件目录
# Zookeeper保存日志文件的目录。
dataLogDir=/usr/local/zookeeper/log


# the port at which the clients will connect
# clientPort：客户端连接端口
# 客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
clientPort=2181

# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60

#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

# 服务器名称与地址：集群信息（服务器编号，服务器地址，LF通信端口，选举端口）
# 这个配置项的书写格式比较特殊，规则如下：

# server.N=YYY:A:B  

# 其中N表示服务器编号，YYY表示服务器的IP地址，A为LF通信端口，表示该服务器与集群中的leader交换的信息的端口。B为选举端口，表示选举新leader时服务器间相互通信的端口（当leader挂掉时，其余服务器会相互通信，选择出新的leader）。一般来说，集群中每个服务器的A端口都是一样，每个服务器的B端口也是一样。但是当所采用的为伪集群时，IP地址都一样，只能时A端口和B端口不一样。
```



1.5 启动
-----------
由于已经配置过环境变量，直接使用下面命令启动即可：
> cd $ZOOKEEPER_HOME
> ./bin/zkServer.sh start

其它
```sh
停止命令：
./bin/zkServer.sh stop　　

重启命令：
./bin/zkServer.sh restart

状态查看命令：（同时可看到当前节点是leader还是follower）
./bin/zkServer.sh status
```


1.6 验证
-----------
使用 JPS 验证进程是否已经启动，出现 QuorumPeerMain 则代表启动成功。
```sh
[root@hadoop001 bin]# jps
3814 QuorumPeerMain
```



二、集群环境搭建
==========
为保证集群高可用，Zookeeper集群的节点数最好是奇数，最少有三个节点，所以这里演示搭建一个三个节点的集群。这里我使用三台主机进行搭建，主机名分别为 hadoop001，hadoop002，hadoop003。


2.1 修改配置
-----------
解压一份 zookeeper 安装包，修改其配置文件 zoo.cfg，内容如下。之后使用 scp 命令将安装包分发到三台服务器上：

这里假设zookeeper的安装目录为/usr/local/zookeeper-cluster
> cd /usr/local/zookeeper-cluster
> ll conf/
> vim conf/zoo.cfg
```sh
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper-cluster/data/
dataLogDir=/usr/local/zookeeper-cluster/log/
clientPort=2181

# server.N=YYY:A:B  这个N是服务器的标识，可以是任意有效数字，标识这是第几个服务器节点，这个标识要写到dataDir目录下面myid文件里
# 指名集群间通讯端口和选举端口
server.1=hadoop001:2287:3387
server.2=hadoop002:2287:3387
server.3=hadoop003:2287:3387
```

scp 命令将安装包发到其它的几台服务器上
> scp -r /usr/local/zookeeper-cluster/ xxuser@xxx机器:/usr/local/


2.2 标识节点
-----------
分别在三台主机的 dataDir 目录下新建 myid 文件,并写入对应的节点标识。Zookeeper 集群通过 myid 文件识别集群节点，并通过上文配置的节点通信端口和选举端口来进行节点通信，选举出 Leader 节点。

创建存储目录：

# 三台主机均执行该命令
> mkdir -p  /usr/local/zookeeper-cluster/data
> mkdir -p  /usr/local/zookeeper-cluster/log

创建并写入节点标识到 myid 文件：
```sh
# hadoop001主机
echo "1" > /usr/local/zookeeper-cluster/data/myid
# hadoop002主机
echo "2" > /usr/local/zookeeper-cluster/data/myid
# hadoop003主机
echo "3" > /usr/local/zookeeper-cluster/data/myid
```

2.3 启动集群
-----------
分别在三台主机上，执行如下命令启动服务：
> /usr/app/zookeeper-cluster/zookeeper/bin/zkServer.sh start


2.4 集群验证
-----------
启动后使用 zkServer.sh status 查看集群各个节点状态。 

通过上面的status命令， 可以看到哪个节点是 leader 节点， 哪些节点是 follower 节点。




三、单机集群环境搭建(伪集群)
==========
同集群环境搭建搭建类似，只不过是在一个机器上装多个zookeeper，然后通过端口区分不同的zookeeper。

注意：此种是因为没有过多的机器，仅仅一台机器上测试用用而已，真实的生产环境不会这样。  


集群配置 与 单机配置 的区别：

配置文件中加上serever.x=ip:端口，同时需要在准备各自的myid文件。


1、在同一台主机上，通过复制得到三个zookeeper实例
----------
```sh
cd /usr/app
ll
mkdir zookeeper-cluster
cd zookeeper-cluster

# 在下面分别准备多个zookeeper 
zookeeper-3.4.12-12181
zookeeper-3.4.12-12182
zookeeper-3.4.12-12183
```
　　　　

2、对三个zookeeper节点进行配置  
----------
修改dataDir，添加server.0、server.1、server.2  
> mkdir data
> mkdir logs


zookeeper1配置文件conf/zoo.cfg修改如下：
```sh
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/data/soft/zookeeper-cluster/zookeeper-3.4.12-12181/data
dataLogDir=/data/soft/zookeeper-cluster/zookeeper-3.4.12-12181/logs
clientPort=12181

server.1=127.0.0.1:12888:13888
server.2=127.0.0.1:14888:15888
server.3=127.0.0.1:16888:17888
```
　注：server.1中的数字1为服务器的ID，需要与myid文件中的id一致，下一步将配置myid

zookeeper1的data/myid配置，使用如下命令（即新建一个文件data/myid，在其中添加内容为：1）：
```
echo '1' > data/myid
```


zookeeper2配置文件conf/zoo.cfg修改如下：
```sh
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/data/soft/zookeeper-cluster/zookeeper-3.4.12-12182/data
dataLogDir=/data/soft/zookeeper-cluster/zookeeper-3.4.12-12182/logs
clientPort=12182

server.1=127.0.0.1:12888:13888
server.2=127.0.0.1:14888:15888
server.3=127.0.0.1:16888:17888
```
zookeeper2的data/myid配置，使用如下命令：
```
echo '2' > data/myid
```


zookeeper3配置文件conf/zoo.cfg修改如下：
```sh
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/data/soft/zookeeper-cluster/zookeeper-3.4.12-12183/data
dataLogDir=/data/soft/zookeeper-cluster/zookeeper-3.4.12-12183/logs
clientPort=12183

server.1=127.0.0.1:12888:13888
server.2=127.0.0.1:14888:15888
server.3=127.0.0.1:16888:17888
```

zookeeper3的data/myid配置，使用如下命令：
```
echo '3' > data/myid
```


3、分别启动三个zookeeper节点
----------
> ./zookeeper-3.4.12-12181/bin/zkServer.sh start
> ./zookeeper-3.4.12-12182/bin/zkServer.sh start
> ./zookeeper-3.4.12-12183/bin/zkServer.sh start


4、查看节点状态
----------
> ./zookeeper-3.4.12-12181/bin/zkServer.sh status
> ./zookeeper-3.4.12-12182/bin/zkServer.sh status
> ./zookeeper-3.4.12-12183/bin/zkServer.sh status














