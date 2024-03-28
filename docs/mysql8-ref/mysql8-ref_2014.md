> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-actors-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-actors-table.html)

#### 29.12.2.1 设置 _actors 表

`setup_actors`表包含确定是否为新的前台服务器线程（与客户端连接关联的线程）启用监视和历史事件记录的信息。默认情况下，此表的最大大小为 100 行。要更改表大小，请在服务器启动时修改`performance_schema_setup_actors_size`系统变量。

对于每个新的前台线程，性能模式将线程的用户和主机与`setup_actors`表的行进行匹配。如果该表中的行匹配，则使用其`ENABLED`和`HISTORY`列值来设置线程的`threads`行的`INSTRUMENTED`和`HISTORY`列，分别。这使得可以根据主机、用户或帐户（用户和主机组合）有选择地应用仪器化和历史事件记录。如果没有匹配，则线程的`INSTRUMENTED`和`HISTORY`列设置为`NO`。

对于后台线程，没有关联的用户。`INSTRUMENTED`和`HISTORY`默认为`YES`，并且不会查询`setup_actors`。

`setup_actors`表的初始内容匹配任何用户和主机组合，因此默认情况下为所有前台线程启用监视和历史事件收集：

```sql
mysql> SELECT * FROM performance_schema.setup_actors;
+------+------+------+---------+---------+
| HOST | USER | ROLE | ENABLED | HISTORY |
+------+------+------+---------+---------+
| %    | %    | %    | YES     | YES     |
+------+------+------+---------+---------+
```

有关如何使用`setup_actors`表影响事件监视的信息，请参见第 29.4.6 节，“按线程进行预过滤”。

对`setup_actors`表的修改仅影响在修改后创建的前台线程，而不影响现有线程。要影响现有线程，请修改`threads`表行的`INSTRUMENTED`和`HISTORY`列。

`setup_actors`表具有以下列：

+   `主机`

    主机名。这应该是一个字面名称，或者`'%'`表示“任何主机”。

+   `用户`

    用户名。这应该是一个字面名称，或者`'%'`表示“任何用户”。

+   `角色`

    未使用。

+   `启用`

    是否为与行匹配的前台线程启用仪表化。该值为`YES`或`NO`。

+   `HISTORY`

    是否记录与行匹配的前台线程的历史事件。该值为`YES`或`NO`。

`setup_actors`表具有以下索引：

+   主键为(`HOST`, `USER`, `ROLE`)

允许对`setup_actors`表执行`TRUNCATE TABLE`语句。它会删除行。
