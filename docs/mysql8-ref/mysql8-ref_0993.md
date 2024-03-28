> 原文：[`dev.mysql.com/doc/refman/8.0/en/change-replication-filter.html`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-filter.html)

#### 15.4.2.2 CHANGE REPLICATION FILTER Statement

```sql
CHANGE REPLICATION FILTER *filter*[, *filter*]
	[, ...] [FOR CHANNEL *channel*]

*filter*: {
    REPLICATE_DO_DB = (*db_list*)
  | REPLICATE_IGNORE_DB = (*db_list*)
  | REPLICATE_DO_TABLE = (*tbl_list*)
  | REPLICATE_IGNORE_TABLE = (*tbl_list*)
  | REPLICATE_WILD_DO_TABLE = (*wild_tbl_list*)
  | REPLICATE_WILD_IGNORE_TABLE = (*wild_tbl_list*)
  | REPLICATE_REWRITE_DB = (*db_pair_list*)
}

*db_list*:
    *db_name*[, *db_name*][, ...]

*tbl_list*:
    *db_name.table_name*[, *db_name.table_name*][, ...]
*wild_tbl_list*:
    '*db_pattern.table_pattern*'[, '*db_pattern.table_pattern*'][, ...]

*db_pair_list*:
    (*db_pair*)[, (*db_pair*)][, ...]

*db_pair*:
    *from_db*, *to_db*
```

`CHANGE REPLICATION FILTER`设置一个或多个复制过滤规则，与使用复制过滤选项（如`--replicate-do-db`或`--replicate-wild-ignore-table`的方式相同。使用此语句设置的过滤器与使用服务器选项设置的过滤器在两个关键方面有所不同：

1.  该语句不需要重新启动服务器才能生效，只需首先停止复制 SQL 线程使用`STOP REPLICA SQL_THREAD`（然后使用`START REPLICA SQL_THREAD`重新启动）。

1.  该语句的效果不是持久的；使用`CHANGE REPLICATION FILTER`设置的任何过滤器在重启复制**mysqld**后会丢失。

`CHANGE REPLICATION FILTER`需要`REPLICATION_SLAVE_ADMIN`权限（或已弃用的`SUPER`权限）。

使用`FOR CHANNEL *`channel`*`子句使复制过滤器特定于复制通道，例如在多源副本上。未附加特定`FOR CHANNEL`子句应用的过滤器被视为全局过滤器，这意味着它们适用于所有复制通道。

注意

不能在配置为组复制的 MySQL 服务器实例上设置全局复制过滤器，因为在某些服务器上过滤事务会使组无法达成一致状态。可以在未直接涉及组复制的复制通道上设置特定通道的复制过滤器，例如，其中一个组成员同时充当到组外源的副本。不能在`group_replication_applier`或`group_replication_recovery`通道上设置。

以下列表显示了`CHANGE REPLICATION FILTER`选项及其与`--replicate-*`服务器选项的关系：

+   `REPLICATE_DO_DB`: 根据数据库名称包含更新。等同于`--replicate-do-db`。

+   `REPLICATE_IGNORE_DB`: 根据数据库名称排除更新。等同于`--replicate-ignore-db`。

+   `REPLICATE_DO_TABLE`: 根据表名包含更新。等同于`--replicate-do-table`。

+   `REPLICATE_IGNORE_TABLE`: 根据表名排除更新。等同于`--replicate-ignore-table`。

+   `REPLICATE_WILD_DO_TABLE`: 根据通配符模式匹配表名包含更新。等同于`--replicate-wild-do-table`。

+   `REPLICATE_WILD_IGNORE_TABLE`: 根据通配符模式匹配表名排除更新。等同于`--replicate-wild-ignore-table`。

+   `REPLICATE_REWRITE_DB`: 在源端替换指定数据库的名称后，在副本上执行更新。等同于`--replicate-rewrite-db`。

`REPLICATE_DO_DB`和`REPLICATE_IGNORE_DB`过滤器的确切效果取决于是基于语句还是基于行的复制。有关更多信息，请参见第 19.2.5 节，“服务器如何评估复制过滤规则”。

可以通过在单个`CHANGE REPLICATION FILTER`语句中用逗号分隔规则来创建多个复制过滤规则，如下所示：

```sql
CHANGE REPLICATION FILTER
    REPLICATE_DO_DB = (d1), REPLICATE_IGNORE_DB = (d2);
```

发出刚才显示的语句等同于使用选项`--replicate-do-db=d1` `--replicate-ignore-db=d2`启动副本**mysqld**。

