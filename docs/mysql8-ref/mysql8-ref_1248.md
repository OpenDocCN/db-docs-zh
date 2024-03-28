# 17.15 InnoDB INFORMATION_SCHEMA 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema.html)

17.15.1 InnoDB INFORMATION_SCHEMA 关于压缩的表

17.15.2 InnoDB INFORMATION_SCHEMA 事务和锁信息表

17.15.3 InnoDB INFORMATION_SCHEMA 模式对象表

17.15.4 InnoDB INFORMATION_SCHEMA 全文索引表

17.15.5 InnoDB INFORMATION_SCHEMA 缓冲池表

17.15.6 InnoDB INFORMATION_SCHEMA 指标表

17.15.7 InnoDB INFORMATION_SCHEMA 临时表信息表

17.15.8 从 INFORMATION_SCHEMA.FILES 检索 InnoDB 表空间元数据

本节提供了关于`InnoDB` `INFORMATION_SCHEMA` 表的信息和使用示例。

`InnoDB` `INFORMATION_SCHEMA` 表提供了关于`InnoDB` 存储引擎各个方面的元数据、状态信息和统计信息。您可以通过在`INFORMATION_SCHEMA` 数据库上发出`SHOW TABLES`语句来查看`InnoDB` `INFORMATION_SCHEMA` 表的列表：

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB%';
```

对于表定义，请参阅第 28.4 节，“INFORMATION_SCHEMA InnoDB 表”。关于`MySQL` `INFORMATION_SCHEMA` 数据库的一般信息，请参阅第二十八章，“*INFORMATION_SCHEMA 表*”。
