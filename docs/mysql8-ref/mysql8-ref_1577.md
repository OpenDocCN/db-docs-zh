> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-documents-remove.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-documents-remove.html)

#### 22.3.3.5 删除文档

您可以使用`remove()`方法从模式中的集合中删除一些或所有文档。X DevAPI 提供了与`remove()`方法一起使用的其他方法，用于过滤和排序要删除的文档。

##### 使用条件删除文档

以下示例向`remove()`方法传递了搜索条件。匹配条件的所有文档都将从`countryinfo`集合中删除。在此示例中，有一个文档符合条件。

```sql
mysql-js> db.countryinfo.remove("Code = 'SEA'")
```

##### 删除第一个文档

要删除`countryinfo`集合中的第一个文档，请使用值为 1 的`limit()`方法。

```sql
mysql-js> db.countryinfo.remove("true").limit(1)
```

##### 删除订单中的最后一个文档

以下示例按国家名称删除了`countryinfo`集合中的最后一个文档。

```sql
mysql-js> db.countryinfo.remove("true").sort(["Name desc"]).limit(1)
```

##### 删除集合中的所有文档

您可以删除集合中的所有文档。要这样做，请使用`remove("true")`方法，而不指定搜索条件。

注意

在删除文档时，请谨慎操作，不指定搜索条件会删除集合中的所有文档。

或者，使用`db.drop_collection('countryinfo')`操作来删除`countryinfo`集合。

##### 相关信息

+   请参阅 CollectionRemoveFunction 以获取完整的语法定义。

+   请参阅第 22.3.2 节，“下载和导入 world_x 数据库”以获取重新创建`world_x`模式的说明。
