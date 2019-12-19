[toc]

# 安装部署
## 单节点部署3节点zk集群
- 环境
```
centos7.4
zookeeper-3.4.13
```

- 配置
```
mkdir zookeeper
tar xf zookeeper-3.4.13.tar.gz --strip 1 -C zookeeper/
chown -R root:root zookeeper/
mkdir -p zk_cluster/{zk1,zk2,zk3}/{conf,data,dataLog}
```

- zk1
```
vim /home/data/zk_cluster/zk1/conf/zoo.cfg
tickTime = 2000
dataDir =/home/data/zk_cluster/zk1/data
dataLogDir = /home/data/zk_cluster/zk1/dataLog
clientPort = 2181
initLimit = 5
syncLimit = 2

server.0=10.43.82.12:2888:3888
server.1=10.43.82.12:2889:3889
server.2=10.43.82.12:2890:3890
```
```
echo "0" > zk_cluster/zk1/data/myid
```

- zk2
```
vim /home/data/zk_cluster/zk2/conf/zoo.cfg
tickTime = 2000
dataDir =/home/data/zk_cluster/zk2/data
dataLogDir = /home/data/zk_cluster/zk2/dataLog
clientPort = 2182
initLimit = 5
syncLimit = 2

server.0=10.43.82.12:2888:3888
server.1=10.43.82.12:2889:3889
server.2=10.43.82.12:2890:3890
```
```
echo "1" > zk_cluster/zk2/data/myid
```

- zk3
```
/home/data/zk_cluster/zk3/conf/zoo.cfg
tickTime = 2000
dataDir =/home/data/zk_cluster/zk3/data
dataLogDir = /home/data/zk_cluster/zk3/dataLog
clientPort = 2183
initLimit = 5
syncLimit = 2

server.0=10.43.82.12:2888:3888
server.1=10.43.82.12:2889:3889
server.2=10.43.82.12:2890:3890
```
```
echo "2" > zk_cluster/zk3/data/myid
```

- 启动
```
/home/data/zookeeper/bin/zkServer.sh start /home/data/zk_cluster/zk1/conf/zoo.cfg
/home/data/zookeeper/bin/zkServer.sh start /home/data/zk_cluster/zk2/conf/zoo.cfg
/home/data/zookeeper/bin/zkServer.sh start /home/data/zk_cluster/zk3/conf/zoo.cfg
```

- 检查
```
jps
```
```
QuorumPeerMain
QuorumPeerMain
QuorumPeerMain
```

- 查看日志
```
tail -100f zookeeper.out
```

- 查看状态
```
/home/data/zookeeper/bin/zkServer.sh status /home/data/zk_cluster/zk1/conf/zoo.cfg
/home/data/zookeeper/bin/zkServer.sh status /home/data/zk_cluster/zk2/conf/zoo.cfg
/home/data/zookeeper/bin/zkServer.sh status /home/data/zk_cluster/zk3/conf/zoo.cfg
```
提示：因为选举需要时间，所以等一段时间后再查看集群状态。

- 连入
```
/home/data/zookeeper/bin/zkCli.sh -server 10.43.82.12:2181,10.43.82.12:2182,10.43.82.12:2183
```

# 配置使用
- 创建节点
```
create /n1 "m1"
```

- 获取节点
```
get /n1
```

- 删除节点
```
delete /n1
```