在使用多个复制通道处理来自不同源的事务的多源复制中，使用`FOR CHANNEL *`channel`*`子句在复制通道上设置复制过滤器：

```sql
CHANGE REPLICATION FILTER REPLICATE_DO_DB = (d1) FOR CHANNEL channel_1;
```

这使您可以创建特定通道的复制过滤器，以从源中筛选出选定的数据。提供`FOR CHANNEL`子句时，复制过滤器语句将作用于该复制通道，删除具有与指定复制过滤器相同过滤器类型的任何现有复制过滤器，并用指定的过滤器替换它们。未在语句中明确列出的过滤器类型不会被修改。如果针对未配置的复制通道发出该语句，则该语句将失败，并显示 ER_SLAVE_CONFIGURATION 错误。如果针对 Group Replication 通道发出该语句，则该语句将失败，并显示 ER_SLAVE_CHANNEL_OPERATION_NOT_ALLOWED 错误。

在配置了多个复制通道的副本中，发出不带`FOR CHANNEL`子句的`CHANGE REPLICATION FILTER`会为每个配置的复制通道以及全局复制过滤器配置复制过滤器。对于每种过滤器类型，如果语句中列出了过滤器类型，则该类型的任何现有过滤规则都将被最近发出的语句中指定的过滤规则替换，否则过滤器类型的旧值将被保留。有关更多信息，请参见 Section 19.2.5.4, “Replication Channel Based Filters”。

如果同一过滤规则被指定多次，则实际上只使用*最后*一条规则。例如，这里显示的两个语句具有完全相同的效果，因为第一个语句中的第一条`REPLICATE_DO_DB`规则被忽略：

```sql
CHANGE REPLICATION FILTER
    REPLICATE_DO_DB = (db1, db2), REPLICATE_DO_DB = (db3, db4);

CHANGE REPLICATION FILTER
    REPLICATE_DO_DB = (db3, db4);
```

注意

这种行为与`--replicate-*`过滤选项的行为不同，其中多次指定相同选项会导致创建多个过滤规则。

不包含任何特殊字符的表和数据库名称无需加引号。与`REPLICATION_WILD_TABLE`和`REPLICATION_WILD_IGNORE_TABLE`一起使用的值是字符串表达式，可能包含（特殊）通配符字符，因此必须加引号。以下示例语句中显示了这一点：

```sql
CHANGE REPLICATION FILTER
    REPLICATE_WILD_DO_TABLE = ('db1.old%');

CHANGE REPLICATION FILTER
    REPLICATE_WILD_IGNORE_TABLE = ('db1.new%', 'db2.new%');
```

与`REPLICATE_REWRITE_DB`一起使用的值表示数据库名称的*对*；每个这样的值必须用括号括起来。以下语句将源数据库`db1`上发生的语句重写为副本数据库`db2`：

```sql
CHANGE REPLICATION FILTER REPLICATE_REWRITE_DB = ((db1, db2));
```

刚刚显示的语句包含两组括号，一组括住数据库名称的对，另一组括住整个列表。这在下面的示例中可能更容易看到，该示例创建了两个`rewrite-db`规则，一个将数据库`dbA`重写为`dbB`，另一个将数据库`dbC`重写为`dbD`：

```sql
CHANGE REPLICATION FILTER
  REPLICATE_REWRITE_DB = ((dbA, dbB), (dbC, dbD));
```

`CHANGE REPLICATION FILTER`语句仅替换受语句影响的过滤类型和复制通道的复制过滤规则，而保持其他规则和通道不变。如果要取消给定类型的所有过滤器，请将过滤器的值设置为显式空列表，如此示例所示，它删除了所有现有的`REPLICATE_DO_DB`和`REPLICATE_IGNORE_DB`规则：

```sql
CHANGE REPLICATION FILTER
    REPLICATE_DO_DB = (), REPLICATE_IGNORE_DB = ();
```

以这种方式将过滤器设置为空会删除所有现有规则，不会创建任何新规则，并且不会恢复在启动时使用`--replicate-*`选项在命令行或配置文件中设置的任何规则。

`RESET REPLICA ALL`语句会移除在被语句删除的通道上设置的特定通道复制过滤器。当被删除的通道或通道重新创建时，任何为副本指定的全局复制过滤器都会被复制到它们，而不会应用任何特定通道的复制过滤器。

欲了解更多信息，请参阅第 19.2.5 节，“服务器如何评估复制过滤规则”。
