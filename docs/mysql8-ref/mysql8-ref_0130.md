> 原文：[`dev.mysql.com/doc/refman/8.0/en/sorting-rows.html`](https://dev.mysql.com/doc/refman/8.0/en/sorting-rows.html)

#### 5.3.4.4 排序行

您可能已经注意到在前面的示例中，结果行没有按特定顺序显示。当行以某种有意义的方式排序时，检查查询输出通常更容易。要对结果进行排序，请使用`ORDER BY`子句。

这里是按日期排序的动物生日：

```sql
mysql> SELECT name, birth FROM pet ORDER BY birth;
+----------+------------+
| name     | birth      |
+----------+------------+
| Buffy    | 1989-05-13 |
| Bowser   | 1989-08-31 |
| Fang     | 1990-08-27 |
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Slim     | 1996-04-29 |
| Whistler | 1997-12-09 |
| Chirpy   | 1998-09-11 |
| Puffball | 1999-03-30 |
+----------+------------+
```

在字符类型列上，排序——就像所有其他比较操作一样——通常以不区分大小写的方式执行。这意味着对于除大小写外完全相同的列，排序是未定义的。您可以通过使用`BINARY`来强制对列进行区分大小写排序，如下所示：`ORDER BY BINARY *`col_name`*`。

默认排序顺序是升序，最小值优先。要以相反（降序）顺序排序，请在要排序的列名后添加`DESC`关键字：

```sql
mysql> SELECT name, birth FROM pet ORDER BY birth DESC;
+----------+------------+
| name     | birth      |
+----------+------------+
| Puffball | 1999-03-30 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Claws    | 1994-03-17 |
| Fluffy   | 1993-02-04 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Buffy    | 1989-05-13 |
+----------+------------+
```

您可以按多个列进行排序，并且可以按不同方向对不同列进行排序。例如，要按动物类型升序排序，然后按动物类型内的出生日期降序排序（最年轻的动物优先），请使用以下查询：

```sql
mysql> SELECT name, species, birth FROM pet
       ORDER BY species, birth DESC;
+----------+---------+------------+
| name     | species | birth      |
+----------+---------+------------+
| Chirpy   | bird    | 1998-09-11 |
| Whistler | bird    | 1997-12-09 |
| Claws    | cat     | 1994-03-17 |
| Fluffy   | cat     | 1993-02-04 |
| Fang     | dog     | 1990-08-27 |
| Bowser   | dog     | 1989-08-31 |
| Buffy    | dog     | 1989-05-13 |
| Puffball | hamster | 1999-03-30 |
| Slim     | snake   | 1996-04-29 |
+----------+---------+------------+
```

`DESC`关键字仅适用于紧随其后的列名（`birth`）；它不会影响`species`列的排序顺序。
