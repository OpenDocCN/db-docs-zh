> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-table-generated-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-table-generated-columns.html)

#### 15.1.9.2 ALTER TABLE 和 Generated Columns

允许对生成列执行的`ALTER TABLE`操作为`ADD`、`MODIFY`和`CHANGE`。

+   可以添加生成列。

    ```sql
    CREATE TABLE t1 (c1 INT);
    ALTER TABLE t1 ADD COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) STORED;
    ```

+   可以修改生成列的数据类型和表达式。

    ```sql
    CREATE TABLE t1 (c1 INT, c2 INT GENERATED ALWAYS AS (c1 + 1) STORED);
    ALTER TABLE t1 MODIFY COLUMN c2 TINYINT GENERATED ALWAYS AS (c1 + 5) STORED;
    ```

+   如果没有其他列引用生成列，则可以重命名或删除生成列。

    ```sql
    CREATE TABLE t1 (c1 INT, c2 INT GENERATED ALWAYS AS (c1 + 1) STORED);
    ALTER TABLE t1 CHANGE c2 c3 INT GENERATED ALWAYS AS (c1 + 1) STORED;
    ALTER TABLE t1 DROP COLUMN c3;
    ```

+   无法将虚拟生成列更改为存储生成列，反之亦然。要解决此问题，请删除列，然后使用新定义添加列。

    ```sql
    CREATE TABLE t1 (c1 INT, c2 INT GENERATED ALWAYS AS (c1 + 1) VIRTUAL);
    ALTER TABLE t1 DROP COLUMN c2;
    ALTER TABLE t1 ADD COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) STORED;
    ```

+   可以将非生成列更改为存储但不是虚拟生成列。

    ```sql
    CREATE TABLE t1 (c1 INT, c2 INT);
    ALTER TABLE t1 MODIFY COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) STORED;
    ```

+   可以将存储但不是虚拟生成列更改为非生成列。存储生成的值成为非生成列的值。

    ```sql
    CREATE TABLE t1 (c1 INT, c2 INT GENERATED ALWAYS AS (c1 + 1) STORED);
    ALTER TABLE t1 MODIFY COLUMN c2 INT;
    ```

+   `ADD COLUMN` 不是对存储列进行原地操作（不使用临时表完成）的操作，因为表达式必须由服务器评估。对于存储列，索引更改是原地完成的，而表达式更改不是原地完成的。对列注释的更改是原地完成的。

+   对于非分区表，对虚拟列进行`ADD COLUMN`和`DROP COLUMN`是原地操作。但是，无法在与其他`ALTER TABLE`操作组合时原地执行添加或删除虚拟列。

    对于分区表，对虚拟列进行`ADD COLUMN`和`DROP COLUMN`不是原地操作。

+   `InnoDB`支持虚拟生成列的辅助索引。在虚拟生成列上添加或删除辅助索引是原地操作。有关更多信息，请参见 Section 15.1.20.9, “Secondary Indexes and Generated Columns”。

+   当向表中添加或修改`VIRTUAL`生成列时，不能确保由生成列表达式计算的数据不超出列的范围。这可能导致返回不一致的数据和意外失败的语句。为了允许对这些列进行验证的控制，`ALTER TABLE`支持`WITHOUT VALIDATION`和`WITH VALIDATION`子句：

    +   使用`WITHOUT VALIDATION`（如果未指定任何子句，则为默认值），将执行原地操作（如果可能），不会检查数据完整性，并且语句完成得更快。但是，如果值超出范围，则稍后从表中读取的值可能会报告警告或错误。

    +   使用`WITH VALIDATION`，`ALTER TABLE`会复制表。如果发生超出范围或任何其他错误，则语句失败。由于执行了表复制，因此语句需要更长时间。

    `WITHOUT VALIDATION`和`WITH VALIDATION`只允许与`ADD COLUMN`、`CHANGE COLUMN`和`MODIFY COLUMN`操作一起使用。否则，会出现`ER_WRONG_USAGE`错误。

+   如果表达式评估导致截断或向函数提供不正确的输入，则`ALTER TABLE`语句将以错误终止，DDL 操作将被拒绝。

+   一个`ALTER TABLE`语句，改变列*`col_name`*的默认值可能会改变引用该列的生成列表达式的值，该生成列表达式使用*`col_name`*，这可能会改变引用该列的生成列表达式的值，该生成列表达式使用`DEFAULT(*`col_name`*)`。因此，如果任何生成列表达式使用`DEFAULT()`，那么改变列定义的`ALTER TABLE`操作会导致表重建。
