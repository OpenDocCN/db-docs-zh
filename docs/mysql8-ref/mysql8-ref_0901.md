# 15.1.15 创建索引语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-index.html`](https://dev.mysql.com/doc/refman/8.0/en/create-index.html)

```sql
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX *index_name*
    [*index_type*]
    ON *tbl_name* (*key_part*,...)
    [*index_option*]
    [*algorithm_option* | *lock_option*] ...

*key_part*: {*col_name* [(*length*)] | (*expr*)} [ASC | DESC]

*index_option*: {
    KEY_BLOCK_SIZE [=] *value*
  | *index_type*
  | WITH PARSER *parser_name*
  | COMMENT '*string*'
  | {VISIBLE | INVISIBLE}
  | ENGINE_ATTRIBUTE [=] '*string*'
  | SECONDARY_ENGINE_ATTRIBUTE [=] '*string*'
}

*index_type*:
    USING {BTREE | HASH}

*algorithm_option*:
    ALGORITHM [=] {DEFAULT | INPLACE | COPY}

*lock_option*:
    LOCK [=] {DEFAULT | NONE | SHARED | EXCLUSIVE}
```

通常，在创建表本身时，您会在表上创建所有索引，使用`创建表`。参见第 15.1.20 节，“创建表语句”。这个指导原则对于`InnoDB`表尤为重要，因为主键决定了数据文件中行的物理布局。`创建索引`使您能够向现有表添加索引。

`创建索引`被映射到一个`修改表`语句来创建索引。参见第 15.1.9 节，“修改表语句”。`创建索引`不能用于创建`主键`；请使用`修改表`。有关索引的更多信息，请参见第 10.3.1 节，“MySQL 如何使用索引”。

`InnoDB`支持虚拟列上的辅助索引。有关更多信息，请参见第 15.1.20.9 节，“辅助索引和生成列”。

当启用`innodb_stats_persistent`设置时，在表上创建索引后，运行`分析表`语句来分析`InnoDB`表。

从 MySQL 8.0.17 开始，*`expr`*的*`key_part`*规范可以采用`(CAST *`json_expression`* AS *`type`* ARRAY)`的形式，在`JSON`列上创建多值索引。参见多值索引。

一个形如`(*key_part1*, *key_part2*, ...)`的索引规范创建了一个具有多个关键部分的索引。索引键值是通过连接给定关键部分的值形成的。例如`(col1, col2, col3)`指定了一个多列索引，其索引键由`col1`、`col2`和`col3`的值组成。

*`key_part`* 规范可以以 `ASC` 或 `DESC` 结尾，以指定索引值是按升序还是降序存储。如果没有给出顺序说明符，则默认为升序。对于 `HASH` 索引，不允许使用 `ASC` 和 `DESC`。对于多值索引，也不支持 `ASC` 和 `DESC`。从 MySQL 8.0.12 开始，不允许对 `SPATIAL` 索引使用 `ASC` 和 `DESC`。

以下各节描述了 `CREATE INDEX` 语句的不同方面：

+   列前缀键部分

+   功能键部分

+   唯一索引

+   全文索引

+   多值索引

+   空间索引

+   索引选项

+   表复制和锁定选项

#### 列前缀键部分

对于字符串列，可以创建仅使用列值前导部分的索引，使用 `*`col_name`*(*`length`*)` 语法来指定索引前缀长度：

+   对于 `CHAR`、`VARCHAR`、`BINARY` 和 `VARBINARY` 键部分，可以指定前缀。

+   对于 `BLOB` 和 `TEXT` 键部分，必须指定前缀。此外，`BLOB` 和 `TEXT` 列只能为 `InnoDB`、`MyISAM` 和 `BLACKHOLE` 表创建索引。

+   前缀*限制*以字节为单位。但是，在`CREATE TABLE`、`ALTER TABLE`和`CREATE INDEX`语句中的索引规范中，对于非二进制字符串类型（`CHAR`、`VARCHAR`、`TEXT`）的索引长度被解释为字符数，对于二进制字符串类型（`BINARY`、`VARBINARY`、`BLOB`）的索引长度被解释为字节数。在为使用多字节字符集的非二进制字符串列指定前缀长度时，请考虑这一点。

    前缀支持和前缀长度（在支持的情况下）取决于存储引擎。例如，对于使用`REDUNDANT`或`COMPACT`行格式的`InnoDB`表，前缀可以长达 767 字节。对于使用`DYNAMIC`或`COMPRESSED`行格式的`InnoDB`表，前缀长度限制为 3072 字节。对于`MyISAM`表，前缀长度限制为 1000 字节。`NDB`存储引擎不支持前缀（参见 Section 25.2.7.6, “Unsupported or Missing Features in NDB Cluster”）。

如果指定的索引前缀超过最大列数据类型大小，`CREATE INDEX`将处理索引如下：

+   对于非唯一索引，如果启用了严格的 SQL 模式，将会出现错误；如果未启用严格的 SQL 模式，则会将索引长度减少到不超过最大列数据类型大小，并产生警告。

