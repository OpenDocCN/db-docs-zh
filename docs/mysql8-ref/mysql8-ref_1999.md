# 29.4.6 通过线程进行预过滤

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-thread-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-thread-filtering.html)

`threads`表包含每个服务器线程的一行。每行包含有关线程的信息，并指示是否为其启用了监视。要使性能模式监视线程，必须满足以下条件：

+   `setup_consumers`表中的`thread_instrumentation`消费者必须为`YES`。

+   `threads.INSTRUMENTED`列必须为`YES`。

+   仅对在`setup_instruments`表中启用的工具产生的线程事件进行监视。

`threads`表还指示每个服务器线程是否执行历史事件记录。这包括等待、阶段、语句和事务事件，并影响到这些表的记录：

```sql
events_waits_history
events_waits_history_long
events_stages_history
events_stages_history_long
events_statements_history
events_statements_history_long
events_transactions_history
events_transactions_history_long
```

要发生历史事件记录，必须满足以下条件：

+   必须启用`setup_consumers`表中适当的与历史相关的消费者。例如，在`events_waits_history`和`events_waits_history_long`表中等待事件记录需要相应的`events_waits_history`和`events_waits_history_long`消费者为`YES`。

+   `threads.HISTORY`列必须为`YES`。

+   仅对在`setup_instruments`表中启用的工具产生的线程事件进行记录。

对于前台线程（由客户端连接产生），`threads`表行中`INSTRUMENTED`和`HISTORY`列的初始值取决于与线程关联的用户帐户是否与`setup_actors`表中的任何行匹配。这些值来自匹配的`setup_actors`表行的`ENABLED`和`HISTORY`列。

对于后台线程，没有关联的用户。`INSTRUMENTED`和`HISTORY`默认为`YES`，不会查询`setup_actors`。

初始`setup_actors`内容如下所示：

```sql
mysql> SELECT * FROM performance_schema.setup_actors;
+------+------+------+---------+---------+
| HOST | USER | ROLE | ENABLED | HISTORY |
+------+------+------+---------+---------+
| %    | %    | %    | YES     | YES     |
+------+------+------+---------+---------+
```

`HOST`和`USER`列应包含文字主机或用户名，或`'%'`以匹配任何名称。

`ENABLED`和`HISTORY`列指示是否为匹配的线程启用仪表化和历史事件记录，受先前描述的其他条件的约束。

当性能模式在`setup_actors`中为每个新前台线程检查匹配时，首先尝试找到更具体的匹配，使用`USER`和`HOST`列（`ROLE`未使用）：

+   具有`USER='*`literal`*'`和`HOST='*`literal`*'`的行。

+   具有`USER='*`literal`*'`和`HOST='%'`的行。

+   具有`USER='%'`和`HOST='*`literal`*'`的行。

+   具有`USER='%'`和`HOST='%'`的行。

匹配顺序很重要，因为不同匹配的`setup_actors`行可以具有不同的`USER`和`HOST`值。这使得可以根据`ENABLED`和`HISTORY`列的值，基于主机、用户或帐户（用户和主机组合）有选择地应用仪表化和历史事件记录：

+   当最佳匹配是`ENABLED=YES`的行时，线程的`INSTRUMENTED`值变为`YES`。当最佳匹配是`HISTORY=YES`的行时，线程的`HISTORY`值变为`YES`。

+   当最佳匹配是`ENABLED=NO`的行时，线程的`INSTRUMENTED`值变为`NO`。当最佳匹配是`HISTORY=NO`的行时，线程的`HISTORY`值变为`NO`。

+   当找不到匹配时，线程的`INSTRUMENTED`和`HISTORY`值变为`NO`。

在`setup_actors`行中，`ENABLED`和`HISTORY`列可以独立设置为`YES`或`NO`。这意味着您可以单独启用仪表化，而不考虑是否收集历史事件。

默认情况下，对所有新前台线程启用监视和历史事件收集，因为`setup_actors`表最初包含`HOST`和`USER`都为`'%'`的行。要执行更有限的匹配，例如仅为某些前台线程启用监视，必须更改此行，因为它匹配任何连接，并添加更具体的`HOST`/`USER`组合的行。

假设您修改`setup_actors`如下：

```sql
UPDATE performance_schema.setup_actors
SET ENABLED = 'NO', HISTORY = 'NO'
WHERE HOST = '%' AND USER = '%';
INSERT INTO performance_schema.setup_actors
(HOST,USER,ROLE,ENABLED,HISTORY)
VALUES('localhost','joe','%','YES','YES');
INSERT INTO performance_schema.setup_actors
(HOST,USER,ROLE,ENABLED,HISTORY)
VALUES('hosta.example.com','joe','%','YES','NO');
INSERT INTO performance_schema.setup_actors
(HOST,USER,ROLE,ENABLED,HISTORY)
VALUES('%','sam','%','NO','YES');
```

`UPDATE`语句更改默认匹配以禁用仪表化和历史事件收集。`INSERT`语句为更具体的匹配添加行。

现在性能模式确定如何设置新连接线程的`INSTRUMENTED`和`HISTORY`值如下：

+   如果`joe`从本地主机连接，则连接与第一行插入的行匹配。线程的`INSTRUMENTED`和`HISTORY`值变为`YES`。

+   如果`joe`从`hosta.example.com`连接，则连接与第二行插入的行匹配。线程的`INSTRUMENTED`值变为`YES`，`HISTORY`值变为`NO`。

+   如果`joe`从任何其他主机连接，则没有匹配。线程的`INSTRUMENTED`和`HISTORY`值变为`NO`。

+   如果`sam`从任何主机连接，则连接与第三行插入的行匹配。线程的`INSTRUMENTED`值变为`NO`，`HISTORY`值变为`YES`。

+   对于任何其他连接，`HOST`和`USER`设置为`'%'`的行匹配。此行现在的`ENABLED`和`HISTORY`设置为`NO`，因此线程的`INSTRUMENTED`和`HISTORY`值变为`NO`。

对`setup_actors`表的修改仅影响在修改后创建的前台线程，而不影响现有线程。要影响现有线程，请修改`threads`表行的`INSTRUMENTED`和`HISTORY`列。
