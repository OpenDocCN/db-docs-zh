# 14.23 杂项函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html)

**表 14.33 杂项函数**

| 名称 | 描述 |
| --- | --- |
| `ANY_VALUE()` | 抑制`ONLY_FULL_GROUP_BY`值拒绝 |
| `BIN_TO_UUID()` | 将二进制 UUID 转换为字符串 |
| `DEFAULT()` | 返回表列的默认值 |
| `GROUPING()` | 区分超级聚合 ROLLUP 行和常规行 |
| `INET_ATON()` | 返回 IP 地址的数值 |
| `INET_NTOA()` | 返回数值的 IP 地址 |
| `INET6_ATON()` | 返回 IPv6 地址的数值 |
| `INET6_NTOA()` | 返回数值的 IPv6 地址 |
| `IS_IPV4()` | 参数是否为 IPv4 地址 |
| `IS_IPV4_COMPAT()` | 参数是否为 IPv4 兼容地址 |
| `IS_IPV4_MAPPED()` | 参数是否为 IPv4 映射地址 |
| `IS_IPV6()` | 参数是否为 IPv6 地址 |
| `IS_UUID()` | 参数是否为有效的 UUID |
| `NAME_CONST()` | 使列具有给定名称 |
| `SLEEP()` | 休眠若干秒 |
| `UUID()` | 返回通用唯一标识符（UUID） |
| `UUID_SHORT()` | 返回整数值的通用标识符 |
| `UUID_TO_BIN()` | 将字符串 UUID 转换为二进制 |
| `VALUES()` | 定义在插入期间要使用的值 |
| 名称 | 描述 |

+   `ANY_VALUE(*`arg`*)`

    当启用`ONLY_FULL_GROUP_BY` SQL 模式时，此函数对`GROUP BY`查询很有用，用于 MySQL 拒绝你知道是有效的查询的情况，但 MySQL 无法确定拒绝的原因。函数的返回值和类型与其参数的返回值和类型相同，但函数结果不会被检查`ONLY_FULL_GROUP_BY` SQL 模式。

    例如，如果`name`是一个非索引列，在启用`ONLY_FULL_GROUP_BY`的情况下，以下查询将失败：

    ```sql
    mysql> SELECT name, address, MAX(age) FROM t GROUP BY name;
    ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP
    BY clause and contains nonaggregated column 'mydb.t.address' which
    is not functionally dependent on columns in GROUP BY clause; this
    is incompatible with sql_mode=only_full_group_by
    ```

    失败的原因是`address`是一个非聚合列，既不在`GROUP BY`列中，也不在函数上依赖于它们。因此，每个`name`组内的行的`address`值是不确定的。有多种方法可以使 MySQL 接受查询：

    +   修改表，使`name`成为主键或唯一的`NOT NULL`列。这样 MySQL 就可以确定`address`在`name`上是函数上依赖的；也就是说，`address`是由`name`唯一确定的。（如果`NULL`必须被允许作为有效的`name`值，则此技术不适用。）

    +   使用`ANY_VALUE()`来引用`address`：

        ```sql
        SELECT name, ANY_VALUE(address), MAX(age) FROM t GROUP BY name;
        ```

        在这种情况下，MySQL 忽略了每个`name`组内`address`值的不确定性，并接受了查询。如果您只是不关心为每个组选择哪个非聚合列的值，那么这可能是有用的。`ANY_VALUE()`不是一个聚合函数，不像`SUM()`或`COUNT()`等函数。它只是用来抑制不确定性测试的。

    +   禁用`ONLY_FULL_GROUP_BY`。这相当于在启用`ONLY_FULL_GROUP_BY`的情况下使用`ANY_VALUE()`，如前一项所述。

    如果列之间存在函数依赖关系，但 MySQL 无法确定，那么`ANY_VALUE()`也是有用的。以下查询是有效的，因为`age`在分组列`age-1`上是函数上依赖的，但 MySQL 无法判断，并在启用`ONLY_FULL_GROUP_BY`时拒绝查询：

    ```sql
    SELECT age FROM t GROUP BY age-1;
    ```

    要使 MySQL 接受查询，请使用`ANY_VALUE()`：

    ```sql
    SELECT ANY_VALUE(age) FROM t GROUP BY age-1;
    ```

    在没有`GROUP BY`子句的情况下，可以使用`ANY_VALUE()`来引用聚合函数：

    ```sql
    mysql> SELECT name, MAX(age) FROM t;
    ERROR 1140 (42000): In aggregated query without GROUP BY, expression
    #1 of SELECT list contains nonaggregated column 'mydb.t.name'; this
    is incompatible with sql_mode=only_full_group_by
    ```

    没有`GROUP BY`，只有一个组，选择哪个`name`值对于该组是不确定的。`ANY_VALUE()`告诉 MySQL 接受查询：

    ```sql
    SELECT ANY_VALUE(name), MAX(age) FROM t;
    ```

    也许，由于给定数据集的某些属性，您知道所选的非聚合列实际上是函数上依赖于`GROUP BY`列的。例如，一个应用程序可能强制一个列相对于另一个列的唯一性。在这种情况下，对于实际上是函数上依赖的列使用`ANY_VALUE()`可能是有意义的。

    有关更多讨论，请参阅 第 14.19.3 节，“MySQL 对 GROUP BY 的处理”。

