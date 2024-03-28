# 27.1 定义存储程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-programs-defining.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-defining.html)

每个存储程序包含一个由 SQL 语句组成的主体。该语句可以是由分号(`;`)字符分隔的多个语句组成的复合语句。例如，以下存储过程具有由`BEGIN ... END`块组成的主体，其中包含一个`SET`语句和一个包含另一个`SET`语句的`REPEAT`循环：

```sql
CREATE PROCEDURE dorepeat(p1 INT)
BEGIN
  SET @x = 0;
  REPEAT SET @x = @x + 1; UNTIL @x > p1 END REPEAT;
END;
```

如果您使用**mysql**客户端程序定义包含分号字符的存储程序，则会出现问题。默认情况下，**mysql**本身将分号识别为语句分隔符，因此您必须临时重新定义分隔符，以使**mysql**将整个存储程序定义传递给服务器。

要重新定义**mysql**的分隔符，请使用`delimiter`命令。以下示例展示了如何为刚刚显示的`dorepeat()`过程执行此操作。将分隔符更改为`//`以便将整个定义作为单个语句传递给服务器，然后在调用过程之前将其恢复为`;`。这样可以使过程体中使用的`;`分隔符传递到服务器，而不是被**mysql**本身解释。

```sql
mysql> delimiter //

mysql> CREATE PROCEDURE dorepeat(p1 INT)
 -> BEGIN
 ->   SET @x = 0;
 ->   REPEAT SET @x = @x + 1; UNTIL @x > p1 END REPEAT;
 -> END
 -> //
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;

mysql> CALL dorepeat(1000);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @x;
+------+
| @x   |
+------+
| 1001 |
+------+
1 row in set (0.00 sec)
```

您可以将分隔符重新定义为除`//`之外的其他字符串，并且分隔符可以由单个字符或多个字符组成。应避免使用反斜杠（`\`）字符，因为这是 MySQL 的转义字符。

以下是一个接受参数、使用 SQL 函数执行操作并返回结果的函数示例。在这种情况下，不需要使用`delimiter`，因为函数定义不包含内部的`;`语句分隔符：

```sql
mysql> CREATE FUNCTION hello (s CHAR(20))
mysql> RETURNS CHAR(50) DETERMINISTIC
 -> RETURN CONCAT('Hello, ',s,'!');
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT hello('world');
+----------------+
| hello('world') |
+----------------+
| Hello, world!  |
+----------------+
1 row in set (0.00 sec)
```
