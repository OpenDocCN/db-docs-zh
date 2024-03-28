# 18.5 ARCHIVE 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/archive-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/archive-storage-engine.html)

`ARCHIVE` 存储引擎生成专用表，以非常小的占用空间存储大量未索引的数据。

**表 18.5 ARCHIVE 存储引擎特性**

| 功能 | 支持 |
| --- | --- |
| **B-树索引** | 否 |
| **备份/时间点恢复**（在服务器中实现，而不是在存储引擎中。） | 是 |
| **集群数据库支持** | 否 |
| **集群索引** | 否 |
| **压缩数据** | 是 |
| **数据缓存** | 否 |
| **加密数据** | 是（通过加密函数在服务器中实现。） |
| **外键支持** | 否 |
| **全文搜索索引** | 否 |
| **地理空间数据类型支持** | 是 |
| **地理空间索引支持** | 否 |
| **哈希索引** | 否 |
| **索引缓存** | 否 |
| **锁定粒度** | 行 |
| **MVCC** | 否 |
| **复制支持**（在服务器中实现，而不是在存储引擎中。） | 是 |
| **存储限制** | 无 |
| **T-树索引** | 否 |
| **事务** | 否 |
| **数据字典的更新统计信息** | 是 |
| 功能 | 支持 |

`ARCHIVE` 存储引擎包含在 MySQL 二进制发行版中。如果您从源代码构建 MySQL，可以通过使用 `-DWITH_ARCHIVE_STORAGE_ENGINE` 选项在 **CMake** 中启用此存储引擎。

要查看 `ARCHIVE` 引擎的源代码，请查看 MySQL 源代码分发中的 `storage/archive` 目录。

您可以使用 `SHOW ENGINES` 语句检查 `ARCHIVE` 存储引擎是否可用。

创建 `ARCHIVE` 表时，存储引擎会创建以表名开头的文件。数据文件的扩展名为 `.ARZ`。在优化操作期间可能会出现 `.ARN` 文件。

`ARCHIVE` 引擎支持 `INSERT`, `REPLACE`, 和 `SELECT`，但不支持 `DELETE` 或 `UPDATE`。它支持 `ORDER BY` 操作，`BLOB` 列，以及空间数据类型（参见 第 13.4.1 节，“空间数据类型”）。不支持地理空间参考系统。`ARCHIVE` 引擎使用行级锁定。

`ARCHIVE` 引擎支持 `AUTO_INCREMENT` 列属性。`AUTO_INCREMENT` 列可以有唯一或非唯一索引。在任何其他列上创建索引会导致错误。`ARCHIVE` 引擎还支持 `CREATE TABLE` 语句中的 `AUTO_INCREMENT` 表选项，用于指定新表的初始序列值或重置现有表的序列值。

`ARCHIVE` 不支持将值插入到小于当前最大列值的 `AUTO_INCREMENT` 列中。尝试这样做会导致 `ER_DUP_KEY` 错误。

如果未请求，`ARCHIVE` 引擎会忽略 `BLOB` 列，并在读取时跳过它们。

`ARCHIVE` 存储引擎不支持分区。

**存储：** 行在插入时会被压缩。`ARCHIVE` 引擎使用 `zlib` 无损数据压缩（参见 [`www.zlib.net/`](http://www.zlib.net/)）。您可以使用 `OPTIMIZE TABLE` 来分析表并将其打包成更小的格式（关于使用 `OPTIMIZE TABLE` 的原因，请参见本节后面）。该引擎还支持 `CHECK TABLE`。有几种类型的插入操作被使用：

+   `INSERT` 语句只是将行推送到压缩缓冲区中，并在必要时刷新该缓冲区。对缓冲区的插入受到锁的保护。`SELECT` 强制刷新发生。

+   仅在批量插入完成后才能看到，除非同时发生其他插入操作，此时可能部分可见。`SELECT` 永远不会导致批量插入的刷新，除非在加载时发生正常插入。

**检索**：在检索时，行会按需解压缩；没有行缓存。`SELECT` 操作执行完整的表扫描：当发生 `SELECT` 时，它会找出当前有多少行并读取这些行。`SELECT` 作为一致性读取执行。请注意，插入期间大量的 `SELECT` 语句可能会降低压缩效果，除非只使用批量插入。为了获得更好的压缩效果，您可以使用 `OPTIMIZE TABLE` 或 `REPAIR TABLE`。`SHOW TABLE STATUS` 报告的 `ARCHIVE` 表中的行数始终是准确的。请参阅 Section 15.7.3.4, “OPTIMIZE TABLE Statement”，Section 15.7.3.5, “REPAIR TABLE Statement” 和 Section 15.7.7.38, “SHOW TABLE STATUS Statement”。

### 其他资源

+   专门针对 `ARCHIVE` 存储引擎的论坛可在 [`forums.mysql.com/list.php?112`](https://forums.mysql.com/list.php?112) 找到。
