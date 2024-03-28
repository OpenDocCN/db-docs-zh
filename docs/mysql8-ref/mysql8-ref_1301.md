# 18.2.1 MyISAM 启动选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-start.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-start.html)

下列选项可用于更改`MyISAM`表的行为。有关更多信息，请参见**mysqld**的第 7.1.7 节，“服务器命令选项”。

**表 18.3 MyISAM 选项和变量参考**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| bulk_insert_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| concurrent_insert | 是 | 是 | 是 |  | 全局 | 是 |
| delay_key_write | 是 | 是 | 是 |  | 全局 | 是 |
| have_rtree_keys |  |  | 是 |  | 全局 | 否 |
| key_buffer_size | 是 | 是 | 是 |  | 全局 | 是 |
| log-isam | 是 | 是 |  |  |  |  |
| myisam-block-size | 是 | 是 |  |  |  |  |
| myisam_data_pointer_size | 是 | 是 | 是 |  | 全局 | 是 |
| myisam_max_sort_file_size | 是 | 是 | 是 |  | 全局 | 是 |
| myisam_mmap_size | 是 | 是 | 是 |  | 全局 | 否 |
| myisam_recover_options | 是 | 是 | 是 |  | 全局 | 否 |
| myisam_repair_threads | 是 | 是 | 是 |  | 两者 | 是 |
| myisam_sort_buffer_size | 是 | 是 | 是 |  | 两者 | 是 |
| myisam_stats_method | 是 | 是 | 是 |  | 两者 | 是 |
| myisam_use_mmap | 是 | 是 | 是 |  | 全局 | 是 |
| tmp_table_size | 是 | 是 | 是 |  | 两者 | 是 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |

下列系统变量影响`MyISAM`表的行为。有关更多信息，请参见第 7.1.8 节，“服务器系统变量”。 

+   `bulk_insert_buffer_size`

    用于批量插入优化中使用的树缓存大小。

    注

    这是每个线程的限制！

+   `delay_key_write=ALL`

    在任何`MyISAM`表的写入之间不要刷新键缓冲区。

    注

    如果这样做，您不应该从另一个程序（例如从另一个 MySQL 服务器或使用**myisamchk**）访问`MyISAM`表格，当表格正在使用时。这样做会导致索引损坏。使用`--external-locking`并不能消除这种风险。

+   `myisam_max_sort_file_size`

    MySQL 在重新创建`MyISAM`索引时允许使用的临时文件的最大大小（在`REPAIR TABLE`、`ALTER TABLE`或`LOAD DATA`期间）。如果文件大小大于此值，则使用键缓存而不是创建索引，这会更慢。该值以字节为单位给出。

+   `myisam_recover_options=*`mode`*`

    设置自动恢复崩溃的`MyISAM`表的模式。

+   `myisam_sort_buffer_size`

    设置在恢复表格时使用的缓冲区的大小。

如果您使用设置了`myisam_recover_options`系统变量的值启动**mysqld**，则自动恢复将被激活。在这种情况下，当服务器打开`MyISAM`表时，它会检查表是否标记为崩溃，或者表的打开计数变量不为 0 且您正在以禁用外部锁定的方式运行服务器。如果这两个条件中的任何一个为真，则会发生以下情况：

+   服务器检查表格是否有错误。

+   如果服务器发现错误，则尝试进行快速表修复（带排序但不重新创建数据文件）。

+   如果由于数据文件中的错误（例如重复键错误）而导致修复失败，则服务器会再次尝试，这次重新创建数据文��。

+   如果修复仍然失败，服务器会再次尝试使用旧的修复选项方法（逐行写入而不排序）。这种方法应该能够修复任何类型的错误，并且对磁盘空间要求低。

如果恢复无法从先前完成的语句中恢复所有行，并且您没有在`myisam_recover_options`系统变量的值中指定`FORCE`，则自动修复将在错误日志中中止并显示错误消息：

```sql
Error: Couldn't repair table: test.g00pages
```

如果您指定了`FORCE`，则会写入类似于以下的警告：

```sql
Warning: Found 344 of 354 rows when repairing ./test/g00pages
```

如果自动恢复值包括`BACKUP`，则恢复过程会创建形如`*`tbl_name-datetime`*.BAK`的文件。您应该有一个**cron**脚本，自动将这些文件从数据库目录移动到备份介质。
