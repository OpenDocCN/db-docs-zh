> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-check-constraints.html)

#### 15.1.20.6 CHECK 约束

在 MySQL 8.0.16 之前，`CREATE TABLE`仅允许以下有限版本的表`CHECK`约束语法，该语法被解析并忽略：

```sql
CHECK (*expr*)
```

截至 MySQL 8.0.16，`CREATE TABLE`允许所有存储引擎的表和列`CHECK`约束的核心特性。`CREATE TABLE`允许以下`CHECK`约束语法，适用于表约束和列约束：

```sql
[CONSTRAINT [*symbol*]] CHECK (*expr*) [[NOT] ENFORCED]
```

可选的*`symbol`*指定了约束的名称。如果省略，MySQL 会从表名、字面量`_chk_`和一个序号（1、2、3、...）生成一个名称。约束名称最长为 64 个字符。它们区分大小写，但不区分重音符号。

*`expr`*指定约束条件为一个布尔表达式，每行必须评估为`TRUE`或`UNKNOWN`（对于`NULL`值）。如果条件评估为`FALSE`，则失败并发生约束违反。违反的效果取决于正在执行的语句，如本节后面所述。

可选的强制执行子句指示约束是否被执行：

+   如果省略或指定为`ENFORCED`，则创建并执行约束。

+   如果指定为`NOT ENFORCED`，则创建约束但不执行。

`CHECK`约束可以指定为表约束或列约束：

+   表约束不出现在列定义中，可以引用任何表列或列。允许对稍后出现在表定义中的列进行前向引用。

+   列约束出现在列定义中，只能引用该列。

考虑这个表定义：

```sql
CREATE TABLE t1
(
  CHECK (c1 <> c2),
  c1 INT CHECK (c1 > 10),
  c2 INT CONSTRAINT c2_positive CHECK (c2 > 0),
  c3 INT CHECK (c3 < 100),
  CONSTRAINT c1_nonzero CHECK (c1 <> 0),
  CHECK (c1 > c3)
);
```

定义包括命名和未命名格式的表约束和列约束：

+   第一个约束��表约束：它出现在任何列定义之外，因此可以（并且确实）引用多个表列。此约束包含对尚未定义的列的前向引用。未指定约束名称，因此 MySQL 生成一个名称。

+   接下来的三个约束是列约束：每个出现在列定义中，因此只能引用正在定义的列。其中一个约束明确命名。MySQL 为另外两个生成名称。

+   最后两个约束是表约束。其中一个明确命名。MySQL 为另一个生成一个名称。

如前所述，MySQL 为未指定名称的任何`CHECK`约束生成一个名称。要查看前述表定义生成的名称，请使用`SHOW CREATE TABLE`：

```sql
mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` int(11) DEFAULT NULL,
  `c2` int(11) DEFAULT NULL,
  `c3` int(11) DEFAULT NULL,
  CONSTRAINT `c1_nonzero` CHECK ((`c1` <> 0)),
  CONSTRAINT `c2_positive` CHECK ((`c2` > 0)),
  CONSTRAINT `t1_chk_1` CHECK ((`c1` <> `c2`)),
  CONSTRAINT `t1_chk_2` CHECK ((`c1` > 10)),
  CONSTRAINT `t1_chk_3` CHECK ((`c3` < 100)),
  CONSTRAINT `t1_chk_4` CHECK ((`c1` > `c3`))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

SQL 标准规定所有类型的约束（主键，唯一索引，外键，检查）属于同一命名空间。在 MySQL 中，每种约束类型在每个模式（数据库）中都有自己的命名空间。因此，`CHECK`约束名称必须在每个模式中是唯一的；同一模式中的两个表不能共享`CHECK`约束名称。（例外情况：`TEMPORARY`表隐藏了同名的非`TEMPORARY`表，因此它也可以具有相同的`CHECK`约束名称。）

以表名开头生成的约束名称有助于确保模式的唯一性，因为表名在模式内也必须是唯一的。

`CHECK`条件表达式必须遵守以下规则。如果表达式包含不允许的结构，则会发生错误。

+   允许非生成和生成列，除了具有`AUTO_INCREMENT`属性的列和其他表中的列。

+   允许使用文字，确定性内置函数和运算符。如果给定表中的数据相同，则函数是确定性的，多次调用会产生相同的结果，与连接的用户无关。不符合此定义的函数的示例包括：`CONNECTION_ID()`，`CURRENT_USER()`，`NOW()`。

+   不允许存储函数和可加载函数。

+   不允许存储过程和函数参数。

+   变量（系统变量，用户定义变量和存储程序本地变量）是不允许的。

+   不允许子查询。

外键参照操作（`ON UPDATE`，`ON DELETE`）在用于`CHECK`约束的列上是被禁止的。同样，`CHECK`约束在用于外键参照操作的列上也是被禁止的。

`CHECK`约束会在`INSERT`，`UPDATE`，`REPLACE`，`LOAD DATA`和`LOAD XML`语句中进行评估，如果约束评估为`FALSE`，则会发生错误。如果发生错误，已应用更改的处理方式对于事务性和非事务性存储引擎有所不同，并且还取决于是否启用了严格的 SQL 模式，如严格的 SQL 模式中所述。

`CHECK`约束会在`INSERT IGNORE`，`UPDATE IGNORE`，`LOAD DATA ... IGNORE`和`LOAD XML ... IGNORE`语句中进行评估，如果约束评估为`FALSE`，则会发出警告。对于任何违反约束的行，插入或更新将被跳过。

如果约束表达式评估为与声明的列类型不同的数据类型，则根据通常的 MySQL 类型转换规则发生对声明类型的隐式强制转换。请参阅第 14.3 节，“表达式评估中的类型转换”。如果类型转换失败或导致精度丢失，则会发生错误。

注意

约束表达式评估在评估时使用当前的 SQL 模式。如果表达式的任何组件依赖于 SQL 模式，则除非在所有使用期间 SQL 模式相同，否则对表的不同使用可能导致不同的结果。

信息模式`CHECK_CONSTRAINTS`表提供有关在表上定义的 `CHECK` 约束的信息。请参阅第 28.3.5 节，“INFORMATION_SCHEMA CHECK_CONSTRAINTS 表”。
