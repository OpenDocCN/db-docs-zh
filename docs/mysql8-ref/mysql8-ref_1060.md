> 原文：[`dev.mysql.com/doc/refman/8.0/en/check-table.html`](https://dev.mysql.com/doc/refman/8.0/en/check-table.html)

#### 15.7.3.2 CHECK TABLE Statement

```sql
CHECK TABLE *tbl_name* [, *tbl_name*] ... [*option*] ...

*option*: {
    FOR UPGRADE
  | QUICK
  | FAST
  | MEDIUM
  | EXTENDED
  | CHANGED
}
```

`CHECK TABLE` 检查一个或多个表中的错误。`CHECK TABLE` 也可以检查视图是否存在问题，例如在视图定义中引用的表已不存在。

要检查一个表，您必须���其具有某些权限。

`CHECK TABLE` 适用于 `InnoDB`, `MyISAM`, `ARCHIVE`, 和 `CSV` 表。

在对 `InnoDB` 表运行 `CHECK TABLE` 之前，请参阅 InnoDB 表的 CHECK TABLE 使用注意事项。

`CHECK TABLE` 支持分区表，并且您可以使用 `ALTER TABLE ... CHECK PARTITION` 来检查一个或多个分区；有关更多信息，请参阅 第 15.1.9 节，“ALTER TABLE 语句” 和 第 26.3.4 节，“分区的维护”。

`CHECK TABLE` 忽略未建立索引的虚拟生成列。

+   检查表输出

+   检查版本兼容性

+   检查数据一致性

+   InnoDB 表的 CHECK TABLE 使用注意事项

+   MyISAM 表的 CHECK TABLE 使用注意事项

##### 检查表输出

`CHECK TABLE` 返回一个结果集，其中包含以下表中显示的列。

| 列 | 值 |
| --- | --- |
| `Table` | 表名 |
| `Op` | 始终为 `check` |
| `Msg_type` | `status`, `error`, `info`, `note`, 或 `warning` |
| `Msg_text` | 一个信息性消息 |

该语句可能为每个检查的表产生许多行信息。最后一行的 `Msg_type` 值为 `status`，`Msg_text` 通常应为 `OK`。 `Table is already up to date` 表示表的存储引擎指示无需检查表。

##### 检查版本兼容性

`FOR UPGRADE`选项检查指定表是否与当前版本的 MySQL 兼容。使用`FOR UPGRADE`，服务器会检查每个表，以确定自创建表以来是否有任何数据类型或索引的不兼容更改。如果没有，则检查成功。否则，如果存在可能的不兼容性，服务器会对表进行全面检查（可能需要一些时间）。

不兼容性可能是因为数据类型的存储格式已更改或其排序顺序已更改。我们的目标是避免这些更改，但偶尔它们是必要的，以纠正比发布之间的不兼容性更糟糕的问题。

`FOR UPGRADE`会发现这些不兼容性：

+   在`InnoDB`和`MyISAM`表中，`TEXT`列的末尾空格索引顺序在 MySQL 4.1 和 5.0 之间发生了变化。

+   新`DECIMAL`数据类型的存储方法在 MySQL 5.0.3 和 5.0.5 之间发生了变化。

+   有时会对字符集或校对规则进行更改，需要重建表索引。有关此类更改的详细信息，请参见第 3.5 节，“MySQL 8.0 中的更改”。有关重建表的信息，请参见第 3.14 节，“重建或修复表或索引”。

+   MySQL 8.0 不支持旧版本 MySQL 中允许的 2 位数`YEAR(2)`数据类型。对于包含`YEAR(2)`列的表，`CHECK TABLE`建议使用`REPAIR TABLE`，将 2 位数`YEAR(2)`列转换为 4 位数`YEAR`列。

+   触发器创建时间保持不变。

+   如果表中包含旧的时间列（不支持分数秒精度的`TIME`、`DATETIME`和`TIMESTAMP`列）且`avoid_temporal_upgrade`系统变量已禁用，则会报告需要重建表。这有助于 MySQL 升级过程检测和升级包含旧时间列的表。如果启用了`avoid_temporal_upgrade`，`FOR UPGRADE`会忽略表中存在的旧时间列；因此，升级过程不会对其进行升级。

    要检查包含这种时间列并需要重建的表格，请在执行`CHECK TABLE ... FOR UPGRADE`之前禁用`avoid_temporal_upgrade`。

+   对于使用非本机分区的表格会发出警告，因为 MySQL 8.0 中移除了非本机分区。请参阅第二十六章，*分区*。

##### 检查数据一致性

下表显示了可以提供的其他检查选项。这些选项将传递给存储引擎，存储引擎可能会使用或忽略它们。

| 类型 | 意义 |
| --- | --- |
| `QUICK` | 不扫描行以检查不正确的链接。适用于`InnoDB`和`MyISAM`表格和视图。 |
| `FAST` | 仅检查未正确关闭的表格。对`InnoDB`无效；仅适用于`MyISAM`表格和视图。 |
| `CHANGED` | 仅检查自上次检查以来已更改或未正确关闭的表格。对`InnoDB`无效；仅适用于`MyISAM`表格和视图。 |
| `MEDIUM` | 扫描行以验证已删除链接是否有效。这还为行计算一个键校验和，并将其与键的计算校验和进行验证。对`InnoDB`无效；仅适用于`MyISAM`表格和视图。 |
| `EXTENDED` | 对每一行的所有键进行完整的键查找。这确保表格是 100%一致的，但需要很长时间。对`InnoDB`无效；仅适用于`MyISAM`表格和视图。 |

您可以组合检查选项，如下例所示，对表格进行快速检查以确定是否已正确关闭：

```sql
CHECK TABLE test_table FAST QUICK;
```

注意

如果`CHECK TABLE`在标记为“损坏”或“未正确关闭”的表格中未发现问题，`CHECK TABLE`可能会移除标记。

如果表格损坏，问题很可能在索引中而不是数据部分。所有前面的检查类型都会彻底检查索引，因此应该能找到大多数错误。

要检查一个您认为没问题的表格，请不使用检查选项或使用`QUICK`选项。当您匆忙时可以使用后者，并且可以承担`QUICK`在数据文件中找不到错误的极小风险。（在大多数情况下，在正常使用情况下，MySQL 应该能找到数据文件中的任何错误。如果发生这种情况，表格将被标记为“损坏”，直到修复为止。）

`FAST`和`CHANGED`主要用于从脚本（例如从**cron**中执行）定期检查表格。在大多数情况下，`FAST`优于`CHANGED`。（唯一不优选的情况是当您怀疑在`MyISAM`代码中发现了错误时。）

仅在运行正常检查但 MySQL 尝试更新行或按键查找行时仍然从表中获得错误时才使用`EXTENDED`。如果正常检查成功，这是非常不可能的。

使用`CHECK TABLE ... EXTENDED`可能会影响查询优化器生成的执行计划。

`CHECK TABLE`报告的一些问题无法自动纠正：

+   `找到行，其中自增列的值为 0`。

    这意味着表中有一行，其中`AUTO_INCREMENT`索引列包含值 0。（可以通过使用`UPDATE`语句显式将列设置为 0 来创建`AUTO_INCREMENT`列为 0 的行。）

    这本身不是错误，但如果您决定转储表并恢复它，或对表进行`ALTER TABLE`操作可能会引起麻烦。在这种情况下，`AUTO_INCREMENT`列根据`AUTO_INCREMENT`列的规则更改值，可能会导致诸如重复键错误之类的问题。

    要消除警告，请执行`UPDATE`语句将列设置为非 0 值。

##### InnoDB 表的`CHECK TABLE`使用注意事项

以下注意事项适用于`InnoDB`表：

+   如果`CHECK TABLE`遇到损坏的页，服务器会退出以防止错误传播（Bug #10132）。如果损坏发生在辅助索引中但表数据可读，运行`CHECK TABLE`仍可能导致服务器退出。

+   如果`CHECK TABLE`在聚簇索引中遇到损坏的`DB_TRX_ID`或`DB_ROLL_PTR`字段，`CHECK TABLE`可能会导致`InnoDB`访问无效的撤消日志记录，导致与 MVCC 相关的服务器退出。

+   如果`CHECK TABLE`在`InnoDB`表或索引中遇到错误，它会报告错误，并通常标记索引，有时标记表为损坏，阻止进一步使用索引或表。此类错误包括辅助索引中不正确的条目数或不正确的链接。

+   如果`CHECK TABLE`在辅助索引中发现不正确的条目数，它会报告错误，但不会导致服务器退出或阻止访问文件。

+   `CHECK TABLE` 调查索引页结构，然后调查每个键入。它不验证指向聚簇记录的键指针，也不遵循 `BLOB` 指针的路径。

+   当 `InnoDB` 表存储在自己的 `.ibd` 文件 中时，`.ibd` 文件的前 3 个 页 包含头部信息而不是表或索引数据。`CHECK TABLE` 语句不会检测仅影响头部数据的不一致性。要验证整个 `InnoDB` `.ibd` 文件的内容，使用 **innochecksum** 命令。

+   在大型 `InnoDB` 表上运行 `CHECK TABLE` 时，其他线程可能在 `CHECK TABLE` 执行期间被阻塞。为避免超时，信号量等待阈值（600 秒）在 `CHECK TABLE` 操作期间延长 2 小时（7200 秒）。如果 `InnoDB` 检测到 240 秒或更长时间的信号量等待，它开始将 `InnoDB` 监视器输出打印到错误日志中。如果锁请求超出信号量等待阈值，`InnoDB` 将中止该进程。为完全避免信号量等待超时的可能性，运行 `CHECK TABLE QUICK` 而不是 `CHECK TABLE`。

+   `InnoDB` `SPATIAL` 索引的 `CHECK TABLE` 功能包括 R 树有效性检查和确保 R 树行数与聚簇索引匹配的检查。

+   `CHECK TABLE` 支持虚拟生成列上的辅助索引，这些索引由 `InnoDB` 支持。

+   截至 MySQL 8.0.14，`InnoDB` 支持并行聚簇索引读取，可以提高 `CHECK TABLE` 的性能。`InnoDB` 在 `CHECK TABLE` 操作期间两次读取聚簇索引。第二次读取可以并行执行。`innodb_parallel_read_threads` 会话变量必须设置为大于 1 的值，才能进行并行聚簇索引读取。默认值为 4。用于执行并行聚簇索引读取的实际线程数由 `innodb_parallel_read_threads` 设置或要扫描的索引子树数量决定，以较小者为准。

##### MyISAM 表的 `CHECK TABLE` 用法注意事项

以下注意事项适用于`MyISAM`表：

+   `CHECK TABLE`更新`MyISAM`表的关键统计信息。

+   如果`CHECK TABLE`输出不返回`OK`或`Table is already up to date`，通常应该对表进行修复。请参阅第 9.6 节，“MyISAM 表维护和崩溃恢复”。

+   如果未指定`CHECK TABLE`选项`QUICK`、`MEDIUM`或`EXTENDED`，动态格式`MyISAM`表的默认检查类型为`MEDIUM`。这与在表上运行**myisamchk --medium-check *`tbl_name`***的结果相同。对于静态格式`MyISAM`表，默认的检查类型也是`MEDIUM`，除非指定了`CHANGED`或`FAST`。在这种情况下，默认值为`QUICK`。对于`CHANGED`和`FAST`，行扫描被跳过，因为行很少损坏。
