> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-table-delete.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-table-delete.html)

#### 22.3.4.4 删除表

您可以使用 `delete()` 方法从数据库中的表中删除一些或所有记录。X DevAPI 提供了额外的方法与 `delete()` 方法一起使用，以过滤和排序要删除的记录。

##### 使用条件删除记录

以下示例向 `delete()` 方法传递搜索条件。与条件匹配的所有记录都将从 city 表中删除。在此示例中，有一条记录符合条件。

```sql
mysql-js> db.city.delete().where("Name = 'Olympia'")
```

##### 删除第一条记录

要删除 city 表中的第一条记录，请使用值为 1 的 `limit()` 方法。

```sql
mysql-js> db.city.delete().limit(1)
```

##### 删除表中的所有记录

您可以删除表中的所有记录。要这样做，请使用不指定搜索条件的 `delete()` 方法。

注意

在不指定搜索条件的情况下删除记录时要小心；这样做会删除表中的所有记录。

##### 删除表

`dropCollection()` 方法也可用于 MySQL Shell 中从数据库中删除关系表。例如，要从 `world_x` 数据库中删除 `citytest` 表，执行以下命令：

```sql
mysql-js> session.dropCollection("world_x", "citytest")
```

##### 相关信息

+   参见 TableDeleteFunction 以获取完整的语法定义。

+   参见 第 22.3.2 节，“下载和导入 world_x 数据库”，了解重新创建 `world_x` 数据库的说明。
