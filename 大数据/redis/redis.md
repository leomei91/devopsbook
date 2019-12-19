[toc]

# 安装部署
```
cd /tmp
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar xf redis-5.0.5.tar.gz
mkdir /app/redis
cd redis-5.0.5
make&&make PREFIX=/app/redis install
ln -s /app/redis/bin/redis-cli /usr/bin/
ln -s /app/redis/bin/redis-server /usr/bin/
```
```
cd /app/redis
mkdir -p /app/redis/conf/{7001,7002}
mkdir -p /app/redis/logs
```
```
vim /app/redis/conf/7001/redis.conf
```
```
bind 10.43.75.134 #
protected-mode yes
port 7001 #
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile /var/run/redis_7001.pid #
logfile "/app/redis/logs/7001.log" #
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /app/redis 
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-timeout 60
repl-disable-tcp-nodelay no
repl-backlog-size 1048576 
repl-backlog-ttl 3600
maxclients 65000 
maxmemory-policy volatile-lru 
appendonly yes 
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
cluster-enabled yes 
cluster-config-file nodes-7001.conf #
cluster-node-timeout 5000 
slowlog-log-slower-than 10 
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
hz 10
aof-rewrite-incremental-fsync yes
```
需要修改的配置项用`#`标记了。
- 启动
```
redis-server /app/redis/conf/7001/redis.conf
```
- 创建集群
```
redis-cli --cluster create 10.43.75.134:7001 10.43.75.134:7002 10.43.75.135:7001 10.43.75.135:7002 10.43.75.136:7001 10.43.75.136:7002 --cluster-replicas 1
```
```
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
- 测试集群

写入
```
redis-cli -c -h 10.43.75.134 -p 7001 set a '{"aa":"a","bb":"b"}'
```
```
OK
```
读取
```
redis-cli -c -h 10.43.75.136 -p 7001 get a
```
```
"{\"aa\":\"a\",\"bb\":\"b\"}"
```
删除
```
redis-cli -c -h 10.43.75.135 -p 7001 del a
```
```
(integer) 1
```