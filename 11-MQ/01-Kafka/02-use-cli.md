

Kafka命令行常用的几个命令如下：
============
- kafka-server-start.sh
- kafka-topics.sh
- kafka-console-consumer.sh
- kafka-console-producer.sh


在这几个命令中，  
kafka-server-start.sh仅用于启动Kafka;  
kafka-topics.sh用的最多;  
两个kafka-console-xxx.sh常用于测试生产消息与消费消息；  



kafka-server-start.sh
------------
用法：  
> bin/kafka-server-start.sh [-daemon] server.properties [--override property=value]*

这个命令后面可以有多个参数，第一个是可选参数，该参数可以让当前命令以后台服务方式执行，第二个必须是 Kafka 的配置文件。后面还可以有多个--override开头的参数，其中的property可以是Broker Configs中提供的所有参数。这些额外的参数会覆盖配置文件中的设置。

例如下面使用同一个配置文件，通过参数覆盖启动多个Broker。
```sh
#最简单用法
bin/kafka-server-start.sh -daemon config/server.properties

#带上--override参娄
bin/kafka-server-start.sh -daemon config/server.properties --override broker.id=0 --override log.dirs=/tmp/kafka-logs-1 --override listeners=PLAINTEXT://:9092 --override advertised.listeners=PLAINTEXT://192.168.16.150:9092

bin/kafka-server-start.sh -daemon config/server.properties --override broker.id=1 --override log.dirs=/tmp/kafka-logs-2 --override listeners=PLAINTEXT://:9093 --override advertised.listeners=PLAINTEXT://192.168.16.150:9093
```
上面这种用法只是用于演示，真正要启动多个Broker 应该针对不同的 Broker 创建相应的 server.properties 配置。





kafka-topics.sh
-----------
kafka-topics.sh 相对来说最常用。

下面是几种常用的 topic 命令。

```sh
#用kafka-topics.sh命令时，服务地址用下面两个均可，但是推荐用--zookeeper

--bootstrap-server hadoop001:9092

--zookeeper localhost:2181
```

列出主题
> bin/kafka-topics.sh --list --zookeeper localhost:2181

创建主题
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic test_topic

主题信息  
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test_topic

添加分区
> bin/kafka-topics.sh --alter --zookeeper localhost:2181 --topic test_topic --partitions 3

删除主题  
> bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test_topic

注意：需要在Broker的配置文件server.properties中配置 delete.topic.enable=true 才能删除主题。


描述主题的配置
> bin/kafka-configs.sh --describe --zookeeper localhost:2181 --entity-type topics --entity-name test_topic


设置保留时间
```sh
#Deprecated way
bin/kafka-topics.sh  --zookeeper localhost:2181 --alter --topic test_topic --config retention.ms=1000

#Modern way
bin/kafka-configs.sh --zookeeper localhost:2181 --alter --entity-type topics --entity-name test_topic --add-config retention.ms=1000
```

如果您需要删除主题中的所有消息，则可以利用保留时间。首先将保留时间设置为非常低（1000ms），等待几秒钟，然后将保留时间恢复为上一个值。

注意：默认保留时间为24小时（86400000毫秒）。



