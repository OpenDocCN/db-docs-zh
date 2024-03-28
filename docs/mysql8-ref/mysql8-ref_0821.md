# 14.15 信息函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html)

**表 14.20 信息函数**

| 名称 | 描述 |
| --- | --- |
| `BENCHMARK()` | 反复执行一个表达式 |
| `CHARSET()` | 返回参数的字符集 |
| `COERCIBILITY()` | 返回字符串参数的排序强制性值 |
| `COLLATION()` | 返回字符串参数的排序规则 |
| `CONNECTION_ID()` | 返回连接的连接 ID（线程 ID） |
| `CURRENT_ROLE()` | 返回当前活动角色 |
| `CURRENT_USER()`, `CURRENT_USER` | 认证用户的用户名和主机名 |
| `DATABASE()` | 返回默认（当前）数据库名称 |
| `FOUND_ROWS()` | 对于带有 LIMIT 子句的 SELECT，如果没有 LIMIT 子句，将返回的行数 |
| `ICU_VERSION()` | ICU 库版本 |
| `LAST_INSERT_ID()` | 最后一次 INSERT 的 AUTOINCREMENT 列的值 |
| `ROLES_GRAPHML()` | 返回表示内存角色子图的 GraphML 文档 |
| `ROW_COUNT()` | 更新的行数 |
| `SCHEMA()` | DATABASE() 的同义词 |
| `SESSION_USER()` | USER() 的同义词 |
| `SYSTEM_USER()` | USER() 的同义词 |
| `USER()` | 客户端提供的用户名和主机名 |
| `VERSION()` | 返回指示 MySQL 服务器版本的字符串 |
| 名称 | 描述 |

+   `BENCHMARK(*`count`*,*`expr`*)`

    `BENCHMARK()` 函数重复执行表达式 *`expr`* *`count`* 次。它可用于计算 MySQL 处理表达式的速度。结果值为 `0`，对于不合适的参数（如 `NULL` 或负重复计数）为 `NULL`。

    预期的用法是在**mysql**客户端内部，报告查询执行时间：

    ```sql
    mysql> SELECT BENCHMARK(1000000,AES_ENCRYPT('hello','goodbye'));
    +---------------------------------------------------+
    | BENCHMARK(1000000,AES_ENCRYPT('hello','goodbye')) |
    +---------------------------------------------------+
    |                                                 0 |
    +---------------------------------------------------+
    1 row in set (4.74 sec)
    ```

    报告的时间是客户端端的经过时间，而不是服务器端的 CPU 时间。建议多次执行`BENCHMARK()`，并根据服务器机器的负载情况解释结果。

    `BENCHMARK()`旨在衡量标量表达式的运行时性能，这对于您使用它和解释结果有一些重要影响：

    +   只能使用标量表达式。虽然表达式可以是子查询，但必须返回单列且最多一行。例如，如果表`t`有多于一列或多于一行，则`BENCHMARK(10, (SELECT * FROM t))`会失败。

    +   执行`SELECT *`expr`*`语句*`N`*次与执行`SELECT BENCHMARK(*`N`*, *`expr`*)`在涉及的开销量方面有所不同。两者具有非常不同的执行概况，您不应该期望它们花费相同的时间。前者涉及解析器、优化器、表锁定和运行时评估各执行*N*次。后者仅涉及运行时评估*N*次，而所有其他组件仅执行一次。已分配的内存结构将被重用，并且运行时优化，例如对已为聚合函数评估的结果进行本地缓存，可能会改变结果。因此，使用`BENCHMARK()`通过给予该组件更多权重来衡量运行时组件的性能，并消除网络、解析器、优化器等引入的“噪音”。

