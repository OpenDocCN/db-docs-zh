# 19.4.4 使用不同源和副本存储引擎的复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-solutions-diffengines.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-diffengines.html)

在复制过程中，原始源上的表和副本上的复制表使用不同的存储引擎类型并不重要。实际上，`default_storage_engine` 系统变量不会被复制。

这在复制过程中提供了许多好处，因为您可以针对不同的复制场景利用不同的引擎类型。例如，在典型的扩展场景中（参见 第 19.4.5 节，“使用复制进行扩展”），您希望在源上使用 `InnoDB` 表以利用事务功能，但在只读数据的副本上使用 `MyISAM`，因为不需要事务支持。在数据记录环境中使用复制时，您可能希望在副本上使用 `Archive` 存储引擎。

在源和副本上配置不同的引擎取决于您如何设置初始复制过程：

+   如果您使用 **mysqldump** 在源上创建数据库快照，您可以编辑转储文件文本以更改每个表使用的引擎类型。

    另一个 **mysqldump** 的替代方法是在使用转储构建副本数据之前禁用不想在副本上使用的引擎类型。例如，您可以在副本上添加 `--skip-federated` 选项来禁用 `FEDERATED` 引擎。如果要创建的表没有特定的引擎存在，MySQL 将使用默认的引擎类型，通常是 `InnoDB`。（这要求未启用 `NO_ENGINE_SUBSTITUTION` SQL 模式。）如果您想以这种方式禁用其他引擎，您可能需要考虑构建一个专用的二进制文件，该文件仅支持您想要的引擎。

+   如果您使用原始数据文件（二进制备份）设置副本，则无法更改初始表格式。相反，在启动副本后，请使用 `ALTER TABLE` 来更改表类型。

+   对于新的源/复制复制设置，如果源上当前没有表，请在创建新表时避免指定引擎类型。

如果您已经在运行复制解决方案，并希望将现有表转换为另一种引擎类型，请按照以下步骤操作：

1.  停止副本运行复制更新：

    ```sql
    mysql> STOP SLAVE;
    Or from MySQL 8.0.22:
    mysql> STOP REPLICA;
    ```

    这样可以在不中断的情况下更改引擎类型。

1.  对每个要更改的表执行`ALTER TABLE ... ENGINE='*`engine_type`*'`。

1.  重新开始复制过程：

    ```sql
    mysql> START SLAVE;
    ```

    或者，从 MySQL 8.0.22 开始：

    ```sql
    mysql> START REPLICA;
    ```

尽管`default_storage_engine`变量不会被复制，但要注意包含引擎规范的`CREATE TABLE`和`ALTER TABLE`语句会正确地被复制到副本中。例如，在`CSV`表的情况下，如果执行以下语句：

```sql
mysql> ALTER TABLE csvtable ENGINE='MyISAM';
```

这个语句会被复制；副本上的表引擎类型会被转换为`InnoDB`，即使你之前已经将副本上的表类型更改为`CSV`之外的其他引擎。如果你想保留源和副本上的引擎差异，创建新表时应当小心使用`default_storage_engine`变量。例如，不要使用：

```sql
mysql> CREATE TABLE tablea (columna int) Engine=MyISAM;
```

使用这种格式：

```sql
mysql> SET default_storage_engine=MyISAM;
mysql> CREATE TABLE tablea (columna int);
```

在复制时，`default_storage_engine`变量会被忽略，`CREATE TABLE`语句会在副本上使用副本的默认引擎执行。
