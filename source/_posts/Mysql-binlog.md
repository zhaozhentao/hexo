---
title: Mysql binlog
date: 2021-09-27 15:05:58
tags:
---

## 开启Binlog

#### `/etc/my.cnf` 加入配置，需要重启mysql才能生效

``` mysql
[mysqld]
log-bin         = my-binlog-name # binlog文件名
server-id       = 1 # mysql id，区分主库备库
```

## Binlog相关参数

``` mysql
# 查看biglog格式(Statement, Row, Mixed三种)
mysql> show variables like 'binlog_format'

# 是否启用binlog日志
mysql> show variables like 'log_bin';

# 查看binlog的目录
mysql> show global variables like "%log_bin%";

# 查看当前服务器使用的biglog文件及大小
mysql> show binary logs;
```

## Binlog恢复数据库相关操作

``` mysql
产生一个新的binlog文件，避免旧的binlog文件继续增长不好排查
mysql> flush logs;

重置(清除)binlog文件
mysql> reset master;

列出binlog文件(检查上面执行的reset或者flush结果)
mysql> show master logs;

查看某个binlog文件pos信息
mysql> show binlog events in 'my-binlog-name.000004'

查看binlog文件
mysqlbinlog --base64-output=DECODE-ROWS -v #{要查看的binlog文件路径}

备份数据库
mysqldump -h{mysql ip} -uroot -p -B -F -R -x --master-data=2 {数据库名} | gzip > /Users/zhaotao/Desktop/db_name_$(date +%F).sql.gz

指定binlog停止的位置，生成要导入的文件
mysqlbinlog --stop-position=509 --skip-gtids --database=buhang my-binlog-name.000004 > /Users/zhaotao/Desktop/haha.sql

将上面生成的文件导入到数据库 完成恢复
mysql -h 127.0.0.1 -u root -p123456 buhang < haha.sql
```