该命令包含以下参数。
```sh
Create, delete, describe, or change a topic.
Option                                   Description                            
------                                   -----------                            
--alter                                  Alter the number of partitions,        
                                           replica assignment, and/or           
                                           configuration for the topic.         
--config <String: name=value>            A topic configuration override for the 
                                           topic being created or altered.The   
                                           following is a list of valid         
                                           configurations:                      
                                            cleanup.policy                        
                                            compression.type                      
                                            delete.retention.ms                   
                                            file.delete.delay.ms                  
                                            flush.messages                        
                                            flush.ms                              
                                            follower.replication.throttled.       
                                           replicas                             
                                            index.interval.bytes                  
                                            leader.replication.throttled.replicas 
                                            max.message.bytes                     
                                            message.format.version                
                                            message.timestamp.difference.max.ms   
                                            message.timestamp.type                
                                            min.cleanable.dirty.ratio             
                                            min.compaction.lag.ms                 
                                            min.insync.replicas                   
                                            preallocate                           
                                            retention.bytes                       
                                            retention.ms                          
                                            segment.bytes                         
                                            segment.index.bytes                   
                                            segment.jitter.ms                     
                                            segment.ms                            
                                            unclean.leader.election.enable        
                                         See the Kafka documentation for full   
                                           details on the topic configs.        
--create                                 Create a new topic.                    
--delete                                 Delete a topic                         
--delete-config <String: name>           A topic configuration override to be   
                                           removed for an existing topic (see   
                                           the list of configurations under the 
                                           --config option).                    
--describe                               List details for the given topics.     
--disable-rack-aware                     Disable rack aware replica assignment  
--force                                  Suppress console prompts               
--help                                   Print usage information.               
--if-exists                              if set when altering or deleting       
                                           topics, the action will only execute 
                                           if the topic exists                  
--if-not-exists                          if set when creating topics, the       
                                           action will only execute if the      
                                           topic does not already exist         
--list                                   List all available topics.             
--partitions <Integer: # of partitions>  正在创建或更改主题的分区数
                                         （警告：如果为具有密钥的主题   
                                         （分区）增加了分区  
                                          消息的逻辑或排序将受到影响                    
--replica-assignment <String:            A list of manual partition-to-broker   
  broker_id_for_part1_replica1 :           assignments for the topic being      
  broker_id_for_part1_replica2 ,           created or altered.                  
  broker_id_for_part2_replica1 :                                                
  broker_id_for_part2_replica2 , ...>                                           
--replication-factor <Integer:           正在创建的主题中每个分区的复制因子。        
  replication factor>                    
--topic <String: topic>                  The topic to be create, alter or       
                                           describe. Can also accept a regular  
                                           expression except for --create option
--topics-with-overrides                  if set when describing topics, only    
                                           show topics that have overridden     
                                           configs                              
--unavailable-partitions                 if set when describing topics, only    
                                           show partitions whose leader is not  
                                           available                            
--under-replicated-partitions            if set when describing topics, only    
                                           show under replicated partitions     
--zookeeper <String: urls>               REQUIRED: The connection string for    
                                           the zookeeper connection in the form 
                                           host:port. Multiple URLS can be      
                                           given to allow fail-over.         
```





kafka-console-producer.sh
------------
这个命令可以将文件或标准输入的内容发送到Kafka集群。

常用命令如下。

使用标准输入方式。
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

从文件读取：
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test < file-input.txt

--broker-list 和 --topic 是两个必须提供的参数。


该命令更多参数如下
```sh
Option                                   Description                            
------                                   -----------                            
--batch-size <Integer: size>             Number of messages to send in a single 
                                           batch if they are not being sent     
                                           synchronously. (default: 200)        
--broker-list <String: broker-list>      REQUIRED: The broker list string in    
                                           the form HOST1:PORT1,HOST2:PORT2.    
--compression-codec [String:             The compression codec: either 'none',  
  compression-codec]                       'gzip', 'snappy', or 'lz4'.If        
                                           specified without value, then it     
                                           defaults to 'gzip'                   
--key-serializer <String:                The class name of the message encoder  
  encoder_class>                           implementation to use for            
                                           serializing keys. (default: kafka.   
                                           serializer.DefaultEncoder)           
--line-reader <String: reader_class>     The class name of the class to use for 
                                           reading lines from standard in. By   
                                           default each line is read as a       
                                           separate message. (default: kafka.   
                                           tools.                               
                                           ConsoleProducer$LineMessageReader)   
--max-block-ms <Long: max block on       The max time that the producer will    
  send>                                    block for during a send request      
                                           (default: 60000)                     
--max-memory-bytes <Long: total memory   The total memory used by the producer  
  in bytes>                                to buffer records waiting to be sent 
                                           to the server. (default: 33554432)   
--max-partition-memory-bytes <Long:      The buffer size allocated for a        
  memory in bytes per partition>           partition. When records are received 
                                           which are smaller than this size the 
                                           producer will attempt to             
                                           optimistically group them together   
                                           until this size is reached.          
                                           (default: 16384)                     
--message-send-max-retries <Integer>     Brokers can fail receiving the message 
                                           for multiple reasons, and being      
                                           unavailable transiently is just one  
                                           of them. This property specifies the 
                                           number of retires before the         
                                           producer give up and drop this       
                                           message. (default: 3)                
--metadata-expiry-ms <Long: metadata     The period of time in milliseconds     
  expiration interval>                     after which we force a refresh of    
                                           metadata even if we haven't seen any 
                                           leadership changes. (default: 300000)
--old-producer                           Use the old producer implementation.   
--producer-property <String:             A mechanism to pass user-defined       
  producer_prop>                           properties in the form key=value to  
                                           the producer.                        
--producer.config <String: config file>  Producer config properties file. Note  
                                           that [producer-property] takes       
                                           precedence over this config.         
--property <String: prop>                A mechanism to pass user-defined       
                                           properties in the form key=value to  
                                           the message reader. This allows      
                                           custom configuration for a user-     
                                           defined message reader.              
--queue-enqueuetimeout-ms <Integer:      Timeout for event enqueue (default:    
  queue enqueuetimeout ms>                 2147483647)                          
--queue-size <Integer: queue_size>       If set and the producer is running in  
                                           asynchronous mode, this gives the    
                                           maximum amount of  messages will     
                                           queue awaiting sufficient batch      
                                           size. (default: 10000)               
--request-required-acks <String:         The required acks of the producer      
  request required acks>                   requests (default: 1)                
--request-timeout-ms <Integer: request   The ack timeout of the producer        
  timeout ms>                              requests. Value must be non-negative 
                                           and non-zero (default: 1500)         
--retry-backoff-ms <Integer>             Before each retry, the producer        
                                           refreshes the metadata of relevant   
                                           topics. Since leader election takes  
                                           a bit of time, this property         
                                           specifies the amount of time that    
                                           the producer waits before refreshing 
                                           the metadata. (default: 100)         
--socket-buffer-size <Integer: size>     The size of the tcp RECV size.         
                                           (default: 102400)                    
--sync                                   If set message send requests to the    
                                           brokers are synchronously, one at a  
                                           time as they arrive.                 
--timeout <Integer: timeout_ms>          If set and the producer is running in  
                                           asynchronous mode, this gives the    
                                           maximum amount of time a message     
                                           will queue awaiting sufficient batch 
                                           size. The value is given in ms.      
                                           (default: 1000)                      
--topic <String: topic>                  REQUIRED: The topic id to produce      
                                           messages to.                         
--value-serializer <String:              The class name of the message encoder  
  encoder_class>                           implementation to use for            
                                           serializing values. (default: kafka. 
                                           serializer.DefaultEncoder)   
```


