# 18.2 MyISAM 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html)

18.2.1 MyISAM 启动选项

18.2.2 键所需的空间

18.2.3 MyISAM 表存储格式

18.2.4 MyISAM 表问题

`MyISAM`基于旧的（不再可用）`ISAM`存储引擎，但具有许多有用的扩展。

**表 18.2 MyISAM 存储引擎特性**

| 功能 | 支持 |
| --- | --- |
| **B 树索引** | 是 |
| **备份/时间点恢复**（在服务器中实现，而不是在存储引擎中。） | 是 |
| **集群数据库支持** | 否 |
| **聚集索引** | 否 |
| **压缩数据** | 是（仅在使用压缩行格式时支持压缩的 MyISAM 表。使用压缩行格式和 MyISAM 的表是只读的。） |
| **数据缓存** | 否 |
| **加密数据** | 是（通过加密函数在服务器中实现。） |
| **外键支持** | 否 |
| **全文搜索索引** | 是 |
| **地理空间数据类型支持** | 是 |
| **地理空间索引支持** | 是 |
| **哈希索引** | 否 |
| **索引缓存** | 是 |
| **锁定粒度** | 表 |
| **MVCC** | 否 |
| **复制支持**（在服务器中实现，而不是在存储引擎中。） | 是 |
| **存储限制** | 256TB |
| **T 树索引** | 否 |
| **事务** | 否 |
| **更新数据字典的统计信息** | 是 |
| 功能 | 支持 |

每个`MyISAM`表在磁盘上以两个文件存储。这些文件的名称以表名开头，并具有指示文件类型的扩展名。数据文件具有`.MYD`（`MYData`）扩展名。索引文件具有`.MYI`（`MYIndex`）扩展名。表定义存储在 MySQL 数据字典中。

要明确指定要使用`MyISAM`表，可以使用`ENGINE`表选项指示：

```sql
CREATE TABLE t (i INT) ENGINE = MYISAM;
```

在 MySQL 8.0 中，通常需要使用`ENGINE`来指定`MyISAM`存储引擎，因为`InnoDB`是默认引擎。

您可以使用**mysqlcheck**客户端或**myisamchk**实用程序检查或修复`MyISAM`表。您还可以使用**myisampack**压缩`MyISAM`表，以占用更少的空间。请参阅 Section 6.5.3, “mysqlcheck — A Table Maintenance Program”，Section 6.6.4, “myisamchk — MyISAM Table-Maintenance Utility”和 Section 6.6.6, “myisampack — Generate Compressed, Read-Only MyISAM Tables”。

在 MySQL 8.0 中，`MyISAM`存储引擎不提供分区支持。*在以前的 MySQL 版本中创建的分区`MyISAM`表无法在 MySQL 8.0 中使用*。有关更多信息，请参见第 26.6.2 节，“与存储引擎相关的分区限制”。有关升级这些表以便在 MySQL 8.0 中使用的帮助，请参见第 3.5 节，“MySQL 8.0 中的更改”。

`MyISAM`表具有以下特点：

+   所有数据值都以低字节优先存储。这使得数据与机器和操作系统无关。对于二进制可移植性的唯一要求是机器使用二进制补码有符号整数和 IEEE 浮点格式。这些要求在主流机器中被广泛使用。二进制兼容性可能不适用于有时具有特殊处理器的嵌入式系统。

    以低字节优先存储数据不会带来显著的速度惩罚；表行中的字节通常是不对齐的，读取不对齐字节按顺序比按相反顺序需要更少的处理。此外，服务器中提取列值的代码与其他代码相比并不是时间关键。

+   所有数值键值都以高字节优先存储，以便更好地压缩索引。

+   支持大文件（最多 63 位文件长度）的文件系统和操作系统。

+   `MyISAM`表中最多有(2³²)²（1.844E+19）行。

+   每个`MyISAM`表的最大索引数为 64。

    每个索引的最大列数为 16。

+   最大键长度为 1000 字节。这可以通过更改源代码并重新编译来改变。对于长度超过 250 字节的键，使用比默认的 1024 字节更大的键块大小。

+   当按排序顺序插入行时（例如使用`AUTO_INCREMENT`列时），索引树会分裂，使高节点仅包含一个键。这提高了索引树的空间利用率。

+   每个表支持一个`AUTO_INCREMENT`列的内部处理。`MyISAM`自动更新此列的`INSERT`和`UPDATE`操作。这使得`AUTO_INCREMENT`列更快（至少快 10%）。在删除后，顺序顶部的值不会被重用。（当`AUTO_INCREMENT`列被定义为多列索引的最后一列时，从顺序顶部删除的值会被重用。）`AUTO_INCREMENT`值可以通过`ALTER TABLE`或**myisamchk**重置。

+   在混合删除、更新和插入时，动态大小的行碎片化较少。这是通过自动组合相邻的已删除块和在下一个块被删除时扩展块来完成的。

+   `MyISAM`支持并发插入：如果表在数据文件中间没有空闲块，您可以在其他线程从表中读取数据的同时向其插入新行。空闲块可能是由于删除行或更新动态长度行而导致的，其数据超过当前内容。当所有空闲块都被使用完（填满）时，未来的插入操作再次变得并发。参见 Section 10.11.3, “Concurrent Inserts”。

+   将数据文件和索引文件放在不同的目录中，放在不同的物理设备上，可以通过`DATA DIRECTORY`和`INDEX DIRECTORY`表选项来加快速度，`CREATE TABLE`。参见 Section 15.1.20, “CREATE TABLE Statement”。

+   `BLOB`和`TEXT`列可以建立索引。

+   索引列中允许存在`NULL`值。每个键占用 0 到 1 个字节。

+   每个字符列可以有不同的字符集。参见 Chapter 12, *Character Sets, Collations, Unicode*。

+   `MyISAM`索引文件中有一个标志，指示表是否正确关闭。如果使用`myisam_recover_options`系统变量启动**mysqld**，则在打开时会自动检查`MyISAM`表，并在表未正确关闭时进行修复。

+   **myisamchk**如果使用`--update-state`选项运行，则会标记表为已检查。**myisamchk --fast**仅检查那些没有此标记的表。

+   **myisamchk --analyze**存储部分键的统计信息，以及整个键的统计信息。

+   **myisampack**可以压缩`BLOB`和`VARCHAR`列。

`MyISAM`还支持以下功能：

+   支持真正的`VARCHAR`类型；`VARCHAR`列以一个或两个字节存储的长度开始。

+   带有`VARCHAR`列的表可能具有固定或动态行长度。

+   表中`VARCHAR`和`CHAR`列的长度之和可以达到 64KB。

+   任意长度的`UNIQUE`约束。

### 附加资源

+   专门针对`MyISAM`存储引擎的论坛可在[`forums.mysql.com/list.php?21`](https://forums.mysql.com/list.php?21)找到。
