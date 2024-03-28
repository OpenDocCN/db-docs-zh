> 原文：[`dev.mysql.com/doc/refman/8.0/en/while.html`](https://dev.mysql.com/doc/refman/8.0/en/while.html)

#### 15.6.5.8 WHILE Statement

```sql
[*begin_label*:] WHILE *search_condition* DO
    *statement_list*
END WHILE [*end_label*]
```

在`WHILE`语句中的语句列表会重复执行，只要*`search_condition`*表达式为真。*`statement_list`*由一个或多个 SQL 语句组成，每个语句以分号(`;`)作为语句分隔符。

一个带标签的`WHILE`语句。关于标签使用的规则，请参见第 15.6.2 节，“语句标签”。

示例：

```sql
CREATE PROCEDURE dowhile()
BEGIN
  DECLARE v1 INT DEFAULT 5;

  WHILE v1 > 0 DO
    ...
    SET v1 = v1 - 1;
  END WHILE;
END;
```
