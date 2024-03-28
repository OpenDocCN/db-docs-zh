> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html)

#### 15.1.20.8 CREATE TABLE and Generated Columns

`CREATE TABLE`支持生成列的规范。生成列的值是从列定义中包含的表达式计算出来的。

生成列也受到`NDB`存储引擎的支持。

以下简单示例显示了一个表，该表存储直角三角形的边长在`sidea`和`sideb`列中，并在`sidec`中计算斜边的长度（其他两边平方和的平方根）：

```sql
CREATE TABLE triangle (
  sidea DOUBLE,
  sideb DOUBLE,
  sidec DOUBLE AS (SQRT(sidea * sidea + sideb * sideb))
);
INSERT INTO triangle (sidea, sideb) VALUES(1,1),(3,4),(6,8);
```

从表中选择会产生以下结果：

```sql
mysql> SELECT * FROM triangle;
+-------+-------+--------------------+
| sidea | sideb | sidec              |
+-------+-------+--------------------+
|     1 |     1 | 1.4142135623730951 |
|     3 |     4 |                  5 |
|     6 |     8 |                 10 |
+-------+-------+--------------------+
```

使用`triangle`表的任何应用程序都可以访问斜边值，而无需指定计算它们的表达式。

生成列的定义具有以下语法：

```sql
*col_name* *data_type* [GENERATED ALWAYS] AS (*expr*)
  [VIRTUAL | STORED] [NOT NULL | NULL]
  [UNIQUE [KEY]] [[PRIMARY] KEY]
  [COMMENT '*string*']
```

`AS (*`expr`*)`表示该列是生成的，并定义用于计算列值的表达式。`AS`之前可以加上`GENERATED ALWAYS`以使列的生成性质更加明确。表达式中允许或禁止的结构将在后面讨论。

`VIRTUAL`或`STORED`关键字指示列值的存储方式，这对列的使用有影响：

+   `VIRTUAL`：列值不存储，但在读取行时立即计算，紧随任何`BEFORE`触发器之后。虚拟列不占用存储空间。

    `InnoDB`支持虚拟列上的二级索引。请参阅 Section 15.1.20.9, “Secondary Indexes and Generated Columns”。

+   `STORED`：在插入或更新行时，列值会被计算并存储。存储列确实需要存储空间，并且可以被索引。

如果未指定关键字，则默认为`VIRTUAL`。

在表中混合使用`VIRTUAL`和`STORED`列是允许的。

可以提供其他属性以指示列是否被索引或可以为`NULL`，或提供注释。

生成列表达式必须遵守以下规则。如果表达式包含不允许的结构，则会发生错误。

+   文本，确定性内置函数和运算符是允许的。如果给定相同的表中数据，多次调用产生相同结果，则函数是确定性的，与连接的用户无关。不确定性并不符合此定义的函数示例：`CONNECTION_ID()`，`CURRENT_USER()`，`NOW()`。

+   不允许存储函数和可加载函数。

+   不允许存储过程和函数参数。

+   不允许变量（系统变量、用户定义变量和存储过程局部变量）。

+   不允许子查询。

+   生成列定义可以引用其他生成列，但只能引用表定义中较早出现的列。生成列定义可以引用表中的任何基本（非生成的）列，无论其定义是早于还是晚于。

+   `AUTO_INCREMENT`属性不能在生成列定义中使用。

+   `AUTO_INCREMENT`列不能作为生成列定义中的基本列使用。

+   如果表达式评估导致截断或向函数提供不正确的输入，则`CREATE TABLE`语句将以错误终止，并拒绝 DDL 操作。

如果表达式评估为与声明的列类型不同的数据类型，则根据通常的 MySQL 类型转换规则隐式强制转换为声明的类型。请参见第 14.3 节，“表达式评估中的类型转换”。

如果生成的列使用`TIMESTAMP`数据类型，则`explicit_defaults_for_timestamp`设置将被忽略。在这种情况下，如果此变量被禁用，则`NULL`不会转换为`CURRENT_TIMESTAMP`。在 MySQL 8.0.22 及更高版本中，如果列还声明为`NOT NULL`，则尝试插入`NULL`将明确拒绝，并显示`ER_BAD_NULL_ERROR`。

注意

表达式评估使用评估时有效的 SQL 模式。如果表达式的任何组件依赖于 SQL 模式，则除非在所有使用期间 SQL 模式相同，否则可能会出现不同的结果。

对于`CREATE TABLE ... LIKE`，目标表保留原始表的生成列信息。

对于`CREATE TABLE ... SELECT`，目标表不保留所选自表中列是否为生成列的信息。语句的`SELECT`部分不能为目标表中的生成列分配值。

允许通过生成列进行分区。请参见表分区。

存储生成列上的外键约束不能使用`CASCADE`、`SET NULL`或`SET DEFAULT`作为`ON UPDATE`参照操作，也不能使用`SET NULL`或`SET DEFAULT`作为`ON DELETE`参照操作。

存储生成列的基列上的外键约束不能使用`CASCADE`、`SET NULL`或`SET DEFAULT`作为`ON UPDATE`或`ON DELETE`引用动作。

外键约束不能引用虚拟生成列。

触发器不能使用`NEW.*col_name*`或使用`OLD.*col_name*`来引用生成列。 

对于`INSERT`、`REPLACE`和`UPDATE`，如果显式插入、替换或更新生成列，则唯一允许的值是`DEFAULT`。

视图中的生成列被视为可更新，因为可以对其进行赋值。但是，如果显式更新此类列，则唯一允许的值是`DEFAULT`。

生成列有几种用途，例如：

+   虚拟生成列可用作简化和统一查询的一种方式。可以将复杂条件定义为生成列，并从表上的多个查询中引用该条件，以确保它们都使用完全相同的条件。

+   存储生成列可用作复杂条件的物化缓存，这些条件在实时计算时成本高昂。

+   生成列可以模拟函数索引：使用生成列定义函数表达式并对其进行索引。这对于无法直接索引的类型列（例如`JSON`列）非常有用；请参见使用生成列创建 JSON 列索引，以获取详细示例。

    对于存储生成列，这种方法的缺点是值存储两次；一次作为生成列的值，一次作为索引中的值。

+   如果生成列被索引，优化器会识别与列定义匹配的查询表达式，并在查询执行期间适当地使用列的索引，即使查询没有直接按名称引用该列。有关详细信息，请参见第 10.3.11 节，“生成列索引的优化器使用”。

示例：

假设表`t1`包含`first_name`和`last_name`列，并且应用程序经常使用类似以下表达式构建全名：

```sql
SELECT CONCAT(first_name,' ',last_name) AS full_name FROM t1;
```

避免编写表达式的一种方法是在`t1`上创建视图`v1`，这样可以通过直接选择`full_name`来简化应用程序，而无需使用表达式：

```sql
CREATE VIEW v1 AS
SELECT *, CONCAT(first_name,' ',last_name) AS full_name FROM t1;

SELECT full_name FROM v1;
```

生成列还使应用程序能够直接选择`full_name`，而无需定义视图：

```sql
CREATE TABLE t1 (
  first_name VARCHAR(10),
  last_name VARCHAR(10),
  full_name VARCHAR(255) AS (CONCAT(first_name,' ',last_name))
);

SELECT full_name FROM t1;
```
