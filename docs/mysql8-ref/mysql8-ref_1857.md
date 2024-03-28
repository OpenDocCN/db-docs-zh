# 26.3.4 分区的维护

> 译文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-maintenance.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-maintenance.html)

可以使用针对此类目的而设计的 SQL 语句在分区表上执行许多表和分区维护任务。

可以使用支持分区表的 `CHECK TABLE`、`OPTIMIZE TABLE`、`ANALYZE TABLE` 和 `REPAIR TABLE` 语句来完成分区表的表维护。

您可以使用一些扩展功能来直接在一个或多个分区上执行此类操作，如下列表所述：

+   **重建分区。** 重建分区；这与删除分区中存储的所有记录，然后重新插入它们具有相同效果。这对于碎片整理很有用。

    示例：

    ```sql
    ALTER TABLE t1 REBUILD PARTITION p0, p1;
    ```

+   **优化分区。** 如果您从分区中删除了大量行，或者对具有可变长度行的分区表进行了许多更改（即具有 `VARCHAR`、`BLOB` 或 `TEXT` 列），您可以使用 `ALTER TABLE ... OPTIMIZE PARTITION` 来回收未使用的空间并对分区数据文件进行碎片整理。

    示例：

    ```sql
    ALTER TABLE t1 OPTIMIZE PARTITION p0, p1;
    ```

    在给定分区上使用 `OPTIMIZE PARTITION` 相当于在该分区上运行 `CHECK PARTITION`、`ANALYZE PARTITION` 和 `REPAIR PARTITION`。

    一些 MySQL 存储引擎，包括 `InnoDB`，不支持每个分区的优化；在这些情况下，`ALTER TABLE ... OPTIMIZE PARTITION` 会分析并重建整个表，并发出适当的警告（Bug #11751825, Bug #42822）。为避免此问题，请改用 `ALTER TABLE ... REBUILD PARTITION` 和 `ALTER TABLE ... ANALYZE PARTITION`。

+   **分析分区。** 这读取并存储分区的键分布。

    示例：

    ```sql
    ALTER TABLE t1 ANALYZE PARTITION p3;
    ```

+   **修复分区。** 这修复损坏的分区。

    示例：

    ```sql
    ALTER TABLE t1 REPAIR PARTITION p0,p1;
    ```

    通常情况下，当分区包含重复键错误时，`REPAIR PARTITION` 操作会失败。您可以使用 `ALTER IGNORE TABLE` 选项，此时由于存在重复键而无法移动的所有行将从分区中删除（Bug #16900947）。

+   **检查分区。** 您可以以与对非分区表使用`CHECK TABLE`相同的方式检查分区中的错误。

    示例：

    ```sql
    ALTER TABLE trb3 CHECK PARTITION p1;
    ```

    这个语句告诉您表`t1`中分区`p1`中的数据或索引是否损坏。如果是这种情况，请使用`ALTER TABLE ... REPAIR PARTITION`来修复该分区。

    通常，当分区包含重复键错误时，`CHECK PARTITION`会失败。您可以在此选项中使用`ALTER IGNORE TABLE`，在这种情况下，该语句将返回在发现重复键违规的分区中每一行的内容。仅报告表的分区表达式中的列的值。（Bug #16900947）

列出的每个语句还支持关键字`ALL`代替分区名称列表。使用`ALL`会导致该语句作用于表中的所有分区。

您还可以使用`ALTER TABLE ... TRUNCATE PARTITION`截断分区。该语句可用于删除一个或多个分区中的所有行，方式与`TRUNCATE TABLE`删除表中的所有行类似。

`ALTER TABLE ... TRUNCATE PARTITION ALL`截断表中的所有分区。
