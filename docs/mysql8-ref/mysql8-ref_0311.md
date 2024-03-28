> 译文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-concurrent-ddl.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-concurrent-ddl.html)

#### 7.6.7.4 克隆和并发 DDL

在 MySQL 8.0.27 之前，在克隆操作期间，包括`TRUNCATE TABLE`在内的捐赠者和接收者 MySQL 服务器实例上的 DDL 操作是不允许的。在选择数据源时应考虑此限制。一种解决方法是使用专用的捐赠者实例，可以在克隆数据时阻止 DDL 操作。

为防止在克隆操作期间进行并发 DDL，捐赠者和接收者上会获取独占的备份锁。`clone_ddl_timeout`变量定义了在捐赠者和接收者上，克隆操作等待备份锁的时间（以秒为单位）。默认设置为 300 秒。如果在指定的时间限制内未获得备份锁，则克隆操作将因错误而失败。

从 MySQL 8.0.27 开始，默认情况下允许在捐赠者上进行并发 DDL。捐赠者上的并发 DDL 支持由`clone_block_ddl`变量控制。可以使用`SET`语句动态启用和禁用捐赠者上的并发 DDL 支持。

```sql
SET GLOBAL clone_block_ddl={OFF|ON}
```

默认设置为`clone_block_ddl=OFF`，允许在捐赠者上进行并发 DDL。

并发 DDL 操作的效果是否被克隆取决于 DDL 操作是否在克隆操作获取动态快照之前完成。

不允许在克隆操作期间进行的 DDL 操作，无论`clone_block_ddl`设置如何，包括：

+   `ALTER TABLE *`tbl_name`* DISCARD TABLESPACE;`

+   `ALTER TABLE *`tbl_name`* IMPORT TABLESPACE;`

+   `ALTER INSTANCE DISABLE INNODB REDO_LOG;`
