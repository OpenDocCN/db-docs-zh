# 12.14.1 整理实现类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collation-implementations.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-implementations.html)

MySQL 实现了几种类型的整理：

**8 位字符集的简单整理**

这种整理使用一个包含 256 个权重的数组来实现，定义了从字符代码到权重的一对一映射。`latin1_swedish_ci`是一个例子。它是一个不区分大小写的整理，因此字符的大写和小写版本具有相同的权重，它们比较相等。

```sql
mysql> SET NAMES 'latin1' COLLATE 'latin1_swedish_ci';
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT HEX(WEIGHT_STRING('a')), HEX(WEIGHT_STRING('A'));
+-------------------------+-------------------------+
| HEX(WEIGHT_STRING('a')) | HEX(WEIGHT_STRING('A')) |
+-------------------------+-------------------------+
| 41                      | 41                      |
+-------------------------+-------------------------+
1 row in set (0.01 sec)

mysql> SELECT 'a' = 'A';
+-----------+
| 'a' = 'A' |
+-----------+
|         1 |
+-----------+
1 row in set (0.12 sec)
```

有关实现说明，请参见第 12.14.3 节，“向 8 位字符集添加简单整理”。

**8 位字符集的复杂整理**

这种整理使用 C 源文件中的函数来定义如何对字符进行排序，如第 12.13 节，“添加字符集”中所述。

**非 Unicode 多字节字符集的整理**

对于这种类型的整理，8 位（单字节）和多字节字符的处理方式不同。对于 8 位字符，字符代码以不区分大小写的方式映射到权重。（例如，单字节字符`'a'`和`'A'`都具有权重`0x41`。）对于多字节字符，字符代码和权重之间有两种关系：

+   权重等于字符代码。`sjis_japanese_ci`是这种整理的一个例子。多字节字符`'ぢ'`的字符代码为`0x82C0`，权重也是`0x82C0`。

    ```sql
    mysql> CREATE TABLE t1
           (c1 VARCHAR(2) CHARACTER SET sjis COLLATE sjis_japanese_ci);
    Query OK, 0 rows affected (0.01 sec)

    mysql> INSERT INTO t1 VALUES ('a'),('A'),(0x82C0);
    Query OK, 3 rows affected (0.00 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> SELECT c1, HEX(c1), HEX(WEIGHT_STRING(c1)) FROM t1;
    +------+---------+------------------------+
    | c1   | HEX(c1) | HEX(WEIGHT_STRING(c1)) |
    +------+---------+------------------------+
    | a    | 61      | 41                     |
    | A    | 41      | 41                     |
    | ぢ    | 82C0    | 82C0                   |
    +------+---------+------------------------+
    3 rows in set (0.00 sec)
    ```

+   字符代码一对一映射到权重，但代码不一定等于权重。`gbk_chinese_ci`是这种整理的一个例子。多字节字符`'膰'`的字符代码为`0x81B0`，但权重为`0xC286`。

    ```sql
    mysql> CREATE TABLE t1
           (c1 VARCHAR(2) CHARACTER SET gbk COLLATE gbk_chinese_ci);
    Query OK, 0 rows affected (0.33 sec)

    mysql> INSERT INTO t1 VALUES ('a'),('A'),(0x81B0);
    Query OK, 3 rows affected (0.00 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> SELECT c1, HEX(c1), HEX(WEIGHT_STRING(c1)) FROM t1;
    +------+---------+------------------------+
    | c1   | HEX(c1) | HEX(WEIGHT_STRING(c1)) |
    +------+---------+------------------------+
    | a    | 61      | 41                     |
    | A    | 41      | 41                     |
    | 膰    | 81B0    | C286                   |
    +------+---------+------------------------+
    3 rows in set (0.00 sec)
    ```

有关实现说明，请参见第 12.13 节，“添加字符集”。

**Unicode 多字节字符集的整理**

这些整理中有一些基于 Unicode 整理算法（UCA），而其他一些则不是。

非 UCA 整理将字符代码一对一映射到权重。在 MySQL 中，这种整理不区分大小写，也不区分重音。`utf8mb4_general_ci`是一个例子：`'a'`、`'A'`、`'À'`和`'á'`每个具有不同的字符代码，但都具有权重`0x0041`，并且比较相等。

```sql
mysql> SET NAMES 'utf8mb4' COLLATE 'utf8mb4_general_ci';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t1
       (c1 CHAR(1) CHARACTER SET UTF8MB4 COLLATE utf8mb4_general_ci);
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO t1 VALUES ('a'),('A'),('À'),('á');
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> SELECT c1, HEX(c1), HEX(WEIGHT_STRING(c1)) FROM t1;
+------+---------+------------------------+
| c1   | HEX(c1) | HEX(WEIGHT_STRING(c1)) |
+------+---------+------------------------+
| a    | 61      | 0041                   |
| A    | 41      | 0041                   |
| À    | C380    | 0041                   |
| á    | C3A1    | 0041                   |
+------+---------+------------------------+
4 rows in set (0.00 sec)
```

MySQL 中基于 UCA 的整理具有以下特性：

+   如果一个字符有权重，每个权重使用 2 个字节（16 位）。

+   一个字符可能有零个权重（或空权重）。在这种情况下，该字符是可忽略的。例如："U+0000 NULL"没有权重，是可忽略的。

+   一个字符可能有一个权重。例如：`'a'`的权重为`0x0E33`。

    ```sql
    mysql> SET NAMES 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
    Query OK, 0 rows affected (0.05 sec)

    mysql> SELECT HEX('a'), HEX(WEIGHT_STRING('a'));
    +----------+-------------------------+
    | HEX('a') | HEX(WEIGHT_STRING('a')) |
    +----------+-------------------------+
    | 61       | 0E33                    |
    +----------+-------------------------+
    1 row in set (0.02 sec)
    ```

+   一个字符可能有多个权重。这是一个扩展。例如：德语字母`'ß'`（SZ 连字，或 SHARP S）的权重为`0x0FEA0FEA`。

    ```sql
    mysql> SET NAMES 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
    Query OK, 0 rows affected (0.11 sec)

    mysql> SELECT HEX('ß'), HEX(WEIGHT_STRING('ß'));
    +-----------+--------------------------+
    | HEX('ß')  | HEX(WEIGHT_STRING('ß'))  |
    +-----------+--------------------------+
    | C39F      | 0FEA0FEA                 |
    +-----------+--------------------------+
    1 row in set (0.00 sec)
    ```

+   多个字符可能有一个权重。这是一个收缩。例如：`'ch'`是捷克语中的一个字母，权重为`0x0EE2`。

    ```sql
    mysql> SET NAMES 'utf8mb4' COLLATE 'utf8mb4_czech_ci';
    Query OK, 0 rows affected (0.09 sec)

    mysql> SELECT HEX('ch'), HEX(WEIGHT_STRING('ch'));
    +-----------+--------------------------+
    | HEX('ch') | HEX(WEIGHT_STRING('ch')) |
    +-----------+--------------------------+
    | 6368      | 0EE2                     |
    +-----------+--------------------------+
    1 row in set (0.00 sec)
    ```

也可能存在多字符对多权重的映射（这是扩展与收缩），但 MySQL 不支持。

对于非 UCA 排序的实现说明，请参见第 12.13 节，“添加字符集”。对于 UCA 排序，请参见第 12.14.4 节，“向 Unicode 字符集添加 UCA 排序”。

**杂项排序**

还有一些排序不属于前述任何类别。
