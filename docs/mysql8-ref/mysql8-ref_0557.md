# 10.3.6 多列索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-column-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-column-indexes.html)

MySQL 可以创建复合索引（即，多列索引）。一个索引最多可以包含 16 列。对于某些数据类型，您可以对列的前缀进行索引（参见 第 10.3.5 节，“列索引”）。

MySQL 可以为测试索引中的所有列或仅测试第一列、前两列、前三列等的查询使用多列索引。如果您在索引定义中以正确的顺序指定列，单个复合索引可以加速对同一表的多种查询。

多列索引可以被视为排序数组，其行包含通过连接索引列的值创建的值。

注意

作为复合索引的替代方案，您可以引入一个根据其他列信息“哈希”生成的列。如果此列较短、相对唯一且已建立索引，则可能比对许多列进行“宽”索引更快。在 MySQL 中，使用此额外列非常容易：

```sql
SELECT * FROM *tbl_name*
  WHERE *hash_col*=MD5(CONCAT(*val1*,*val2*))
  AND *col1*=*val1* AND *col2*=*val2*;
```

假设表具有以下规范：

```sql
CREATE TABLE test (
    id         INT NOT NULL,
    last_name  CHAR(30) NOT NULL,
    first_name CHAR(30) NOT NULL,
    PRIMARY KEY (id),
    INDEX name (last_name,first_name)
);
```

`name` 索引是在 `last_name` 和 `first_name` 列上的索引。该索引可用于查询中指定已知范围值的 `last_name` 和 `first_name` 值的组合的查找。它也可用于仅指定 `last_name` 值的查询，因为该列是索引的最左前缀（如本节后面所述）。因此，`name` 索引用于以下查询中的查找：

```sql
SELECT * FROM test WHERE last_name='Jones';

SELECT * FROM test
  WHERE last_name='Jones' AND first_name='John';

SELECT * FROM test
  WHERE last_name='Jones'
  AND (first_name='John' OR first_name='Jon');

SELECT * FROM test
  WHERE last_name='Jones'
  AND first_name >='M' AND first_name < 'N';
```

然而，`name` 索引*不*用于以下查询中的查找：

```sql
SELECT * FROM test WHERE first_name='John';

SELECT * FROM test
  WHERE last_name='Jones' OR first_name='John';
```

假设您发出以下 `SELECT` 语句：

```sql
SELECT * FROM *tbl_name*
  WHERE col1=*val1* AND col2=*val2*;
```

如果在 `col1` 和 `col2` 上存在多列索引，则可以直接获取适当的行。如果在 `col1` 和 `col2` 上存在单列索引，则优化器尝试使用索引合并优化（参见 第 10.2.1.3 节，“索引合并优化”），或者尝试通过决定哪个索引排除更多行并使用该索引来获取行。

如果表具有多列索引，则优化器可以使用索引的任何最左前缀来查找行。例如，如果您在 `(col1, col2, col3)` 上有一个三列索引，则可以在 `(col1)`、`(col1, col2)` 和 `(col1, col2, col3)` 上进行索引搜索。

如果列不形成索引的最左前缀，则 MySQL 无法使用索引进行查找。假设您有以下所示的 `SELECT` 语句：

```sql
SELECT * FROM *tbl_name* WHERE col1=*val1*;
SELECT * FROM *tbl_name* WHERE col1=*val1* AND col2=*val2*;

SELECT * FROM *tbl_name* WHERE col2=*val2*;
SELECT * FROM *tbl_name* WHERE col2=*val2* AND col3=*val3*;
```

如果在`(col1, col2, col3)`上存在索引，只有前两个查询会使用该索引。第三个和第四个查询涉及索引列，但不使用索引进行查找，因为`(col2)`和`(col2, col3)`不是`(col1, col2, col3)`的最左前缀。
