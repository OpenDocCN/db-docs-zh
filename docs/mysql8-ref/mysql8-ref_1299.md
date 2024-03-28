# 18.1 设置存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/storage-engine-setting.html`](https://dev.mysql.com/doc/refman/8.0/en/storage-engine-setting.html)

当您创建新表时，可以通过在`CREATE TABLE`语句中添加`ENGINE`表选项来指定要使用的存储引擎：

```sql
-- ENGINE=INNODB not needed unless you have set a different
-- default storage engine.
CREATE TABLE t1 (i INT) ENGINE = INNODB;
-- Simple table definitions can be switched from one to another.
CREATE TABLE t2 (i INT) ENGINE = CSV;
CREATE TABLE t3 (i INT) ENGINE = MEMORY;
```

当您省略`ENGINE`选项时，将使用默认存储引擎。在 MySQL 8.0 中，默认引擎是`InnoDB`。您可以通过使用`--default-storage-engine`服务器启动选项或通过在`my.cnf`配置文件中设置`default-storage-engine`选项来指定默认引擎。

您可以通过设置`default_storage_engine`变量为当前会话设置默认存储引擎：

```sql
SET default_storage_engine=NDBCLUSTER;
```

使用`CREATE TEMPORARY TABLE`创建的`TEMPORARY`表的存储引擎可以通过在启动时或运行时设置`default_tmp_storage_engine`来单独设置，与永久表的引擎不同。

要将表从一种存储引擎转换为另一种，使用指示新引擎的`ALTER TABLE`语句：

```sql
ALTER TABLE t ENGINE = InnoDB;
```

参见第 15.1.20 节，“CREATE TABLE Statement”和第 15.1.9 节，“ALTER TABLE Statement”。

如果您尝试使用未编译或已编译但已停用的存储引擎，MySQL 会使用默认存储引擎创建表。例如，在复制设置中，也许您的源服务器使用`InnoDB`表以获得最大安全性，但副本服务器使用其他存储引擎以换取速度而牺牲耐久性或并发性。

默认情况下，当`CREATE TABLE`或`ALTER TABLE`无法使用默认存储引擎时会生成警告。为了防止混淆和意外行为，如果所需引擎不可用，请启用`NO_ENGINE_SUBSTITUTION` SQL 模式。如果所需引擎不可用，此设置会产生错误而不是警告，并且表不会被创建或更改。参见第 7.1.11 节，“服务器 SQL 模式”。

MySQL 可能会根据存储引擎的不同，将表的索引和数据存储在一个或多个其他文件中。表和列的定义存储在 MySQL 数据字典中。各个存储引擎会为其管理的表创建所需的任何额外文件。如果表名包含特殊字符，则表文件的名称会包含这些字符的编码版本，如 Section 11.2.4，“标识符映射到文件名”中所述。
