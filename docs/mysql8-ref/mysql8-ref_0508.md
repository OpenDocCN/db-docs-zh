# 9.6.4 MyISAM 表优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-optimization.html)

为了合并碎片化的行并消除由于删除或更新行而导致的空间浪费，以恢复模式运行**myisamchk**：

```sql
$> myisamchk -r *tbl_name*
```

你可以通过使用`OPTIMIZE TABLE` SQL 语句以相同的方式优化表。`OPTIMIZE TABLE` 进行表修复和关键分析，并对索引树进行排序，以使关键查找更快。使用`OPTIMIZE TABLE`时，没有可能发生工具与服务器之间的不良交互，因为当您使用`OPTIMIZE TABLE`时，服务器会完成所有工作。参见 Section 15.7.3.4, “OPTIMIZE TABLE Statement”。

**myisamchk** 还有许多其他选项，可用于提高表的性能：

+   `--analyze` 或 `-a`：执行关键分布分析。这通过使连接优化器更好地选择连接表的顺序和应该使用的索引来提高连接性能。

+   `--sort-index` 或 `-S`：对索引块进行排序。这样可以优化查找并使使用索引的表扫描更快。

+   `--sort-records=*`index_num`*` 或 `-R *`index_num`*`：根据给定的索引对数据行进行排序。这样可以使您的数据更加局部化，并可能加快使用该索引的基于范围的`SELECT`和`ORDER BY`操作。

有关所有可用选项的完整描述，请参见 Section 6.6.4, “myisamchk — MyISAM Table-Maintenance Utility”。
