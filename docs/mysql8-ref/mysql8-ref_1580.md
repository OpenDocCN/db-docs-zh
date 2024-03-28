> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-table-insert.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-table-insert.html)

#### 22.3.4.1 插入记录到表中

你可以使用`insert()`方法与`values()`方法将记录插入到现有的关系表中。`insert()`方法接受单个列或表中的所有列。使用一个或多个`values()`方法指定要插入的值。

##### 插入完整记录

要插入完整记录，将表中的所有列传递给`insert()`方法。然后对`values()`方法传递表中每列的一个值。例如，要向`world_x`模式中的 city 表添加新记录，请插入以下记录并按两次**Enter**。

```sql
mysql-js> db.city.insert("ID", "Name", "CountryCode", "District", "Info").values(
None, "Olympia", "USA", "Washington", '{"Population": 5000}')
```

city 表有五列：ID、Name、CountryCode、District 和 Info。每个值必须与它所代表的列的数据类型匹配。

##### 插入部分记录

以下示例将值插入到 city 表的 ID、Name 和 CountryCode 列中。

```sql
mysql-js> db.city.insert("ID", "Name", "CountryCode").values(
None, "Little Falls", "USA").values(None, "Happy Valley", "USA")
```

当你使用`insert()`方法指定列时，值的数量必须与列的数量相匹配。在前面的示例中，你必须提供三个值以匹配指定的三列。

##### 相关信息

+   查看 TableInsertFunction 以获取完整的语法定义。