+   对于唯一索引，无论 SQL 模式如何，都会出现错误，因为减少索引长度可能会导致插入不符合指定唯一性要求的非唯一条目。

此处显示的语句使用`name`列的前 10 个字符创建索引（假设`name`是非二进制字符串类型）。

```sql
CREATE INDEX part_of_name ON customer (name(10));
```

如果列中的名称通常在前 10 个字符不同，使用此索引进行查找的速度不应比使用从整个`name`列创建的索引慢得多。此外，使用列前缀进行索引可以使索引文件更小，这可以节省大量磁盘空间，也可能加快`INSERT`操作的速度。

#### 函数键部分

“普通”索引索引列值或列值的前缀。例如，在以下表中，对于给定的`t1`行，索引条目包括完整的`col1`值和由其前 10 个字符组成的`col2`值的前缀：

```sql
CREATE TABLE t1 (
  col1 VARCHAR(10),
  col2 VARCHAR(20),
  INDEX (col1, col2(10))
);
```

MySQL 8.0.13 及更高版本支持索引表达式值而不是列或列前缀值的函数键部分。使用函数键部分可以索引表中未直接存储的值。例如：

```sql
CREATE TABLE t1 (col1 INT, col2 INT, INDEX func_index ((ABS(col1))));
CREATE INDEX idx1 ON t1 ((col1 + col2));
CREATE INDEX idx2 ON t1 ((col1 + col2), (col1 - col2), col1);
ALTER TABLE t1 ADD INDEX ((col1 * 40) DESC);
```

具有多个键部分的索引可以混合非函数和函数键部分。

`ASC`和`DESC`对于函数键部分是支持的。

函数键部分必须遵循以下规则。如果键部分定义包含不允许的结构，则会发生错误。

+   在索引定义中，将表达式括在括号中以区分它们与列或列前缀。例如，这是被允许的；表达式被括在括号中：

    ```sql
    INDEX ((col1 + col2), (col3 - col4))
    ```

    这会产生一个错误；表达式没有被括在括号中：

    ```sql
    INDEX (col1 + col2, col3 - col4)
    ```

+   函数键部分不能仅由列名组成。例如，这是不被允许的：

    ```sql
    INDEX ((col1), (col2))
    ```

    相反，将键部分写为非函数键部分，不使用括号：

    ```sql
    INDEX (col1, col2)
    ```

+   函数键部分表达式不能引用列前缀。有关解决方法，请参阅本节后面关于`SUBSTRING()`和`CAST()`的讨论。

+   函数键部分在外键规范中是不被允许的。

对于`CREATE TABLE ... LIKE`，目标表会保留原始表的函数键部分。

函数索引被实现为隐藏的虚拟生成列，这带来了以下影响：

+   每个函数键部分都计入表列总数的限制；参见第 10.4.7 节，“表列数和行大小限制”。

+   函数键部分继承了适用于生成列的所有限制。例如：

    +   只有对于生成列允许的函数才允许用于函数键部分。

    +   子查询、参数、变量、存储函数和可加载函数不被允许。

    有关适用限制的更多信息，请参见第 15.1.20.8 节，“CREATE TABLE and Generated Columns”，以及第 15.1.9.2 节，“ALTER TABLE and Generated Columns”。

+   虚拟生成列本身不需要存储。索引本身占用存储空间，就像任何其他索引一样。

对于包含功能键部分的索引，支持`UNIQUE`。但是，主键不能包含功能键部分。主键需要存储生成列，但功能键部分实现为虚拟生成列，而不是存储生成列。

`SPATIAL`和`FULLTEXT`索引不能具有功能键部分。

如果表不包含主键，则`InnoDB`会自动将第一个`UNIQUE NOT NULL`索引提升为主键。对于具有功能键部分的`UNIQUE NOT NULL`索引，不支持此操作。

如果存在重复索引，非功能索引会引发警告。包含功能键部分的索引不具有此功能。

要删除被功能键部分引用的列，必须首先删除索引。否则，会出现错误。

虽然非功能键部分支持前缀长度规范，但对于功能键部分则不可能。解决方法是使用`SUBSTRING()`（或`CAST()`，如本节后面所述）。要在查询中使用包含`SUBSTRING()`函数的功能键部分，`WHERE`子句必须包含具有相同参数的`SUBSTRING()`。在以下示例中，只有第二个`SELECT`能够使用索引，因为这是唯一一个参数与`SUBSTRING()`函数匹配索引规范的查询：

```sql
CREATE TABLE tbl (
  col1 LONGTEXT,
  INDEX idx1 ((SUBSTRING(col1, 1, 10)))
);
SELECT * FROM tbl WHERE SUBSTRING(col1, 1, 9) = '123456789';
SELECT * FROM tbl WHERE SUBSTRING(col1, 1, 10) = '1234567890';
```

