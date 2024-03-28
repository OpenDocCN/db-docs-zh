> 原文：[`dev.mysql.com/doc/refman/8.0/en/selecting-all.html`](https://dev.mysql.com/doc/refman/8.0/en/selecting-all.html)

#### 5.3.4.1 选择所有数据

最简单的`SELECT`形式从表中检索所有内容：

```sql
mysql> SELECT * FROM pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
| Puffball | Diane  | hamster | f    | 1999-03-30 | NULL       |
+----------+--------+---------+------+------------+------------+
```

这种形式的`SELECT`使用`*`，它是“选择所有列”的简写。如果您想要查看整个表格，例如，在刚刚加载初始数据集后，这是很有用的。例如，您可能会觉得 Bowser 的出生日期似乎不太对。查阅您的原始血统文件，您发现正确的出生年份应该是 1989 年，而不是 1979 年。

至少有两种方法可以解决这个问题：

+   编辑文件`pet.txt`以更正错误，然后使用`DELETE`和`LOAD DATA`清空表格并重新加载：

    ```sql
    mysql> DELETE FROM pet;
    mysql> LOAD DATA LOCAL INFILE 'pet.txt' INTO TABLE pet;
    ```

    但是，如果这样做，您还必须重新输入 Puffball 的记录。

+   仅使用`UPDATE`语句修复错误记录：

    ```sql
    mysql> UPDATE pet SET birth = '1989-08-31' WHERE name = 'Bowser';
    ```

    `UPDATE`仅更改相关记录，无需重新加载表格。

有一个例外，即`SELECT *`选择所有列的原则。如果表中包含不可见列，则`*`不包括它们。有关更多信息，请参见第 15.1.20.10 节，“不可见列”。
