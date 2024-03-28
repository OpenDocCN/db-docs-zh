# 10.3.3 空间索引优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-index-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-index-optimization.html)

MySQL 允许在`NOT NULL`几何值列上创建`SPATIAL`索引（参见第 13.4.10 节，“创建空间索引”）。优化器检查索引列的`SRID`属性，以确定用于比较的空间参考系统（SRS），并使用适合 SRS 的计算。 （在 MySQL 8.0 之前，优化器使用笛卡尔计算对`SPATIAL`索引值进行比较；如果列包含具有非笛卡尔`SRID`的值，则此类操作的结果是未定义的。）

为了使比较正常工作，`SPATIAL`索引中的每一列必须受到`SRID`限制。也就是说，列定义必须包含显式的`SRID`属性，并且所有列值必须具有相同的`SRID`。

优化器仅考虑具有`SRID`限制的列的`SPATIAL`索引：

+   对于受笛卡尔`SRID`限制的列上的索引，使笛卡尔边界框计算成为可能。

+   对于受地理`SRID`限制的列上的索引，使地理边界框计算成为可能。

优化器忽略没有`SRID`属性（因此不受`SRID`限制）的列上的`SPATIAL`索引。MySQL 仍然维护这样的索引，如下所示：

+   它们用于表修改（`INSERT`，`UPDATE`，`DELETE`等）。即使列可能包含混合的笛卡尔和地理值，更新也会像索引是笛卡尔的一样进行。

+   它们仅用于向后兼容性（例如，在 MySQL 5.7 中执行转储并在 MySQL 8.0 中恢复的能力）。因为`SPATIAL`索引在不受`SRID`限制的列上对优化器没有用处，因此应修改每个这样的列：

    +   验证列中的所有值是否具有相同的`SRID`。要确定几何列*`col_name`*中包含的`SRID`，请使用以下查询：

        ```sql
        SELECT DISTINCT ST_SRID(*col_name*) FROM *tbl_name*;
        ```

        如果查询返回多行，则该列包含混合`SRID`的值。在这种情况下，修改其内容以使所有值具有相同的`SRID`。

    +   重新定义列以具有显式的`SRID`属性。

    +   重新创建`SPATIAL`索引。
