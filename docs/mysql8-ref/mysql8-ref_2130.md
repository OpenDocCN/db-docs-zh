# 30.3 sys Schema 进度报告

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-progress-reporting.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-progress-reporting.html)

以下`sys`模式视图为长时间运行的事务提供进度报告：

```sql
processlist
session
x$processlist
x$session
```

假设所需的仪器和消费者已启用，则这些视图的`progress`列显示支持进度报告的阶段已完成工作的百分比。

阶段进度报告要求启用`events_stages_current`消费者，以及需要进度信息的仪器。目前支持进度报告的阶段的仪器有：

```sql
stage/sql/Copying to tmp table
stage/innodb/alter table (end)
stage/innodb/alter table (flush)
stage/innodb/alter table (insert)
stage/innodb/alter table (log apply index)
stage/innodb/alter table (log apply table)
stage/innodb/alter table (merge sort)
stage/innodb/alter table (read PK and internal sort)
stage/innodb/buffer pool load
```

对于不支持估计和已完成工作报告的阶段，或者如果所需的仪器或消费者未启用，则`progress`列为`NULL`。
