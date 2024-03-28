> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-insert.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-insert.html)

#### 22.4.4.1 向表中插入记录

您可以使用`insert()`方法与`values()`方法将记录插入到现有的关系表中。`insert()`方法接受单个列或表中的所有列。使用一个或多个`values()`方法来指定要插入的值。

##### 插入完整记录

要插入完整记录，将表中的所有列传递给`insert()`方法。然后，对于每个列，将一个值传递给`values()`方法。例如，要向`world_x`数据库中的 city 表添加新记录，请插入以下记录并按两次**Enter**。

```sql
mysql-py> db.city.insert("ID", "Name", "CountryCode", "District", "Info").values(
None, "Olympia", "USA", "Washington", '{"Population": 5000}')
```

city 表有五列：ID、Name、CountryCode、District 和 Info。每个值必须与其代表的列的数据类型匹配。

##### 插入部分记录

以下示例将值插入到 city 表的 ID、Name 和 CountryCode 列中。

```sql
mysql-py> db.city.insert("ID", "Name", "CountryCode").values(
None, "Little Falls", "USA").values(None, "Happy Valley", "USA")
```

当您使用`insert()`方法指定列时，值的数量必须与列的数量匹配。在前面的示例中，您必须提供三个值以匹配指定的三列。

##### 相关信息

+   请参阅 TableInsertFunction 以获取完整的语法定义。
