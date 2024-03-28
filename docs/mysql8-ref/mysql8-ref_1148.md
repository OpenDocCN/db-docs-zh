# 17.1.4 使用 InnoDB 进行测试和基准测试

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-benchmarking.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-benchmarking.html)

如果 `InnoDB` 不是默认存储引擎，您可以通过在命令行上定义 `--default-storage-engine=InnoDB` 或在 MySQL 服务器选项文件的 `[mysqld]` 部分定义 `default-storage-engine=innodb` 来重新启动服务器，以确定数据库服务器和应用程序是否正确使用 `InnoDB`。

由于更改默认存储引擎仅影响新创建的表，运行应用程序安装和设置步骤以确认一切安装正确，然后运行应用程序功能以确保数据加载、编辑和查询功能正常工作。如果表依赖于特定于另一个存储引擎的功能，您将收到错误信息。在这种情况下，将 `ENGINE=*`other_engine_name`*` 子句添加到 `CREATE TABLE` 语句中以避免错误。

如果您没有对存储引擎做出明确决定，并且想要预览使用 `InnoDB` 创建某些表时的工作情况，为每个表发出命令 `ALTER TABLE table_name ENGINE=InnoDB;`。或者，为了在不干扰原始表的情况下运行测试查询和其他语句，制作一份副本：

```sql
CREATE TABLE ... ENGINE=InnoDB AS SELECT * FROM *other_engine_table*;
```

为了在真实工作负载下评估完整应用程序的性能，请安装最新的 MySQL 服务器并运行基准测试。

测试完整的应用程序生命周期，从安装、重度使用到服务器重启。在数据库繁忙时杀死服务器进程以模拟断电，验证在重新启动服务器时数据是否成功恢复。

测试任何复制配置，特别是如果您在源服务器和副本上使用不同的 MySQL 版本和选项。