功能键部分使得可以对无法以其他方式索引的值进行索引，例如`JSON`值。但是，必须正确执行才能实现期望的效果。例如，以下语法不起作用：

```sql
CREATE TABLE employees (
  data JSON,
  INDEX ((data->>'$.name'))
);
```

语法失败的原因是：

+   运算符`->>` 转换为`JSON_UNQUOTE(JSON_EXTRACT(...))`。

+   `JSON_UNQUOTE()`返回一个数据类型为`LONGTEXT`的值，因此隐藏的生成列被分配相同的数据类型。

+   MySQL 无法对未在键部分指定前缀长度的`LONGTEXT`列进行索引，并且功能键部分不允许前缀长度。

要对`JSON`列进行索引，您可以尝试使用`CAST()`函数，如下所示：

```sql
CREATE TABLE employees (
  data JSON,
  INDEX ((CAST(data->>'$.name' AS CHAR(30))))
);
```

隐藏的生成列被分配了`VARCHAR(30)`数据类型，可以被索引。但是当尝试使用索引时，这种方法会产生一个新问题：

+   `CAST()`返回一个带有整理`utf8mb4_0900_ai_ci`（服务器默认整理）的字符串。

+   `JSON_UNQUOTE()`返回一个带有整理`utf8mb4_bin`（硬编码）的字符串。

由于在前面的表定义中索引表达式与后续查询中的`WHERE`子句表达式之间存在整理不匹配，因此索引未被使用：

```sql
SELECT * FROM employees WHERE data->>'$.name' = 'James';
```

由于查询和索引中的表达式不同，索引未被使用。为了支持这种情况下的功能键部分，优化器在寻找要使用的索引时会自动剥离`CAST()`，但*仅当*索引表达式的整理与查询表达式的整理匹配时。为了使用具有功能键部分的索引，以下两种解决方案都有效（尽管在效果上略有不同）：

+   解决方案 1. 将索引表达式分配与`JSON_UNQUOTE()`相同的整理：

    ```sql
    CREATE TABLE employees (
      data JSON,
      INDEX idx ((CAST(data->>"$.name" AS CHAR(30)) COLLATE utf8mb4_bin))
    );
    INSERT INTO employees VALUES
      ('{ "name": "james", "salary": 9000 }'),
      ('{ "name": "James", "salary": 10000 }'),
      ('{ "name": "Mary", "salary": 12000 }'),
      ('{ "name": "Peter", "salary": 8000 }');
    SELECT * FROM employees WHERE data->>'$.name' = 'James';
    ```

    `->>`运算符与`JSON_UNQUOTE(JSON_EXTRACT(...))`相同，而`JSON_UNQUOTE()`返回一个带有整理`utf8mb4_bin`的字符串。因此比较是区分大小写的，只有一行匹配：

    ```sql
    +------------------------------------+
    | data                               |
    +------------------------------------+
    | {"name": "James", "salary": 10000} |
    +------------------------------------+
    ```

+   解决方案 2. 在查询中指定完整表达式：

    ```sql
    CREATE TABLE employees (
      data JSON,
      INDEX idx ((CAST(data->>"$.name" AS CHAR(30))))
    );
    INSERT INTO employees VALUES
      ('{ "name": "james", "salary": 9000 }'),
      ('{ "name": "James", "salary": 10000 }'),
      ('{ "name": "Mary", "salary": 12000 }'),
      ('{ "name": "Peter", "salary": 8000 }');
    SELECT * FROM employees WHERE CAST(data->>'$.name' AS CHAR(30)) = 'James';
    ```

    `CAST()`返回一个带有整理`utf8mb4_0900_ai_ci`的字符串，因此比较不区分大小写，两行匹配：

    ```sql
    +------------------------------------+
    | data                               |
    +------------------------------------+
    | {"name": "james", "salary": 9000}  |
    | {"name": "James", "salary": 10000} |
    +------------------------------------+
    ```

请注意，尽管优化器支持自动剥离`CAST()`与索引生成列，但以下方法不起作用，因为它在有索引和无索引时产生不同的结果（Bug#27337092）：

```sql
mysql> CREATE TABLE employees (
         data JSON,
         generated_col VARCHAR(30) AS (CAST(data->>'$.name' AS CHAR(30)))
       );
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> INSERT INTO employees (data)
       VALUES ('{"name": "james"}'), ('{"name": "James"}');
Query OK, 2 rows affected, 1 warning (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 1

mysql> SELECT * FROM employees WHERE data->>'$.name' = 'James';
+-------------------+---------------+
| data              | generated_col |
+-------------------+---------------+
| {"name": "James"} | James         |
+-------------------+---------------+
1 row in set (0.00 sec)

mysql> ALTER TABLE employees ADD INDEX idx (generated_col);
Query OK, 0 rows affected, 1 warning (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 1

mysql> SELECT * FROM employees WHERE data->>'$.name' = 'James';
+-------------------+---------------+
| data              | generated_col |
+-------------------+---------------+
| {"name": "james"} | james         |
| {"name": "James"} | James         |
+-------------------+---------------+
2 rows in set (0.01 sec)
```