kafka-console-consumer.sh
------------
这个命令只是简单的将消息输出到标准输出中，一般使用的命令如下:

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

还可以通过下面的命令指定分区查看：

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning --partition 0

- --bootstrap-server 必须指定kafka服务地址
- --topic 也要指定查看的主题
- --from-beginning 如果想要从头查看消息


该命令更多参数如下
```sh
option                                   Description                            
------                                   -----------                            
--blacklist <String: blacklist>          Blacklist of topics to exclude from    
                                           consumption.                         
--bootstrap-server <String: server to    REQUIRED (unless old consumer is       
  connect to>                              used): The server to connect to.     
--consumer-property <String:             A mechanism to pass user-defined       
  consumer_prop>                           properties in the form key=value to  
                                           the consumer.                        
--consumer.config <String: config file>  Consumer config properties file. Note  
                                           that [consumer-property] takes       
                                           precedence over this config.         
--csv-reporter-enabled                   If set, the CSV metrics reporter will  
                                           be enabled                           
--delete-consumer-offsets                If specified, the consumer path in     
                                           zookeeper is deleted when starting up
--enable-systest-events                  Log lifecycle events of the consumer   
                                           in addition to logging consumed      
                                           messages. (This is specific for      
                                           system tests.)                       
--formatter <String: class>              The name of a class to use for         
                                           formatting kafka messages for        
                                           display. (default: kafka.tools.      
                                           DefaultMessageFormatter)             
--from-beginning                         If the consumer does not already have  
                                           an established offset to consume     
                                           from, start with the earliest        
                                           message present in the log rather    
                                           than the latest message.             
--key-deserializer <String:                                                     
  deserializer for key>                                                         
--max-messages <Integer: num_messages>   The maximum number of messages to      
                                           consume before exiting. If not set,  
                                           consumption is continual.            
--metrics-dir <String: metrics           If csv-reporter-enable is set, and     
  directory>                               this parameter isset, the csv        
                                           metrics will be outputed here        
--new-consumer                           Use the new consumer implementation.   
                                           This is the default.                 
--offset <String: consume offset>        The offset id to consume from (a non-  
                                           negative number), or 'earliest'      
                                           which means from beginning, or       
                                           'latest' which means from end        
                                           (default: latest)                    
--partition <Integer: partition>         The partition to consume from.         
--property <String: prop>                The properties to initialize the       
                                           message formatter.                   
--skip-message-on-error                  If there is an error when processing a 
                                           message, skip it instead of halt.    
--timeout-ms <Integer: timeout_ms>       If specified, exit if no message is    
                                           available for consumption for the    
                                           specified interval.                  
--topic <String: topic>                  The topic id to consume on.            
--value-deserializer <String:                                                   
  deserializer for values>                                                      
--whitelist <String: whitelist>          Whitelist of topics to include for     
                                           consumption.                         
--zookeeper <String: urls>               REQUIRED (only when using old          
                                           consumer): The connection string for 
                                           the zookeeper connection in the form 
                                           host:port. Multiple URLS can be      
                                           given to allow fail-over. 
```








