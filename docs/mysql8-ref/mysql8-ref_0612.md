> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-key-caches.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-key-caches.html)

#### 10.10.2.2 多个关键缓存

注意

截至 MySQL 8.0，讨论在此处引用多个`MyISAM`关键缓存的复合部分结构化变量语法已被弃用。

关键缓存的共享访问可以提高性能，但并不能完全消除会话之间的争用。它们仍然竞争管理对关键缓存缓冲区访问的控制结构。为了进一步减少关键缓存访问的争用，MySQL 还提供了多个关键缓存。此功能使您能够将不同的表索引分配给不同的关键缓存。

当存在多个关键缓存时，服务器必须知道在处理给定`MyISAM`表的查询时使用哪个缓存。默认情况下，所有`MyISAM`表索引都缓存在默认关键缓存中。要将表索引分配给特定的关键缓存，请使用`CACHE INDEX`语句（请参见 Section 15.7.8.2, “CACHE INDEX Statement”）。例如，以下语句将表`t1`、`t2`和`t3`的索引分配给名为`hot_cache`的关键缓存：

```sql
mysql> CACHE INDEX t1, t2, t3 IN hot_cache;
+---------+--------------------+----------+----------+
| Table   | Op                 | Msg_type | Msg_text |
+---------+--------------------+----------+----------+
| test.t1 | assign_to_keycache | status   | OK       |
| test.t2 | assign_to_keycache | status   | OK       |
| test.t3 | assign_to_keycache | status   | OK       |
+---------+--------------------+----------+----------+
```

在`CACHE INDEX`语句中引用的关键缓存可以通过使用`SET GLOBAL`参数设置语句设置其大小，或通过使用服务器启动选项创建。例如：

```sql
mysql> SET GLOBAL keycache1.key_buffer_size=128*1024;
```

要销毁关键缓存，请将其大小设置为零：

```sql
mysql> SET GLOBAL keycache1.key_buffer_size=0;
```

无法销毁默认关键缓存。任何尝试这样做都将被忽略：

```sql
mysql> SET GLOBAL key_buffer_size = 0;

mysql> SHOW VARIABLES LIKE 'key_buffer_size';
+-----------------+---------+
| Variable_name   | Value   |
+-----------------+---------+
| key_buffer_size | 8384512 |
+-----------------+---------+
```

关键缓存变量是具有名称和组件的结构化系统变量。对于`keycache1.key_buffer_size`，`keycache1`是缓存变量名称，`key_buffer_size`是缓存组件。有关引用结构化关键缓存系统变量的语法描述，请参见 Section 7.1.9.5, “Structured System Variables”。

默认情况下，表索引分配给在服务器启动时创建的主（默认）关键缓存。当关键缓存被销毁时，分配给它的所有索引将重新分配给默认关键缓存。

对于繁忙的服务器，您可以使用涉及三个关键缓存的策略：

+   一个占据分配给所有关键缓存空间 20%的“热”关键缓存。用于经常用于搜索但不更新的表。

+   一个占据分配给所有关键缓存空间 20%的“冷”关键缓存。用于中等大小、经常修改的表，例如临时表。

+   一个占据关键缓存空间 60%的“热”关键缓存。将其作为默认关键缓存，用于默认情况下所有其他表。

使用三个关键缓存是有益的一个原因是，对一个关键缓存结构的访问不会阻塞对其他缓存的访问。访问分配给一个缓存的表的语句不会与访问分配给另一个缓存的表的语句竞争。性能提升也出现在其他方面：

+   热缓存仅用于检索查询，因此其内容永远不会被修改。因此，每当需要从磁盘中拉入一个索引块时，被选中用于替换的缓存块的内容无需首先刷新。

+   对于分配给热缓存的索引，如果没有需要进行索引扫描的查询，那么非叶节点对应的索引 B 树块很可能仍然保留在缓存中。

+   对于临时表最频繁执行的更新操作，当更新的节点在缓存中且无需首先从磁盘读取时，操作速度会快得多。如果临时表的索引大小与冷键缓存的大小相当，那么更新的节点在缓存中的概率非常高。

`CACHE INDEX` 语句建立了表与关键缓存之间的关联，但每次服务器重新启动时该关联会丢失。如果希望每次服务器启动时关联生效，一种实现方法是使用选项文件：包含配置关键缓存的变量设置，以及一个命名为包含要执行的`CACHE INDEX` 语句的文件的 `init_file` 系统变量。例如：

```sql
key_buffer_size = 4G
hot_cache.key_buffer_size = 2G
cold_cache.key_buffer_size = 2G
init_file=/*path*/*to*/*data-directory*/mysqld_init.sql
```

`mysqld_init.sql` 中的语句在每次服务器启动时执行。该文件应该每行包含一个 SQL 语句。以下示例将几个表分配给了`hot_cache`和`cold_cache`：

```sql
CACHE INDEX db1.t1, db1.t2, db2.t3 IN hot_cache
CACHE INDEX db1.t4, db2.t5, db2.t6 IN cold_cache
```
