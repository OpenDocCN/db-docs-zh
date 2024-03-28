> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-functions.html)

#### 19.5.1.14 复制和系统函数

在某些情况下，某些函数无法很好地进行复制：

+   `USER()`、`CURRENT_USER()`（或`CURRENT_USER`）、`UUID()`、`VERSION()`和`LOAD_FILE()`函数在复制时不会更改，因此在副本上无法可靠工作，除非启用了基于行的复制。（参见 Section 19.2.1, “复制格式”。）

    在`MIXED`模式下使用基于行的复制时，`USER()`和`CURRENT_USER()`会自动复制，并在`STATEMENT`模式下生成警告。（另请参阅 Section 19.5.1.8, “CURRENT_USER()的复制”的复制")。）对于`VERSION()`和`RAND()`也是如此。

+   对于`NOW()`，二进制日志包括时间戳。这意味着*源上调用此函数返回的值*会被复制到副本中。为了避免在不同时区的 MySQL 服务器之间复制时出现意外结果，请在源和副本上都设置时区。更多信息，请参见 Section 19.5.1.33, “复制和时区”。

    为了解释在不同时区的服务器之间复制时可能出现的问题，假设源位于纽约，副本位于斯德哥尔摩，两台服务器都使用当地时间。进一步假设，在源上，你创建了一个表`mytable`，对该表执行了一个`INSERT`语句，然后从表中选择，如下所示：

    ```sql
    mysql> CREATE TABLE mytable (mycol TEXT);
    Query OK, 0 rows affected (0.06 sec)

    mysql> INSERT INTO mytable VALUES ( NOW() );
    Query OK, 1 row affected (0.00 sec)

    mysql> SELECT * FROM mytable;
    +---------------------+
    | mycol               |
    +---------------------+
    | 2009-09-01 12:00:00 |
    +---------------------+
    1 row in set (0.00 sec)
    ```

    斯德哥尔摩的当地时间比纽约晚 6 小时；因此，如果你在副本上的确切时刻发出`SELECT NOW()`，将返回值`2009-09-01 18:00:00`。因此，如果在复制了刚刚显示的`CREATE TABLE`和`INSERT`语句之后，从副本的`mytable`复制，你可能期望`mycol`包含值`2009-09-01 18:00:00`。然而，事实并非如此；当你从副本的`mytable`复制时，你得到的结果与源上完全相同：

    ```sql
    mysql> SELECT * FROM mytable;
    +---------------------+
    | mycol               |
    +---------------------+
    | 2009-09-01 12:00:00 |
    +---------------------+
    1 row in set (0.00 sec)
    ```

    与`NOW()`不同，`SYSDATE()`函数不是复制安全的，因为它不受二进制日志中`SET TIMESTAMP`语句的影响，并且在使用基于语句的日志记录时是不确定的。如果使用基于行的日志记录，则不会出现问题。

    另一种选择是使用`--sysdate-is-now`选项，使`SYSDATE()`成为`NOW()`的别名。这必须在源和副本上都执行才能正常工作。在这种情况下，此函数仍会发出警告，但只要在源和副本上都使用`--sysdate-is-now`，就可以安全地忽略。

    当使用`MIXED`模式时，`SYSDATE()`会自动使用基于行的复制进行复制，并在`STATEMENT`模式下生成警告。

    另请参阅 Section 19.5.1.33, “Replication and Time Zones”。

+   *以下限制仅适用于基于语句的复制，不适用于基于行的复制。* 处理用户级锁的`GET_LOCK()`、`RELEASE_LOCK()`、`IS_FREE_LOCK()`和`IS_USED_LOCK()`函数在源上处理并在副本上不知道并发上下文。因此，这些函数不应用于插入源表，因为副本上的内容会有所不同。例如，不要发出类似`INSERT INTO mytable VALUES(GET_LOCK(...))`的语句。

    当使用`MIXED`模式时，这些函数会自动使用基于行的复制进行复制，并在`STATEMENT`模式下生成警告。

作为解决方案，当基于语句的复制生效时，您可以使用将有问题的函数结果保存在用户变量中，并在后续语句中引用该变量的策略。例如，以下单行`INSERT`由于引用`UUID()`函数而存在问题：

```sql
INSERT INTO t VALUES(UUID());
```

要解决这个问题，可以这样做��

```sql
SET @my_uuid = UUID();
INSERT INTO t VALUES(@my_uuid);
```

该序列的语句之所以能够复制，是因为`@my_uuid`的值作为用户变量事件存储在二进制日志中，先于`INSERT`语句，并且可以在`INSERT`中使用。

相同的思路也适用于多行插入，但使用起来更加繁琐。对于两行插入，您可以这样做：

```sql
SET @my_uuid1 = UUID(); @my_uuid2 = UUID();
INSERT INTO t VALUES(@my_uuid1),(@my_uuid2);
```

但是，如果行数很大或者未知，解决方法就会变得困难或者不可行。例如，你无法将以下语句转换为一个语句，其中一个给定的个人用户变量与每一行相关联：

```sql
INSERT INTO t2 SELECT UUID(), * FROM t1;
```

在存储函数中，`RAND()` 只要在函数执行过程中仅调用一次，就能正确复制。（你可以将函数执行时间戳和随机数种子视为在源和副本上相同的隐式输入。）

`FOUND_ROWS()` 和 `ROW_COUNT()` 函数在基于语句的复制中无法可靠复制。一个解决方法是将函数调用的结果存储在用户变量中，然后在 `INSERT` 语句中使用。例如，如果你希望将结果存储在名为 `mytable` 的表中，你可能会像这样做：

```sql
SELECT SQL_CALC_FOUND_ROWS FROM mytable LIMIT 1;
INSERT INTO mytable VALUES( FOUND_ROWS() );
```

但是，如果你正在复制 `mytable`，你应该使用 `SELECT ... INTO`，然后将变量存储在表中，就像这样：

```sql
SELECT SQL_CALC_FOUND_ROWS INTO @found_rows FROM mytable LIMIT 1;
INSERT INTO mytable VALUES(@found_rows);
```

这样，用户变量将作为上下文的一部分进行复制，并在副本上正确应用。

当使用 `MIXED` 模式时，这些函数会在基于行的复制中自动复制，并在 `STATEMENT` 模式下生成警告。（Bug #12092, Bug #30244）
