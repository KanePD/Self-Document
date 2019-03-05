# 配置kafka

- 下载

wget [**http://mirror.bit.edu.cn/apache/kafka/2.1.0/kafka_2.11-2.1.0.tgz**](http://mirror.bit.edu.cn/apache/kafka/2.1.0/kafka_2.11-2.1.0.tgz) 

- 解压

tar -xf kafka_2.11-2.1.0.tgz

- 下载zookeeper

wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz

- 解压后到conf下 将conf文件copy一份

cp zoo_sample.cfg zoo.cfg

- 查看是否成功

到bin目录下 ./zkServer.sh start 

注：提示一下 显示started不代表成功可以用如下语句查看是否开启成功

netstat -tunlp|egrep 2181

如果没有结果可查看同级目录下的zookeeper.out 文件 看看报什么错误

-----



```bash
cd /usr/local/kafka/config/
cp -a server.properties server1.properties
cp -a server.properties server2.properties

vim server.properties
#更改 
broker.id=1
log.dirs=''
listeners=PLAINTEXT://:9092
port=9092

```

启动

```bash
# Broker1
cd /usr/local/kafka
./bin/kafka-server-start.sh ./config/server.properties &

# Broker2
cd /usr/local/kafka
./bin/kafka-server-start.sh ./config/server1.properties &

# Broker3
cd /usr/local/kafka
./bin/kafka-server-start.sh ./config/server2.properties &
```

http://blog.51cto.com/13634837/2087600

```bash
# 开启zookeeper
> bin/zookeeper-server-start.sh config/zookeeper.properties
```

```bash
# 开启kafka
> bin/kafka-server-start.sh config/server.properties

```



```bash
#create topic
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

```bash
#list topic
> bin/kafka-topics.sh --list --zookeeper localhost:2181
```

```bash
# producer
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

```bash
# consumer
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

netstat -tunlp|egrep "(2181|9092)"

- python访问的时候 需要配置host文件 ip hostname，因为匹配的时候 不是ip而是根据域名匹配的

-----

多个partitions 时 会分配到不同的broker中去，replication-factor会告诉kafka有多少的副本

- 副本因子不能大于 Broker 的个数；
- 第一个分区（编号为0）的第一个副本放置位置是随机从 brokerList 选择的；
- https://blog.csdn.net/yangyutong0506/article/details/78970972