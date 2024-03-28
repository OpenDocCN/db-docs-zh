> 原文：[`dev.mysql.com/doc/refman/8.0/en/selecting-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/selecting-columns.html)

#### 5.3.4.3 选择特定列

如果您不想看到表中的整行数据，只需按逗号分隔感兴趣的列名即可。例如，如果您想知道动物的出生日期，选择`name`和`birth`列：

```sql
mysql> SELECT name, birth FROM pet;
+----------+------------+
| name     | birth      |
+----------+------------+
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Buffy    | 1989-05-13 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Puffball | 1999-03-30 |
+----------+------------+
```

要找出谁拥有宠物，请使用此查询：

```sql
mysql> SELECT owner FROM pet;
+--------+
| owner  |
+--------+
| Harold |
| Gwen   |
| Harold |
| Benny  |
| Diane  |
| Gwen   |
| Gwen   |
| Benny  |
| Diane  |
+--------+
```

注意，查询仅从每条记录中检索`owner`列，并且有些列会出现多次。为了最小化输出，通过添加关键字`DISTINCT`仅检索每个唯一的输出记录一次：

```sql
mysql> SELECT DISTINCT owner FROM pet;
+--------+
| owner  |
+--------+
| Benny  |
| Diane  |
| Gwen   |
| Harold |
+--------+
```

您可以使用`WHERE`子句将行选择与列选择结合起来。例如，要仅获取狗和猫的出生日期，请使用以下查询：

```sql
mysql> SELECT name, species, birth FROM pet
       WHERE species = 'dog' OR species = 'cat';
+--------+---------+------------+
| name   | species | birth      |
+--------+---------+------------+
| Fluffy | cat     | 1993-02-04 |
| Claws  | cat     | 1994-03-17 |
| Buffy  | dog     | 1989-05-13 |
| Fang   | dog     | 1990-08-27 |
| Bowser | dog     | 1989-08-31 |
+--------+---------+------------+
```
