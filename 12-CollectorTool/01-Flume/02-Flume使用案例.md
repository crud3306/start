


Flume使用案例
=========
```sh
1 案例一：监听文件内容变动，将新增加的内容输出到控制台。
	1.1 配置
	1.2 启动
	1.3 测试

2 案例二：监听指定目录，将目录下新增加的文件存储到HDFS
	2.1 配置
	2.2 启动
	2.3 测试

3 案例三：使用Avro将本服务器收集到的日志数据发送到另外一台服务器。
	3.1 配置日志收集Flume
	3.2 配置日志聚合Flume
	3.3 启动
	3.4 测试
```



1 案例一
=========
需求： 监听文件内容变动，将新增加的内容输出到控制台。

实现： 主要使用 Exec Source 配合 tail 命令实现。

1. 配置
--------
新建配置文件 exec-memory-logger.properties,其内容如下：
```sh
#指定agent的sources,sinks,channels
a1.sources = s1  
a1.sinks = k1  
a1.channels = c1  

#配置sources属性
a1.sources.s1.type = exec
a1.sources.s1.command = tail -F /tmp/log.txt
a1.sources.s1.shell = /bin/bash -c
#将sources与channels进行绑定
a1.sources.s1.channels = c1

#配置sink 
a1.sinks.k1.type = logger
#将sinks与channels进行绑定  
a1.sinks.k1.channel = c1  

#配置channel类型
a1.channels.c1.type = memory
```

2. 启动　
--------
```sh
flume-ng agent \
--conf conf \
--conf-file /usr/app/apache-flume-1.6.0-cdh5.15.2-bin/examples/exec-memory-logger.properties \
--name a1 \
-Dflume.root.logger=INFO,console
```

3. 测试
--------
向文件中追加数据：

控制台的显示：



2 案例二
============
需求： 监听指定目录，将目录下新增加的文件存储到HDFS。

实现：使用 Spooling Directory Source 和 HDFS Sink。

1. 配置
--------
```sh
#指定agent的sources,sinks,channels
a1.sources = s1  
a1.sinks = k1  
a1.channels = c1  

#配置sources属性
a1.sources.s1.type =spooldir  
a1.sources.s1.spoolDir =/tmp/logs
a1.sources.s1.basenameHeader = true
a1.sources.s1.basenameHeaderKey = fileName 
#将sources与channels进行绑定  
a1.sources.s1.channels =c1 

#配置sink 
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H/
a1.sinks.k1.hdfs.filePrefix = %{fileName}
#生成的文件类型，默认是Sequencefile，可用DataStream，则为普通文本
a1.sinks.k1.hdfs.fileType = DataStream  
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#将sinks与channels进行绑定  
a1.sinks.k1.channel = c1

#配置channel类型
a1.channels.c1.type = memory
```


2. 启动
--------
```sh
flume-ng agent \
--conf conf \
--conf-file /usr/app/apache-flume-1.6.0-cdh5.15.2-bin/examples/spooling-memory-hdfs.properties \
--name a1 -Dflume.root.logger=INFO,console
```

3. 测试
--------
拷贝任意文件到监听目录下，可以从日志看到文件上传到 HDFS 的路径：
```sh
cp log.txt logs/
```


查看上传到 HDFS 上的文件内容与本地是否一致：
```sh
hdfs dfs -cat /flume/events/19-04-09/13/log.txt.1554788567801
```


3 案例三
=============
需求： 将本服务器收集到的数据发送到另外一台服务器。

实现：使用 avro sources 和 avro Sink 实现。

1. 配置日志收集Flume
--------
新建配置 netcat-memory-avro.properties，监听文件内容变化，然后将新的文件内容通过 avro sink 发送到 hadoop001 这台服务器的 8888 端口：
```sh
#指定agent的sources,sinks,channels
a1.sources = s1
a1.sinks = k1
a1.channels = c1

#配置sources属性
a1.sources.s1.type = exec
a1.sources.s1.command = tail -F /tmp/log.txt
a1.sources.s1.shell = /bin/bash -c
a1.sources.s1.channels = c1

#配置sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop001
a1.sinks.k1.port = 8888
a1.sinks.k1.batch-size = 1
a1.sinks.k1.channel = c1

#配置channel类型
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
```


2. 配置日志聚合Flume
--------
使用 avro source 监听 hadoop001 服务器的 8888 端口，将获取到内容输出到控制台：
```sh
#指定agent的sources,sinks,channels
a2.sources = s2
a2.sinks = k2
a2.channels = c2

#配置sources属性
a2.sources.s2.type = avro
a2.sources.s2.bind = hadoop001
a2.sources.s2.port = 8888
#将sources与channels进行绑定
a2.sources.s2.channels = c2

#配置sink
a2.sinks.k2.type = logger
#将sinks与channels进行绑定
a2.sinks.k2.channel = c2

#配置channel类型
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100
```

3. 启动
--------
启动日志聚集 Flume：
```sh
flume-ng agent \
--conf conf \
--conf-file /usr/app/apache-flume-1.6.0-cdh5.15.2-bin/examples/avro-memory-logger.properties \
--name a2 -Dflume.root.logger=INFO,console
```

在启动日志收集 Flume:
```sh
flume-ng agent \
--conf conf \
--conf-file /usr/app/apache-flume-1.6.0-cdh5.15.2-bin/examples/netcat-memory-avro.properties \
--name a1 -Dflume.root.logger=INFO,console
```

这里建议按以上顺序启动，原因是 avro.source 会先与端口进行绑定，这样 avro sink 连接时才不会报无法连接的异常。但是即使不按顺序启动也是没关系的，sink 会一直重试，直至建立好连接。


4. 测试
--------
向文件 tmp/log.txt 中追加内容：

可以看到已经从 8888 端口监听到内容，并成功输出到控制台：





