> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-objects-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-objects-table.html)

#### 29.12.2.4 `setup_objects`表

`setup_objects`表控制着性能模式监视特定对象的行为。默认情况下，该表的最大大小为 100 行。要更改表的大小，请在服务器启动时修改`performance_schema_setup_objects_size`系统变量。

初始的`setup_objects`内容如下：

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

对于`setup_objects`中列出的对象类型，性能模式使用该表来监视它们。对象匹配基于`OBJECT_SCHEMA`和`OBJECT_NAME`列。没有匹配的对象不会被监视。

默认对象配置的效果是对除了`mysql`、`INFORMATION_SCHEMA`和`performance_schema`数据库之外的所有表进行仪器化。（无论`setup_objects`的内容如何，`INFORMATION_SCHEMA`数据库中的表都不会被仪器化；`information_schema.%`的行只是明确了这一默认设置。）

当性能模式在`setup_objects`中查找匹配时，它首先尝试找到更具体的匹配。例如，对于表`db1.t1`，它会先查找`'db1'`和`'t1'`的匹配，然后是`'db1'`和`'%'`，最后是`'%'`和`'%'`。匹配发生的顺序很重要，因为不同的匹配`setup_objects`行可能具有不同的`ENABLED`和`TIMED`值。

用户具有对表的`INSERT`或`DELETE`权限可以向`setup_objects`中插入或删除行。对于现有行，只有具有对表的`UPDATE`权限的用户可以修改`ENABLED`和`TIMED`列。

有关`setup_objects`表在事件过滤中的作用的更多信息，请参见 Section 29.4.3, “Event Pre-Filtering”。

`setup_objects`表具有以下列：

+   `OBJECT_TYPE`

    要检测的对象类型。值为`'EVENT'`（事件调度器事件）、`'FUNCTION'`（存储函数）、`'PROCEDURE'`（存储过程）、`'TABLE'`（基本表）或`'TRIGGER'`（触发器）。

    `TABLE`过滤影响表 I/O 事件（`wait/io/table/sql/handler`工具）和表锁事件（`wait/lock/table/sql/handler`工具）。

+   `OBJECT_SCHEMA`

    包含对象的模式。这应该是一个字面名称，或者`'%'`表示“任何模式”。

+   `OBJECT_NAME`

    被检测对象的名称。这应该是一个字面名称，或者`'%'`表示“任何对象”。

+   `ENABLED`

    对象的事件是否被检测。值为`YES`或`NO`。此列可以修改。

+   `TIMED`

    对象的事件是否被计时。此列可以修改。

`setup_objects`表具有以下索引：

+   在(`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`)上的索引。

允许对`setup_objects`表进行`TRUNCATE TABLE`。它会删除行。
