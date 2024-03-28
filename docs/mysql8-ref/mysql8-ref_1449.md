> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-memory.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-memory.html)

#### 19.5.1.21 复制和 MEMORY 表

当复制源服务器关闭并重新启动时，其`MEMORY`表会变为空。为了将这种效果复制到副本中，源在启动后第一次使用给定的`MEMORY`表时，会记录一个事件，通知副本该表必须通过向二进制日志写入`DELETE`或（从 MySQL 8.0.22 开始）`TRUNCATE TABLE`语句来清空该表。这个生成的事件在二进制日志中通过注释可识别，如果服务器上使用了 GTID，则会分配一个 GTID。该语句始终以语句格式记录，即使二进制日志格式设置为`ROW`，并且即使服务器设置为`read_only`或`super_read_only`模式，也会写入。请注意，在源重新启动并首次使用表之间的间隔期间，副本仍然具有`MEMORY`表中的过时数据。为了避免这种间隔，当直接查询副本可能返回陈旧数据时，您可以设置`init_file`系统变量以命名一个包含在启动时在源上填充`MEMORY`表的语句的文件。

当副本服务器关闭并重新启动时，其`MEMORY`表会变为空。这会导致副本与源不同步，并可能导致其他故障或导致副本停止：

+   来自源的行格式更新和删除可能会因为`Can't find record in '*`memory_table`*'`而失败。

+   诸如`INSERT INTO ... SELECT FROM *`memory_table`*`之类的语句可能在源和副本上插入不同的行集。

副本还会将一个`DELETE`或（从 MySQL 8.0.22 开始）`TRUNCATE TABLE`语句写入其自己的二进制日志，传递给任何下游副本，导致它们清空自己的`MEMORY`表。

重新启动正在复制`MEMORY`表的副本的安全方法是首先在源上删除或清空所有`MEMORY`表中的行，并等待这些更改复制到副本。然后才能安全地重新启动副本。

在某些情况下可能适用另一种重启方法。当`binlog_format=ROW`时，如果在重新启动复制之前设置`replica_exec_mode=IDEMPOTENT`（从 MySQL 8.0.26 开始）或`slave_exec_mode=IDEMPOTENT`（在 MySQL 8.0.26 之前），则可以防止复制停止。这样可以使复制继续进行，但其`MEMORY`表仍然与源端不同。如果应用逻辑允许`MEMORY`表的内容安全丢失（例如，如果`MEMORY`表用于缓存），那么这是可以接受的。`replica_exec_mode=IDEMPOTENT`或`slave_exec_mode=IDEMPOTENT`对所有表都适用，因此可能会隐藏非`MEMORY`表中的其他复制错误。

（刚刚描述的方法在 NDB Cluster 中不适用，那里的`replica_exec_mode`或`slave_exec_mode`始终为`IDEMPOTENT`，且无法更改。）

`MEMORY`表的大小受`max_heap_table_size`系统变量的值限制，该值不会被复制（参见 Section 19.5.1.39，“复制和变量”）。更改`max_heap_table_size`对于使用`ALTER TABLE ... ENGINE = MEMORY`或`TRUNCATE TABLE`创建或更新的`MEMORY`表会生效，或者对于所有在服务器重新启动后的`MEMORY`表也会生效。如果你在源端增加了此变量的值而在复制端没有这样做，那么源端的表可能会比复制端的表更大，导致在源端成功插入但在复制端出现“表已满”错误。这是一个已知问题（Bug #48666）。在这种情况下，你必须在复制端和源端都设置`max_heap_table_size`的全局值，然后重新启动复制。建议同时重新启动源端和复制端的 MySQL 服务器，以确保新值在它们各自上完全生效。

查看第 18.3 节，“MEMORY 存储引擎”，了解有关`MEMORY`表的更多信息。
