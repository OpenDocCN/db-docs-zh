> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-remove.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-remove.html)

#### 22.4.3.5 删除文档

您可以使用`remove()`方法从模式中的集合中删除一些或所有文档。X DevAPI 提供了额外的方法与`remove()`方法一起使用，以过滤和排序要删除的文档。

##### 使用条件删除文档

以下示例向`remove()`方法传递了搜索条件。与条件匹配的所有文档都将从`countryinfo`集合中删除。在此示例中，有一个文档符合条件。

```sql
mysql-py> db.countryinfo.remove("Code = 'SEA'")
```

##### 删除第一个文档

要删除`countryinfo`集合中的第一个文档，请使用值为 1 的`limit()`方法。

```sql
mysql-py> db.countryinfo.remove("true").limit(1)
```

##### 删除订单中的最后一个文档

以下示例通过国家名称删除`countryinfo`集合中的最后一个文档。

```sql
mysql-py> db.countryinfo.remove("true").sort(["Name desc"]).limit(1)
```

##### 删除集合中的所有文档

您可以删除集合中的所有文档。要这样做，请使用不指定搜索条件的`remove("true")`方法。

注意

在不指定搜索条件的情况下删除文档时要小心。此操作会从集合中删除所有文档。

或者，使用`db.drop_collection('countryinfo')`操作删除`countryinfo`集合。

##### 相关信息

+   请参阅 CollectionRemoveFunction 获取完整的语法定义。

+   请参阅 Section 22.4.2，“下载和导入 world_x 数据库”，了解重新创建`world_x`模式的说明。
