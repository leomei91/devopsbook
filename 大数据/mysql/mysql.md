# 配置使用
## 用户创建
- 应用用户
```
create user 'appuser'@'%' identified by 'xxx';
```
授权
```
grant create,select,update,delete,insert on <db_name>.* to 'appuser'@'%';
```

- 备份用户
```
GRANT SELECT,SHOW VIEW,EVENT,TRIGGER,LOCK TABLES,RELOAD, PROCESS, SUPER, REPLICATION CLIENT ON *.* TO 'backupuser'@'127.0.0.1' identified by 'xxx';
```

- 只读用户
```
create user 'readuser'@'%' identified by 'xxx';
```
授权
```
grant select on <db_name>.* to 'readuser'@'%';
```

## 重置密码
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'xxx';
FLUSH PRIVILEGES;
```

## 查看所有用户
```
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;
```

## 查看用户权限
```
show grants for 'appuser'@'%';
```

## 创建数据库
```
CREATE DATABASE `mydb` CHARACTER SET utf8 COLLATE utf8_general_ci;
```

## 创建表
```
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_id` INT UNSIGNED AUTO_INCREMENT,
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 插入数据
```
INSERT INTO runoob_tbl (runoob_title, runoob_author, submission_date) VALUES ("学习 PHP", "菜鸟教程", NOW());
```

## 清空表数据
```
truncate table <table_name>;
```

## 数据备份和恢复
### 备份
```
mkdir -pv /data/backup/mysql/<db_name>/
```
```
./bin/mydumper -u backupuser -h ip -P 3306 -p 'xxx' -t 16 -F 64 -B <db_name> --skip-tz-utc -o /data/backup/mysql/<db_name>/
```
- 备份脚本
```
vim /data/shell/mysql_backup.sh
```
```
#!/bin/bash
DIR=/data/backup/mysql/<db_name>_`date +%Y%m%d%H%M%S`/
mydumper -u backupuser -h ip -P 3306 -p 'xxx' -t 16 -F 64 -B <db_name> --skip-tz-utc -o $DIR
```
```
chmod +x /data/shell/mysql_backup.sh
```

- 计划任务
```
0 2 * * * /data/shell/mysql_backup.sh
```

## 恢复
- tidb
```
./bin/loader -u backupuser -h 127.0.0.1 -P 4000 -p 'xxx' -t 16 -d /data/backup/mysql/<db_name>/
```