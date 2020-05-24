

Flume 简介及基本使用
===========
```sh
一、Flume简介

二、Flume架构和基本概念
	2.1 基本架构
	2.2 基本概念
	2.3 组件种类

三、Flume架构模式
	3.1 multi-agent flow
	3.2 Consolidation
	3.3 Multiplexing the flow

四、Flume配置格式
```


一、Flume简介
===========
Apache Flume是一个分布式，高可用的数据收集系统。它可以从不同的数据源收集数据，经过聚合后发送到存储系统中，通常用于日志数据的收集。

Flume 分为 NG 和 OG (1.0 之前) 两个版本，NG在OG的基础上进行了完全的重构，是目前使用最为广泛的版本。下面的介绍均以 NG 为基础。


二、Flume架构和基本概念
===========
```sh
数据 --->      Source ->  Channel -> Sink       --> hdfs
```

2.1 基本架构
-----------
外部数据源以特定格式向 Flume 发送 events (事件)，当 source 接收到 events 时，它将其存储到一个或多个 channel，channe 会一直保存 events 直到它被 sink 所消费。sink 的主要功能从 channel 中读取 events，并将其存入外部存储系统或转发到下一个 source，成功后再从 channel 中移除 events。


2.2 基本概念
-----------
1. Event  
Event 是 Flume NG 数据传输的基本单元。类似于 JMS 和消息系统中的消息。一个 Event 由标题和正文组成：前者是键/值映射，后者是任意字节数组。


2. Source  
数据收集组件，从外部数据源收集数据，并存储到 Channel 中。


3. Channel      
Channel 是源和接收器之间的管道，用于临时存储数据。可以是内存或持久化的文件系统：

- Memory Channel : 使用内存，优点是速度快，但数据可能会丢失 (如突然宕机)；  

- File Channel : 使用持久化的文件系统，优点是能保证数据不丢失，但是速度慢。


4. Sink  
Sink 的主要功能从 Channel 中读取 Event，并将其存入外部存储系统或将其转发到下一个 Source，成功后再从 Channel 中移除 Event。


5. Agent   
是一个独立的 (JVM) 进程，包含 Source、 Channel、 Sink 等组件。



2.3 组件种类
-----------
Flume 中的每一个组件都提供了丰富的类型，适用于不同场景：

- Source 类型 ：内置了几十种类型，如 Avro Source，Thrift Source，Kafka Source，JMS Source；

- Sink 类型 ：HDFS Sink，Hive Sink，HBaseSinks，Avro Sink 等；

- Channel 类型 ：Memory Channel，JDBC Channel，Kafka Channel，File Channel 等。

对于 Flume 的使用，除非有特别的需求，否则通过组合内置的各种类型的 Source，Sink 和 Channel 就能满足大多数的需求。

在 Flume 官网 上对所有类型组件的配置参数均以表格的方式做了详尽的介绍，并附有配置样例；同时不同版本的参数可能略有所不同，所以使用时建议选取官网对应版本的 User Guide 作为主要参考资料。



三、Flume架构模式
===========
Flume 支持多种架构模式，分别介绍如下

3.1 multi-agent flow
-----------
Flume 支持跨越多个 Agent 的数据传递，这要求前一个 Agent 的 Sink 和下一个 Agent 的 Source 都必须是 Avro 类型，Sink 指向 Source 所在主机名 (或 IP 地址) 和端口（详细配置见下文案例三）。


3.2 Consolidation
-----------
日志收集中常常存在大量的客户端（比如分布式 web 服务），Flume 支持使用多个 Agent 分别收集日志，然后通过一个或者多个 Agent 聚合后再存储到文件系统中。


3.3 Multiplexing the flow
-----------
Flume 支持从一个 Source 向多个 Channel，也就是向多个 Sink 传递事件，这个操作称之为 Fan Out(扇出)。默认情况下 Fan Out 是向所有的 Channel 复制 Event，即所有 Channel 收到的数据都是相同的。同时 Flume 也支持在 Source 上自定义一个复用选择器 (multiplexing selector) 来实现自定义的路由规则。


四、Flume配置格式
===========
Flume 配置通常需要以下两个步骤：

分别定义好 Agent 的 Sources，Sinks，Channels，然后将 Sources 和 Sinks 与通道进行绑定。

需要注意的是：一个 Source 可以配置多个 Channel，但一个 Sink 只能配置一个 Channel。

创建一个xxx.properties文件，然后在里里配置内容

基本格式如下：
```sh
<Agent>.sources = <Source>
<Agent>.sinks = <Sink>
<Agent>.channels = <Channel1> <Channel2>


# set channel for source
<Agent>.sources.<Source>.channels = <Channel1> <Channel2> ...

# set channel for sink
<Agent>.sinks.<Sink>.channel = <Channel1>
```	

分别定义 Source，Sink，Channel 的具体属性。基本格式如下：
```sh
# properties for sources
<Agent>.sources.<Source>.<someProperty> = <someValue>

# properties for channels
<Agent>.channel.<Channel>.<someProperty> = <someValue>

# properties for sinks
<Agent>.sources.<Sink>.<someProperty> = <someValue>
```

一个具体的例子：监听文件内容变动，将新增加的内容输出到控制台
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


更多内容，请看：02-Flume使用案例。