命令那么多，怎么记？
--------------
Kafka 的命令行工具提供了非常丰富的提示信息，所以只需要记住上面大概的几个用法，知道怎么写就行。当需要用到某个命令时，通过命令提示进行操作。

比如说，如何使用 kafka-configs.sh 查看主题（Topic）的配置？

首先，在命令行中输入bin/kafka-configs.sh，然后或输出下面的命令提示信息。
```sh
Add/Remove entity config for a topic, client, user or broker
Option                      Description                                        
------                      -----------                                        
--add-config <String>       Key Value pairs of configs to add. Square brackets 
                              can be used to group values which contain commas:
                              'k1=v1,k2=[v1,v2,v2],k3=v3'. The following is a  
                              list of valid configurations: For entity_type    
                              'topics':                                        
                                cleanup.policy                                    
                                compression.type                                  
                                delete.retention.ms                               
                                file.delete.delay.ms                              
                                flush.messages                                    
                                flush.ms                                          
                                follower.replication.throttled.replicas           
                                index.interval.bytes                              
                                leader.replication.throttled.replicas             
                                max.message.bytes                                 
                                message.format.version                            
                                message.timestamp.difference.max.ms               
                                message.timestamp.type                            
                                min.cleanable.dirty.ratio                         
                                min.compaction.lag.ms                             
                                min.insync.replicas                               
                                preallocate                                       
                                retention.bytes                                   
                                retention.ms                                      
                                segment.bytes                                     
                                segment.index.bytes                               
                                segment.jitter.ms                                 
                                segment.ms                                        
                                unclean.leader.election.enable                    
                            For entity_type 'brokers':                         
                                follower.replication.throttled.rate               
                                leader.replication.throttled.rate                 
                            For entity_type 'users':                           
                                producer_byte_rate                                
                                SCRAM-SHA-256                                     
                                SCRAM-SHA-512                                     
                                consumer_byte_rate                                
                            For entity_type 'clients':                         
                                producer_byte_rate                                
                                consumer_byte_rate                                
                            Entity types 'users' and 'clients' may be specified
                              together to update config for clients of a       
                              specific user.                                   
--alter                     Alter the configuration for the entity.            
--delete-config <String>    config keys to remove 'k1,k2'                      
--describe                  List configs for the given entity.                 
--entity-default            Default entity name for clients/users (applies to  
                              corresponding entity type in command line)       
--entity-name <String>      Name of entity (topic name/client id/user principal
                              name/broker id)                                  
--entity-type <String>      Type of entity (topics/clients/users/brokers)      
--force                     Suppress console prompts                           
--help                      Print usage information.                           
--zookeeper <String: urls>  REQUIRED: The connection string for the zookeeper  
                              connection in the form host:port. Multiple URLS  
                              can be given to allow fail-over.
```
从第一行可以看到这个命令可以修改 topic, client, user 或 broker 的配置。

如果要设置 topic，就需要设置 entity-type 为topics，输入如下命令：
```sh
bin/kafka-configs.sh --entity-type topics
Command must include exactly one action: --describe, --alter
```

命令提示需要指定一个操作（不只是上面提示的两个操作），增加--describe试试：
```sh
bin/kafka-configs.sh --entity-type topics --describe
[root@localhost kafka_2.11-0.10.2.1]# bin/kafka-configs.sh --entity-type topics --describe
Missing required argument "[zookeeper]"
```

继续增加 --zookeeper：
```sh
bin/kafka-configs.sh --entity-type topics --describe --zookeeper localhost:2181
Configs for topic '__consumer_offsets' are segment.bytes=104857600,cleanup.policy=compact,compression.type=producer
```

由于没有指定主题名，这里显示了__consumer_offsets的信息。下面指定一个topic试试。
```sh
bin/kafka-configs.sh --entity-type topics --describe --zookeeper localhost:2181 --entity-name test
Configs for topic 'test' are
```
此时显示了test主题的信息，这里是空。

因为Kafka完善的命令提示，可以很轻松的通过提示信息来进行下一步操作，运用熟练后，基本上很快就能实现自己想要的命令。