#### 唯一索引

`UNIQUE`索引创建一个约束，使索引中的所有值必须是不同的。如果尝试添加一个具有与现有行匹配的键值的新行，则会发生错误。如果在`UNIQUE`索引中为列指定前缀值，则列值必须在前缀长度内是唯一的。`UNIQUE`索引允许对可以包含`NULL`的列有多个`NULL`值。

如果一个表有一个由单个整数类型列组成的`PRIMARY KEY`或`UNIQUE NOT NULL`索引，您可以在`SELECT`语句中使用`_rowid`来引用索引列，如下所示：

+   `_rowid`指的是如果有由单个整数列组成的`PRIMARY KEY`，则指的是`PRIMARY KEY`列。如果有`PRIMARY KEY`但它不由单个整数列组成，则无法使用`_rowid`。

+   否则，`_rowid`指的是第一个`UNIQUE NOT NULL`索引中的列，如果该索引由单个整数列组成。如果第一个`UNIQUE NOT NULL`索引不包含单个整数列，则无法使用`_rowid`。

#### 全文索引

仅支持`InnoDB`和`MyISAM`表的`FULLTEXT`索引，只能包括`CHAR`、`VARCHAR`和`TEXT`列。索引始终在整个列上进行；不支持列前缀索引，如果指定了任何前缀长度，则会被忽略。有关操作的详细信息，请参见第 14.9 节，“全文搜索函数”。

#### 多值索引

截至 MySQL 8.0.17，`InnoDB`支持多值索引。多值索引是定义在存储值数组的列上的辅助索引。一个“正常”的索引对应每个数据记录一个索引记录（1:1）。一个多值索引可以对应单个数据记录多个索引记录（N:1）。多值索引用于对`JSON`数组进行索引。例如，在以下 JSON 文档中对邮政编码数组定义的多值索引为每个邮政编码创建一个索引记录，每个索引记录引用相同的数据���录。

```sql
{
    "user":"Bob",
    "user_id":31,
    "zipcode":[94477,94536]
}
```

##### 创建多值索引

您可以在`CREATE TABLE`、`ALTER TABLE`或`CREATE INDEX`语句中创建多值索引。这需要在索引定义中使用`CAST(... AS ... ARRAY)`，将`JSON`数组中的相同类型的标量值转换为 SQL 数据类型数组。然后，一个虚拟列会透明地生成，其中包含 SQL 数据类型数组中的值；最后，在虚拟列上创建一个函数索引（也称为虚拟索引）。这是在来自 SQL 数据类型数组的值的虚拟列上定义的函数索引形成了多值索引。

下面的示例展示了在名为`customers`的表中的`JSON`列`custinfo`上的数组`$.zipcode`上可以创建多值索引`zips`的三种不同方式。在每种情况下，JSON 数组被转换为`UNSIGNED`整数值的 SQL 数据类型数组。

+   仅`CREATE TABLE`：

    ```sql
    CREATE TABLE customers (
        id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
        modified DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        custinfo JSON,
        INDEX zips( (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)) )
        );
    ```

+   `CREATE TABLE` 加上 `ALTER TABLE`：

    ```sql
    CREATE TABLE customers (
        id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
        modified DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        custinfo JSON
        );

    ALTER TABLE customers ADD INDEX zips( (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)) );
    ```

+   `CREATE TABLE` 加上 `CREATE INDEX`：

    ```sql
    CREATE TABLE customers (
        id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
        modified DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        custinfo JSON
        );

    CREATE INDEX zips ON customers ( (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)) );
    ```

多值索引也可以作为复合索引的一部分定义。此示例显示了一个包含两个单值部分（用于`id`和`modified`列）和一个多值部分（用于`custinfo`列）的复合索引：

```sql
CREATE TABLE customers (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    modified DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    custinfo JSON
    );

ALTER TABLE customers ADD INDEX comp(id, modified,
    (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)) );
```

复合索引中只能使用一个多值键部分。多值键部分可以相对于键的其他部分以任何顺序使用。换句话说，刚刚展示的`ALTER TABLE`语句可以使用`comp(id, (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY), modified))`（或任何其他顺序）仍然有效。

##### 使用多值索引

当在`WHERE`子句中指定以下函数时，优化器使用多值索引来获取记录：

+   `MEMBER OF()`

+   `JSON_CONTAINS()`

+   `JSON_OVERLAPS()`

我们可以通过使用以下`CREATE TABLE`和`INSERT`语句创建和填充`customers`表来演示这一点：

