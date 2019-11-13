# 配置使用
## 数据的备份和恢复
### 备份
下载 tool 压缩包：
```
wget http://download.pingcap.org/tidb-enterprise-tools-latest-linux-amd64.tar.gz &&
wget http://download.pingcap.org/tidb-enterprise-tools-latest-linux-amd64.sha256
```
检查文件完整性，返回 ok 则正确：
```
sha256sum -c tidb-enterprise-tools-latest-linux-amd64.sha256
```
解开压缩包：
```
tar -xzf tidb-enterprise-tools-latest-linux-amd64.tar.gz &&
cd tidb-enterprise-tools-latest-linux-amd64
```
可使用 mydumper 从 TiDB 导出数据进行备份，然后用 loader 将其导入到 TiDB 里面进行恢复。

- 创建备份用户并授权
```
create user 'backupuser'@'%' identified by 'xxx';
GRANT SELECT,SHOW VIEW,EVENT,TRIGGER,LOCK TABLES,RELOAD, PROCESS, SUPER, REPLICATION CLIENT ON *.* TO 'backupuser'@'%';
```

- 从 TiDB 备份数据
```
mkdir -pv /data/backup/tidb/<db_name>
```
备份指定数据库
```
./bin/mydumper -u backupuser -h localhost -P 4000 -p 'xxx' -t 16 -F 64 -B ecarx --skip-tz-utc -o /data/backup/tidb/<db_name>/
```
```
-t 指定线程数
-F  是将实际的 table 切分成多大的 chunk，这里就是 64MB 一个 chunk
-B 指定数据库
--skip-tz-utc 添加这个参数忽略掉 TiDB 与导数据的机器之间时区设置不一致的情况，禁止自动转换
```
全库备份
```
mkdir -pv /data/backup/tidb/all
```
```
./bin/mydumper -u backupuser -h localhost -P 4000 -p 'xxx' -t 16 -F 64 --skip-tz-utc -o /data/backup/tidb/all/
```

### 恢复
- tidb
```
./bin/loader -u backupuser -h 127.0.0.1 -P 4000 -p 'xxx' -t 16 -d /data/backup/tidb/<db_name>
```

- mysql
```
./bin/loader -u admin -h ip -P 3306 -p 'xxx' -t 16 -d /data/backup/tidb/<db_name>
```