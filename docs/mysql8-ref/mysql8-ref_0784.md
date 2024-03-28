# 13.6 数据类型默认值

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html`](https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html)

数据类型规范可以具有显式或隐式默认值。

数据类型规范中的`DEFAULT *`value`*`子句明确指示了列的默认值。例如：

```sql
CREATE TABLE t1 (
  i     INT DEFAULT -1,
  c     VARCHAR(10) DEFAULT '',
  price DOUBLE(16,2) DEFAULT 0.00
);
```

`SERIAL DEFAULT VALUE`是一个特殊情况。在整数列的定义中，它是`NOT NULL AUTO_INCREMENT UNIQUE`的别名。

显式`DEFAULT`子句处理的某些方面取决于版本，如下所述。

+   [MySQL 8.0.13 之后的显式默认处理](https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html#data-type-defaults-explicit "MySQL 8.0.13 之后的显式默认处理")

+   [MySQL 8.0.13 之前的显式默认处理](https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html#data-type-defaults-explicit-old "MySQL 8.0.13 之前的显式默认处理")

+   [隐式默认处理](https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html#data-type-defaults-implicit "隐式默认处理")

### MySQL 8.0.13 之后的显式默认处理

在`DEFAULT`子句中指定的默认值可以是字面常量或表达式。除了一个例外，将表达式默认值括在括号中以区分它们与字面常量默认值。例如：

```sql
CREATE TABLE t1 (
  -- literal defaults
  i INT         DEFAULT 0,
  c VARCHAR(10) DEFAULT '',
  -- expression defaults
  f FLOAT       DEFAULT (RAND() * RAND()),
  b BINARY(16)  DEFAULT (UUID_TO_BIN(UUID())),
  d DATE        DEFAULT (CURRENT_DATE + INTERVAL 1 YEAR),
  p POINT       DEFAULT (Point(0,0)),
  j JSON        DEFAULT (JSON_ARRAY())
);
```

例外情况是，对于`TIMESTAMP`和`DATETIME`列，您可以指定`CURRENT_TIMESTAMP`函数作为默认值，而无需括号。请参阅第 13.2.5 节，“TIMESTAMP 和 DATETIME 的自动初始化和更新”。

只有将`BLOB`、`TEXT`、`GEOMETRY`和`JSON`数据类型写为表达式时，才能为其分配默认值，即使表达式值是字面值：

+   这是允许的（将字面默认指定为表达式）：

    ```sql
    CREATE TABLE t2 (b BLOB DEFAULT ('abc'));
    ```

+   这会产生一个错误（未将字面默认指定为表达式）：

    ```sql
    CREATE TABLE t2 (b BLOB DEFAULT 'abc');
    ```

表达式默认值必须遵守以下规则。如果表达式包含不允许的结构，则会发生错误。

+   字面常量、内置函数（确定性和非确定性）和运算符是允许的。

+   子查询、参数、变量、存储函数和可加载函数不被允许。

+   表达式默认值不能依赖于具有`AUTO_INCREMENT`属性的列。

+   一个列的表达式默认值可以引用其他表列，但是不能引用生成列或具有表达式默认值的列，除非这些列在表定义中出现在前面。也就是说，表达式默认值不能包含对生成列或具有表达式默认值的列的前向引用。

    排序约束也适用于使用`ALTER TABLE`重新排序表列。如果结果表会有一个包含对生成列或具有表达式默认值的列的前向引用的表达式默认值，则该语句将失败。

注意

如果表达式默认值的任何组件依赖于 SQL 模式，除非在所有使用期间 SQL 模式相同，否则对表的不同使用可能会导致不同的结果。

对于`CREATE TABLE ... LIKE`和`CREATE TABLE ... SELECT`，目标表会保留原始表的表达式默认值。

如果表达式默认值引用了一个非确定性函数，任何导致表达式被评估的语句对于基于语句的复制都是不安全的。这包括诸如`INSERT`和`UPDATE`之类的语句。在这种情况下，如果二进制日志记录被禁用，该语句将正常执行。如果启用了二进制日志记录并且`binlog_format`设置为`STATEMENT`，则该语句将被记录并执行，但会向错误日志写入警告消息，因为复制从机可能会发散。当`binlog_format`设置为`MIXED`或`ROW`时，该语句将正常执行。

在插入新行时，具有表达式默认值的列的默认值可以通过省略列名或将列指定为`DEFAULT`来插入（与具有文字默认值的列一样）：

```sql
mysql> CREATE TABLE t4 (uid BINARY(16) DEFAULT (UUID_TO_BIN(UUID())));
mysql> INSERT INTO t4 () VALUES();
mysql> INSERT INTO t4 () VALUES(DEFAULT);
mysql> SELECT BIN_TO_UUID(uid) AS uid FROM t4;
+--------------------------------------+
| uid                                  |
+--------------------------------------+
| f1109174-94c9-11e8-971d-3bf1095aa633 |
| f110cf9a-94c9-11e8-971d-3bf1095aa633 |
+--------------------------------------+
```

然而，使用`DEFAULT(*`col_name`*)`为命名列指定默认值仅适用于具有文字默认值的列，而不适用于具有表达式默认值的列。

不是所有存储引擎都允许表达式默认值。对于那些不允许的引擎，会出现`ER_UNSUPPORTED_ACTION_ON_DEFAULT_VAL_GENERATED`错误。

如果默认值计算结果的数据类型与声明的列类型不同，根据通常的 MySQL 类型转换规则会发生隐式强制转换到声明的类型。参见 第 14.3 节，“表达式评估中的类型转换”。

### 显式默认处理在 MySQL 8.0.13 之前

除了一个例外，`DEFAULT`子句中指定的默认值必须是字面常量；它不能是函数或表达式。这意味着，例如，你不能将日期列的默认值设置为函数值，如`NOW()`或`CURRENT_DATE`。例外是，对于`TIMESTAMP`和`DATETIME`列，你可以指定`CURRENT_TIMESTAMP`作为默认值。参见 第 13.2.5 节，“TIMESTAMP 和 DATETIME 的自动初始化和更新”。

`BLOB`、`TEXT`、`GEOMETRY` 和 `JSON` 数据类型不能被分配默认值。

如果默认值计算结果的数据类型与声明的列类型不同，根据通常的 MySQL 类型转换规则会发生隐式强制转换到声明的类型。参见 第 14.3 节，“表达式评估中的类型转换”。

### 隐式默认处理

如果数据类型规范中不包含显式`DEFAULT`值，MySQL 将确定默认值如下：

如果列可以接受`NULL`作为值，则该列将定义为带有显式`DEFAULT NULL`子句的列。

如果列不能接受`NULL`作为值，MySQL 将定义不带显式`DEFAULT`子句的列。

对于没有显式`DEFAULT`子句的`NOT NULL`列的数据输入，如果`INSERT`或`REPLACE`语句不包含该列的值，或者`UPDATE`语句将列设置为`NULL`，MySQL 根据当时有效的 SQL 模式处理该列：

+   如果启用了严格的 SQL 模式，对于事务表会发生错误并回滚该语句。对于非事务表，会发生错误，但如果这发生在多行语句的第二行或后续行，前面的行将被插入。

+   如果未启用严格模式，MySQL 会将列设置为列数据类型的隐式默认值。

假设表`t`定义如下：

```sql
CREATE TABLE t (i INT NOT NULL);
```

在这种情况下，`i`没有显式默认值，因此在严格模式下，以下每个语句都会产生错误，且不会插入行。当不使用严格模式时，只有第三个语句会产生错误；前两个语句会插入隐式默认值，但第三个会失败，因为`DEFAULT(i)`无法产生值：

```sql
INSERT INTO t VALUES();
INSERT INTO t VALUES(DEFAULT);
INSERT INTO t VALUES(DEFAULT(i));
```

参见第 7.1.11 节，“服务器 SQL 模式”。

对于给定表，`SHOW CREATE TABLE`语句显示哪些列具有显式的`DEFAULT`子句。

隐式默认值定义如下：

+   对于数值类型，默认值为`0`，但对于声明了`AUTO_INCREMENT`属性的整数或浮点数类型，其默认值为序列中的下一个值。

+   对于除`TIMESTAMP`之外的日期和时间类型，其默认值为该类型的适当“零”值。如果启用了`explicit_defaults_for_timestamp`系统变量（参见第 7.1.8 节，“服务器系统变量”），则对于`TIMESTAMP`也是如此。否则，对于表中的第一个`TIMESTAMP`列，其默认值为当前日期和时间。参见第 13.2 节，“日期和时间数据类型”。

+   对于除`ENUM`之外的字符串类型，默认值为空字符串。对于`ENUM`，默认值为第一个枚举值。
