


Hadoop不同版本
=======
1. Apache Hadoop
2. Cloudera Hadoop  (简称 CDH版本) 优先选择

Apache当前的版本管理是比较混乱的，各种版本层出不穷，让很多初学者不知所措，相比之下，Cloudera公司的Hadoop版本管理的要很多。

我们知道，Hadoop遵从Apache开源协议，用户可以免费地任意使用和修改Hadoop，也正因此，市面上出现了很多Hadoop版本，其中 比较出名的一是Cloudera公司的发行版，我们将该版本称为CDH（Cloudera Distribution Hadoop）。截至目前为止，CDH共有4个版本，其中，前两个已经不再更新，最近的两个，分别是CDH3（在Apache Hadoop 0.20.2版本基础上演化而来的）和CDH4在Apache Hadoop 2.0.0版本基础上演化而来的），分别对应Apache的Hadoop 1.0和Hadoop 2.0，它们每隔一段时间便会更新一次。

Cloudera以patch level划分小版本，比如patch level为923.142表示在原生态Apache Hadoop 0.20.2基础上添加了1065个patch（这些patch是各个公司或者个人贡献的，在Hadoop jira上均有记录），其中923个是最后一个beta版本添加的patch，而142个是稳定版发行后新添加的patch。由此可见，patch level越高，功能越完备且解决的bug越多。

Cloudera版本层次更加清晰，且它提供了适用于各种操作系统的Hadoop安装包，可直接使用apt-get或者yum命令进行安装，更加省事。



Hadoop单机伪集群环境搭建 安装步骤
=====
```sh
Hadoop单机版环境准备
一、前置条件 jdk安装

二、配置免密登录
    2.1 配置映射
    2.2 生成公私钥
    3.3 授权

三、Hadoop(HDFS)环境搭建
    3.1 下载并解压
    3.2 配置环境变量
    3.3 修改Hadoop配置
        1. hadoop-env.sh
        2. core-site.xml
        3. hdfs-site.xml
        4. slaves
    3.4 关闭防火墙
    3.5 初始化
    3.6 启动HDFS
    3.7 验证是否启动成功

四、Hadoop(YARN)环境搭建
    4.1 修改配置
        1. yarn-site.xml
        2. mapred-site.xml
    4.2 启动服务
    4.3 验证是否启动成功
```


环境准备
======
一、前置条件  
Hadoop 的运行依赖 JDK，需要预先安装  

jdk版本要求：  
hadoop版本>=2.7：要求>=Java 7(openjdk/oracle)  
hadoop版本<=2.6：要求Java 6(openjdk/oracle)  

所以目前选jdk1.8即可  


二、配置免密登录  
Hadoop 组件之间需要基于 SSH 进行通讯。

2.1 配置映射  
------
配置 ip 地址和主机名映射：  
> vim /etc/hosts  
```sh
# 文件末尾增加
192.168.43.202  hadoop001
```


2.2 生成公私钥
------
执行下面命令行生成公匙和私匙：
> ssh-keygen -t rsa


2.3 授权
------
```sh
进入 ~/.ssh 目录下，查看生成的公匙和私匙，并将公匙写入到授权文件：

[root@@hadoop001 sbin]#  cd ~/.ssh
[root@@hadoop001 .ssh]# ll
-rw-------. 1 root root 1675 3 月  15 09:48 id_rsa
-rw-r--r--. 1 root root  388 3 月  15 09:48 id_rsa.pub

# 写入公匙到授权文件
[root@hadoop001 .ssh]# cat id_rsa.pub >> authorized_keys
[root@hadoop001 .ssh]# chmod 600 authorized_keys
```


安装hadoop
======

3.1 下载并解压
------
下载 Hadoop 安装包，这里我下载的是 CDH 版本的，下载地址为：http://archive.cloudera.com/cdh5/cdh/5/

首先找到下载的地址，然后再后面加上.tar.gz就可以下载了
CDH的下载地址 http://archive.cloudera.com/cdh5/ 上，现在是cdh5的版本。

下载方式
比如http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.9.3 
改成http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.9.3.tar.gz

再比如 hadoop-2.6.0-cdh5.15.2，下载地址为
http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.15.2.tar.gz

解压  
> tar -zvxf hadoop-2.6.0-cdh5.15.2.tar.gz -c /user/local/

```sh
# 创建hadoop用户，注意：现在创建用户时直接就创建了同名的用户组。所以可以不用单独执行groupadd xx
useradd hadoop   

# 设置密码
passwd hadoop

查看某用户所属的用户组
groups usernamexxx

如果没有组，则
# 创建hadoop用户组
groupadd hadoop

# 添加用户到用户组
usermod -G hadoop hadoop
```

