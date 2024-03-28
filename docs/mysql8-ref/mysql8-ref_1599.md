> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-delete.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-delete.html)

#### 22.4.4.4 删除表

你可以使用`delete()`方法从数据库中的表中删除一些或所有记录。X DevAPI 提供了额外的方法与`delete()`方法一起使用，以过滤和排序要删除的记录。

##### 使用条件删除记录

以下示例将搜索条件传递给`delete()`方法。所有匹配条件的记录都将从`city`表中删除。在这个示例中，有一条记录符合条件。

```sql
mysql-py> db.city.delete().where("Name = 'Olympia'")
```

##### 删除第一条记录

要删除城市表中的第一条记录，使用值为 1 的`limit()`方法。

```sql
mysql-py> db.city.delete().limit(1)
```

##### 删除表中的所有记录

你可以删除表中的所有记录。要这样做，使用`delete()`方法而不指定搜索条件。

注意

当你删除记录而不指定搜索条件时要小心；这样会删除表中的所有记录。

##### 删除表

`drop_collection()`方法也可用于在 MySQL Shell 中从数据库中删除关系表。例如，要从`world_x`数据库中删除`citytest`表，执行：

```sql
mysql-py> db.drop_collection("citytest")
```

##### 相关信息

+   查看 TableDeleteFunction 获取完整的语法定义。

+   查看第 22.4.2 节，“下载和导入 world_x 数据库”以获取重新创建`world_x`数据库的说明。
