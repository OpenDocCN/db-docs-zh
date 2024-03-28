# 29.19.2 获取父事件信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-obtaining-parent-events.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-obtaining-parent-events.html)

`data_locks` 表显示持有和请求的数据锁。该表的行具有`THREAD_ID`列，指示拥有锁的会话的线程 ID，以及`EVENT_ID`列，指示导致锁的性能模式事件。(`THREAD_ID`, `EVENT_ID`)值的元组在其他性能模式表中隐式标识父事件：

+   `events_waits_*`xxx`*` 表中的父等待事件

+   `events_stages_*`xxx`*` 表中的父阶段事件

+   `events_statements_*`xxx`*` 表中的父语句事件

+   `events_transactions_current` 表中的父事务事件

要获取有关父事件的详细信息，请将`THREAD_ID`和`EVENT_ID`列与适当的父事件表中同名列进行连接。关系基于嵌套集数据模型，因此连接具有几个子句。给定由`parent`和`child`表示的父表和子表，连接如下所示：

```sql
WHERE
  parent.THREAD_ID = child.THREAD_ID        /* 1 */
  AND parent.EVENT_ID < child.EVENT_ID      /* 2 */
  AND (
    child.EVENT_ID <= parent.END_EVENT_ID   /* 3a */
    OR parent.END_EVENT_ID IS NULL          /* 3b */
  )
```

连接的条件是：

1.  父事件和子事件在同一线程中。

1.  子事件在父事件之后开始，因此其`EVENT_ID`值大于父事件的值。

1.  父事件已经完成或仍在运行。

要查找锁信息，`data_locks` 是包含子事件的表。

`data_locks` 表仅显示现有锁定，因此关于哪个表包含父事件的考虑如下：

+   对于事务，唯一的选择是`events_transactions_current`。如果事务已完成，则可能在事务历史表中，但锁已经消失。

+   对于语句，一切取决于获取锁的语句是否是已经完成的事务中的语句（使用`events_statements_history`）还是语句仍在运行（使用`events_statements_current`）。

+   对于阶段，逻辑与语句类似；使用`events_stages_history`或`events_stages_current`。

+   对于等待，逻辑与语句类似；使用`events_waits_history`或`events_waits_current`。然而，记录的等待太多，导致引起锁定的等待很可能已经从历史表中消失。

等待、阶段和语句事件很快就会从历史中消失。如果很久之前执行的语句获取了锁但仍在打开的事务中，可能无法找到该语句，但可以找到事务。

这就是为什么嵌套集数据模型更适合定位父事件。在父/子关系（数据锁 -> 父等待 -> 父阶段 -> 父事务）中跟随链接时，当中间节点已经从历史表中消失时，效果不佳。

以下场景说明了如何找到在其中获取锁的语句的父事务：

会话 A：

```sql
[1] START TRANSACTION;
[2] SELECT * FROM t1 WHERE pk = 1;
[3] SELECT 'Hello, world';
```

会话 B：

```sql
SELECT ...
FROM performance_schema.events_transactions_current AS parent
  INNER JOIN performance_schema.data_locks AS child
WHERE
  parent.THREAD_ID = child.THREAD_ID
  AND parent.EVENT_ID < child.EVENT_ID
  AND (
    child.EVENT_ID <= parent.END_EVENT_ID
    OR parent.END_EVENT_ID IS NULL
  );
```

会话 B 的查询应该显示语句 [2] 拥有记录中 `pk=1` 的数据锁。

如果会话 A 执行更多语句，[2] 将从历史表中消失。

查询应该显示从 [1] 开始的事务，无论执行了多少语句、阶段或等待。

要查看更多数据，还可以使用`events_*`xxx`*_history_long`表，除了事务，假设服务器中没有运行其他查询（以便保留历史记录）。
