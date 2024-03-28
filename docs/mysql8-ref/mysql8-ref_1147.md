# 17.1.3 验证 InnoDB 是否是默认存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-check-availability.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-check-availability.html)

发出 `SHOW ENGINES` 语句以查看可用的 MySQL 存储引擎。在 `SUPPORT` 列中查找 `DEFAULT`。

```sql
mysql> SHOW ENGINES;
```

或者查询信息模式 `ENGINES` 表。

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.ENGINES;
```
