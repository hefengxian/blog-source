---
title: MySQL开启主从复制
date: 2018-08-31 16:32:13
categories: DB
tags: 
    - MySQL
    - MySQL Replication
---

## 必要条件

- 在主库上要开启 bin log
- 从库的 server-id 必须和主库不一样
- 主库创建一个专门的主从复制的用户
- 从主库获得 binlog 的位置
- 从库设置同步

创建用户
```sql
CREATE USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY 'poms@db';
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%';
```

获取 binlog 位置
```sql
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
UNLOCK TABLES;
```

配置从库
```sql
CHANGE MASTER TO
    MASTER_HOST='master_host_name',
    MASTER_USER='replication_user_name',
    MASTER_PASSWORD='replication_password',
    MASTER_LOG_FILE='recorded_log_file_name',
    MASTER_LOG_POS=recorded_log_position;
```

参考文档：

+ [Setting Up Binary Log File Position Based Replication](https://dev.mysql.com/doc/refman/8.0/en/replication-howto.html)

<!--more-->