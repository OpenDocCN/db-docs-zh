# 22.4.5 表中的文档

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-in-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-in-tables.html)

在 MySQL 中，表可以包含传统关系数据、JSON 值或两者兼有。您可以通过将文档存储在具有本机`JSON`数据类型的列中，将传统数据与 JSON 文档结合起来。

本节示例使用`world_x`模式中的城市表。

#### 城市表描述

城市表有五列（或字段）。

```sql
+---------------+------------+-------+-------+---------+------------------+
| Field         | Type       | Null  | Key   | Default | Extra            |
+---------------+------------+-------+-------+---------+------------------+
| ID            | int(11)    | NO    | PRI   | null    | auto_increment   |
| Name          | char(35)   | NO    |       |         |                  |
| CountryCode   | char(3)    | NO    |       |         |                  |
| District      | char(20)   | NO    |       |         |                  |
| Info          | json       | YES   |       | null    |                  |
+---------------+------------+-------+-------+---------+------------------+

```

#### 插入一条记录

要将文档插入表的列中，请按正确顺序将格式良好的 JSON 文档传递给`values()`方法。在下面的示例中，一个文档作为最终值传递，将插入到 Info 列中。

```sql
mysql-py> db.city.insert().values(
None, "San Francisco", "USA", "California", '{"Population":830000}')
```

#### 选择一条记录

您可以发出带有评估表达式中文档值的搜索条件的查询。

```sql
mysql-py> db.city.select(["ID", "Name", "CountryCode", "District", "Info"]).where(
"CountryCode = :country and Info->'$.Population' > 1000000").bind(
'country', 'USA')
+------+----------------+-------------+----------------+-----------------------------+
| ID   | Name           | CountryCode | District       | Info                        |
+------+----------------+-------------+----------------+-----------------------------+
| 3793 | New York       | USA         | New York       | {"Population": 8008278}     |
| 3794 | Los Angeles    | USA         | California     | {"Population": 3694820}     |
| 3795 | Chicago        | USA         | Illinois       | {"Population": 2896016}     |
| 3796 | Houston        | USA         | Texas          | {"Population": 1953631}     |
| 3797 | Philadelphia   | USA         | Pennsylvania   | {"Population": 1517550}     |
| 3798 | Phoenix        | USA         | Arizona        | {"Population": 1321045}     |
| 3799 | San Diego      | USA         | California     | {"Population": 1223400}     |
| 3800 | Dallas         | USA         | Texas          | {"Population": 1188580}     |
| 3801 | San Antonio    | USA         | Texas          | {"Population": 1144646}     |
+------+----------------+-------------+----------------+-----------------------------+
9 rows in set (0.01 sec)
```

#### 相关信息

+   有关更多信息，请参阅与关系表和文档一起工作。

+   详细描述数据类型，请参阅第 13.5 节，“JSON 数据类型”。
