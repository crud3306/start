

kafka安装与测试
============

1、配置JDK环境
------------
Kafka 使用Zookeeper 来保存相关配置信息，Kafka及Zookeeper 依赖Java 运行环境，从oracle网站下载JDK 安装包，解压安装  
下载地十：https://www.oracle.com/cn/java/technologies/javase/javase-jdk8-downloads.html

```sh
tar -zxvf jdk-8u171-linux-x64.tar.gz
mv jdk1.8.0_171 /usr/local/java/
```

设置Java 环境变量：
```sh
#java 
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
````


2、安装kafka
-------------
下载地址：http://kafka.apache.org/downloads

下载解压即可，无须编译安装，默认端口：9092
```sh
cd /opt
wget http://mirror.bit.edu.cn/apache/kafka/2.3.0/kafka_2.11-2.3.0.tgz
tar zxvf kafka_2.11-2.3.0.tgz
mv kafka_2.11-2.3.0 /usr/local/apps/

cd /usr/local/apps/
ln -s kafka_2.11-2.3.0 kafka
```


3、启动测试
--------------
需先启动zookeeper，再启动kafka。

（1）启动Zookeeper服务，可以用kafka中自带的zookeeper，也可单独安装，默认端口：2181
```sh
cd /usr/local/apps/kafka

#执行脚本
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

#查看进程
jps
```


（2）启动单机Kafka服务
```sh
#执行脚本
bin/kafka-server-start.sh config/server.properties
#查看进程
4 jps
```


（3）创建topic进行测试
```sh
#执行脚本
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```


（4）查看topic列表
```sh
#执行脚本
bin/kafka-topics.sh --list --zookeeper localhost:2181

#输出：
test
```


（5）生产者消息测试
```sh
#执行脚本(使用kafka-console-producer.sh 发送消息)
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```


（6）消费者消息测试
```sh
#执行脚本(使用kafka-console-consumer.sh 接收消息并在终端打印)
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```




4、单机多broker集群配置
-----------
单机部署多个broker，不同的broker，设置不同的id、监听端口、日志目录

```sh
cp config/server.properties config/server-1.properties 

vim server-1.properties
#修改：
broker.id=1
port=9093
log.dir=/tmp/kafka-logs-1


#启动Kafka服务
bin/kafka-server-start.sh config/server-1.properties &
```





独立安装zookeeper服务
================
如果不用kafka自带的zookeeper，需单独安装

下载ZooKeeper
要在您的计算机上安装ZooKeeper框架，请访问以下链接并下载最新版本的ZooKeeper。
http://zookeeper.apache.org/releases.html

```sh
cd /usr/local/src
wget https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
tar -zxf zookeeper-3.6.2.tar.gz
cd zookeeper-3.6.2
mkdir data
```

创建配置文件
```sh
vi conf/zoo.cfg
tickTime=2000
dataDir=/usr/local/zookeeper-3.6.2/data
clientPort=2181
initLimit=5
syncLimit=2
```
或者
```sh
cp conf/zoo_sample.cfg conf/zoo.cfg

#更改dataDir，或默认
dataDir=/usr/local/zookeeper-3.6.2/data
```


启动ZooKeeper服务器
```sh
./bin/zkServer.sh start
```

停止Zookeeper服务器
```sh
./bin/zkServer.sh stop
```
