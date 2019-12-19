[toc]

# 安装部署
```
# 下载rpm包
yum install https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-redhat11-11-2.noarch.rpm -y

# 安装
yum -y install postgresql11 postgresql11-server postgresql11-libs

# 初始化数据库
/usr/pgsql-11/bin/postgresql-11-setup initdb

# 设置开机自启动PostgreSQL和启动服务
systemctl enable postgresql-11
systemctl start postgresql-11
systemctl status postgresql-11
## 看到控制台输出的Active后有Running的字样说明启动完成
```
- 配置
```
vim /var/lib/pgsql/11/data/pg_hba.conf
```
```
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             10.43.75.0/24           trust
```
```
vim /var/lib/pgsql/11/data/postgresql.conf
```
```
listen_addresses = '*'
```
- 重启
```
systemctl restart postgresql-11
```

## 插件安装
https://www.jianshu.com/p/f4ab815f01f0
```
yum -y install epel-release
yum install postgis25_11
yum install postgis25_11-client
```
- 验证
```
sudo su - postgres
$ psql -h 127.0.0.1
```
```
postgres=# CREATE DATABASE gistest;
postgres=# \c gistest;
gistest=# CREATE EXTENSION postgis;
gistest=# SELECT postgis_full_version();
                                                                                              postgis_full_version                                                                                              
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 POSTGIS="2.5.3 r17699" [EXTENSION] PGSQL="110" GEOS="3.8.0-CAPI-1.13.1 " PROJ="Rel. 6.2.1, November 1st, 2019" GDAL="GDAL 3.0.2, released 2019/10/28" LIBXML="2.9.1" LIBJSON="0.11" LIBPROTOBUF="1.0.2" RASTER
(1 row)
```

# 配置使用
## 基本SQL
- 连入
```
sudo su - postgres
psql -h 127.0.0.1
```
- 列出所有数据库
```
\l
```
- 查看SQL命令的解释
```
\h select
```
- 查看psql命令列表
```
\?
```
- 连接其他数据库
```
\c [database_name]
```
- 列出当前数据库的所有表格
```
\d
```
- 列出某一张表格的结构
```
\d [table_name]
```
- 列出所有用户
```
\du
```
- 打开文本编辑器
```
\e
```
- 列出当前数据库和连接的信息
```
\conninfo
```
- 创建数据库
```
postgres=# CREATE DATABASE admap;
```
注意：`=#`环境才能创建成功

- 导入sql脚本
```
psql addis_imp_mdb < addis_imp_mdb.sql
```

- 备份单个数据库
```
pg_dump -U postgres -d imp_public -Fp -f imp_public.sql
```

- 使用如下命令可对全部pg数据库进行备份。
```
pg_dumpall -U postgres -f db_bak.sql
```

- 恢复方式很简单。执行恢复命令即可：
```
psql -U postgres -f imp_public.sql imp_public
psql -U postgres –f db_bak.sql
```

- 清空表
```
TRUNCATE table_name;
```
- 查看数据
```
select * from "AD_LaneDivider";
```

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
或者
```
psql imp_task < imp_task_clear_task.sql
```