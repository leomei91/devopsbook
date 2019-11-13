# 配置使用
## 数据的备份和恢复
### 备份
```
mkdir -pv /data/backup/pgsql
```
```
pg_dumpall -U postgres -f /data/backup/pgsql/db_all.sql
```
- 配合定时任务
```
0 3 * * * /data/shell/backup.sh
```
```
vim /data/shell/backup.sh
```
```
#!/bin/bash
pg_dumpall -U postgres -f /data/backup/pgsql/db_all_`date +%Y%m%d%H%M%S`.sql
```
```
chmod +x /data/shell/backup.sh
```


### 恢复
```
psql -U postgres -f /data/backup/pgsql/db_all.sql
```