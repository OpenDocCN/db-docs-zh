# 14.9.7 为全文索引添加用户定义的排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/full-text-adding-collation.html`](https://dev.mysql.com/doc/refman/8.0/en/full-text-adding-collation.html)

警告

用户定义的排序规则已被弃用；您应该期望在未来的 MySQL 版本中删除对它们的支持。从 MySQL 8.0.33 开始，服务器对任何 SQL 语句中使用 `COLLATE *`user_defined_collation`*` 都会发出警告；当服务器以 `--collation-server` 设置为用户定义的排序规则的名称时，也会发出警告。

本节描述如何为使用内置全文解析器进行全文搜索添加用户定义的排序规则。示例排序规则类似于 `latin1_swedish_ci`，但将 `'-'` 字符视为字母而不是标点符号，以便将其索引为单词字符。有关添加排序规则的一般信息在 Section 12.14, “Adding a Collation to a Character Set” 中给出；假定您已经阅读并熟悉了涉及的文件。

要为全文索引添加排序规则，请使用以下过程。这里的说明添加了一个简单字符集的排序规则，如 Section 12.14, “Adding a Collation to a Character Set” 中所讨论的，可以使用描述字符集属性的配置文件来创建。对于像 Unicode 这样的复杂字符集，请使用描述字符集属性的 C 源文件创建排序规则。

1.  在 `Index.xml` 文件中添加一个排序规则。用户定义的排序规则的允许 ID 范围在 Section 12.14.2, “Choosing a Collation ID” 中给出。ID 必须未使用，因此如果系统中已经使用了 ID 1025，则选择一个不同的值。

    ```sql
    <charset name="latin1">
    ...
    <collation name="latin1_fulltext_ci" id="1025"/>
    </charset>
    ```

1.  在 `latin1.xml` 文件中声明排序规则的排序顺序。在这种情况下，排序顺序可以从 `latin1_swedish_ci` 复制：

    ```sql
    <collation name="latin1_fulltext_ci">
    <map>
    00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
    10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F
    20 21 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F
    30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F
    40 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F
    50 51 52 53 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F
    60 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F
    50 51 52 53 54 55 56 57 58 59 5A 7B 7C 7D 7E 7F
    80 81 82 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F
    90 91 92 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F
    A0 A1 A2 A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF
    B0 B1 B2 B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF
    41 41 41 41 5C 5B 5C 43 45 45 45 45 49 49 49 49
    44 4E 4F 4F 4F 4F 5D D7 D8 55 55 55 59 59 DE DF
    41 41 41 41 5C 5B 5C 43 45 45 45 45 49 49 49 49
    44 4E 4F 4F 4F 4F 5D F7 D8 55 55 55 59 59 DE FF
    </map>
    </collation>
    ```

1.  修改 `latin1.xml` 中的 `ctype` 数组。将对应于 `'-'` 字符的代码 0x2D（即连字符的代码）的值从 10（标点符号）更改为 01（大写字母）。在下面的数组中，这是从下面第四行开始，从末尾数第三个值的元素。

    ```sql
    <ctype>
    <map>
    00
    20 20 20 20 20 20 20 20 20 28 28 28 28 28 20 20
    20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20
    48 10 10 10 10 10 10 10 10 10 10 10 10 *01* 10 10
    84 84 84 84 84 84 84 84 84 84 10 10 10 10 10 10
    10 81 81 81 81 81 81 01 01 01 01 01 01 01 01 01
    01 01 01 01 01 01 01 01 01 01 01 10 10 10 10 10
    10 82 82 82 82 82 82 02 02 02 02 02 02 02 02 02
    02 02 02 02 02 02 02 02 02 02 02 10 10 10 10 20
    10 00 10 02 10 10 10 10 10 10 01 10 01 00 01 00
    00 10 10 10 10 10 10 10 10 10 02 10 02 00 02 01
    48 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10
    10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10
    01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01
    01 01 01 01 01 01 01 10 01 01 01 01 01 01 01 02
    02 02 02 02 02 02 02 02 02 02 02 02 02 02 02 02
    02 02 02 02 02 02 02 10 02 02 02 02 02 02 02 02
    </map>
    </ctype>
    ```

1.  重新启动服务器。

1.  要使用新的排序规则，将其包含在要使用它的列的定义中：

    ```sql
    mysql> DROP TABLE IF EXISTS t1;
    Query OK, 0 rows affected (0.13 sec)

    mysql> CREATE TABLE t1 (
        a TEXT CHARACTER SET latin1 COLLATE latin1_fulltext_ci,
        FULLTEXT INDEX(a)
        ) ENGINE=InnoDB;
    Query OK, 0 rows affected (0.47 sec)
    ```

1.  测试排序规则，以验证连字符被视为单词字符：

    ```sql
    mysql> INSERT INTO t1 VALUEs ('----'),('....'),('abcd');
    Query OK, 3 rows affected (0.22 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> SELECT * FROM t1 WHERE MATCH a AGAINST ('----' IN BOOLEAN MODE);
    +------+
    | a    |
    +------+
    | ---- |
    +------+
    1 row in set (0.00 sec)
    ```
