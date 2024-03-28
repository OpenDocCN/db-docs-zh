# 12.8.5 二进制排序规则与 _bin 排序规则的比较

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-binary-collations.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-binary-collations.html)

本节描述了二进制字符串的`binary`排序规则与非二进制字符串的`_bin`排序规则的比较。

使用`BINARY`、`VARBINARY`和`BLOB`数据类型存储的二进制字符串具有名为`binary`的字符集和排序规则。二进制字符串是字节序列，这些字节的数值确定了比较和排序顺序。参见第 12.10.8 节，“二进制字符集”。

使用`CHAR`、`VARCHAR`和`TEXT`数据类型存储的非二进制字符串具有除`binary`之外的字符集和排序规则。给定的非二进制字符集可以有多个排序规则，每个规则定义了集合中字符的特定比较和排序顺序。对于大多数字符集，其中一个是二进制排序规则，在排序规则名称中以`_bin`后缀表示。例如，`latin1`和`big5`的二进制排序规则分别命名为`latin1_bin`和`big5_bin`。`utf8mb4`是一个例外，它有两个二进制排序规则，分别是`utf8mb4_bin`和`utf8mb4_0900_bin`；参见第 12.10.1 节，“Unicode 字符集”。

`binary`排序规则在几个方面与`_bin`排序规则不同，将在以下部分讨论：

+   比较和排序的单位

+   字符集转换

+   大小写转换

+   比较中的尾随空格处理

+   插入和检索的尾随空格处理

#### 比较和排序的单位

二进制字符串是字节序列。对于`binary`校对规则，比较和排序基于数字字节值。非二进制字符串是字符序列，可能是多字节的。非二进制字符串的校对规则定义了用于比较和排序的字符值排序。对于`_bin`校对规则，此排序基于数字字符代码值，类似于二进制字符串的排序，只是字符代码值可能是多字节的。

#### 字符集转换

非二进制字符串具有一个字符集，并且在许多情况下会自动转换为另一个字符集，即使字符串具有`_bin`校对规则：

+   将列值分配给具有不同字符集的另一列时：

    ```sql
    UPDATE t1 SET utf8mb4_bin_column=latin1_column;
    INSERT INTO t1 (latin1_column) SELECT utf8mb4_bin_column FROM t2;
    ```

+   当使用字符串字面值为`INSERT`或`UPDATE`分配列值时：

    ```sql
    SET NAMES latin1;
    INSERT INTO t1 (utf8mb4_bin_column) VALUES ('string-in-latin1');
    ```

+   从服务器发送结果到客户端时：

    ```sql
    SET NAMES latin1;
    SELECT utf8mb4_bin_column FROM t2;
    ```

对于二进制字符串列，不会发生转换。对于类似的情况，字符串值会逐字节复制。

#### 大小写转换

非二进制字符集的校对规则提供了关于字符大小写的信息，因此非二进制字符串中的字符可以从一个大小写转换为另一个大小写，即使是对于忽略大小写进行排序的`_bin`校对规则：

```sql
mysql> SET NAMES utf8mb4 COLLATE utf8mb4_bin;
mysql> SELECT LOWER('aA'), UPPER('zZ');
+-------------+-------------+
| LOWER('aA') | UPPER('zZ') |
+-------------+-------------+
| aa          | ZZ          |
+-------------+-------------+
```

字节中的大小写概念不适用于二进制字符串。要执行大小写转换，必须首先使用适合存储在字符串中的数据的字符集将字符串转换为非二进制字符串：

```sql
mysql> SET NAMES binary;
mysql> SELECT LOWER('aA'), LOWER(CONVERT('aA' USING utf8mb4));
+-------------+------------------------------------+
| LOWER('aA') | LOWER(CONVERT('aA' USING utf8mb4)) |
+-------------+------------------------------------+
| aA          | aa                                 |
+-------------+------------------------------------+
```

#### 比较中处理末尾空格

MySQL 校对规则具有一个`PAD SPACE`或`NO PAD`的填充属性：

+   大多数 MySQL 校对规则具有`PAD SPACE`的填充属性。

+   基于 UCA 9.0.0 及更高版本的 Unicode 校对规则具有`NO PAD`的填充属性；参见第 12.10.1 节，“Unicode 字符集”。

对于非二进制字符串（`CHAR`，`VARCHAR`和`TEXT`值），字符串校对填充属性决定了在比较末尾空格时的处理方式：

+   对于`PAD SPACE`校对规则，比较中末尾空格不重要；字符串比较时不考虑末尾空格。

+   `NO PAD`校对规则将末尾空格视为比较中的重要字符，就像任何其他字符一样。

这些不同的行为可以使用两个`utf8mb4`二进制校对规则来演示，其中一个是`PAD SPACE`，另一个是`NO PAD`。示例还展示了如何使用`INFORMATION_SCHEMA` `COLLATIONS`表来确定校对规则的填充属性。

```sql
mysql> SELECT COLLATION_NAME, PAD_ATTRIBUTE
       FROM INFORMATION_SCHEMA.COLLATIONS
       WHERE COLLATION_NAME LIKE 'utf8mb4%bin';
+------------------+---------------+
| COLLATION_NAME   | PAD_ATTRIBUTE |
+------------------+---------------+
| utf8mb4_bin      | PAD SPACE     |
| utf8mb4_0900_bin | NO PAD        |
+------------------+---------------+
mysql> SET NAMES utf8mb4 COLLATE utf8mb4_bin;
mysql> SELECT 'a ' = 'a';
+------------+
| 'a ' = 'a' |
+------------+
|          1 |
+------------+
mysql> SET NAMES utf8mb4 COLLATE utf8mb4_0900_bin;
mysql> SELECT 'a ' = 'a';
+------------+
| 'a ' = 'a' |
+------------+
|          0 |
+------------+
```

注意

在此上下文中的“比较”不包括`LIKE`模式匹配运算符，对于这些运算符，无论校对规则如何，末尾空格都是重要的。

对于二进制字符串（`BINARY`，`VARBINARY`和`BLOB`值），在比较中所有字节都是重要的，包括尾随空格：

```sql
mysql> SET NAMES binary;
mysql> SELECT 'a ' = 'a';
+------------+
| 'a ' = 'a' |
+------------+
|          0 |
+------------+
```

#### 插入和检索的尾随空格处理

`CHAR(*`N`*)`列存储长度为*`N`*个字符的非二进制字符串。对于插入操作，长度小于*`N`*个字符的值将用空格扩展。对于检索，尾随空格将被移除。

`BINARY(*`N`*)`列存储长度为*`N`*字节的二进制字符串。对于插入操作，长度小于*`N`*字节的值将用`0x00`字节扩展。对于检索，不会删除任何内容；始终返回声明长度的值。

```sql
mysql> CREATE TABLE t1 (
         a CHAR(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin,
         b BINARY(10)
       );
mysql> INSERT INTO t1 VALUES ('x','x');
mysql> INSERT INTO t1 VALUES ('x ','x ');
mysql> SELECT a, b, HEX(a), HEX(b) FROM t1;
+------+------------------------+--------+----------------------+
| a    | b                      | HEX(a) | HEX(b)               |
+------+------------------------+--------+----------------------+
| x    | 0x78000000000000000000 | 78     | 78000000000000000000 |
| x    | 0x78200000000000000000 | 78     | 78200000000000000000 |
+------+------------------------+--------+----------------------+
```