> chown -R hadoop:hadoop /user/local/hadoop-2.6.0-cdh5.15.2/
> ln -s /usr/local/hadoop-2.6.0-cdh5.15.2/ /usr/local/hadoop



3.2 配置环境变量
------
> vi /etc/profile
配置环境变量：
```sh
export HADOOP_HOME=/usr/app/hadoop-2.6.0-cdh5.15.2
export  PATH=${HADOOP_HOME}/bin:$PATH
```
执行 source 命令，使得配置的环境变量立即生效：
> source /etc/profile



3.3 修改Hadoop配置
------
进入 ${HADOOP_HOME}/etc/hadoop/ 目录下，修改以下配置：

1. hadoop-env.sh
```sh
# JDK安装路径
export  JAVA_HOME=/usr/java/jdk1.8.0_201/
```

2. core-site.xml
```xml
<configuration>
    <property>
        <!--指定 namenode 的 hdfs 协议文件系统的通信地址-->
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop001:8020</value>
    </property>
    <property>
        <!--指定 hadoop 存储临时文件的目录-->
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/tmp</value>
    </property>
</configuration>
```

3. hdfs-site.xml

指定副本系数和临时文件存储位置：
```xml
<configuration>
    <property>
        <!--由于我们这里搭建是单机版本，所以指定 dfs 的副本系数为 1-->
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

4. slaves

配置所有从属节点的主机名或 IP 地址，由于是单机版本，所以指定本机即可：
> vim slaves
```sh
hadoop001
```


3.4 关闭防火墙
---------
不关闭防火墙可能导致无法访问 Hadoop 的 Web UI 界面：
```sh
# 查看防火墙状态
sudo firewall-cmd --state

# 关闭防火墙:
sudo systemctl stop firewalld.service
```

3.5 初始化
---------
第一次启动 Hadoop 时需要进行初始化，进入 ${HADOOP_HOME}/bin/ 目录下，执行以下命令：

> sudo su - hadoop
> cd ${HADOOP_HOME}/bin/
> ./hdfs namenode -format


3.6 启动HDFS
---------
进入 ${HADOOP_HOME}/sbin/ 目录下，启动 HDFS：

> sudo su - hadoop
> cd ${HADOOP_HOME}/sbin/
> ./start-dfs.sh
```sh
[hadoop@localhost sbin]$ ./start-dfs.sh
Starting namenodes on [hadoop001]
hadoop001: starting namenode, logging to /usr/local/hadoop-2.6.0-cdh5.15.2/logs/hadoop-hadoop-namenode-localhost.localdomain.out
hadoop001: starting datanode, logging to /usr/local/hadoop-2.6.0-cdh5.15.2/logs/hadoop-hadoop-datanode-localhost.localdomain.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.6.0-cdh5.15.2/logs/hadoop-hadoop-secondarynamenode-localhost.localdomain.out
[hadoop@localhost sbin]$
#如果提示输入密码，则免密登录没有配好，重新配置免密
```


3.7 验证是否启动成功
---------
方式一：执行 jps 查看 NameNode 和 DataNode 服务是否已经启动：
```sh
[hadoop@localhost sbin]$ jps
3221 NameNode
3609 Jps
3499 SecondaryNameNode
3309 DataNode
```

方式二：查看 Web UI 界面，端口为 50070：  
网页访问  http://机器ip:50070




四、Hadoop(YARN)环境搭建
=========

4.1 修改配置
---------
进入 ${HADOOP_HOME}/etc/hadoop/ 目录下，修改以下配置：

1. yarn-site.xml
```xml
<configuration>
    <property>
        <!--配置 NodeManager 上运行的附属服务。需要配置成 mapreduce_shuffle 后才可以在 Yarn 上运行 MapReduce 程序。-->
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

