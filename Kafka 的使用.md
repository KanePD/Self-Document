[TOC]

# Kafka 概念、单机搭建与使用

官方网址：[Apache Kafka® is *a distributed streaming platform*](http://kafka.apache.org/intro)

## 基本概念介绍

在Kafka中有一些基本的概念，

### **Topic**

- 简介：Topic在Kafka中是一个抽象的概念，一个主题是已经发布的记录的种类。主题在Kafka中是可以被多重订阅的，这就意味着一个主题可能有0个、一个、或者许多个消费者去订阅这个主题中的消息。

- Partitions：在每一个topic在Kafka中可以有多个分区，增加一个主题的分区可以提高Kafka的吞吐率，但是不是越多越好，因为如果分区数量越多的话生产者插入的效率也会降低。所以真正到生产环境时，需要权衡生产与消费的一个平衡关系，消费稍微大于生产者，不会产生消息的堆积，也能够充分提高Kafka的效率。

- Replication Factor：复制因子，是对于当前的Topic是否需要副本。如果设置成1的话，代表当前Topic在整个Kafka中只有一份。这里有个限制Topic的数量不能够多于当前Kafka的Broker数量。

- 存储方式：在Kafka的配置中(Server.properties)有logs.dir的配置，这个是Kafka存储消息的位置。如果Topic复制因子是1分区是1的话，在对应的文件夹下会有一个名称为topicname的文件夹；如果复制因子是2分区是2，假设存在两个Broker，在每个Broker中将会存在两个文件夹分别为topicname_0 topicname_1的文件夹

- Leader与Follower：由于每个topic如果存在副本的话，是对于partition进行复制。这么多存在在不同的Broker上的副本，其中有一个partition是leader其他的是Followers，当一个broker宕机会在副本中选择一个充当Leader。关于Kafka中的选举机制以及Leader的确认可以查看这两篇文章：[Leader确认](https://blog.csdn.net/qingqing7/article/details/80842511)、[选举机制](https://blog.csdn.net/yanshu2012/article/details/54894629)

### **Producer**

生产者，顾明思议是生产消息，允许应用发布一个流的消息到一个或者多个主题中，

### **Consumer**
  - 简介：消费者是订阅某个topic消息。
  - Group:每个消费者都有个groupid 来标定当前消费者属于哪个group。Group的作用是，当同一个group的两个消费者订阅一个topic的时候，如果当前topic没有分区那么其中一个消费者是获得不了任何消息的；如果有分区的话，将会按照数量进行负载均衡，每个消费者获得不同的分区的消息。
  - 同一个Group下的消费者不会同时订阅一个主题下的同一个分区，如果消费者数量杜宇分区数量，则多出的消费者是不会有任何消息获得的。
### **Broker**

Broker 是一个Kafka的Server，一台单物理机或者集群都可以拥有多个broker一个broker可以容纳多个主题，这个与复制因子、主题的分区都有关系。

## Kafka单机配置，一个Broker

### 环境：

- win10物理机
- Wmare Centos7虚拟机
- XShell 访问虚拟机

### 配置zookeeper

- 下载

```bash
# zookeeper
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
```

- 解压后进入目录

```bash
cd zookeeper-3.4.13/conf
```

- 复制zookeeper的配置文件

```bash
cp zoo_sample.cfg zoo.cfg	
```

- 返回上级进入bin目录下，键入如下命令

```bash
./zkServer.sh start 
```

- 查看是否成功开启zookeeper服务

```bash
#注：这里提示一下开启后提示的成功不一定是真的成功,所以需要查看一下
netstat -tunlp|egrep 2181
# 如果没有结果查看统计目录下的 zookeeper.out文件 查看log信息
# 使用jps命令查看 QuorumPeerMain是zookeeper的守护进程
11089 QuorumPeerMain
11114 Jps
```

### 配置Kafka

- 下载安装包

```bash
# Kafka
wget http://mirror.bit.edu.cn/apache/kafka/2.1.0/kafka_2.11-2.1.0.tgz
```

- 解压后进入文件夹下bin目录下

```bash
# 第一个是start.sh位置第二个是server.rpoperties的位置，所以确认好路径的正确性
./kafka-server-start.sh ./../config/server.properties &
# 我们可以在Kafka的目录下直接执行，而不进入到bin下，命令看着更舒服些
./bin/kafka-server-start.sh ./config/server.properties &
```

- 查看是否开启成功：默认的Kafka端口是9092，zookeeper是2181

```bash
netstat -tunlp|egrep "(2181|9092)"
# 结果如下
[root@localhost ~]# netstat -tunlp|egrep "(2181|9092)"
tcp6      0     0 :::9092               :::*                  LISTEN      1877/java  tcp6      0     0 :::2181               :::*                  LISTEN      1820/java
# jps 查看
11089 QuorumPeerMain
11458 Kafka
11847 Jps
```

- 至此Kafka配置成功

### 使用Kafka

#### 创建topic

```bash
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
# 返回结果
Created topic "test"
```

#### 在虚拟机用**sh脚本**上作为生产者生产消息

- 我们重新开一个Xshell窗口，CD到`Kafka目录/bin`下，我们先介绍这一节会使用到的 **kafka-console-producer.sh**

```bash
# 键入如下命令
./kafka-console-producer.sh --broker-list localhost:9092 --topic test
>today message
>
# 最近本的指定，broker-list与topic是必须的参数
# 成功命令行会进入一个>的情况，键入消息按回车键就是发送消息到Kafka了
# 发送一个【today message】
```

- **kafka-console-producer.sh**参数说明，运行`./kafka-console-producer.sh --help`可查看

#### 在虚拟机上用**sh脚本**作为消费者消费消息

- 重新开另个一Xshell窗口CD到`Kafka目录/bin`下，我们先介绍这一节会使用到的 **kafka-console-consumer.sh**

```bash
# 键入如下命令
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
# 最近本的指定，bootstrap-server与topic/whitelist是必须的参数
# 由于有 from-beginning 参数 会从头load所有消息
# 消费后返回如下
today message
#在生产端键入消息后，消费端会同步消息出现
```

- **kafka-console-consumer.sh**参数说明运行`./kafka-console-consumer.sh --help`可查看

#### 使用**Python**作为生产者、消费者

- 在物理机上写一个Python生产者的脚本

```python
from kafka.producer import KafkaProducer
import time
def send_data(data):
	producer = KafkaProducer(bootstrap_servers='192.168.233.138:9092')
	producer.send("test",b''+str(data)+'')
    producer.flush()
	print ("end")
	
if __name__=="__main__":
	send_data("physics python message");
```

- 查看Xshell上消费的命令行

```bash
[root@localhost ~]# /home/kafka_2.11-2.1.0/bin/kafka-console-consumer.sh --bootstrap-server 192.168.233.138:9092 --topic test --from-beginning
111
333

1
12
physics python message
```

- 在物理机上写一个消费者的脚本

```python
from kafka import KafkaConsumer
import time
def get_data(data):
	consumer = KafkaConsumer('test',bootstrap_servers='192.168.233.138:9092', group_id='my_favorite_group')
	print ("end")
	for msg in consumer:
		print(msg)
	
if __name__=="__main__":
	get_data();
```

- 物理机消费者的结果

```bash
# 我这边是先运行的消费者的脚本，所以实时接收到了物理机产生的消息
ConsumerRecord(topic=u'test', partition=0, offset=5, timestamp=1551762485911L, timestamp_type=0, key=None, value='physics python message', checksum=1520092583, serialized_key_size=-1, serialized_value_size=22)
```

- 测试使用虚拟机sh端的生产者发送123 消息，查看物理机消费者结果

```bash
ConsumerRecord(topic=u'test', partition=0, offset=6, timestamp=1551762784609L, timestamp_type=0, key=None, value='123', checksum=1760815061, serialized_key_size=-1, serialized_value_size=3)
```

- <font color="red">几点注意</font>

```bash
# 物理机连接时可能出现【kafka.errors.NoBrokersAvailable: NoBrokersAvailable】这个错误按照如下顺序依次更改
1. 查看虚拟机防火墙是否关闭
	systemctl status firewalld
	systemctl stop firewalld
2. 更改kafka服务端的server.properties:
	增加 [ listeners=PLAINTEXT://192.168.233.138:9092 ]这一行
3. 修改物理机的hosts文件 C:\Windows\System32\drivers\etc\hosts
	增加 【虚拟机ip 虚拟机主机名】 Eg:[192.168.233.138 localhost]
```

#### 使用**Springboot** 作为生产者、消费者

注：我直接在我的一个寄存的Spring Boot Demo项目上更改

- 在pom.xml中添加kafka依赖

```xml
 <dependency>
 <groupId>org.springframework.kafka</groupId>
 <artifactId>spring-kafka</artifactId>
 </dependency>
<!-- 提示一件事情此处别指定version了，直接用最新的就可以，老的版本一些包找不到 -->
```

- 写一个kafka 生产者配置类

```java
package com.example.kane.config;

import java.util.HashMap;
import java.util.Map;
import java.util.regex.Pattern;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

@Configuration
@EnableKafka
public class kafka_config {
	 public Map<String, Object> producerConfigs() {
	        Map<String, Object> props = new HashMap<>();
	        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.233.138:9092");
	        props.put(ProducerConfig.RETRIES_CONFIG, 0);
	        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 4096);
	        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
	        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 40960);
	        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
	        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
	        return props;
	    }
	 
	    public ProducerFactory<String, String> producerFactory() {
	        return new DefaultKafkaProducerFactory<>(producerConfigs());
	    }
	 
	    @Bean
	    public KafkaTemplate<String, String> kafkaTemplate() {
	        return new KafkaTemplate<String, String>(producerFactory());
	    }

}
```

- 创建一个生产数据的Controller

```java
package com.example.kane.Controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


@RestController
@RequestMapping("/kafka")
public class CollectController {
	 protected final Logger logger = LoggerFactory.getLogger(this.getClass());
	    @Autowired
	    private KafkaTemplate kafkaTemplate;

	    @RequestMapping(value = "/send", method = RequestMethod.GET)
	    public void sendKafka(HttpServletRequest request, HttpServletResponse response) {
	        try {
	            String message = request.getParameter("message");
	            logger.info("kafka的消息={}", message);
	            kafkaTemplate.send("test", "key", message);
	            logger.info("发送kafka成功.");
	        } catch (Exception e) {
	            logger.error("发送kafka失败", e);
	        }
	    }

}
```

- 启动项目后，在浏览器访问http://localhost:8080/kafka/send?message=url_producer

```bash
# 查看结果
2019-03-05 13:57:16.438  INFO 10208 --- [nio-8080-exec-1] c.e.kane.Controller.CollectController    : 发送kafka成功.
2019-03-05 13:57:45.871  INFO 10208 --- [nio-8080-exec-5] c.e.kane.Controller.CollectController    : kafka的消息=url_producer
2019-03-05 13:57:45.872  INFO 10208 --- [nio-8080-exec-5] c.e.kane.Controller.CollectController    : 发送kafka成功.
# 查看虚拟机 Consumer结果

[root@localhost ~]# /home/kafka_2.11-2.1.0/bin/kafka-console-consumer.sh --bootstrap-server 192.168.233.138:9092 --topic test --from-beginning
physics python message
123
null
url_producer
```

- 增加消费者的配置

```java
package com.example.kane.config;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;

import java.util.HashMap;
import java.util.Map;

import com.example.kane.service.kafka_listener;
@Configuration
@EnableKafka
public class kafka_consumer_config {
    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }

    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }


    public Map<String, Object> consumerConfigs() {
        Map<String, Object> propsMap = new HashMap<>();
        propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.233.138:9092");
        propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
        propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        return propsMap;
    }
    @Bean
    public kafka_listener listener() {
        return new kafka_listener();
    }
}
```

- 增加listener类

```java
package com.example.kane.service;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
public class kafka_listener {
    protected final Logger logger = LoggerFactory.getLogger(this.getClass());


    @KafkaListener(topics = {"test"})
    public void listen(ConsumerRecord<?, ?> record) {
    	logger.info(record.toString());
        logger.info("kafka的key: " + record.key());
        logger.info("kafka的value: " + record.value().toString());
    }
}
```

- 同样我们用访问http://localhost:8080/kafka/send?message=url_producer1重新发一个消息

```bash
# 结果
2019-03-05 14:31:04.787  INFO 10208 --- [nio-8080-exec-1] c.e.kane.Controller.CollectController    : 发送kafka成功.
2019-03-05 14:31:04.848  INFO 10208 --- [ntainer#0-0-C-1] com.example.kane.service.kafka_listener  : ConsumerRecord(topic = test, partition = 0, offset = 10, CreateTime = 1551767464787, serialized key size = 3, serialized value size = 13, headers = RecordHeaders(headers = [], isReadOnly = false), key = key, value = url_producer1)
2019-03-05 14:31:04.848  INFO 10208 --- [ntainer#0-0-C-1] com.example.kane.service.kafka_listener  : kafka的key: key
2019-03-05 14:31:04.848  INFO 10208 --- [ntainer#0-0-C-1] com.example.kane.service.kafka_listener  : kafka的value: url_producer1
# 查看虚拟机 消费者信息
physics python message
123
null
url_producer
url_producer1
url_producer1
```

## 一些需要注意的问题

1. 现在kafka官方提供自带zookeeper版本，不建议使用自带的，还是建议自己安装zookeeper
2. 物理机没法访问的时候，看文中的注意事项，依次更改一定能访问

