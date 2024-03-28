> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-select.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-select.html)

#### 22.4.4.2 选择表

您可以使用`select()`方法从数据库中的表中查询并返回记录。X DevAPI 提供了与`select()`方法一起使用的附加方法，以过滤和排序返回的记录。

MySQL 提供以下运算符来指定搜索条件：`OR`（`||`）、`AND`（`&&`）、`XOR`、`IS`、`NOT`、`BETWEEN`、`IN`、`LIKE`、`!=`、`<>`、`>`、`>=`、`<`、`<=`、`&`、`|`、`<<`、`>>`、`+`、`-`、`*`、`/`、`~`和`%`。

##### 选择所有记录

要发出返回现有表中的所有记录的查询，请使用未指定搜索条件的`select()`方法。以下示例从`world_x`数据库中的 city 表中选择所有记录。

注意

限制空的`select()`方法的使用仅限于交互式语句。始终在应用程序代码中使用显式列名选择。

```sql
mysql-py> db.city.select()
+------+------------+-------------+------------+-------------------------+
| ID   | Name       | CountryCode | District   | Info                    |
+------+------------+-------------+------------+-------------------------+
|    1 | Kabul      | AFG         | Kabol      |{"Population": 1780000}  |
|    2 | Qandahar   | AFG         | Qandahar   |{"Population": 237500}   |
|    3 | Herat      | AFG         | Herat      |{"Population": 186800}   |
...    ...          ...           ...          ...
| 4079 | Rafah      | PSE         | Rafah      |{"Population": 92020}    |
+------+------- ----+-------------+------------+-------------------------+
4082 rows in set (0.01 sec)
```

空集（没有匹配记录）返回以下信息：

```sql
Empty set (0.00 sec)

```

##### 过滤搜索

要发出返回一组表列的查询，请使用`select()`方法，并在方括号之间指定要返回的列。此查询从 city 表返回 Name 和 CountryCode 列。

```sql
mysql-py> db.city.select(["Name", "CountryCode"])
+-------------------+-------------+
| Name              | CountryCode |
+-------------------+-------------+
| Kabul             | AFG         |
| Qandahar          | AFG         |
| Herat             | AFG         |
| Mazar-e-Sharif    | AFG         |
| Amsterdam         | NLD         |
...                 ...
| Rafah             | PSE         |
| Olympia           | USA         |
| Little Falls      | USA         |
| Happy Valley      | USA         |
+-------------------+-------------+
4082 rows in set (0.00 sec)
```

要发出返回符合特定搜索条件的行的查询，请使用`where()`方法包含这些条件。例如，以下示例返回以字母 Z 开头的城市的名称和国家代码。

```sql
mysql-py> db.city.select(["Name", "CountryCode"]).where("Name like 'Z%'")
+-------------------+-------------+
| Name              | CountryCode |
+-------------------+-------------+
| Zaanstad          | NLD         |
| Zoetermeer        | NLD         |
| Zwolle            | NLD         |
| Zenica            | BIH         |
| Zagazig           | EGY         |
| Zaragoza          | ESP         |
| Zamboanga         | PHL         |
| Zahedan           | IRN         |
| Zanjan            | IRN         |
| Zabol             | IRN         |
| Zama              | JPN         |
| Zhezqazghan       | KAZ         |
| Zhengzhou         | CHN         |
...                 ...
| Zeleznogorsk      | RUS         |
+-------------------+-------------+
59 rows in set (0.00 sec)
```

您可以使用`bind()`方法将值与搜索条件分开。例如，不要使用"Name = 'Z%' "作为条件，而是替换为以字母开头的名称为前缀的命名占位符，例如*name*。然后将占位符和值包含在`bind()`方法中，如下所示：

```sql
mysql-py> db.city.select(["Name", "CountryCode"]).where(
"Name like :name").bind("name", "Z%")
```

提示

在程序内部，绑定使您可以在表达式中指定占位符，在执行之前用值填充，并可以根据需要受益于自动转义。

始终使用绑定来清理输入。避免使用字符串连接引入查询中的值，这可能会产生无效输入，并且在某些情况下可能会导致安全问题。

##### 项目结果

要使用`AND`运算符发出查询，请在`where()`方法中的搜索条件之间添加运算符。

```sql
mysql-py> db.city.select(["Name", "CountryCode"]).where(
"Name like 'Z%' and CountryCode = 'CHN'")
+----------------+-------------+
| Name           | CountryCode |
+----------------+-------------+
| Zhengzhou      | CHN         |
| Zibo           | CHN         |
| Zhangjiakou    | CHN         |
| Zhuzhou        | CHN         |
| Zhangjiang     | CHN         |
| Zigong         | CHN         |
| Zaozhuang      | CHN         |
...              ...
| Zhangjiagang   | CHN         |
+----------------+-------------+
22 rows in set (0.01 sec)
```

要指定多个条件运算符，可以将搜索条件括在括号中以更改运算符优先级。以下示例演示了`AND`和`OR`运算符的放置位置。

```sql
mysql-py> db.city.select(["Name", "CountryCode"]).where(
"Name like 'Z%' and (CountryCode = 'CHN' or CountryCode = 'RUS')")
+-------------------+-------------+
| Name              | CountryCode |
+-------------------+-------------+
| Zhengzhou         | CHN         |
| Zibo              | CHN         |
| Zhangjiakou       | CHN         |
| Zhuzhou           | CHN         |
...                 ...
| Zeleznogorsk      | RUS         |
+-------------------+-------------+
29 rows in set (0.01 sec)
```

##### 限制、排序和偏移结果

您可以应用`limit()`、`order_by()`和`offset()`方法来管理`select()`方法返回的记录数量和顺序。

要指定结果集中包含的记录数，请将一个值附加到`select()`方法的`limit()`方法。例如，以下查询返回 country 表中的前五条记录。

```sql
mysql-py> db.country.select(["Code", "Name"]).limit(5)
+------+-------------+
| Code | Name        |
+------+-------------+
| ABW  | Aruba       |
| AFG  | Afghanistan |
| AGO  | Angola      |
| AIA  | Anguilla    |
| ALB  | Albania     |
+------+-------------+
5 rows in set (0.00 sec)
```

要指定结果的顺序，请将`order_by()`方法附加到`select()`方法。将一个或多个要排序的列的列表传递给`order_by()`方法，并根据需要选择降序（`desc`）或升序（`asc`）属性。升序是默认的排序类型。

例如，以下查询按 Name 列对所有记录进行排序，然后以降序返回前三条记录。

```sql
mysql-py> db.country.select(["Code", "Name"]).order_by(["Name desc"]).limit(3)
+------+------------+
| Code | Name       |
+------+------------+
| ZWE  | Zimbabwe   |
| ZMB  | Zambia     |
| YUG  | Yugoslavia |
+------+------------+
3 rows in set (0.00 sec)
```

默认情况下，`limit()`方法从表中的第一条记录开始。您可以使用`offset()`方法来更改起始记录。例如，要忽略第一条记录并返回符合条件的接下来三条记录，请将一个值传递给`offset()`方法。

```sql
mysql-py> db.country.select(["Code", "Name"]).order_by(["Name desc"]).limit(3).offset(1)
+------+------------+
| Code | Name       |
+------+------------+
| ZMB  | Zambia     |
| YUG  | Yugoslavia |
| YEM  | Yemen      |
+------+------------+
3 rows in set (0.00 sec)
```

##### 相关信息

+   MySQL 参考手册提供了有关函数和运算符的详细文档。

+   请参阅 TableSelectFunction 以获取完整的语法定义。