```sql
mysql> CREATE TABLE customers (
 ->     id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->     modified DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
 ->     custinfo JSON
 ->     );
Query OK, 0 rows affected (0.51 sec)

mysql> INSERT INTO customers VALUES
 ->     (NULL, NOW(), '{"user":"Jack","user_id":37,"zipcode":[94582,94536]}'),
 ->     (NULL, NOW(), '{"user":"Jill","user_id":22,"zipcode":[94568,94507,94582]}'),
 ->     (NULL, NOW(), '{"user":"Bob","user_id":31,"zipcode":[94477,94507]}'),
 ->     (NULL, NOW(), '{"user":"Mary","user_id":72,"zipcode":[94536]}'),
 ->     (NULL, NOW(), '{"user":"Ted","user_id":56,"zipcode":[94507,94582]}');
Query OK, 5 rows affected (0.07 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

首先，我们在`customers`表上执行三个查询，分别使用`MEMBER OF()`，`JSON_CONTAINS()`和`JSON_OVERLAPS()`，每个查询的结果如下所示：

```sql
mysql> SELECT * FROM customers
 ->     WHERE 94507 MEMBER OF(custinfo->'$.zipcode');
+----+---------------------+-------------------------------------------------------------------+
| id | modified            | custinfo                                                          |
+----+---------------------+-------------------------------------------------------------------+
|  2 | 2019-06-29 22:23:12 | {"user": "Jill", "user_id": 22, "zipcode": [94568, 94507, 94582]} |
|  3 | 2019-06-29 22:23:12 | {"user": "Bob", "user_id": 31, "zipcode": [94477, 94507]}         |
|  5 | 2019-06-29 22:23:12 | {"user": "Ted", "user_id": 56, "zipcode": [94507, 94582]}         |
+----+---------------------+-------------------------------------------------------------------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM customers
 ->     WHERE JSON_CONTAINS(custinfo->'$.zipcode', CAST('[94507,94582]' AS JSON));
+----+---------------------+-------------------------------------------------------------------+
| id | modified            | custinfo                                                          |
+----+---------------------+-------------------------------------------------------------------+
|  2 | 2019-06-29 22:23:12 | {"user": "Jill", "user_id": 22, "zipcode": [94568, 94507, 94582]} |
|  5 | 2019-06-29 22:23:12 | {"user": "Ted", "user_id": 56, "zipcode": [94507, 94582]}         |
+----+---------------------+-------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> SELECT * FROM customers
 ->     WHERE JSON_OVERLAPS(custinfo->'$.zipcode', CAST('[94507,94582]' AS JSON));
+----+---------------------+-------------------------------------------------------------------+
| id | modified            | custinfo                                                          |
+----+---------------------+-------------------------------------------------------------------+
|  1 | 2019-06-29 22:23:12 | {"user": "Jack", "user_id": 37, "zipcode": [94582, 94536]}        |
|  2 | 2019-06-29 22:23:12 | {"user": "Jill", "user_id": 22, "zipcode": [94568, 94507, 94582]} |
|  3 | 2019-06-29 22:23:12 | {"user": "Bob", "user_id": 31, "zipcode": [94477, 94507]}         |
|  5 | 2019-06-29 22:23:12 | {"user": "Ted", "user_id": 56, "zipcode": [94507, 94582]}         |
+----+---------------------+-------------------------------------------------------------------+
4 rows in set (0.00 sec)
```

接下来，我们对之前的三个查询中的每个运行`EXPLAIN`：

```sql
mysql> EXPLAIN SELECT * FROM customers
 ->     WHERE 94507 MEMBER OF(custinfo->'$.zipcode');
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | customers | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | Using where |
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT * FROM customers
 ->     WHERE JSON_CONTAINS(custinfo->'$.zipcode', CAST('[94507,94582]' AS JSON));
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | customers | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | Using where |
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT * FROM customers
 ->     WHERE JSON_OVERLAPS(custinfo->'$.zipcode', CAST('[94507,94582]' AS JSON));
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | customers | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | Using where |
+----+-------------+-----------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)
```

刚刚展示的三个查询都无法使用任何键。为了解决这个问题，我们可以在`JSON`列（`custinfo`）中的`zipcode`数组上添加一个多值索引，如下所示：

```sql
mysql> ALTER TABLE customers
 ->     ADD INDEX zips( (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)) );
