> 原文：[`dev.mysql.com/doc/refman/8.0/en/repair-table.html`](https://dev.mysql.com/doc/refman/8.0/en/repair-table.html)

#### 15.7.3.5 修复表语句

```sql
REPAIR [NO_WRITE_TO_BINLOG | LOCAL]
    TABLE *tbl_name* [, *tbl_name*] ...
    [QUICK] [EXTENDED] [USE_FRM]
```

`REPAIR TABLE` 修复可能损坏的表，仅适用于某些存储引擎。

此语句需要表的`SELECT`和`INSERT`权限。

虽然通常情况下您不应该经常运行`REPAIR TABLE`，但如果发生灾难，这个语句很可能从`MyISAM`表中恢复所有数据。如果您的表经常损坏，请尝试找出原因，以消除使用`REPAIR TABLE`的必要性。参见第 B.3.3.3 节，“如果 MySQL 经常崩溃怎么办”，以及第 18.2.4 节，“MyISAM 表问题”。

`REPAIR TABLE` 检查表以查看是否需要升级。如果需要，它执行升级，遵循与`CHECK TABLE ... FOR UPGRADE`相同的规则。有关更多信息，请参见第 15.7.3.2 节，���检查表语句”。

重要提示

+   在执行表修复操作之前备份表；在某些情况下，该操作可能导致数据丢失。可能的原因包括但不限于文件系统错误。请参见第九章，“备份和恢复”。

+   如果服务器在`REPAIR TABLE`操作期间退出，在重新启动后，立即执行另一个`REPAIR TABLE`语句对该表进行修复是至关重要的，然后再对其执行其他操作。在最坏的情况下，您可能会得到一个没有关于数据文件信息的新干净索引文件，然后您执行的下一个操作可能会覆盖数据文件。这是一个不太可能但可能发生的情况，强调了首先进行备份的价值。

+   如果源上的表损坏并且您在其上运行`REPAIR TABLE`，则对原始表的任何更改*不会*传播到副本。

+   修复表存储引擎和分区支持

+   修复表选项

+   修复表输出

+   表修复注意事项

##### 修复表存储引擎和分区支持

`REPAIR TABLE`适用于`MyISAM`、`ARCHIVE`和`CSV`表。对于`MyISAM`表，默认情况下具有与**myisamchk --recover *`tbl_name`***相同的效果。此语句不适用于视图。

`REPAIR TABLE`支持分区表。但是，在分区表上不能使用`USE_FRM`选项。

您可以使用`ALTER TABLE ... REPAIR PARTITION`来修复一个或多个分区；有关更多信息，请参见第 15.1.9 节，“ALTER TABLE 语句”和第 26.3.4 节，“分区维护”。

##### 修复表选项

+   `NO_WRITE_TO_BINLOG`或`LOCAL`

    默认情况下，服务器将`REPAIR TABLE`语句写入二进制日志，以便它们复制到副本。要禁止记录日志，请指定可选的`NO_WRITE_TO_BINLOG`关键字或其别名`LOCAL`。

+   `QUICK`

    如果使用`QUICK`选项，`REPAIR TABLE`尝试仅修复索引文件，而不是数据文件。这种类型的修复类似于**myisamchk --recover --quick**所做的操作。

+   `EXTENDED`

    如果使用`EXTENDED`选项，MySQL 会逐行创建索引行，而不是一次性创建一个索引并进行排序。这种类型的修复类似于**myisamchk --safe-recover**所做的操作。

+   `USE_FRM`

    如果`.MYI`索引文件丢失或其头部损坏，可以使用`USE_FRM`选项。此选项告诉 MySQL 不要信任`.MYI`文件头中的信息，并使用数据字典中的信息重新创建它。这种修复无法使用**myisamchk**进行。

    注意

    仅在无法使用常规`REPAIR`模式时才使用`USE_FRM`选项。告诉服务器忽略`.MYI`文件会使存储在`.MYI`中的重要表元数据对修复过程不可用，这可能会产生有害后果：

    +   当前的`AUTO_INCREMENT`值丢失了。

    +   表中已删除记录的链接丢失了，这意味着删除记录后的空闲空间仍然未被占用。

    +   `.MYI` 头部指示表是否被压缩。如果服务器忽略这些信息，它就无法知道表是否被压缩，修复可能会导致表内容的更改或丢失。这意味着不应该在压缩表上使用 `USE_FRM`。无论如何，这是不必要的：压缩表是只读的，因此它们不应该变得损坏。

    如果您对由当前运行的 MySQL 服务器的不同版本创建的表使用 `USE_FRM`，`REPAIR TABLE` 不会尝试修复表。在这种情况下，`REPAIR TABLE` 返回的结果集包含一个 `Msg_type` 值为 `error` 和 `Msg_text` 值为 `Failed repairing incompatible .FRM file` 的行。

    如果使用 `USE_FRM`，`REPAIR TABLE` 不会检查表以查看是否需要升级。

##### 修复表输出

`REPAIR TABLE` 返回一个包含以下表中列的结果集。

| 列 | 值 |
| --- | --- |
| `Table` | 表名 |
| `Op` | 始终为 `repair` |
| `Msg_type` | `status`、`error`、`info`、`note` 或 `warning` |
| `Msg_text` | 一个信息性消息 |

`REPAIR TABLE` 语句可能为每个修复的表产生许多行信息。最后一行的 `Msg_type` 值为 `status`，`Msg_test` 通常应为 `OK`。对于 `MyISAM` 表，如果没有得到 `OK`，应尝试使用 **myisamchk --safe-recover** 进行修复。(`REPAIR TABLE` 没有实现所有 **myisamchk** 的选项。使用 **myisamchk --safe-recover**，您还可以使用 `--max-record-length` 等 `REPAIR TABLE` 不支持的选项。)

`REPAIR TABLE` 表捕获并抛出在从旧损坏文件复制表统计信息到新创建文件时发生的任何错误。例如，如果 `.MYD` 或 `.MYI` 文件的所有者的用户 ID 与 **mysqld** 进程的用户 ID 不同，`REPAIR TABLE` 会生成一个“无法更改文件所有权”的错误，除非 **mysqld** 是由 `root` 用户启动的。

##### 表修复考虑事项

`修复表` 会升级表格，如果它包含旧的时间列，格式为 5.6.4 之前的格式（`TIME`，`DATETIME` 和 `TIMESTAMP` 列，不支持分数秒精度），并且 `avoid_temporal_upgrade` 系统变量被禁用。如果 `avoid_temporal_upgrade` 被启用，`修复表` 会忽略表中存在的旧时间列，并且不会升级它们。

要升级包含这些时间列的表格，请在执行 `修复表` 前禁用 `avoid_temporal_upgrade`。

通过设置特定的系统变量，您可以提高 `修复表` 的性能。请参阅 第 10.6.3 节，“优化修复表语句”。
