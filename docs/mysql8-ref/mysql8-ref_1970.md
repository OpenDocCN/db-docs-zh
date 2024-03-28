# 28.4.24 `INFORMATION_SCHEMA INNODB_TABLESPACES` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tablespaces-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tablespaces-table.html)

`INNODB_TABLESPACES` 表提供关于 `InnoDB` 文件表空间、通用表空间和撤销表空间的元数据。

有关相关用法信息和示例，请参见 第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。

注意

`INFORMATION_SCHEMA` `FILES` 表报告了 `InnoDB` 表空间类型的元数据，包括文件表空间、通用表空间、系统表空间、全局临时表空间和撤销表空间。

`INNODB_TABLESPACES` 表具有以下列：

+   `SPACE`

    表空间 ID。

+   `NAME`

    模式（数据库）和表名。

+   `FLAG`

    代表表空间格式和存储特性的位级信息的数值。

+   `ROW_FORMAT`

    表空间行格式（`Compact or Redundant`，`Dynamic` 或 `Compressed`，或 `Undo`）。此列中的数据是从数据文件中的表空间标志信息解释而来。

    无法从此标志信息确定表空间行格式是 `Redundant` 还是 `Compact`，这就是为什么可能的 `ROW_FORMAT` 值之一是 `Compact or Redundant`。

+   `PAGE_SIZE`

    表空间页大小。此列中的数据是从 `.ibd` 文件 中的表空间标志信息解释而来。

+   `ZIP_PAGE_SIZE`

    表空间 zip 页大小。此列中的数据是从 `.ibd` 文件 中的表空间标志信息解释而来。

+   `SPACE_TYPE`

    表空间类型。可能的值包括通用表空间的 `General`，文件表空间的 `Single`，系统表空间的 `System`，以及撤销表空间的 `Undo`。

+   `FS_BLOCK_SIZE`

    文件系统块大小，用于空洞打孔的单位大小。此列与 `InnoDB` 透明页压缩 功能相关。

+   `FILE_SIZE`

    文件的表面大小，表示文件的最大未压缩大小。此列与 `InnoDB` 透明页压缩 功能相关。

+   `ALLOCATED_SIZE`

    文件的实际大小，即在磁盘上分配的空间。此列与`InnoDB`的透明页压缩功能有关。

+   `AUTOEXTEND_SIZE`

    表空间的自动扩展大小。此列在 MySQL 8.0.23 中添加。

+   `SERVER_VERSION`

    创建表空间的 MySQL 版本，或导入表空间的 MySQL 版本，或最后一次主要 MySQL 版本升级的版本。该值不会受到发行系列升级的影响，例如从 MySQL 8.0.*`x`*升级到 8.0.*`y`*。该值可以被视为表空间的“创建”标记或“认证”标记。

+   `SPACE_VERSION`

    表空间版本，用于跟踪表空间格式的更改。

+   `ENCRYPTION`

    表空间是否加密。此列在 MySQL 8.0.13 中添加。

+   `STATE`

    表空间状态。此列在 MySQL 8.0.14 中添加。

    对于每个表和通用表空间，状态包括：

    +   `normal`：表空间正常且活动。

    +   `discarded`：表空间被`ALTER TABLE ... DISCARD TABLESPACE`语句丢弃。

    +   `corrupted`：表空间被`InnoDB`标识为损坏。

    对于撤销表空间，状态包括：

    +   `active`：撤销表空间中的回滚段可以分配给新事务。

    +   `inactive`：撤销表空间中的回滚段不再被新事务使用。截断过程正在进行中。撤销表空间要么被清理线程隐式选择，要么通过`ALTER UNDO TABLESPACE ... SET INACTIVE`语句被设置为不活动。

    +   `empty`：撤销表空间被截断，不再活动。可以通过`ALTER UNDO TABLESPACE ... SET INACTIVE`语句将其删除或重新激活。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE SPACE = 26\G
*************************** 1\. row ***************************
         SPACE: 26
          NAME: test/t1
          FLAG: 0
    ROW_FORMAT: Compact or Redundant
     PAGE_SIZE: 16384
 ZIP_PAGE_SIZE: 0
    SPACE_TYPE: Single
 FS_BLOCK_SIZE: 4096
     FILE_SIZE: 98304
ALLOCATED_SIZE: 65536
AUTOEXTEND_SIZE: 0
SERVER_VERSION: 8.0.23
 SPACE_VERSION: 1
    ENCRYPTION: N
         STATE: normal
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA`的`COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