Query OK, 0 rows affected (0.47 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

当我们再次运行之前的`EXPLAIN`语句时，我们现在可以观察到查询可以（并且确实）使用刚刚创建的索引`zips`：

```sql
mysql> EXPLAIN SELECT * FROM customers
 ->     WHERE 94507 MEMBER OF(custinfo->'$.zipcode');
+----+-------------+-----------+------------+------+---------------+------+---------+-------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key  | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | customers | NULL       | ref  | zips          | zips | 9       | const |    1 |   100.00 | Using where |
+----+-------------+-----------+------------+------+---------------+------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT * FROM customers
 ->     WHERE JSON_CONTAINS(custinfo->'$.zipcode', CAST('[94507,94582]' AS JSON));
+----+-------------+-----------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table     | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-----------+------------+-------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | customers | NULL       | range | zips          | zips | 9       | NULL |    6 |   100.00 | Using where |
+----+-------------+-----------+------------+-------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT * FROM customers
 ->     WHERE JSON_OVERLAPS(custinfo->'$.zipcode', CAST('[94507,94582]' AS JSON));
+----+-------------+-----------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table     | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-----------+------------+-------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | customers | NULL       | range | zips          | zips | 9       | NULL |    6 |   100.00 | Using where |
+----+-------------+-----------+------------+-------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)
```

多值索引可以定义为唯一键。如果定义为唯一键，并尝试插入已经存在于多值索引中的值，则会返回重复键错误。如果已经存在重复值，则尝试添加唯一多值索引将失败，如下所示：

```sql
mysql> ALTER TABLE customers DROP INDEX zips;
Query OK, 0 rows affected (0.55 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE customers
 ->     ADD UNIQUE INDEX zips((CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)));
ERROR 1062 (23000): Duplicate entry '94507, ' for key 'customers.zips'
mysql> ALTER TABLE customers
 ->     ADD INDEX zips((CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)));
Query OK, 0 rows affected (0.36 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

##### 多值索引的特性

多值索引具有以下附加特性：

+   影响多值索引的 DML 操作与影响普通索引的 DML 操作处理方式相同，唯一的区别是对于单个聚集索引记录可能有多个插入或更新。

+   空值和多值索引：

    +   如果多值键部分具有空数组，则不会向索引添加任何条目，并且数据记录不可通过索引扫描访问。

    +   如果多值键部分生成返回`NULL`值，则将添加一个包含`NULL`的条目到多值索引中。如果键部分被定义为`NOT NULL`，则会报告错误。

    +   如果类型化数组列设置为`NULL`，存储引擎将存储一个指向数据记录的包含`NULL`的单个记录。

    +   在索引数组中不允许`JSON`空值。如果任何返回值为`NULL`，则将其视为 JSON 空值，并报告无效的 JSON 值错误。

+   因为多值索引是虚拟列上的虚拟索引，所以它们必须遵守与虚拟生成列上的二级索引相同的规则。

+   对于空数组不会添加索引记录。

##### 多值索引的限制和限制。

多值索引受到以下列出的限制和限制的约束：

+   每个多值索引只允许一个多值键部分。但是，[`CAST(... AS ... ARRAY)`表达式可以引用`JSON`文档中的多个数组，如下所示：

    ```sql
    CAST(data->'$.arr[*][*]' AS UNSIGNED ARRAY)
    ```

    在这种情况下，与`JSON`表达式匹配的所有值都作为单个扁平数组存储在索引中。

+   具有多值键部分的索引不支持排序，因此不能用作主键。出于同样的原因，不能使用`ASC`或`DESC`关键字定义多值索引。

+   多值索引不能是覆盖索引。

+   多值索引每个记录的最大值数量由可以存储在单个撤销日志页上的数据量确定，即 65221 字节（64K 减去 315 字节的开销），这意味着键值的最大总长度也是 65221 字节。键的最大数量取决于各种因素，这阻止了定义特定限制。例如，测试表明，多值索引允许每个记录最多有 1604 个整数键。当达到限制时，会报告类似以下的错误：ERROR 3905 (HY000): Exceeded max number of values per record for multi-valued index 'idx' by 1 value(s)。

+   多值键部分中允许的唯一类型表达式是`JSON`表达式。表达式不需要引用插入到索引列中的`JSON`文档中的现有元素，但必须本身在语法上有效。

+   因为相同聚集索引记录的索引记录分散在多值索引中，所以多值索引不支持范围扫描或仅索引扫描。

+   外键规范中不允许多值索引。

+   索引前缀不能为多值索引定义。

+   不能在转换为`BINARY`的数据上定义多值索引（请参阅`CAST()`函数的描述）。

+   不支持在线创建多值索引，这意味着操作使用`ALGORITHM=COPY`。请参阅性能和空间要求。

+   不支持以下两种字符集和排序规则以外的字符集和排序规则用于多值索引：

    1.  使用默认`binary`排序规则的`binary`字符集

    1.  使用默认`utf8mb4_0900_as_cs`排序规则的`utf8mb4`字符集。

+   与`InnoDB`表列上的其他索引一样，多值索引不能使用`USING HASH`创建；尝试这样做会导致警告：This storage engine does not support the HASH index algorithm, storage engine default was used instead.（`USING BTREE`像往常一样受支持。）

#### 空间索引

`MyISAM`、`InnoDB`、`NDB`和`ARCHIVE`存储引擎支持诸如`POINT`和`GEOMETRY`之类的空间列。(第 13.4 节，“空间数据类型”，描述了空间数据类型。)然而，对于不同存储引擎，对空间列索引的支持有所不同。根据以下规则，空间列上的空间和非空间索引是可用的。

空间列上的空间索引具有以下特点：

+   仅适用于`InnoDB`和`MyISAM`表。为其他存储引擎指定`SPATIAL INDEX`会导致错误。

+   从 MySQL 8.0.12 开始，空间列上的索引*必须*是`SPATIAL`索引。因此，对于在空间列上创建索引，`SPATIAL`关键字是可选的，但是隐含的。

+   仅适用于单个空间列。空间索引不能在多个空间列上创建。

+   索引列必须是`NOT NULL`。

+   列前缀长度是被禁止的。每列的完整宽度都被索引。

+   不允许用于主键或唯一索引。

空间列上的非空间索引（使用`INDEX`、`UNIQUE`或`PRIMARY KEY`创建）具有以下特点：

+   除了`ARCHIVE`之外，任何支持空间列的存储引擎都允许。

+   列可以是`NULL`，除非索引是主键。

+   非`SPATIAL`索引的索引类型取决于存储引擎。目前使用的是 B-tree。

+   仅适用于只能具有`NULL`值的列，对于`InnoDB`、`MyISAM`和`MEMORY`表。

#### 索引选项

在键部分列表之后，可以给出索引选项。*`index_option`*值可以是以下任何一种：

+   `KEY_BLOCK_SIZE [=] *`value`*`

    对于`MyISAM`表，`KEY_BLOCK_SIZE`可选地指定用于索引键块的字节大小。该值被视为提示；如果需要，可以使用不同的大小。为单个索引定义指定的`KEY_BLOCK_SIZE`值会覆盖表级别的`KEY_BLOCK_SIZE`值。

    对于`InnoDB`表，不支持在索引级别上使用`KEY_BLOCK_SIZE`。请参阅第 15.1.20 节，“CREATE TABLE 语句”。

+   *`index_type`*

    一些存储引擎允许您在创建索引时指定索引类型。例如：

    ```sql
    CREATE TABLE lookup (id INT) ENGINE = MEMORY;
    CREATE INDEX id_index ON lookup (id) USING BTREE;
    ```

    表 15.1，“存储引擎的索引类型”显示了不同存储引擎支持的允许的索引类型值。当没有索引类型说明符时，默认情况下使用第一个索引类型。表中未列出的存储引擎不支持索引定义中的*`index_type`*子句。

    **表 15.1 存储引擎的索引类型**

    | 存储引擎 | 允许的索引类型 |
    | --- | --- |
    | `InnoDB` | `BTREE` |
    | `MyISAM` | `BTREE` |
    | `MEMORY`/`HEAP` | `HASH`, `BTREE` |
    | `NDB` | `HASH`, `BTREE`（见文本中的注释） |

    *`index_type`*子句不能用于`FULLTEXT INDEX`或（在 MySQL 8.0.12 之前）`SPATIAL INDEX`规范。全文索引实现取决于存储引擎。空间索引实现为 R 树索引。

    如果您指定了对于给定存储引擎无效的索引类型，但是引擎可以使用另一种可用类型而不影响查询结果，则引擎将使用可用类型。解析器将`RTREE`识别为一种类型名称。从 MySQL 8.0.12 开始，这仅允许用于`SPATIAL`索引。在 8.0.12 之前，`RTREE`不能为任何存储引擎指定。

    `BTREE`索引由`NDB`存储引擎实现为 T 树索引。

    注意

    对于`NDB`表列上的索引，`USING`选项只能用于唯一索引或主键。`USING HASH`阻止有序索引的创建；否则，在`NDB`表上创建唯一索引或主键将自动导致有序索引和哈希索引的创建，每个索引相同的列集。

    对于包含一个或多个`NULL`列的`NDB`表的唯一索引，哈希索引只能用于查找文字值，这意味着`IS [NOT] NULL`条件需要对表进行全面扫描。一个解决方法是确保在这种表上始终以包含有序索引的方式创建使用一个或多个`NULL`列的唯一索引；也就是说，在创建索引时避免使用`USING HASH`。

    如果您指定了对于给定存储引擎无效的索引类型，但另一种索引类型可用且不会影响查询结果，那么引擎将使用可用的类型。解析器将`RTREE`识别为一种类型名称，但目前不能为任何存储引擎指定此类型。

    注意

    在`ON *`tbl_name`*`子句之前使用*`index_type`*选项已被弃用；预计在未来的 MySQL 版本中将删除在此位置使用该选项的支持。如果在较早和较晚的位置都给出了*`index_type`*选项，则最终选项生效。

    `TYPE *`type_name`*` 被识别为`USING *`type_name`*`的同义词。然而，`USING`是首选形式。

    下表显示了支持*`index_type`*选项的存储引擎的索引特性。

    **表 15.2 InnoDB 存储引擎索引特性**

    | 索引类别 | 索引类型 | 存储 NULL 值 | 允许多个 NULL 值 | IS NULL 扫描类型 | IS NOT NULL 扫描类型 |
    | --- | --- | --- | --- | --- | --- |
    | 主键 | `BTREE` | 否 | 否 | N/A | N/A |
    | 独特 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | 键 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | `FULLTEXT` | N/A | 是 | 是 | 表 | 表 |
    | `SPATIAL` | N/A | 否 | 否 | N/A | N/A |

    **表 15.3 MyISAM 存储引擎索引特性**

    | 索引类别 | 索引类型 | 存储 NULL 值 | 允���多个 NULL 值 | IS NULL 扫描类型 | IS NOT NULL 扫描类型 |
    | --- | --- | --- | --- | --- | --- |
    | 主键 | `BTREE` | 否 | 否 | N/A | N/A |
    | 独特 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | 键 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | `FULLTEXT` | N/A | 是 | 是 | 表 | 表 |
    | `SPATIAL` | N/A | 否 | 否 | N/A | N/A |

    **表 15.4 MEMORY 存储引擎索引特性**

    | 索引类别 | 索引类型 | 存储 NULL 值 | 允许多个 NULL 值 | IS NULL 扫描类型 | IS NOT NULL 扫描类型 |
    | --- | --- | --- | --- | --- | --- |
    | 主键 | `BTREE` | 否 | 否 | N/A | N/A |
    | 独特 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | 键 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | 主键 | `HASH` | 否 | 否 | N/A | N/A |
    | 独特 | `HASH` | 是 | 是 | 索引 | 索引 |
    | 键 | `HASH` | 是 | 是 | 索引 | 索引 |

    **表 15.5 NDB 存储引擎索引特性**

    | 索引类别 | 索引类型 | 存储 NULL 值 | 允许多个 NULL 值 | IS NULL 扫描类型 | IS NOT NULL 扫描类型 |
    | --- | --- | --- | --- | --- | --- |
    | 主键 | `BTREE` | 否 | 否 | 索引 | 索引 |
    | 独特 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | 键 | `BTREE` | 是 | 是 | 索引 | 索引 |
    | 主键 | `HASH` | 否 | 否 | 表（见注 1） | 表（见注 1） |
    | 唯一 | `HASH` | 是 | 是 | 表（见注 1） | 表（见注 1） |
    | 键 | `HASH` | 是 | 是 | 表（见注 1） | 表（见注 1） |

    表注释：

    1\. `USING HASH`防止创建隐式有序索引。

+   `WITH PARSER *`parser_name`*`

    此选项仅适用于`FULLTEXT`索引。如果全文索引和搜索操作需要特殊处理，则将解析器插件与索引关联起来。`InnoDB`和`MyISAM`支持全文解析器插件。如果您有一个关联有全文解析器插件的`MyISAM`表，您可以使用`ALTER TABLE`将表转换为`InnoDB`。有关更多信息，请参见全文解析器插件和编写全文解析器插件。

+   `COMMENT '*`string`*'`

    索引定义可以包括最多 1024 个字符的可选注释。

    索引页的`MERGE_THRESHOLD`可以通过`CREATE INDEX`语句的*`index_option`* `COMMENT`子句为单个索引进行配置。例如：

    ```sql
    CREATE TABLE t1 (id INT);
    CREATE INDEX id_index ON t1 (id) COMMENT 'MERGE_THRESHOLD=40';
    ```

    如果索引页的页面满百分比低于`MERGE_THRESHOLD`值，当删除行或更新操作缩短行时，`InnoDB`会尝试将索引页与相邻的索引页合并。默认的`MERGE_THRESHOLD`值为 50，这是以前硬编码的值。

    `MERGE_THRESHOLD`也可以在索引级别和表级别使用`CREATE TABLE`和`ALTER TABLE`语句进行定义。有关更多信息，请参见第 17.8.11 节，“配置索引页合并阈值”。

+   `VISIBLE`，`INVISIBLE`

    指定索引可见性。索引默认可见。不可见索引不会被优化器使用。索引可见性的指定适用于主键以外的索引（显式或隐式）。有关更多信息，请参见第 10.3.12 节，“不可见索引”。

+   `ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`选项（自 MySQL 8.0.21 起可用）用于指定主要和次要存储引擎的索引属性。这些选项保留供将来使用。

    允许的值是包含有效`JSON`文档的字符串文字或空字符串（''）。无效的`JSON`将被拒绝。

    ```sql
    CREATE INDEX i1 ON t1 (c1) ENGINE_ATTRIBUTE='{"*key*":"*value*"}';
    ```

    `ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`值可以重复而不会出错。在这种情况下，将使用最后指定的值。

    服务器不会检查`ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`值，也不会在更改表的存储引擎时清除它们。

#### 表复制和锁定选项

可以提供`ALGORITHM`和`LOCK`子句以影响表复制方法和在修改其索引时读写表的并发级别。它们的含义与`ALTER TABLE`语句相同。有关更多信息，请参见第 15.1.9 节，“ALTER TABLE 语句”。

NDB Cluster 支持使用与标准 MySQL 服务器相同的`ALGORITHM=INPLACE`语法进行在线操作。有关更多信息，请参见第 25.6.12 节，“NDB Cluster 中的 ALTER TABLE 在线操作”。
