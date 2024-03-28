> 原文：[`dev.mysql.com/doc/refman/8.0/en/loop.html`](https://dev.mysql.com/doc/refman/8.0/en/loop.html)

#### 15.6.5.5 LOOP Statement

```sql
[*begin_label*:] LOOP
    *statement_list*
END LOOP [*end_label*]
```

`LOOP` 实现了一个简单的循环结构，允许对由一个或多个语句组成的语句列表重复执行，每个语句以分号 (`;`) 作为语句分隔符。循环内的语句将重复执行，直到循环被终止。通常，可以通过 `LEAVE` 语句来实现循环终止。在存储函数内部，也可以使用 `RETURN` 语句，该语句完全退出函数。

忽略包含循环终止语句会导致无限循环。

`LOOP` 语句可以被标记。有关标签使用规则，请参见 第 15.6.2 节，“语句标签”。

示例：

```sql
CREATE PROCEDURE doiterate(p1 INT)
BEGIN
  label1: LOOP
    SET p1 = p1 + 1;
    IF p1 < 10 THEN
      ITERATE label1;
    END IF;
    LEAVE label1;
  END LOOP label1;
  SET @x = p1;
END;
```
