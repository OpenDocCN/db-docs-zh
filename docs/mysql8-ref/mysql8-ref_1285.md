> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-dml.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-dml.html)

#### 17.20.6.5 将 DML 语句调整为 memcached 操作

基准测试表明，`daemon_memcached` 插件加速了 DML 操作（插入、更新和删除）比加速查询更多。因此，考虑将初始开发工作重点放在写入密集型且 I/O 限制的应用程序上，并寻找机会在新的写入密集型应用程序中使用带有 `daemon_memcached` 插件的 MySQL。

单行 DML 语句是最容易转换为 `memcached` 操作的语句类型。`INSERT` 变为 `add`，`UPDATE` 变为 `set`，`incr` 或 `decr`，而 `DELETE` 变为 `delete`。这些操作通过 **memcached** 接口发出时，保证只影响一行，因为 *`key`* 在表内是唯一的。

在以下的 SQL 示例中，`t1` 指的是基于 `innodb_memcache.containers` 表中配置的 **memcached** 操作所使用的表。`key` 指的是列在 `key_columns` 下列出的列，`val` 指的是列在 `value_columns` 下列出的列。

```sql
INSERT INTO t1 (key,val) VALUES (*some_key*,*some_value*);
SELECT val FROM t1 WHERE key = *some_key*;
UPDATE t1 SET val = *new_value* WHERE key = *some_key*;
UPDATE t1 SET val = val + x WHERE key = *some_key*;
DELETE FROM t1 WHERE key = *some_key*;
```

下面的 `TRUNCATE TABLE` 和 `DELETE` 语句，用于从表中删除所有行，对应于 `flush_all` 操作，其中 `t1` 被配置为 **memcached** 操作的表，就像前面的示例中一样。

```sql
TRUNCATE TABLE t1;
DELETE FROM t1;
```