+   `BIN_TO_UUID(*`binary_uuid`*)`, `BIN_TO_UUID(*`binary_uuid`*, *`swap_flag`*)`

    `BIN_TO_UUID()` 是 `UUID_TO_BIN()` 的逆操作。它将二进制 UUID 转换为字符串 UUID 并返回结果。二进制值应为 `VARBINARY(16)` 值的 UUID。返回值是由短横线分隔的五个十六进制数字组成的字符串。（有关此格式的详细信息，请参阅 `UUID()` 函数描述。）如果 UUID 参数为 `NULL`，则返回值为 `NULL`。如果任何参数无效，则会出现错误。

    `BIN_TO_UUID()` 接受一个或两个参数：

    +   一参数形式接受一个二进制 UUID 值。假定 UUID 值未交换其时间低位和时间高位部分。字符串结果与二进制参数的顺序相同。

    +   两参数形式接受一个二进制 UUID 值和一个交换标志值：

        +   如果 *`swap_flag`* 为 0，则两参数形式等同于一参数形式。字符串结果与二进制参数的顺序相同。

        +   如果 *`swap_flag`* 为 1，则假定 UUID 值已交换其时间低位和时间高位部分。这些部分在结果值中被交换回其原始位置。

    有关用法示例和有关时间部分交换的信息，请参阅 `UUID_TO_BIN()` 函数描述。

+   `DEFAULT(*`col_name`*)`

    返回表列的默认值。如果列没有默认值，则会出现错误。

    使用 `DEFAULT(*`col_name`*)` 来指定命名列的默认值仅适用于具有文字默认值而不是表达式默认值的列。

    ```sql
    mysql> UPDATE t SET i = DEFAULT(i)+1 WHERE id < 100;
    ```

+   `FORMAT(*`X`*,*`D`*)`

    将数字 *`X`* 格式化为类似 `'#,###,###.##'` 的格式，四舍五入到 *`D`* 小数位，并将结果作为字符串返回。有关详细信息，请参阅 第 14.8 节，“字符串函数和运算符”。

