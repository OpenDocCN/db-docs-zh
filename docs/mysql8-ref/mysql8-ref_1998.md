# 29.4.5 对象预过滤

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-object-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-object-filtering.html)

`setup_objects`表控制性能模式监视特定表格和存储程序对象。初始`setup_objects`内容如下：

```sql
mysql> SELECT * FROM performance_schema.setup_objects;
+-------------+--------------------+-------------+---------+-------+
| OBJECT_TYPE | OBJECT_SCHEMA      | OBJECT_NAME | ENABLED | TIMED |
+-------------+--------------------+-------------+---------+-------+
| EVENT       | mysql              | %           | NO      | NO    |
| EVENT       | performance_schema | %           | NO      | NO    |
| EVENT       | information_schema | %           | NO      | NO    |
| EVENT       | %                  | %           | YES     | YES   |
| FUNCTION    | mysql              | %           | NO      | NO    |
| FUNCTION    | performance_schema | %           | NO      | NO    |
| FUNCTION    | information_schema | %           | NO      | NO    |
| FUNCTION    | %                  | %           | YES     | YES   |
| PROCEDURE   | mysql              | %           | NO      | NO    |
| PROCEDURE   | performance_schema | %           | NO      | NO    |
| PROCEDURE   | information_schema | %           | NO      | NO    |
| PROCEDURE   | %                  | %           | YES     | YES   |
| TABLE       | mysql              | %           | NO      | NO    |
| TABLE       | performance_schema | %           | NO      | NO    |
| TABLE       | information_schema | %           | NO      | NO    |
| TABLE       | %                  | %           | YES     | YES   |
| TRIGGER     | mysql              | %           | NO      | NO    |
| TRIGGER     | performance_schema | %           | NO      | NO    |
| TRIGGER     | information_schema | %           | NO      | NO    |
| TRIGGER     | %                  | %           | YES     | YES   |
+-------------+--------------------+-------------+---------+-------+
```

对`setup_objects`表的修改会立即影响对象监视。

`OBJECT_TYPE`列指示行适用的对象类型。`TABLE`过滤影响表格 I/O 事件(`wait/io/table/sql/handler`仪器)和表格锁事件(`wait/lock/table/sql/handler`仪器)。

`OBJECT_SCHEMA`和`OBJECT_NAME`列应包含文字模式或对象名称，或`'%'`以匹配任何名称。

`ENABLED`列指示是否监视匹配对象，`TIMED`指示是否收集时间信息。设置`TIMED`列会影响性能模式表内容，如第 29.4.1 节“性能模式事件定时”中所述。

默认对象配置的效果是对除`mysql`、`INFORMATION_SCHEMA`和`performance_schema`数据库中的对象之外的所有对象进行仪器化。 (无论`setup_objects`的内容如何，`INFORMATION_SCHEMA`数据库中的表格都不会被仪器化；`information_schema.%`的行只是明确说明了这一默认值。)

当性能模式在`setup_objects`中查找匹配时，它首先尝试找到更具体的匹配。对于匹配给定`OBJECT_TYPE`的行，性能模式按照以下顺序检查行：

+   行中的`OBJECT_SCHEMA='*`文字`*'`和`OBJECT_NAME='*`文字`*'`。

+   行中的`OBJECT_SCHEMA='*`文字`*'`和`OBJECT_NAME='%'`。

+   表格中的`OBJECT_SCHEMA='%'`和`OBJECT_NAME='%'`。

例如，对于表格`db1.t1`，性能模式会在`TABLE`行中查找匹配`'db1'`和`'t1'`，然后是`'db1'`和`'%'`，最后是`'%'`和`'%'`。匹配发生的顺序很重要，因为不同的匹配`setup_objects`行可能具有不同的`ENABLED`和`TIMED`值。

对于与表相关的事件，性能模式将 `setup_objects` 的内容与 `setup_instruments` 结合起来，以确定是否启用仪器以及是否计时启用的仪器：

+   对于与 `setup_objects` 中的行匹配的表，只有在 `setup_instruments` 和 `setup_objects` 中的 `ENABLED` 都是 `YES` 时，表仪器才会产生事件。

+   两个表中的 `TIMED` 值被合并，只有当两个值都是 `YES` 时才收集时间信息。

对于存储过程对象，性能模式直接从 `setup_objects` 行中获取 `ENABLED` 和 `TIMED` 列。 不会将值与 `setup_instruments` 结合。

假设 `setup_objects` 包含适用于 `db1`、`db2` 和 `db3` 的以下 `TABLE` 行：

```sql
+-------------+---------------+-------------+---------+-------+
| OBJECT_TYPE | OBJECT_SCHEMA | OBJECT_NAME | ENABLED | TIMED |
+-------------+---------------+-------------+---------+-------+
| TABLE       | db1           | t1          | YES     | YES   |
| TABLE       | db1           | t2          | NO      | NO    |
| TABLE       | db2           | %           | YES     | YES   |
| TABLE       | db3           | %           | NO      | NO    |
| TABLE       | %             | %           | YES     | YES   |
+-------------+---------------+-------------+---------+-------+
```

如果 `setup_instruments` 中与对象相关的仪器具有 `ENABLED` 值为 `NO`，则不会监视对象的事件。 如果 `ENABLED` 值为 `YES`，则根据相关 `setup_objects` 行中的 `ENABLED` 值进行事件监视：

+   `db1.t1` 事件被监视

+   `db1.t2` 事件不被监视

+   `db2.t3` 事件被监视

+   `db3.t4` 事件不被监视

+   `db4.t5` 事件被监视

类似的逻辑也适用于从 `setup_instruments` 和 `setup_objects` 表中合并 `TIMED` 列以确定是否收集事件时间信息。

如果一个持久表和一个临时表具有相同的名称，则对 `setup_objects` 行进行匹配的方式对两者都是相同的。 不可能为一个表启用监视而不为另一个表启用。 但是，每个表都是单独进行仪器化的。
