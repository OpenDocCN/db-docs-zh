> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-index.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-index.html)

#### 22.4.3.6 创建和删除索引

索引用于快速查找具有特定字段值的文档。没有索引，MySQL 必须从第一个文档开始，然后逐个读取整个集合以查找相关字段。集合越大，成本就越高。如果集合很大且对特定字段的查询很常见，则考虑在文档内的特定字段上创建索引。

例如，以下查询在 Population 字段上使用索引性能更好：

```sql
mysql-py> db.countryinfo.find("demographics.Population < 100")
...[*output removed*]
8 documents in set (0.00 sec)
```

`create_index()`方法创建一个索引，您可以使用指定要使用的字段的 JSON 文档进行定义。本节是索引的高级概述。有关更多信息，请参阅索引集合。

##### 添加一个非唯一索引

要创建非唯一索引，请将索引名称和索引信息传递给`create_index()`方法。禁止重复索引名称。

以下示例指定了一个名为`popul`的索引，针对`demographics`对象中的`Population`字段进行定义，作为`Integer`数值进行索引。最后一个参数指示字段是否应该需要`NOT NULL`约束。如果值为`false`，则该字段可以包含`NULL`值。索引信息是一个包含一个或多个字段详细信息的 JSON 文档，用于包含在索引中。每个字段定义必须包括字段的完整文档路径，并指定字段的类型。

```sql
mysql-py> db.countryinfo.createIndex("popul", {fields:
[{field: '$.demographics.Population', type: 'INTEGER'}]})
```

在这里，索引使用整数数值创建。还有其他选项可用，包括用于 GeoJSON 数据的选项。您还可以指定索引类型，这里省略了，因为默认类型“index”是合适的。

##### 添加一个唯一索引

要创建唯一索引，请将索引名称、索引定义和索引类型“unique”传递给`create_index()`方法。此示例显示了在国家名称（`"Name"`）上创建的唯一索引，这是`countryinfo`集合中另一个常见字段进行索引。在索引字段描述中，`"TEXT(40)"`表示要索引的字符数，`"required": True`指定该字段必须存在于文档中。

```sql
mysql-py> db.countryinfo.create_index("name",
{"fields": [{"field": "$.Name", "type": "TEXT(40)", "required": True}], "unique": True})
```

##### 删除索引

要删除索引，请将要删除的索引名称传递给`drop_index()`方法。例如，您可以按如下方式删除“popul”索引：

```sql
mysql-py> db.countryinfo.drop_index("popul")
```

##### 相关信息

+   有关更多信息，请参阅索引集合。

+   有关定义索引的更多信息，请参阅定义索引中定义索引的 JSON 文档。

+   请参阅集合索引管理函数以获取完整的语法定义。
