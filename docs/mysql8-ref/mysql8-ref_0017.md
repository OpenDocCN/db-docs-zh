> 原文：[`dev.mysql.com/doc/refman/8.0/en/ansi-diff-comments.html`](https://dev.mysql.com/doc/refman/8.0/en/ansi-diff-comments.html)

#### 1.6.2.4 `'--'` 作为注释的起始

标准 SQL 使用 C 语法`/* 这是一个注释 */`作为注释，MySQL 服务器也支持这种语法。MySQL 还支持这种语法的扩展，允许将 MySQL 特定的 SQL 嵌入到注释中；参见第 11.7 节，“注释”。

MySQL 服务器还使用`#`作为起始注释字符。这是非标准的。

标准 SQL 还使用“`--`”作为起始注释序列。MySQL 服务器支持`--`注释风格的变体；`--`起始注释序列被接受为这样，但必须跟随一个空格字符，如空格或换行符。这个空格旨在防止生成的 SQL 查询出现问题，使用以下结构，更新余额以反映一个费用：

```sql
UPDATE account SET balance=balance-charge
WHERE account_id=user_id
```

当`charge`为负值时会发生什么，比如`-1`，这可能是在向账户存入金额时的情况。在这种情况下，生成的语句如下：

```sql
UPDATE account SET balance=balance--1
WHERE account_id=5752;
```

`balance--1`是有效的标准 SQL，但`--`被解释为注释的起始，并且表达式的一部分被丢弃。结果是语句的含义完全不同于预期：

```sql
UPDATE account SET balance=balance
WHERE account_id=5752;
```

这个语句根本不会改变值。为了防止这种情况发生，MySQL 要求在`--`后面跟随一个空格字符，以便在 MySQL 服务器中被识别为起始注释序列，这样像`balance--1`这样的表达式总是安全的使用。
