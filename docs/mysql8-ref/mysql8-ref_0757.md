# 13.3.6 `SET`类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/set.html`](https://dev.mysql.com/doc/refman/8.0/en/set.html)

`SET`是一个字符串对象，可以具有零个或多个值，每个值必须从创建表时指定的允许值列表中选择。由于`SET`列值由逗号（`,`）分隔的成员指定，因此`SET`成员值本身不应包含逗号。

例如，指定为`SET('one', 'two') NOT NULL`的列可以具有以下任何值：

```sql
''
'one'
'two'
'one,two'
```

`SET`列最多可以有 64 个不同的成员。

定义中的重复值会导致警告，如果启用了严格的 SQL 模式，则会导致错误。

在创建表时，`SET`成员值的尾随空格会自动删除。

有关`SET`类型的存储要求，请参见字符串类型存储要求。

有关`SET`类型的语法和长度限制，请参见第 13.3.1 节，“字符串数据类型语法”。

当检索时，存储在`SET`列中的值将以列定义中使用的大小写形式显示。请注意，`SET`列可以分配字符集和排序规则。对于二进制或区分大小写的排序规则，分配值给列时会考虑大小写。

MySQL 以数字形式存储`SET`值，存储值的低位对应于第一个集合成员。如果在数字上下文中检索`SET`值，则检索到的值具有对应于构成列值的集合成员的位设置。例如，可以像这样从`SET`列中检索数值：

```sql
mysql> SELECT *set_col*+0 FROM *tbl_name*;
```

如果将数字存储到`SET`列中，则在数字的二进制表示中设置的位确定列值中的集合成员。对于指定为`SET('a','b','c','d')`的列，成员具有以下十进制和二进制值。

| `SET`成员 | 十进制值 | 二进制值 |
| --- | --- | --- |
| `'a'` | `1` | `0001` |
| `'b'` | `2` | `0010` |
| `'c'` | `4` | `0100` |
| `'d'` | `8` | `1000` |

如果将值`9`分配给此列，即二进制为`1001`，因此选择第一个和第四个`SET`值成员`'a'`和`'d'`，得到的值为`'a,d'`。

对于包含多个`SET`元素的值，插入值时元素的顺序无关紧要。同样，给定元素在值中列出的次数也无关紧要。稍后检索值时，值中的每个元素只出现一次，并且元素按照在创建表时指定的顺序列出。假设列被指定为`SET('a','b','c','d')`：

```sql
mysql> CREATE TABLE myset (col SET('a', 'b', 'c', 'd'));
```

如果插入值`'a,d'`、`'d,a'`、`'a,d,d'`、`'a,d,a'`和`'d,a,d'`：

```sql
mysql> INSERT INTO myset (col) VALUES 
-> ('a,d'), ('d,a'), ('a,d,a'), ('a,d,d'), ('d,a,d');
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

然后检索这些值时，所有这些值都显示为`'a,d'`：

```sql
mysql> SELECT col FROM myset;
+------+
| col  |
+------+
| a,d  |
| a,d  |
| a,d  |
| a,d  |
| a,d  |
+------+
5 rows in set (0.04 sec)
```

如果将`SET`列设置为不支持的值，则该值将被忽略并发出警告：

```sql
mysql> INSERT INTO myset (col) VALUES ('a,d,d,s');
Query OK, 1 row affected, 1 warning (0.03 sec)

mysql> SHOW WARNINGS;
+---------+------+------------------------------------------+
| Level   | Code | Message                                  |
+---------+------+------------------------------------------+
| Warning | 1265 | Data truncated for column 'col' at row 1 |
+---------+------+------------------------------------------+
1 row in set (0.04 sec)

mysql> SELECT col FROM myset;
+------+
| col  |
+------+
| a,d  |
| a,d  |
| a,d  |
| a,d  |
| a,d  |
| a,d  |
+------+
6 rows in set (0.01 sec)
```

如果启用了严格的 SQL 模式，则尝试插入无效的`SET`值将导致错误。

`SET`值按数字顺序排序。`NULL`值在非`NULL` `SET`值之前排序。

诸如`SUM()`或`AVG()`之类的期望数值参数的函数，如果需要，将参数转换为数字。对于`SET`值，转换操作会导致使用数值值。

通常，您可以使用`FIND_IN_SET()`函数或`LIKE`运算符来搜索`SET`值：

```sql
mysql> SELECT * FROM *tbl_name* WHERE FIND_IN_SET('*value*',*set_col*)>0;
mysql> SELECT * FROM *tbl_name* WHERE *set_col* LIKE '%*value*%';
```

第一条语句查找包含*`value`*集合成员的行。第二条类似，但不完全相同：它查找包含*`value`*的任何地方的行，即使作为另一个集合成员的子字符串。

以下语句也是允许的：

```sql
mysql> SELECT * FROM *tbl_name* WHERE *set_col* & 1;
mysql> SELECT * FROM *tbl_name* WHERE *set_col* = '*val1*,*val2*';
```

这些语句中的第一个查找包含第一个集合成员的值。第二个查找精确匹配。对于第二种类型的比较要小心。将集合值与`'*`val1`*,*`val2`*'`进行比较会产生不同的结果，与将值与`'*`val2`*,*`val1`*'`进行比较不同。您应该按照它们在列定义中列出的顺序指定值。

要确定`SET`列的所有可能值，请使用`SHOW COLUMNS FROM *`tbl_name`* LIKE *`set_col`*`并解析输出的`Type`列中的`SET`定义。

在 C API 中，`SET`值以字符串形式返回。有关使用结果集元数据将其与其他字符串区分开的信息，请参阅 C API 基本数据结构。
