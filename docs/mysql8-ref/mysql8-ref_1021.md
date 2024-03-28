> 原文：[`dev.mysql.com/doc/refman/8.0/en/repeat.html`](https://dev.mysql.com/doc/refman/8.0/en/repeat.html)

#### 15.6.5.6 REPEAT Statement

```sql
[*begin_label*:] REPEAT
    *statement_list*
UNTIL *search_condition*
END REPEAT [*end_label*]
```

在`REPEAT`语句中的语句列表将重复执行，直到*`search_condition`*表达式为真。因此，`REPEAT`总是至少进入循环一次。*`statement_list`*由一个或多个语句组成，每个语句以分号(`;`)作为语句分隔符。

一个`REPEAT`语句可以被标记。有关标签使用的规则，请参见第 15.6.2 节，“语句标签”。

示例:

```sql
mysql> delimiter //

mysql> CREATE PROCEDURE dorepeat(p1 INT)
       BEGIN
         SET @x = 0;
         REPEAT
           SET @x = @x + 1;
         UNTIL @x > p1 END REPEAT;
       END
       //
Query OK, 0 rows affected (0.00 sec)

mysql> CALL dorepeat(1000)//
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @x//
+------+
| @x   |
+------+
| 1001 |
+------+
1 row in set (0.00 sec)
```
