# 15.1.34 DROP TRIGGER Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-trigger.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-trigger.html)

```sql
DROP TRIGGER [IF EXISTS] [*schema_name*.]*trigger_name*
```

这个语句会触发一个触发器。模式（数据库）名称是可选的。如果省略了模式，触发器将从默认模式中删除。`DROP TRIGGER`需要与触发器关联的表的`TRIGGER`权限。

使用`IF EXISTS`可以防止出现触发器不存在的错误。在使用`IF EXISTS`时，会为不存在的触发器生成一个`NOTE`。参见 Section 15.7.7.42, “SHOW WARNINGS Statement”。

如果删除表，则该表的触发器也会被删除。
