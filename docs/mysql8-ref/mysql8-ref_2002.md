# 29.4.9 为过滤操作命名仪表或消费者

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-filtering-names.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-filtering-names.html)

为过滤操作提供的名称可以根据需要具体或一般。要指示单个仪表或消费者，请完整指定其名称：

```sql
UPDATE performance_schema.setup_instruments
SET ENABLED = 'NO'
WHERE NAME = 'wait/synch/mutex/myisammrg/MYRG_INFO::mutex';

UPDATE performance_schema.setup_consumers
SET ENABLED = 'NO'
WHERE NAME = 'events_waits_current';
```

要指定一组仪表或消费者，请使用匹配组成员的模式：

```sql
UPDATE performance_schema.setup_instruments
SET ENABLED = 'NO'
WHERE NAME LIKE 'wait/synch/mutex/%';

UPDATE performance_schema.setup_consumers
SET ENABLED = 'NO'
WHERE NAME LIKE '%history%';
```

如果使用模式，应选择能够匹配所有感兴趣的项目但不匹配其他项目的模式。例如，要选择所有文件 I/O 仪表，最好使用包含整个仪表名称前缀的模式：

```sql
... WHERE NAME LIKE 'wait/io/file/%';
```

`'%/file/%'`的模式匹配其他仪表中名称中任何位置包含`'/file/'`的元素。更不合适的是`'%file%'`这种模式，因为它匹配名称中任何位置包含`'file'`的仪表，比如`wait/synch/mutex/innodb/file_open_mutex`。

要检查模式匹配的仪表或消费者名称，请执行简单测试：

```sql
SELECT NAME FROM performance_schema.setup_instruments
WHERE NAME LIKE '*pattern*';

SELECT NAME FROM performance_schema.setup_consumers
WHERE NAME LIKE '*pattern*';
```

有关支持的名称类型的信息，请参阅第 29.6 节，“性能模式仪表命名约定”。