2. mapred-site.xml
```xml
<configuration>
    <property>
        <!--使用YARN来进行运行时候的资源调度-->
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
如果没有mapred-site.xml，则拷贝一份样例文件后再修改  
> cp mapred-site.xml.template mapred-site.xml




4.2 启动服务  
------
进入 ${HADOOP_HOME}/sbin/ 目录下，启动 YARN：

> sudo su - hadoop
> cd ${HADOOP_HOME}/sbin/
> ./start-yarn.sh
```sh
[hadoop@localhost sbin]$ ./start-yarn.sh 
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop-2.6.0-cdh5.15.2/logs/yarn-hadoop-resourcemanager-localhost.localdomain.out
hadoop001: starting nodemanager, logging to /usr/local/hadoop-2.6.0-cdh5.15.2/logs/yarn-hadoop-nodemanager-localhost.localdomain.out
[hadoop@localhost sbin]$
```

4.3 验证是否启动成功
------
方式一：执行 jps 命令查看 NodeManager 和 ResourceManager 服务是否已经启动：
```sh
[hadoop@localhost sbin]$ jps
3221 NameNode
3499 SecondaryNameNode
3819 NodeManager
3309 DataNode
3725 ResourceManager
3885 Jps
[hadoop@localhost sbin]$
```

方式二：查看 Web UI 界面，端口号为 8088：

网页访问  http://机器ip:8088/





stop hdfs
```sh
[hadoop@localhost sbin]$ ./stop-dfs.sh 
Stopping namenodes on [hadoop001]
hadoop001: stopping namenode
hadoop001: stopping datanode
Stopping secondary namenodes [0.0.0.0]
0.0.0.0: stopping secondarynamenode
[hadoop@localhost sbin]$
```


stop yarn
```sh
[hadoop@localhost sbin]$ ./stop-yarn.sh 
stopping yarn daemons
stopping resourcemanager
hadoop001: stopping nodemanager
no proxyserver to stop
[hadoop@localhost sbin]$ 
```







问题 
============
Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
------------
原因：  
Apache提供的hadoop本地库是32位的，而在64位的服务器上就会有问题，因此需要自己编译64位的版本。   

1 下载对应的64位lib包   
方式1（不推荐）   首先找到对应自己hadoop版本的64位的lib包，可以自己手动去编译，但比较麻烦，也可以去网上找，好多都有已经编译好了的。

方式2（推荐）  
可以去网站：http://dl.bintray.com/sequenceiq/sequenceiq-bin/ 下载对应的编译版本

Hadoop 2.6.x 版本用hadoop-native-64-2.6.0.tar即可  
Hadoop 2.7.x 版本用hadoop-native-64-2.7.0.tar即可  


2 将下载好的64位的lib包解压到已经安装好的hadoop安装目录lib/native下  
> mkdir tmp_native
> tar -xvf hadoop-native-64-2.6.0.tar -C tmp_native   
里面是很多.so文件，把它们都放入 HADOOP_HOME/lib/native/ 目录下   
> cp tmp_native/* ${HADOOP_HOME}/lib/native/


3 然后增加环境变量：  
> sudo vi /etc/profile 
```sh
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native  
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"  
```

5 让环境变量生效
> source /etc/profile  

6 自检 指令检查
> hadoop checknative –a



no namenode to stop
----------------
问题描述：
虚拟机环境下，使用stop-yarn.sh和stop-dfs.sh停止yarn和hdfs时出现no resourcemanager to stop、no nodemanager to stop、no namenode to stop、no datanode to stop，但是相关进程都真实存在，并且可用

出现这个问题的原因：  
当初启动的时候没有指定pid的存放位置，hadoop(hbase也是这样)默认会放在Linux的/tmp目录下，进程名命名规则一般是框架名-用户名-角色名.pid,而默认情况下tmp里面的东西，一天会删除一次，由于pid不存在，当执行stop相关命令的时候找不到pid也就无法停止相关进程，所以报no xxx to stop

解决方式：    
当然就是手动指定pid的存放位置，避免放在/tmp目录下，在配置文件中指定其它位置。

修改hadoop-env.sh，如果没有相关配置，可用直接添加
```sh
export HADOOP_PID_DIR=/home/hadoop/pidDir
export HADOOP_SECURE_DN_PID_DIR=/home/hadoop/pidDir

#上述配置，影响
#　　NameNode
#　　DataNode
#　　SecondaryNameNode
#进程pid存储
```

修改mapred-env.sh
```sh
export HADOOP_MAPRED_PID_DIR=/home/hadoop/pidDir

#上述配置，影响
#　　JobHistoryServer
#进程pid存储
```

修改yarn-env.sh
```sh
export YARN_PID_DIR=/home/hadoop/pidDir

#上述配置，影响
#　　NodeManager
#　　ResourceManager
#进程pid存储
```

以上配置好后，启动yarn和hdfs，启动成功后首先jps查看，ok，5个进程都在，然后cd /home/hadoop/pidDir目录下，有如下文件，完美
```sh
-rw-rw-r-- 1 hadoop hadoop 6 Mar 2 17:13 hadoop-hadoop-datanode.pid
-rw-rw-r-- 1 hadoop hadoop 6 Mar 2 17:13 hadoop-hadoop-namenode.pid
-rw-rw-r-- 1 hadoop hadoop 6 Mar 2 17:13 hadoop-hadoop-secondarynamenode.pid
-rw-rw-r-- 1 hadoop hadoop 6 Mar 2 17:13 yarn-hadoop-nodemanager.pid
-rw-rw-r-- 1 hadoop hadoop 6 Mar 2 17:13 yarn-hadoop-resourcemanager.pid
```





hadoop 更多配置说明  
----------
http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/ClusterSetup.html  








