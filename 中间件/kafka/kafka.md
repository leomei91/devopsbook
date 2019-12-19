[toc]

# 安装部署
- 配置
```
cd kafka
mkdir -p logs/{1,2,3}
```

- server1.properties
```
vim config/server1.properties
broker.id=1
listeners=PLAINTEXT://:9092
host.name=10.43.82.12
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/home/data/kafka/logs/1
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=10.43.82.12:2181,10.43.82.12:2182,10.43.82.12:2183
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
```

- server2.properties
```
broker.id=2
listeners=PLAINTEXT://:9093
host.name=10.43.82.12
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/home/data/kafka/logs/2
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=10.43.82.12:2181,10.43.82.12:2182,10.43.82.12:2183
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
```

- server3.properties
```
broker.id=3
listeners=PLAINTEXT://:9094
host.name=10.43.82.12
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/home/data/kafka/logs/3
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=10.43.82.12:2181,10.43.82.12:2182,10.43.82.12:2183
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
```

- 启动
```
bin/kafka-server-start.sh -daemon config/server1.properties
bin/kafka-server-start.sh -daemon config/server2.properties
bin/kafka-server-start.sh -daemon config/server3.properties
```

- 停止
```
bin/kafka-server-stop.sh config/server1.properties
bin/kafka-server-stop.sh config/server2.properties
bin/kafka-server-stop.sh config/server3.properties
```

- 检查
```
jps
```
```
Kafka
Kafka
Kafka
```

# 配置使用
- 创建topic
```
bin/kafka-topics.sh --create --topic topicname --replication-factor 1 --partitions 1 --zookeeper localhost:2181
```

- 查看topic
```
./bin/kafka-topics.sh --list --zookeeper 10.43.82.13:2181
```

- 删除topic
```
./bin/kafka-topics.sh --delete --zookeeper 10.43.82.13:2181 --topic test1
```

- 发送消息
```
bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test1
```

- 接受消息
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic test1
```