+   [`GROUPING(*`expr`* [, *`expr`*] ...)`](miscellaneous-functions.html#function_grouping)

    对于包含`WITH ROLLUP`修饰符的`GROUP BY`查询，`ROLLUP`操作会生成超级聚合输出行，其中`NULL`表示所有值的集合。`GROUPING()`函数使您能够区分超级聚合行中的`NULL`值和常规分组行中的`NULL`值。

    `GROUPING()`允许在选择列表、`HAVING`子句和（自 MySQL 8.0.12 起）`ORDER BY`子句中使用。

    每个`GROUPING()`的参数必须是与`GROUP BY`子句中的表达式完全匹配的表达式。表达式不能是位置指示符。对于每个表达式，如果当前行中表达式的值是代表超级聚合值的`NULL`，则`GROUPING()`会产生 1。否则，`GROUPING()`会产生 0，表示表达式值是常规结果行的`NULL`或不是`NULL`。

    假设表`t1`包含以下行，其中`NULL`表示类似于“其他”或“未知”的内容：

    ```sql
    mysql> SELECT * FROM t1;
    +------+-------+----------+
    | name | size  | quantity |
    +------+-------+----------+
    | ball | small |       10 |
    | ball | large |       20 |
    | ball | NULL  |        5 |
    | hoop | small |       15 |
    | hoop | large |        5 |
    | hoop | NULL  |        3 |
    +------+-------+----------+
    ```

    没有`WITH ROLLUP`的表格摘要如下所示：

    ```sql
    mysql> SELECT name, size, SUM(quantity) AS quantity
           FROM t1
           GROUP BY name, size;
    +------+-------+----------+
    | name | size  | quantity |
    +------+-------+----------+
    | ball | small |       10 |
    | ball | large |       20 |
    | ball | NULL  |        5 |
    | hoop | small |       15 |
    | hoop | large |        5 |
    | hoop | NULL  |        3 |
    +------+-------+----------+
    ```

    结果包含`NULL`值，但这些值不代表超级聚合行，因为查询中没有包含`WITH ROLLUP`。

    添加`WITH ROLLUP`会生成包含额外`NULL`值的超级聚合摘要行。然而，如果不将此结果与先前的结果进行比较，就不容易看出哪些`NULL`值出现在超级聚合行中，哪些出现在常规分组行中：

    ```sql
    mysql> SELECT name, size, SUM(quantity) AS quantity
           FROM t1
           GROUP BY name, size WITH ROLLUP;
    +------+-------+----------+
    | name | size  | quantity |
    +------+-------+----------+
    | ball | NULL  |        5 |
    | ball | large |       20 |
    | ball | small |       10 |
    | ball | NULL  |       35 |
    | hoop | NULL  |        3 |
    | hoop | large |        5 |
    | hoop | small |       15 |
    | hoop | NULL  |       23 |
    | NULL | NULL  |       58 |
    +------+-------+----------+
    ```

    要区分超级聚合行中的`NULL`值和常规分组行中的`NULL`值，使用`GROUPING()`，它仅对超级聚合的`NULL`值返回 1：

    ```sql
    mysql> SELECT
             name, size, SUM(quantity) AS quantity,
             GROUPING(name) AS grp_name,
             GROUPING(size) AS grp_size
           FROM t1
           GROUP BY name, size WITH ROLLUP;
    +------+-------+----------+----------+----------+
    | name | size  | quantity | grp_name | grp_size |
    +------+-------+----------+----------+----------+
    | ball | NULL  |        5 |        0 |        0 |
    | ball | large |       20 |        0 |        0 |
    | ball | small |       10 |        0 |        0 |
    | ball | NULL  |       35 |        0 |        1 |
    | hoop | NULL  |        3 |        0 |        0 |
    | hoop | large |        5 |        0 |        0 |
    | hoop | small |       15 |        0 |        0 |
    | hoop | NULL  |       23 |        0 |        1 |
    | NULL | NULL  |       58 |        1 |        1 |
    +------+-------+----------+----------+----------+
    ```

    `GROUPING()`的常见用途：

    +   为超级聚合的`NULL`值替换标签：

        ```sql
        mysql> SELECT
                 IF(GROUPING(name) = 1, 'All items', name) AS name,
                 IF(GROUPING(size) = 1, 'All sizes', size) AS size,
                 SUM(quantity) AS quantity
               FROM t1
               GROUP BY name, size WITH ROLLUP;
        +-----------+-----------+----------+
        | name      | size      | quantity |
        +-----------+-----------+----------+
        | ball      | NULL      |        5 |
        | ball      | large     |       20 |
        | ball      | small     |       10 |
        | ball      | All sizes |       35 |
        | hoop      | NULL      |        3 |
        | hoop      | large     |        5 |
        | hoop      | small     |       15 |
        | hoop      | All sizes |       23 |
        | All items | All sizes |       58 |
        +-----------+-----------+----------+
        ```

    +   通过过滤掉常规分组行，只返回超级聚合行：

        ```sql
        mysql> SELECT name, size, SUM(quantity) AS quantity
               FROM t1
               GROUP BY name, size WITH ROLLUP
               HAVING GROUPING(name) = 1 OR GROUPING(size) = 1;
        +------+------+----------+
        | name | size | quantity |
        +------+------+----------+
        | ball | NULL |       35 |
        | hoop | NULL |       23 |
        | NULL | NULL |       58 |
        +------+------+----------+
        ```

    `GROUPING()`允许多个表达式参数。在这种情况下，`GROUPING()`的返回值代表从每个表达式的结果组合而成的位掩码，其中最低位对应最右边表达式的结果。例如，对于三个表达式参数，`GROUPING(*`expr1`*, *`expr2`*, *`expr3`*)`的计算如下：

    ```sql
     result for GROUPING(*expr3*)
    + result for GROUPING(*expr2*) << 1
    + result for GROUPING(*expr1*) << 2
    ```

    以下查询展示了单个参数的`GROUPING()`结果如何组合为多参数调用以生成位掩码值：

    ```sql
    mysql> SELECT
             name, size, SUM(quantity) AS quantity,
             GROUPING(name) AS grp_name,
             GROUPING(size) AS grp_size,
           GROUPING(name, size) AS grp_all
           FROM t1
           GROUP BY name, size WITH ROLLUP;
    +------+-------+----------+----------+----------+---------+
    | name | size  | quantity | grp_name | grp_size | grp_all |
    +------+-------+----------+----------+----------+---------+
    | ball | NULL  |        5 |        0 |        0 |       0 |
    | ball | large |       20 |        0 |        0 |       0 |
    | ball | small |       10 |        0 |        0 |       0 |
    | ball | NULL  |       35 |        0 |        1 |       1 |
    | hoop | NULL  |        3 |        0 |        0 |       0 |
    | hoop | large |        5 |        0 |        0 |       0 |
    | hoop | small |       15 |        0 |        0 |       0 |
    | hoop | NULL  |       23 |        0 |        1 |       1 |
    | NULL | NULL  |       58 |        1 |        1 |       3 |
    +------+-------+----------+----------+----------+---------+
    ```

    对于多个表达式参数，如果任何表达式代表超级聚合值，则`GROUPING()`的返回值为非零。因此，多参数`GROUPING()`语法提供了一种更简单的方法来编写仅返回超级聚合行的早期查询，通过使用单个多参数`GROUPING()`调用而不是多个单参数调用：

    ```sql
    mysql> SELECT name, size, SUM(quantity) AS quantity
           FROM t1
           GROUP BY name, size WITH ROLLUP
           HAVING GROUPING(name, size) <> 0;
    +------+------+----------+
    | name | size | quantity |
    +------+------+----------+
    | ball | NULL |       35 |
    | hoop | NULL |       23 |
    | NULL | NULL |       58 |
    +------+------+----------+
    ```

    使用`GROUPING()`受到以下限制：

    +   不要将子查询`GROUP BY`表达式用作`GROUPING()`参数，因为匹配可能失败。例如，对于此查询，匹配失败：

        ```sql
        mysql> SELECT GROUPING((SELECT MAX(name) FROM t1))
               FROM t1
               GROUP BY (SELECT MAX(name) FROM t1) WITH ROLLUP;
        ERROR 3580 (HY000): Argument #1 of GROUPING function is not in GROUP BY
        ```

    +   不应在`HAVING`子句中使用`GROUP BY`文字表达式作为`GROUPING()`参数。由于优化器评估`GROUP BY`和`HAVING`的时间差异，匹配可能成功，但`GROUPING()`的评估并不产生预期的结果。考虑以下查询：

        ```sql
        SELECT a AS f1, 'w' AS f2
        FROM t
        GROUP BY f1, f2 WITH ROLLUP
        HAVING GROUPING(f2) = 1;
        ```

        `GROUPING()`在整个`HAVING`子句之前对文字常量表达式进行评估，并返回 0。要检查是否受到影响，可以使用`EXPLAIN`并查找`Extra`列中的`Impossible having`。

    有关`WITH ROLLUP`和`GROUPING()`的更多信息，请参见第 14.19.2 节，“GROUP BY 修饰符”。

+   `INET_ATON(*`expr`*)`

    给定一个作为字符串的 IPv4 网络地址的点分十进制表示，返回一个代表地址在网络字节顺序（大端）中的数值的整数。如果`INET_ATON()`不理解其参数，或者*`expr`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT INET_ATON('10.0.5.9');
     -> 167773449
    ```

    对于此示例，返回值计算为 10×256³ + 0×256² + 5×256 + 9。

    对于简短形式的 IP 地址（例如`'127.1'`表示`'127.0.0.1'`），`INET_ATON()`可能会或可能不会返回非`NULL`结果。因此，不应该对这样的地址使用`INET_ATON()`。

    注意

    为了存储`INET_ATON()`生成的值，请使用`INT UNSIGNED`列，而不是带有符号的`INT`。如果使用带符号的列，无法正确存储首个八位组大于 127 的 IP 地址对应的值。参见第 13.1.7 节，“超出范围和溢出处理”。

+   `INET_NTOA(*`expr`*)`

    给定以网络字节顺序表示的数字 IPv4 网络地址，返回地址的点分十进制字符串表示形式作为连接字符集中的字符串。如果不理解其参数，`INET_NTOA()`返回`NULL`。

    ```sql
    mysql> SELECT INET_NTOA(167773449);
     -> '10.0.5.9'
    ```

+   `INET6_ATON(*`expr`*)`

    给定一个作为字符串的 IPv6 或 IPv4 网络地址，返回表示地址的数字值的二进制字符串，以网络字节顺序（大端）表示。因为数值格式的 IPv6 地址所需的字节数比最大整数类型还要多，所以此函数返回的表示具有`VARBINARY`数据类型的表示：IPv6 地址为`VARBINARY(16)`，IPv4 地址为`VARBINARY(4)`。如果参数不是有效地址，或者为`NULL`，`INET6_ATON()`返回`NULL`。

    以下示例使用`HEX()`以可打印形式显示`INET6_ATON()`的结果：

    ```sql
    mysql> SELECT HEX(INET6_ATON('fdfe::5a55:caff:fefa:9089'));
     -> 'FDFE0000000000005A55CAFFFEFA9089'
    mysql> SELECT HEX(INET6_ATON('10.0.5.9'));
     -> '0A000509'
    ```

    `INET6_ATON()`对有效参数施加了几个约束。以下列出这些约束以及示例。

    +   不允许使用尾随区域 ID，如`fe80::3%1`或`fe80::3%eth0`。

    +   不允许使用尾随网络掩码，如`2001:45f:3:ba::/64`或`198.51.100.0/24`。

    +   对于表示 IPv4 地址的值，仅支持无类地址。类地址（如`198.51.1`）将被拒绝。不允许使用尾随端口号，如`198.51.100.2:8080`。地址组件中不允许使用十六进制数字，如`198.0xa0.1.2`。不支持八进制数字：`198.51.010.1`被视为`198.51.10.1`，而不是`198.51.8.1`。这些 IPv4 约束也适用于具有 IPv4 地址部分的 IPv6 地址，如 IPv4 兼容或 IPv4 映射地址。

    要将以`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")值表示的 IPv4 地址*`expr`*转换为以`VARBINARY`值表示的 IPv6 地址，使用以下表达式：

    ```sql
    INET6_ATON(INET_NTOA(*expr*))
    ```

    例如：

    ```sql
    mysql> SELECT HEX(INET6_ATON(INET_NTOA(167773449)));
     -> '0A000509'
    ```

    如果在**mysql**客户端中调用`INET6_ATON()`，二进制字符串将使用十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — The MySQL Command-Line Client”。

+   `INET6_NTOA(*`expr`*)`

    给定以二进制字符串形式表示的 IPv6 或 IPv4 网络地址，将返回连接字符集中的地址字符串表示。如果参数不是有效地址，或者为`NULL`，`INET6_NTOA()`将返回`NULL`。

    `INET6_NTOA()`具有以下属性：

    +   它不使用操作系统函数执行转换，因此输出字符串是与平台无关的。

    +   返回字符串的最大长度为 39（4 x 8 + 7）。给出这个语句：

        ```sql
        CREATE TABLE t AS SELECT INET6_NTOA(*expr*) AS c1;
        ```

        结果表将具有以下定义：

        ```sql
        CREATE TABLE t (c1 VARCHAR(39) CHARACTER SET utf8mb3 DEFAULT NULL);
        ```

    +   返回字符串使用小写字母表示 IPv6 地址。

    ```sql
    mysql> SELECT INET6_NTOA(INET6_ATON('fdfe::5a55:caff:fefa:9089'));
     -> 'fdfe::5a55:caff:fefa:9089'
    mysql> SELECT INET6_NTOA(INET6_ATON('10.0.5.9'));
     -> '10.0.5.9'

    mysql> SELECT INET6_NTOA(UNHEX('FDFE0000000000005A55CAFFFEFA9089'));
     -> 'fdfe::5a55:caff:fefa:9089'
    mysql> SELECT INET6_NTOA(UNHEX('0A000509'));
     -> '10.0.5.9'
    ```

    如果在**mysql**客户端中调用`INET6_NTOA()`，二进制字符串将使用十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — The MySQL Command-Line Client”。

+   `IS_IPV4(*`expr`*)`

    如果参数作为字符串指定的 IPv4 地址有效，则返回 1，否则返回 0。如果*`expr`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT IS_IPV4('10.0.5.9'), IS_IPV4('10.0.5.256');
     -> 1, 0
    ```

    对于给定的参数，如果`IS_IPV4()`返回 1，则`INET_ATON()`（以及`INET6_ATON()`）返回非`NULL`。反之则不成立：在某些情况下，当`IS_IPV4()`返回 0 时，`INET_ATON()`返回非`NULL`。

    如前述所示，`IS_IPV4()`对于何为有效的 IPv4 地址更为严格，因此对于需要对无效值进行强检查的应用程序可能很有用。或者，使用`INET6_ATON()`将 IPv4 地址转换为内部形式并检查`NULL`结果（表示无效地址）。`INET6_ATON()`在检查 IPv4 地址方面与`IS_IPV4()`一样强大。

+   `IS_IPV4_COMPAT(*`expr`*)`

    此函数接受以二进制字符串形式表示的数字形式的 IPv6 地址，如`INET6_ATON()`返回的。如果参数是有效的 IPv4 兼容 IPv6 地址，则返回 1，否则返回 0（除非*`expr`*为`NULL`，在这种情况下函数返回`NULL`）。IPv4 兼容地址的形式为`::*`ipv4_address`*`。

    ```sql
    mysql> SELECT IS_IPV4_COMPAT(INET6_ATON('::10.0.5.9'));
     -> 1
    mysql> SELECT IS_IPV4_COMPAT(INET6_ATON('::ffff:10.0.5.9'));
     -> 0
    ```

    IPv4 兼容地址的 IPv4 部分也可以使用十六进制表示。例如，`198.51.100.1`具有以下原始十六进制值：

    ```sql
    mysql> SELECT HEX(INET6_ATON('198.51.100.1'));
     -> 'C6336401'
    ```

    以 IPv4 兼容形式表示，`::198.51.100.1`等同于`::c0a8:0001`或（去掉前导零）`::c0a8:1`

    ```sql
    mysql> SELECT
     ->   IS_IPV4_COMPAT(INET6_ATON('::198.51.100.1')),
     ->   IS_IPV4_COMPAT(INET6_ATON('::c0a8:0001')),
     ->   IS_IPV4_COMPAT(INET6_ATON('::c0a8:1'));
     -> 1, 1, 1
    ```

+   `IS_IPV4_MAPPED(*`expr`*)`

    此函数接受以二进制字符串形式表示的数字形式的 IPv6 地址，如`INET6_ATON()`返回的。如果参数是有效的 IPv4 映射 IPv6 地址，则返回 1，否则返回 0，除非*`expr`*为`NULL`，在这种情况下函数返回`NULL`。IPv4 映射地址的形式为`::ffff:*`ipv4_address`*`。

    ```sql
    mysql> SELECT IS_IPV4_MAPPED(INET6_ATON('::10.0.5.9'));
     -> 0
    mysql> SELECT IS_IPV4_MAPPED(INET6_ATON('::ffff:10.0.5.9'));
     -> 1
    ```

    与`IS_IPV4_COMPAT()`一样，IPv4 映射地址的 IPv4 部分也可以使用十六进制表示：

    ```sql
    mysql> SELECT
     ->   IS_IPV4_MAPPED(INET6_ATON('::ffff:198.51.100.1')),
     ->   IS_IPV4_MAPPED(INET6_ATON('::ffff:c0a8:0001')),
     ->   IS_IPV4_MAPPED(INET6_ATON('::ffff:c0a8:1'));
     -> 1, 1, 1
    ```

+   `IS_IPV6(*`expr`*)`

    如果参数是以字符串形式指定的有效 IPv6 地址，则返回 1，否则返回 0，除非*`expr`*为`NULL`，在这种情况下函数返回`NULL`。此函数不认为 IPv4 地址是有效的 IPv6 地址。

    ```sql
    mysql> SELECT IS_IPV6('10.0.5.9'), IS_IPV6('::1');
     -> 0, 1
    ```

    对于给定的参数，如果`IS_IPV6()`返回 1，则`INET6_ATON()`返回非`NULL`。

+   `IS_UUID(*`string_uuid`*)`

    如果参数是有效的字符串格式 UUID，则返回 1，如果参数不是有效的 UUID，则返回 0，如果参数为`NULL`，则返回`NULL`。

    “有效”意味着该值以可解析的格式存在。也就是说，它具有正确的长度并且仅包含允许的字符（十六进制数字以任何大小写字母形式，可选地包括短横线和大括号）。这种格式最常见：

    ```sql
    aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
    ```

    还允许这些其他格式：

    ```sql
    aaaaaaaabbbbccccddddeeeeeeeeeeee
    {aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee}
    ```

    有关值内字段的含义，请参阅`UUID()`函数描述。

    ```sql
    mysql> SELECT IS_UUID('6ccd780c-baba-1026-9564-5b8c656024db');
    +-------------------------------------------------+
    | IS_UUID('6ccd780c-baba-1026-9564-5b8c656024db') |
    +-------------------------------------------------+
    |                                               1 |
    +-------------------------------------------------+
    mysql> SELECT IS_UUID('6CCD780C-BABA-1026-9564-5B8C656024DB');
    +-------------------------------------------------+
    | IS_UUID('6CCD780C-BABA-1026-9564-5B8C656024DB') |
    +-------------------------------------------------+
    |                                               1 |
    +-------------------------------------------------+
    mysql> SELECT IS_UUID('6ccd780cbaba102695645b8c656024db');
    +---------------------------------------------+
    | IS_UUID('6ccd780cbaba102695645b8c656024db') |
    +---------------------------------------------+
    |                                           1 |
    +---------------------------------------------+
    mysql> SELECT IS_UUID('{6ccd780c-baba-1026-9564-5b8c656024db}');
    +---------------------------------------------------+
    | IS_UUID('{6ccd780c-baba-1026-9564-5b8c656024db}') |
    +---------------------------------------------------+
    |                                                 1 |
    +---------------------------------------------------+
    mysql> SELECT IS_UUID('6ccd780c-baba-1026-9564-5b8c6560');
    +---------------------------------------------+
    | IS_UUID('6ccd780c-baba-1026-9564-5b8c6560') |
    +---------------------------------------------+
    |                                           0 |
    +---------------------------------------------+
    mysql> SELECT IS_UUID(RAND());
    +-----------------+
    | IS_UUID(RAND()) |
    +-----------------+
    |               0 |
    +-----------------+
    ```

+   `NAME_CONST(*`name`*,*`value`*)`

    返回给定的值。当用于生成结果集列时，`NAME_CONST()`使列具有给定的名称。参数应为常量。

    ```sql
    mysql> SELECT NAME_CONST('myname', 14);
    +--------+
    | myname |
    +--------+
    |     14 |
    +--------+
    ```

    此函数仅供内部使用。服务器在编写包含对本地程序变量的引用的存储程序语句时使用它，如第 27.7 节“存储程序二进制日志记录”中所述。您可能会在`mysqlbinlog**的输出中看到此函数。

    对于您的应用程序，您可以通过简单的别名来获得与刚刚显示的示例完全相同的结果，如下所示：

    ```sql
    mysql> SELECT 14 AS myname;
    +--------+
    | myname |
    +--------+
    |     14 |
    +--------+
    1 row in set (0.00 sec)
    ```

    有关列别名的更多信息，请参阅第 15.2.13 节“SELECT 语句”。

+   `SLEEP(*`duration`*)`

    休眠（暂停）由*`duration`*参数给定的秒数，然后返回 0。持续时间可能有小数部分。如果参数为`NULL`或负数，`SLEEP()`会产生警告，在严格的 SQL 模式下会产生错误。

    当休眠正常返回（没有中断）时，它返回 0：

    ```sql
    mysql> SELECT SLEEP(1000);
    +-------------+
    | SLEEP(1000) |
    +-------------+
    |           0 |
    +-------------+
    ```

    当`SLEEP()`是唯一被查询中断的事物时，它返回 1，查询本身不返回错误。无论查询是被终止还是超时，这都是正确的：

    +   这个语句是通过另一个会话中的`KILL QUERY`中断的：

        ```sql
        mysql> SELECT SLEEP(1000);
        +-------------+
        | SLEEP(1000) |
        +-------------+
        |           1 |
        +-------------+
        ```

    +   这个语句由超时中断：

        ```sql
        mysql> SELECT /*+ MAX_EXECUTION_TIME(1) */ SLEEP(1000);
        +-------------+
        | SLEEP(1000) |
        +-------------+
        |           1 |
        +-------------+
        ```

    当`SLEEP()`只是被中断查询的一部分时，查询会返回错误：

    +   这个语句是通过另一个会话中的`KILL QUERY`中断的：

        ```sql
        mysql> SELECT 1 FROM t1 WHERE SLEEP(1000);
        ERROR 1317 (70100): Query execution was interrupted
        ```

    +   这个语句是通过超时中断的：

        ```sql
        mysql> SELECT /*+ MAX_EXECUTION_TIME(1000) */ 1 FROM t1 WHERE SLEEP(1000);
        ERROR 3024 (HY000): Query execution was interrupted, maximum statement
        execution time exceeded
        ```

    此函数对基于语句的复制不安全。如果在`binlog_format`设置为`STATEMENT`时使用此函数，将记录警告。

+   `UUID()`

    返回根据 RFC 4122“通用唯一标识符（UUID）URN 命名空间”（[`www.ietf.org/rfc/rfc4122.txt`](http://www.ietf.org/rfc/rfc4122.txt)）生成的通用唯一标识符（UUID）。

    UUID 被设计为在空间和时间上全局唯一的数字。两次调用`UUID()`预期会生成两个不同的值，即使这些调用是在两个不相互连接的设备上执行的。

    警告

    虽然`UUID()`值旨在是唯一的，但它们不一定是无法猜测或不可预测的。如果需要不可预测性，应以其他方式生成 UUID 值。

    `UUID()`返回一个符合 RFC 4122 中描述的 UUID 版本 1 的值。该值是一个 128 位数字，表示为`utf8mb3`格式的五个十六进制数字，如`aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee`：

    +   前三个数字是从时间戳的低、中和高部分生成的。高部分还包括 UUID 版本号。

    +   第四个数字在时间戳值失去单调性时保留时间上的唯一性（例如，由于夏令时）。

    +   第五个数字是提供空间唯一性的 IEEE 802 节点号。如果后者不可用（例如，因为主机设备没有以太网卡，或者不知道如何在主机操作系统上找到接口的硬件地址），则替换为随机数。在这种情况下，空间唯一性无法保证。尽管如此，碰撞应该具有*非常*低的概率。

        仅在 FreeBSD、Linux 和 Windows 上考虑接口的 MAC 地址。在其他操作系统上，MySQL 使用随机生成的 48 位数字。

    ```sql
    mysql> SELECT UUID();
     -> '6ccd780c-baba-1026-9564-5b8c656024db'
    ```

    要在字符串和二进制 UUID 值之间转换，请使用`UUID_TO_BIN()`和`BIN_TO_UUID()`函数。要检查字符串是否为有效的 UUID 值，请使用`IS_UUID()`函数。

    此函数对基于语句的复制不安全。如果在`binlog_format`设置为`STATEMENT`时使用此函数，将记录警告。

+   `UUID_SHORT()`

    返回一个作为 64 位无符号整数的“短”通用标识符。`UUID_SHORT()`返回的值与`UUID()`函数返回的字符串格式的 128 位标识符不同，并具有不同的唯一性属性。如果满足以下条件，`UUID_SHORT()`的值将保证是唯一的：

    +   当前服务器的`server_id`值介于 0 和 255 之间，并且在您的源服务器和副本服务器集合中是唯一的。

    +   在**mysqld**重新启动之间，不要将系统时间设置回去

    +   在**mysqld**重新启动之间，平均每秒调用`UUID_SHORT()`少于 1600 万次

    `UUID_SHORT()` 返回值构造如下：

    ```sql
     (server_id & 255) << 56
    + (server_startup_time_in_seconds << 24)
    + incremented_variable++;
    ```

    ```sql
    mysql> SELECT UUID_SHORT();
     -> 92395783831158784
    ```

    注意

    `UUID_SHORT()` 不能与基于语句的复制一起使用。

+   `UUID_TO_BIN(*`string_uuid`*)`, `UUID_TO_BIN(*`string_uuid`*, *`swap_flag`*)`

    将字符串 UUID 转换为二进制 UUID 并返回结果。（`IS_UUID()`函数描述列出了允许的字符串 UUID 格式。）返回的二进制 UUID 是`VARBINARY(16)`值。如果 UUID 参数为 `NULL`，则返回值为 `NULL`。如果任何参数无效，则会发生错误。

    `UUID_TO_BIN()` 接受一个或两个参数：

    +   单参数形式接受一个字符串 UUID 值。二进制结果与字符串参数的顺序相同。

    +   两参数形式接受一个字符串 UUID 值和一个标志值：

        +   如果 *`swap_flag`* 为 0，则两参数形式等同于单参数形式。二进制结果与字符串参数的顺序相同。

        +   如果 *`swap_flag`* 为 1，则返回值的格式不同：时间低位和时间高位部分（分别为第一组和第三组十六进制数字）被交换。这将更快变化的部分移到右侧，并且如果结果存储在索引列中，可以提高索引效率。

    时间部分交换假定使用 UUID 版本 1 值，例如由`UUID()`函数生成的值。对于不遵循版本 1 格式的其他方式生成的 UUID 值，时间部分交换不提供任何好处。有关版本 1 格式的详细信息，请参阅`UUID()`函数描述。

    假设您有以下字符串 UUID 值：

    ```sql
    mysql> SET @uuid = '6ccd780c-baba-1026-9564-5b8c656024db';
    ```

    要将字符串 UUID 转换为带有或不带有时间部分交换的二进制，请使用`UUID_TO_BIN()`：

    ```sql
    mysql> SELECT HEX(UUID_TO_BIN(@uuid));
    +----------------------------------+
    | HEX(UUID_TO_BIN(@uuid))          |
    +----------------------------------+
    | 6CCD780CBABA102695645B8C656024DB |
    +----------------------------------+
    mysql> SELECT HEX(UUID_TO_BIN(@uuid, 0));
    +----------------------------------+
    | HEX(UUID_TO_BIN(@uuid, 0))       |
    +----------------------------------+
    | 6CCD780CBABA102695645B8C656024DB |
    +----------------------------------+
    mysql> SELECT HEX(UUID_TO_BIN(@uuid, 1));
    +----------------------------------+
    | HEX(UUID_TO_BIN(@uuid, 1))       |
    +----------------------------------+
    | 1026BABA6CCD780C95645B8C656024DB |
    +----------------------------------+
    ```

    要将`UUID_TO_BIN()`返回的二进制 UUID 转换为字符串 UUID，请使用`BIN_TO_UUID()`。如果通过将第二个参数设置为 1 调用`UUID_TO_BIN()`生成二进制 UUID，则在将二进制 UUID 转换回字符串 UUID 时，还应将第二个参数设置为 1 传递给`BIN_TO_UUID()`以取消时间部分的交换：

    ```sql
    mysql> SELECT BIN_TO_UUID(UUID_TO_BIN(@uuid));
    +--------------------------------------+
    | BIN_TO_UUID(UUID_TO_BIN(@uuid))      |
    +--------------------------------------+
    | 6ccd780c-baba-1026-9564-5b8c656024db |
    +--------------------------------------+
    mysql> SELECT BIN_TO_UUID(UUID_TO_BIN(@uuid,0),0);
    +--------------------------------------+
    | BIN_TO_UUID(UUID_TO_BIN(@uuid,0),0)  |
    +--------------------------------------+
    | 6ccd780c-baba-1026-9564-5b8c656024db |
    +--------------------------------------+
    mysql> SELECT BIN_TO_UUID(UUID_TO_BIN(@uuid,1),1);
    +--------------------------------------+
    | BIN_TO_UUID(UUID_TO_BIN(@uuid,1),1)  |
    +--------------------------------------+
    | 6ccd780c-baba-1026-9564-5b8c656024db |
    +--------------------------------------+
    ```

    如果在两个方向的转换中使用时间部分交换不同，则无法正确恢复原始 UUID：

    ```sql
    mysql> SELECT BIN_TO_UUID(UUID_TO_BIN(@uuid,0),1);
    +--------------------------------------+
    | BIN_TO_UUID(UUID_TO_BIN(@uuid,0),1)  |
    +--------------------------------------+
    | baba1026-780c-6ccd-9564-5b8c656024db |
    +--------------------------------------+
    mysql> SELECT BIN_TO_UUID(UUID_TO_BIN(@uuid,1),0);
    +--------------------------------------+
    | BIN_TO_UUID(UUID_TO_BIN(@uuid,1),0)  |
    +--------------------------------------+
    | 1026baba-6ccd-780c-9564-5b8c656024db |
    +--------------------------------------+
    ```

    如果在**mysql**客户端中调用`UUID_TO_BIN()`，二进制字符串将根据`--binary-as-hex`的值以十六进制表示。有关该选项的更多信息，请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `VALUES(*`col_name`*)`

    在`INSERT ... ON DUPLICATE KEY UPDATE`语句中，您可以在`UPDATE`子句中使用`VALUES(*`col_name`*)`函数来引用语句的`INSERT`部分的列值。换句话说，在`UPDATE`子句中的`VALUES(*`col_name`*)`指的是如果没有发生重复键冲突，将要插入的*`col_name`*的值。这个函数在多行插入中特别有用。`VALUES()`函数只在`INSERT`语句的`ON DUPLICATE KEY UPDATE`子句中有意义，否则返回`NULL`。详见 Section 15.2.7.2, “INSERT ... ON DUPLICATE KEY UPDATE Statement”。

    ```sql
    mysql> INSERT INTO table (a,b,c) VALUES (1,2,3),(4,5,6)
     -> ON DUPLICATE KEY UPDATE c=VALUES(a)+VALUES(b);
    ```

    重要提示

    在 MySQL 8.0.20 中，此用法已被弃用，并可能在将来的 MySQL 版本中被移除。请改用行别名或行和列别名。有关更多信息和示例，请参见 Section 15.2.7.2, “INSERT ... ON DUPLICATE KEY UPDATE Statement”。
