# 14.18.2 与全局事务标识符（GTID）一起使用的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gtid-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html)

本节描述的函数用于基于 GTID 的复制。重要的是要记住，所有这些函数都将 GTID 集的字符串表示作为参数。因此，在使用它们时，GTID 集必须始终用引号括起来。有关更多信息，请参见[GTID Sets](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-concepts.html#replication-gtids-concepts-gtid-sets "GTID Sets")。

两个 GTID 集的并集只是它们作为字符串的表示，用逗号连接在一起。换句话说，您可以定义一个非常简单的函数来获取两个 GTID 集的并集，类似于此处创建的函数：

```sql
CREATE FUNCTION GTID_UNION(g1 TEXT, g2 TEXT)
    RETURNS TEXT DETERMINISTIC
    RETURN CONCAT(g1,',',g2);
```

有关 GTID 及这些 GTID 函数在实践中的使用方式的更多信息，请参见[第 19.1.3 节，“具有全局事务标识符的复制”](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids.html "19.1.3 具有全局事务标识符的复制")。

**表 14.26 GTID 函数**

| 名称 | 描述 | 已弃用 |
| --- | --- | --- |
| [`GTID_SUBSET()`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_gtid-subset) | 如果子集中的所有 GTID 也在集合中，则返回 true；否则返回 false。 |  |
| [`GTID_SUBTRACT()`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_gtid-subtract) | 返回集合中不在子集中的所有 GTID。 |  |
| [`WAIT_FOR_EXECUTED_GTID_SET()`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_wait-for-executed-gtid-set) | 等待给定的 GTID 在副本上执行。 |  |
| [`WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_wait-until-sql-thread-after-gtids) | 使用`WAIT_FOR_EXECUTED_GTID_SET()`。 | 8.0.18 |

+   [`GTID_SUBSET(*`set1`*,*`set2`*)`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_gtid-subset)

    给定两组全局事务标识符 *`set1`* 和 *`set2`*，如果 *`set1`* 中的所有 GTID 也在 *`set2`* 中，则返回 true。如果 *`set1`* 或 *`set2`* 为 `NULL`，则返回 `NULL`。否则返回 false。

    与此函数一起使用的 GTID 集表示为字符串，如以下示例所示：

    ```sql
    mysql> SELECT GTID_SUBSET('3E11FA47-71CA-11E1-9E33-C80AA9429562:23',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57')\G
    *************************** 1\. row ***************************
    GTID_SUBSET('3E11FA47-71CA-11E1-9E33-C80AA9429562:23',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57'): 1 1 row in set (0.00 sec)

    mysql> SELECT GTID_SUBSET('3E11FA47-71CA-11E1-9E33-C80AA9429562:23-25',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57')\G
    *************************** 1\. row ***************************
    GTID_SUBSET('3E11FA47-71CA-11E1-9E33-C80AA9429562:23-25',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57'): 1 1 row in set (0.00 sec)

    mysql> SELECT GTID_SUBSET('3E11FA47-71CA-11E1-9E33-C80AA9429562:20-25',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57')\G
    *************************** 1\. row ***************************
    GTID_SUBSET('3E11FA47-71CA-11E1-9E33-C80AA9429562:20-25',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57'): 0 1 row in set (0.00 sec)
    ```

+   [`GTID_SUBTRACT(*`set1`*,*`set2`*)`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_gtid-subtract)

    给定两组全局事务标识符 *`set1`* 和 *`set2`*，仅返回 *`set1`* 中不在 *`set2`* 中的 GTID。如果 *`set1`* 或 *`set2`* 为 `NULL`，则返回 `NULL`。

    与此函数一起使用的所有 GTID 集表示为字符串，并且必须用引号括起来，如下例所示：

    ```sql
    mysql> SELECT GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:21')\G
    *************************** 1\. row ***************************
    GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:21'): 3e11fa47-71ca-11e1-9e33-c80aa9429562:22-57 1 row in set (0.00 sec)

    mysql> SELECT GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:20-25')\G
    *************************** 1\. row ***************************
    GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:20-25'): 3e11fa47-71ca-11e1-9e33-c80aa9429562:26-57 1 row in set (0.00 sec)

    mysql> SELECT GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:23-24')\G
    *************************** 1\. row ***************************
    GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:23-24'): 3e11fa47-71ca-11e1-9e33-c80aa9429562:21-22:25-57 1 row in set (0.01 sec)
    ```

    从一个 GTID 集中减去它自身会产生一个空集，如下所示：

    ```sql
    mysql> SELECT GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
     ->     '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57')\G
    *************************** 1\. row ***************************
    GTID_SUBTRACT('3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57',
        '3E11FA47-71CA-11E1-9E33-C80AA9429562:21-57'): 1 row in set (0.00 sec)
    ```

+   [`WAIT_FOR_EXECUTED_GTID_SET(*`gtid_set`*[, *`timeout`*])`](https://dev.mysql.com/doc/refman/8.0/en/gtid-functions.html#function_wait-for-executed-gtid-set)

    等待服务器应用所有全局事务标识符包含在*`gtid_set`*中的事务，即直到条件 GTID_SUBSET(*`gtid_subset`*, `@@GLOBAL.gtid_executed`)成立为止。有关 GTID 集的定义，请参见第 19.1.3.1 节，“GTID 格式和存储”。

    如果指定了超时时间，并且在所有 GTID 集中的事务被应用之前经过*`timeout`*秒，函数将停止等待。*`timeout`*是可选的，默认超时时间为 0 秒，在这种情况下，函数始终等待直到所有 GTID 集中的事务被应用。*`timeout`*必须大于或等于 0；在严格 SQL 模式下运行时，负的*`timeout`*值会立即被拒绝并显示错误（`ER_WRONG_ARGUMENTS`）；否则函数返回`NULL`并发出警告。

    `WAIT_FOR_EXECUTED_GTID_SET()`监视服务器上应用的所有 GTID，包括从所有复制通道和用户客户端到达的事务。它不考虑复制通道是否已启动或停止。

    更多信息，请参见第 19.1.3 节，“使用全局事务标识符进行复制”。

    与此函数一起使用的 GTID 集表示为字符串，因此必须像以下示例中所示进行引用：

    ```sql
    mysql> SELECT WAIT_FOR_EXECUTED_GTID_SET('3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5');
     -> 0
    ```

    有关 GTID 集的语法描述，请参见第 19.1.3.1 节，“GTID 格式和存储”。

    对于`WAIT_FOR_EXECUTED_GTID_SET()`，返回值是查询的状态，其中 0 表示成功，1 表示超时。任何其他失败都会生成错误。

    `gtid_mode`在任何客户端使用此函数等待 GTID 被应用时，不能被更改为 OFF。

+   [`WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS(*`gtid_set`*[, *`timeout`*][,*`channel`*])`](gtid-functions.html#function_wait-until-sql-thread-after-gtids)

    `WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()`已被弃用。请改用`WAIT_FOR_EXECUTED_GTID_SET()`，它可以工作，无论复制通道或用户客户端通过哪个指定事务到达服务器。
