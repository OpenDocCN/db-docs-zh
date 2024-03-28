# 15.6.2 语句标签

> 原文：[`dev.mysql.com/doc/refman/8.0/en/statement-labels.html`](https://dev.mysql.com/doc/refman/8.0/en/statement-labels.html)

```sql
[*begin_label*:] BEGIN
    [*statement_list*]
END [*end_label*]

[*begin_label*:] LOOP
    *statement_list*
END LOOP [*end_label*]

[*begin_label*:] REPEAT
    *statement_list*
UNTIL *search_condition*
END REPEAT [*end_label*]

[*begin_label*:] WHILE *search_condition* DO
    *statement_list*
END WHILE [*end_label*]
```

标签允许用于 `BEGIN ... END` 块以及 `LOOP`、`REPEAT` 和 `WHILE` 语句。这些语句的标签使用遵循以下规则：

+   *`begin_label`* 后必须跟着一个冒号。

+   *`begin_label`* 可以单独出现，不需要 *`end_label`*。如果有 *`end_label`*，它必须与 *`begin_label`* 相同。

+   *`end_label`* 不能单独出现，必须有 *`begin_label`*。

+   同一嵌套级别的标签必须是不同的。

+   标签最多可以有 16 个字符长。

要引用标记结构内的标签，请使用 `ITERATE` 或 `LEAVE` 语句。以下示例使用这些语句来继续迭代或终止循环：

```sql
CREATE PROCEDURE doiterate(p1 INT)
BEGIN
  label1: LOOP
    SET p1 = p1 + 1;
    IF p1 < 10 THEN ITERATE label1; END IF;
    LEAVE label1;
  END LOOP label1;
END;
```

块标签的范围不包括在块内声明的处理程序的代码。有关详细信息，请参见 第 15.6.7.2 节，“DECLARE ... HANDLER Statement”。
