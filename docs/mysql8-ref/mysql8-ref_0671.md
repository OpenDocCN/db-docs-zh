# 12.2.1 字符集 repertoire

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-repertoire.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-repertoire.html)

字符集的 repertoire 是集合中的字符。

字符串表达式具有 repertoire 属性，可以有两个值：

+   `ASCII`：表达式只能包含 ASCII 字符；即 Unicode 范围`U+0000`到`U+007F`中的字符。

+   `UNICODE`：表达式可以包含 Unicode 范围`U+0000`到`U+10FFFF`中的字符。这包括基本多文种平面（BMP）范围（`U+0000`到`U+FFFF`）中的字符和 BMP 范围之外的补充字符（`U+10000`到`U+10FFFF`）中的字符。

`ASCII`范围是`UNICODE`范围的子集，因此具有`ASCII`repertoire 的字符串可以安全地转换为具有`UNICODE`repertoire 的任何字符串的字符集，而不会丢失信息。它也可以安全地转换为任何`ascii`字符集的超集。（所有 MySQL 字符集都是`ascii`的超集，除了`swe7`，它重新使用一些标点字符作为瑞典重音字符。）

使用 repertoire 使得在许多情况下可以进行字符集转换，否则 MySQL 在协定强制性规则无法解决歧义时会返回“collations 混合不合法”错误。（有关强制性的信息，请参见 Section 12.8.4, “Collation Coercibility in Expressions”.）

以下讨论提供了表达式及其 repertoire 的示例，并描述了 repertoire 的使用如何改变字符串表达式的评估：

+   字符串常量的 repertoire 取决于字符串内容，可能与字符串字符集的 repertoire 不同。考虑以下语句：

    ```sql
    SET NAMES utf8mb4; SELECT 'abc';
    SELECT _utf8mb4'def';
    ```

    尽管在前述每种情况中字符集为`utf8mb4`，但实际上字符串并不包含任何 ASCII 范围之外的字符，因此它们的 repertoire 是`ASCII`而不是`UNICODE`。

+   具有`ascii`字符集的列具有`ASCII`repertoire，因为其字符集。在下表中，`c1`具有`ASCII`repertoire：

    ```sql
    CREATE TABLE t1 (c1 CHAR(1) CHARACTER SET ascii);
    ```

    以下示例说明了 repertoire 如何使得在没有 repertoire 的情况下发生错误时能够确定结果：

    ```sql
    CREATE TABLE t1 (
      c1 CHAR(1) CHARACTER SET latin1,
      c2 CHAR(1) CHARACTER SET ascii
    );
    INSERT INTO t1 VALUES ('a','b');
    SELECT CONCAT(c1,c2) FROM t1;
    ```

    没有 repertoire，会出现以下错误：

    ```sql
    ERROR 1267 (HY000): Illegal mix of collations (latin1_swedish_ci,IMPLICIT)
    and (ascii_general_ci,IMPLICIT) for operation 'concat'
    ```

    使用 repertoire，可以发生从子集到超集（`ascii`到`latin1`）的转换，并返回结果：

    ```sql
    +---------------+
    | CONCAT(c1,c2) |
    +---------------+
    | ab            |
    +---------------+
    ```

+   具有一个字符串参数的函数继承其参数的 repertoire。`UPPER(_utf8mb4'abc')`的结果具有`ASCII`repertoire，因为其参数具有`ASCII`repertoire。（尽管有`_utf8mb4`引导符，字符串`'abc'`不包含 ASCII 范围之外的字符。）

+   对于返回字符串但不具有字符串参数并使用`character_set_connection`作为结果字符集的函数，如果`character_set_connection`是`ascii`，则结果字符集是`ASCII`，否则是`UNICODE`：

    ```sql
    FORMAT(*numeric_column*, 4);
    ```

    使用字符集会改变 MySQL 评估以下示例的方式：

    ```sql
    SET NAMES ascii;
    CREATE TABLE t1 (a INT, b VARCHAR(10) CHARACTER SET latin1);
    INSERT INTO t1 VALUES (1,'b');
    SELECT CONCAT(FORMAT(a, 4), b) FROM t1;
    ```

    没有字符集，将出现以下错误：

    ```sql
    ERROR 1267 (HY000): Illegal mix of collations (ascii_general_ci,COERCIBLE)
    and (latin1_swedish_ci,IMPLICIT) for operation 'concat'
    ```

    通过字符集，返回结果：

    ```sql
    +-------------------------+
    | CONCAT(FORMAT(a, 4), b) |
    +-------------------------+
    | 1.0000b                 |
    +-------------------------+
    ```

+   具有两个或更多字符串参数的函数使用结果字符集的“最宽”参数字符集，其中`UNICODE`比`ASCII`更宽。考虑以下`CONCAT()`调用：

    ```sql
    CONCAT(_ucs2 X'0041', _ucs2 X'0042')
    CONCAT(_ucs2 X'0041', _ucs2 X'00C2')
    ```

    对于第一个调用，字符集是`ASCII`，因为两个参数都在 ASCII 范围内。对于第二个调用，字符集是`UNICODE`，因为第二个参数超出了 ASCII 范围。

+   函数返回值的字符集是基于仅影响结果字符集和排序的参数的字符集确定的。

    ```sql
    IF(column1 < column2, 'smaller', 'greater')
    ```

    结果字符集是`ASCII`，因为两个字符串参数（第二个参数和第三个参数）都具有`ASCII`字符集。第一个参数对结果字符集没有影响，即使表达式使用字符串值。