+   `CHARSET(*`str`*)`

    返回字符串参数的字符集，如果参数为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT CHARSET('abc');
     -> 'utf8mb3'
    mysql> SELECT CHARSET(CONVERT('abc' USING latin1));
     -> 'latin1'
    mysql> SELECT CHARSET(USER());
     -> 'utf8mb3'
    ```

+   `COERCIBILITY(*`str`*)`

    返回字符串参数的排序强制性值。

    ```sql
    mysql> SELECT COERCIBILITY('abc' COLLATE utf8mb4_swedish_ci);
     -> 0
    mysql> SELECT COERCIBILITY(USER());
     -> 3
    mysql> SELECT COERCIBILITY('abc');
     -> 4
    mysql> SELECT COERCIBILITY(1000);
     -> 5
    ```

    返回值的含义如下表所示。较低的值具有更高的优先级。

    | 强制性 | 含义 | 示例 |
    | --- | --- | --- |
    | `0` | 显式排序 | 带有`COLLATE`子句的值 |
    | `1` | 无排序 | 具有不同排序的字符串连接 |
    | `2` | 隐式排序 | 列值、存储过程参数或本地变量 |
    | `3` | 系统常量 | `USER()`返回值 |
    | `4` | 可强制 | 字面字符串 |
    | `5` | 数值 | 数值或时间值 |
    | `6` | 可忽略 | `NULL`或从`NULL`派生的表达式 |

    更多信息，请参见第 12.8.4 节，“表达式中的排序强制性”。

+   `COLLATION(*`str`*)`

    返回字符串参数的排序。

    ```sql
    mysql> SELECT COLLATION('abc');
     -> 'utf8mb4_0900_ai_ci'
    mysql> SELECT COLLATION(_utf8mb4'abc');
     -> 'utf8mb4_0900_ai_ci'
    mysql> SELECT COLLATION(_latin1'abc');
     -> 'latin1_swedish_ci'
    ```

+   `CONNECTION_ID()`

    返回连接的连接 ID（线程 ID）。每个连接都有一个在当前连接的客户端集合中唯一的 ID。

    `CONNECTION_ID()`返回的值与信息模式`PROCESSLIST`表的`ID`列、`SHOW PROCESSLIST`输出的`Id`列以及性能模式`threads`表的`PROCESSLIST_ID`列中显示的值类型相同。

    ```sql
    mysql> SELECT CONNECTION_ID();
     -> 23786
    ```

    警告

    更改`pseudo_thread_id`系统变量的会话值会更改`CONNECTION_ID()`函数返回的值。

+   `CURRENT_ROLE()`

    返回一个包含当前会话中当前活动角色的`utf8mb3`字符串，用逗号分隔，如果没有则返回`NONE`。该值反映了`sql_quote_show_create`系统变量的设置。

    假设一个账户被授予以下角色：

    ```sql
    GRANT 'r1', 'r2' TO 'u1'@'localhost';
    SET DEFAULT ROLE ALL TO 'u1'@'localhost';
    ```

    在`u1`的会话中，初始`CURRENT_ROLE()`值命名默认账户角色。使用`SET ROLE`更改该值：

    ```sql
    mysql> SELECT CURRENT_ROLE();
    +-------------------+
    | CURRENT_ROLE()    |
    +-------------------+
    | `r1`@`%`,`r2`@`%` |
    +-------------------+
    mysql> SET ROLE 'r1'; SELECT CURRENT_ROLE();
    +----------------+
    | CURRENT_ROLE() |
    +----------------+
    | `r1`@`%`       |
    +----------------+
    ```

+   `CURRENT_USER`, `CURRENT_USER()`

    返回 MySQL 服务器用于验证当前客户端的用户名称和主机名组合的账户。此账户确定您的访问权限。返回值是`utf8mb3`字符集中的字符串。

    `CURRENT_USER()`的值可能与`USER()`的值不同。

    ```sql
    mysql> SELECT USER();
     -> 'davida@localhost'
    mysql> SELECT * FROM mysql.user;
    ERROR 1044: Access denied for user ''@'localhost' to
    database 'mysql'
    mysql> SELECT CURRENT_USER();
     -> '@localhost'
    ```

    该示例说明，尽管客户端指定了用户名为`davida`（如`USER()`函数的值所示），但服务器却使用匿名用户账户对客户端进行了身份验证（如`CURRENT_USER()`值的空用户部分所示）。这种情况可能发生的一种方式是在授权表中没有列出`davida`的账户。

    在存储过程或视图中，`CURRENT_USER()`返回定义对象的用户账户（由其`DEFINER`值给出），除非使用`SQL SECURITY INVOKER`特性定义。在后一种情况下，`CURRENT_USER()`返回对象的调用者。

    触发器和事件没有选项来定义`SQL SECURITY`特性，因此对于这些对象，`CURRENT_USER()`返回定义对象的用户账户。要返回调用者，请使用`USER()`或`SESSION_USER()`。

    以下语句支持使用`CURRENT_USER()`函数来代替受影响用户或定义者的名称（可能还有主机）；在这种情况下，`CURRENT_USER()`会根据需要进行扩展：

    +   `DROP USER`

    +   `RENAME USER`

    +   `GRANT`

    +   `REVOKE`

    +   `CREATE FUNCTION`

    +   `CREATE PROCEDURE`

    +   `CREATE TRIGGER`

    +   `CREATE EVENT`

    +   `CREATE VIEW`

    +   `ALTER EVENT`

    +   `ALTER VIEW`

    +   `SET PASSWORD`

    对于这种扩展`CURRENT_USER()`的含义的信息，参见 Section 19.5.1.8, “CURRENT_USER()的复制”的复制")。

    从 MySQL 8.0.34 开始，此函数可用于`VARCHAR`或`TEXT`列的默认值，如下所示的`CREATE TABLE`语句：

    ```sql
    CREATE TABLE t (c VARCHAR(288) DEFAULT (CURRENT_USER()));
    ```

+   `DATABASE()`

    返回以`utf8mb3`字符集中的字符串形式的默认（当前）数据库名称。如果没有默认数据库，`DATABASE()`返回`NULL`。在存储过程中，默认数据库是与存储过程关联的数据库，并不一定与调用上下文中的默认数据库相同。

    ```sql
    mysql> SELECT DATABASE();
     -> 'test'
    ```

    如果没有默认数据库，`DATABASE()`返回`NULL`。

+   `FOUND_ROWS()`

    注意

    自 MySQL 8.0.17 起，`SQL_CALC_FOUND_ROWS`查询修饰符和相应的`FOUND_ROWS()`函数已被弃用；预计它们将在未来的 MySQL 版本中被移除。作为替代方案，考虑使用带有`LIMIT`的查询，然后再执行一个不带`LIMIT`但带有`COUNT(*)`的第二个查询，以确定是否有额外的行。例如，不要使用这些查询：

    ```sql
    SELECT SQL_CALC_FOUND_ROWS * FROM *tbl_name* WHERE id > 100 LIMIT 10;
    SELECT FOUND_ROWS();
    ```

    改用以下查询：

    ```sql
    SELECT * FROM *tbl_name* WHERE id > 100 LIMIT 10;
    SELECT COUNT(*) FROM *tbl_name* WHERE id > 100;
    ```

    `COUNT(*)`受到某些优化的影响。`SQL_CALC_FOUND_ROWS`会导致某些优化被禁用。

    一个`SELECT`语句可以包括一个`LIMIT`子句，以限制服务器返回给客户端的行数。在某些情况下，希望知道没有`LIMIT`时语句会返回多少行，但又不想再次运行该语句。要获取这个行数，需要在`SELECT`语句中包含一个`SQL_CALC_FOUND_ROWS`选项，然后在之后调用`FOUND_ROWS()`：

    ```sql
    mysql> SELECT SQL_CALC_FOUND_ROWS * FROM *tbl_name*
     -> WHERE id > 100 LIMIT 10;
    mysql> SELECT FOUND_ROWS();
    ```

    第二个`SELECT`返回一个数字，指示第一个不带`LIMIT`子句的`SELECT`将返回多少行。

    在最近成功的不带`SQL_CALC_FOUND_ROWS`选项的`SELECT`语句中，`FOUND_ROWS()`返回该语句返回的结果集中的行数。如果语句包含`LIMIT`子句，`FOUND_ROWS()`返回限制之前的行数。例如，如果语句包含`LIMIT 10`或`LIMIT 50, 10`，则`FOUND_ROWS()`分别返回 10 或 60。

    通过`FOUND_ROWS()`获取的行数是瞬时的，不打算在`SELECT SQL_CALC_FOUND_ROWS`语句后的语句中使用。如果需要稍后引用该值，请保存它：

    ```sql
    mysql> SELECT SQL_CALC_FOUND_ROWS * FROM ... ;
    mysql> SET @rows = FOUND_ROWS();
    ```

    如果你使用`SELECT SQL_CALC_FOUND_ROWS`，MySQL 必须计算完整结果集中有多少行。然而，这比再次运行不带`LIMIT`的查询要快，因为不需要将结果集发送给客户端。

    `SQL_CALC_FOUND_ROWS` 和 `FOUND_ROWS()` 在需要限制查询返回行数的情况下非常有用，同时又要确定完整结果集中的行数而不需要重新运行查询时。例如，一个 Web 脚本显示分页显示，包含指向显示搜索结果其他部分页面的链接。使用 `FOUND_ROWS()` 可以帮助确定还需要多少其他页面来显示剩余的结果。

    对于 `UNION` 语句，使用 `SQL_CALC_FOUND_ROWS` 和 `FOUND_ROWS()` 比简单的 `SELECT` 语句更复杂，因为 `LIMIT` 可能出现在 `UNION` 中的多个位置。它可以应用于 `UNION` 中的各个 `SELECT` 语句，或者全局应用于整个 `UNION` 结果。

    `SQL_CALC_FOUND_ROWS` 用于 `UNION` 的目的是应返回在没有全局 `LIMIT` 的情况下将返回的行数。使用 `SQL_CALC_FOUND_ROWS` 与 `UNION` 的条件是：

    +   `SQL_CALC_FOUND_ROWS` 关键字必须出现在 `UNION` 的第一个 `SELECT` 中。

    +   `FOUND_ROWS()` 的值仅在使用 `UNION ALL` 时是精确的。如果使用不带 `ALL` 的 `UNION`，会发生重复移除，并且 `FOUND_ROWS()` 的值只是近似值。

    +   如果 `UNION` 中没有 `LIMIT`，则会忽略 `SQL_CALC_FOUND_ROWS` 并返回用于处理 `UNION` 的临时表中的行数。

    除了这里描述的情况外，`FOUND_ROWS()` 的行为是未定义的（例如，在出现错误的 `SELECT` 语句后其值是多少）。

    重要

    使用基于语句的复制时，`FOUND_ROWS()` 无法可靠地复制。此函数会在基于行的复制中自动复制。

+   `ICU_VERSION()`

    用于支持正则表达式操作的国际组件库（ICU）的版本（参见 Section 14.8.2, “Regular Expressions”）。此函数主要用于测试案例。

+   `LAST_INSERT_ID()`, `LAST_INSERT_ID(*`expr`*)`

    没有参数时，`LAST_INSERT_ID()` 返回一个`BIGINT UNSIGNED`（64 位）值，表示作为最近执行的`INSERT`语句的结果成功插入的第一个自动生成值，用于`AUTO_INCREMENT`列。如果没有成功插入行，则`LAST_INSERT_ID()`的值保持不变。

    有参数时，`LAST_INSERT_ID()` 返回一个无符号整数，如果参数为`NULL`，则返回`NULL`。

    例如，在插入生成`AUTO_INCREMENT`值的行之后，您可以像这样获取该值：

    ```sql
    mysql> SELECT LAST_INSERT_ID();
     -> 195
    ```

    当前执行的语句不会影响`LAST_INSERT_ID()`的值。假设您使用一个语句生成`AUTO_INCREMENT`值，然后在一个多行`INSERT`语句中引用`LAST_INSERT_ID()`，该语句将行插入到具有自己的`AUTO_INCREMENT`列的表中。`LAST_INSERT_ID()`的值在第二个语句中保持稳定；其值对第二行及后续行不受先前行插入的影响。（您应该注意，如果混合引用`LAST_INSERT_ID()`和`LAST_INSERT_ID(*`expr`*)`，效果是未定义的。）

    如果前一个语句返回错误，则`LAST_INSERT_ID()`的值是未定义的。对于事务表，如果由于错误而回滚语句，则`LAST_INSERT_ID()`的值将保持未定义。对于手动`ROLLBACK`，`LAST_INSERT_ID()`的值不会恢复到事务之前的值；它将保持在`ROLLBACK`点时的值。

    在存储过程（procedure）或触发器的主体内，`LAST_INSERT_ID()` 的值与在这些对象的主体外执行语句时的方式相同。存储过程或触发器对后续语句看到的`LAST_INSERT_ID()`的值的影响取决于存储过程的类型：

    +   如果存储过程执行改变`LAST_INSERT_ID()`值的语句，那么这个改变的值会被调用存储过程后的语句所看到。

    +   对于改变值的存储函数和触发器，在函数或触发器结束时值会被恢复，因此在其后的语句不会看到改变的值。

    生成的 ID 在服务器上以*每个连接为基础*进行维护。这意味着函数返回给特定客户端的值是该客户端最近影响`AUTO_INCREMENT`列的大多数最新语句生成的第一个`AUTO_INCREMENT`值。即使其他客户端生成了自己的`AUTO_INCREMENT`值，这个值也不会受到影响。这种行为确保每个客户端可以检索自己的 ID，而不必担心其他客户端的活动，也不需要锁定或事务。

    如果将行的`AUTO_INCREMENT`列设置为非“魔术”值（即不是`NULL`且不是`0`），则`LAST_INSERT_ID()`的值不会改变。

    重要提示

    如果使用单个`INSERT`语句插入多行，`LAST_INSERT_ID()`仅返回*第一*插入行生成的值。这样做的原因是为了能够轻松地在其他服务器上重现相同的`INSERT`语句。

    例如：

    ```sql
    mysql> USE test;

    mysql> CREATE TABLE t (
           id INT AUTO_INCREMENT NOT NULL PRIMARY KEY,
           name VARCHAR(10) NOT NULL
           );

    mysql> INSERT INTO t VALUES (NULL, 'Bob');

    mysql> SELECT * FROM t;
    +----+------+
    | id | name |
    +----+------+
    |  1 | Bob  |
    +----+------+

    mysql> SELECT LAST_INSERT_ID();
    +------------------+
    | LAST_INSERT_ID() |
    +------------------+
    |                1 |
    +------------------+

    mysql> INSERT INTO t VALUES
           (NULL, 'Mary'), (NULL, 'Jane'), (NULL, 'Lisa');

    mysql> SELECT * FROM t;
    +----+------+
    | id | name |
    +----+------+
    |  1 | Bob  |
    |  2 | Mary |
    |  3 | Jane |
    |  4 | Lisa |
    +----+------+

    mysql> SELECT LAST_INSERT_ID();
    +------------------+
    | LAST_INSERT_ID() |
    +------------------+
    |                2 |
    +------------------+
    ```

    尽管第二个`INSERT`语句向`t`插入了三行新记录，但为这些行中的第一行生成的 ID 是`2`，并且这个值会被后续的`SELECT`语句中的`LAST_INSERT_ID()`返回。

    如果使用`INSERT IGNORE`并且行被忽略，`LAST_INSERT_ID()`保持不变（或者如果连接尚未执行成功的`INSERT`，则返回 0），对于非事务表，`AUTO_INCREMENT`计数器不会增加。对于`InnoDB`表，如果`innodb_autoinc_lock_mode`设置为`1`或`2`，`AUTO_INCREMENT`计数器会增加，如下例所示：

    ```sql
    mysql> USE test;

    mysql> SELECT @@innodb_autoinc_lock_mode;
    +----------------------------+
    | @@innodb_autoinc_lock_mode |
    +----------------------------+
    |                          1 |
    +----------------------------+

    mysql> CREATE TABLE `t` (
           `id` INT(11) NOT NULL AUTO_INCREMENT,
           `val` INT(11) DEFAULT NULL,
           PRIMARY KEY (`id`),
           UNIQUE KEY `i1` (`val`)
           ) ENGINE=InnoDB;

    # Insert two rows

    mysql> INSERT INTO t (val) VALUES (1),(2);

    # With auto_increment_offset=1, the inserted rows
    # result in an AUTO_INCREMENT value of 3

    mysql> SHOW CREATE TABLE t\G
    *************************** 1\. row ***************************
           Table: t
    Create Table: CREATE TABLE `t` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `val` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `i1` (`val`)
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

    # LAST_INSERT_ID() returns the first automatically generated
    # value that is successfully inserted for the AUTO_INCREMENT column 
    mysql> SELECT LAST_INSERT_ID();
    +------------------+
    | LAST_INSERT_ID() |
    +------------------+
    |                1 |
    +------------------+

    # The attempted insertion of duplicate rows fail but errors are ignored

    mysql> INSERT IGNORE INTO t (val) VALUES (1),(2);
    Query OK, 0 rows affected (0.00 sec)
    Records: 2  Duplicates: 2  Warnings: 0

    # With innodb_autoinc_lock_mode=1, the AUTO_INCREMENT counter
    # is incremented for the ignored rows

    mysql> SHOW CREATE TABLE t\G
    *************************** 1\. row ***************************
           Table: t
    Create Table: CREATE TABLE `t` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `val` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `i1` (`val`)
    ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

    # The LAST_INSERT_ID is unchanged because the previous insert was unsuccessful 
    mysql> SELECT LAST_INSERT_ID();
    +------------------+
    | LAST_INSERT_ID() |
    +------------------+
    |                1 |
    +------------------+
    ```

    有关更多信息，请参阅 Section 17.6.1.6, “InnoDB 中的 AUTO_INCREMENT 处理”。

    如果将*`expr`*作为参数传递给`LAST_INSERT_ID()`，则函数将返回参数的值，并记住作为下一个由`LAST_INSERT_ID()`返回的值。这可以用来模拟序列：

    1.  创建一个表来保存序列计数器并初始化它：

        ```sql
        mysql> CREATE TABLE sequence (id INT NOT NULL);
        mysql> INSERT INTO sequence VALUES (0);
        ```

    1.  使用表来生成类似这样的序列号：

        ```sql
        mysql> UPDATE sequence SET id=LAST_INSERT_ID(id+1);
        mysql> SELECT LAST_INSERT_ID();
        ```

        `UPDATE`语句递增序列计数器，并导致下一次调用`LAST_INSERT_ID()`返回更新后的值。`SELECT`语句检索该值。`mysql_insert_id()` C API 函数也可用于获取该值。请参阅 mysql_insert_id()。

    您可以在不调用`LAST_INSERT_ID()`的情况下生成序列，但以这种方式使用函数的实用性在于 ID 值在服务器中作为最后一个自动生成的值保持。它是多用户安全的，因为多个客户端可以发出`UPDATE`语句并使用`SELECT`语句（或`mysql_insert_id()`）获取自己的序列值，而不会影响或受其他生成自己序列值的客户端的影响。

    请注意，只有在执行`INSERT`和`UPDATE`语句后，`mysql_insert_id()`才会更新，因此您不能在执行其他 SQL 语句（如`SELECT`或`SET`）后使用 C API 函数检索`LAST_INSERT_ID(*`expr`*)`的值。

+   `ROLES_GRAPHML()`

    返回一个包含表示内存角色子图的 GraphML 文档的`utf8mb3`字符串。需要`ROLE_ADMIN`权限（或已弃用的`SUPER`权限）才能查看`<graphml>`元素中的内容。否则，结果只显示一个空元素：

    ```sql
    mysql> SELECT ROLES_GRAPHML();
    +---------------------------------------------------+
    | ROLES_GRAPHML()                                   |
    +---------------------------------------------------+
    | <?xml version="1.0" encoding="UTF-8"?><graphml /> |
    +---------------------------------------------------+
    ```

+   `ROW_COUNT()`

    `ROW_COUNT()`返回如下值：

    +   DDL 语句：0。这适用于诸如`CREATE TABLE`或`DROP TABLE`之类的语句。

    +   除了 `SELECT` 之外的 DML 语句：受影响的行数。这适用于诸如 `UPDATE`、`INSERT` 或 `DELETE`（如前所述）的语句，但现在也适用于诸如 `ALTER TABLE` 和 `LOAD DATA` 的语句。

    +   `SELECT`：如果语句返回结果集，则返回 -1，否则返回“受影响”的行数。例如，对于 `SELECT * FROM t1`，`ROW_COUNT()` 返回 -1。对于 `SELECT * FROM t1 INTO OUTFILE '*`file_name`*'`，`ROW_COUNT()` 返回写入文件的行数。

    +   `SIGNAL` 语句：0。

    对于 `UPDATE` 语句，默认情况下受影响的行数是实际更改的行数。如果在连接到 **mysqld** 时使用 `CLIENT_FOUND_ROWS` 标志到 `mysql_real_connect()`，受影响的行数是“找到”的行数；也就是，被 `WHERE` 子句匹配的行数。

    对于 `REPLACE` 语句，如果新行替换了旧行，则受影响的行数为 2，因为在这种情况下，删除重复项后插入了一行。

    对于 `INSERT ... ON DUPLICATE KEY UPDATE` 语句，每行的受影响行数为 1（如果将行插入为新行）、2（如果更新现有行）或 0（如果将现有行设置为当前值）。如果指定了 `CLIENT_FOUND_ROWS` 标志，则如果将现有行设置为当前值，则受影响的行数为 1（而不是 0）。

    `ROW_COUNT()` 的值类似于 `mysql_affected_rows()` C API 函数的值以及 **mysql** 客户端在语句执行后显示的行数。

    ```sql
    mysql> INSERT INTO t VALUES(1),(2),(3);
    Query OK, 3 rows affected (0.00 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> SELECT ROW_COUNT();
    +-------------+
    | ROW_COUNT() |
    +-------------+
    |           3 |
    +-------------+
    1 row in set (0.00 sec)

    mysql> DELETE FROM t WHERE i IN(1,2);
    Query OK, 2 rows affected (0.00 sec)

    mysql> SELECT ROW_COUNT();
    +-------------+
    | ROW_COUNT() |
    +-------------+
    |           2 |
    +-------------+
    1 row in set (0.00 sec)
    ```

    重要

    `ROW_COUNT()` 在基于语句的复制中无法可靠地复制。此函数会自动使用基于行的复制进行复制。

+   `SCHEMA()`

    此函数是 `DATABASE()` 的同义词。

+   `SESSION_USER()`

    `SESSION_USER()` 是 `USER()` 的同义词。

    从 MySQL 8.0.34 开始，类似于`USER()`，这个函数可以用作`VARCHAR`或`TEXT`列的默认值，如下所示的`CREATE TABLE`语句：

    ```sql
    CREATE TABLE t (c VARCHAR(288) DEFAULT (SESSION_USER()));
    ```

+   `SYSTEM_USER()`

    `SYSTEM_USER()`是`USER()`的同义词。

    注意

    `SYSTEM_USER()`函数与`SYSTEM_USER`权限是不同的。前者返回当前的 MySQL 账户名。后者区分系统用户和普通用户账户类别（参见第 8.2.11 节，“账户类别”）。

    从 MySQL 8.0.34 开始，类似于`USER()`，这个函数可以用作`VARCHAR`或`TEXT`列的默认值，如下所示的`CREATE TABLE`语句：

    ```sql
    CREATE TABLE t (c VARCHAR(288) DEFAULT (SYSTEM_USER()));
    ```

+   `USER()`

    返回当前的 MySQL 用户名和主机名作为一个字符串，使用`utf8mb3`字符集。

    ```sql
    mysql> SELECT USER();
     -> 'davida@localhost'
    ```

    该值指示您连接到服务器时指定的用户名，以及您连接的客户端主机。该值可能与`CURRENT_USER()`的值不同。

    从 MySQL 8.0.34 开始，类似于`USER()`，这个函数可以用作`VARCHAR`或`TEXT`列的默认值，如下所示的`CREATE TABLE`语句：

    ```sql
    CREATE TABLE t (c VARCHAR(288) DEFAULT (USER()));
    ```

+   `VERSION()`

    返回一个指示 MySQL 服务器版本的字符串。该字符串使用`utf8mb3`字符集。该值可能除了版本号外还有后缀。请参阅第 7.1.8 节，“服务器系统变量”中的`version`系统变量的描述。

    此函数在基于语句的复制中是不安全的。如果在`binlog_format`设置为`STATEMENT`时使用此函数，将记录警告。

    ```sql
    mysql> SELECT VERSION();
     -> '8.0.36-standard'
    ```
