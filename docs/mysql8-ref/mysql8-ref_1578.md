> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-indexes-create.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-indexes-create.html)

#### 22.3.3.6 创建和删除索引

索引用于快速查找具有特定字段值的文档。没有索引，MySQL 必须从第一个文档开始，然后读取整个集合以查找相关字段。集合越大，成本越高。如果集合很大且对特定字段的查询很常见，则考虑在文档内的特定字段上创建索引。

例如，以下查询在 Population 字段上使用索引性能更好：

```sql
mysql-js> db.countryinfo.find("demographics.Population < 100")
...[*output removed*]
8 documents in set (0.00 sec)
```

`createIndex()`方法创建一个可以用 JSON 文档定义的索引，该文档指定要使用的字段。本节是索引的高级概述。有关更多信息，请参见索引集合。

##### 添加非唯一索引

要创建非唯一索引，请将索引名称和索引信息传递给`createIndex()`方法。禁止重复索引名称。

以下示例指定了一个名为`popul`的索引，针对`demographics`对象中的`Population`字段定义，索引为`Integer`数值。最后一个参数指示字段是否应该需要`NOT NULL`约束。如果值为`false`，则字段可以包含`NULL`值。索引信息是一个包含一个或多个字段详细信息的 JSON 文档。每个字段定义必须包括字段的完整文档路径，并指定字段的类型。

```sql
mysql-js> db.countryinfo.createIndex("popul", {fields:
[{field: '$.demographics.Population', type: 'INTEGER'}]})
```

在这里，索引是使用整数数值创建的。还有其他选项可用，包括用于 GeoJSON 数据的选项。您还可以指定索引类型，这里省略了，因为默认类型“index”是适当的。

##### 添加唯一索引

要创建唯一索引，请将索引名称、索引定义和索引类型“unique”传递给`createIndex()`方法。此示例显示了在国家名称（`"Name"`）上创建的唯一索引，这是`countryinfo`集合中另一个常见字段进行索引。在索引字段描述中，`"TEXT(40)"`表示要索引的字符数，`"required": True`指定字段必须存在于文档中。

```sql
mysql-js> db.countryinfo.createIndex("name",
{"fields": [{"field": "$.Name", "type": "TEXT(40)", "required": true}], "unique": true})
```

##### 删除索引

要删除索引，请将要删除的索引名称传递给`dropIndex()`方法。例如，您可以按如下方式删除“popul”索引：

```sql
mysql-js> db.countryinfo.dropIndex("popul")
```

##### 相关信息

+   有关更多信息，请参见索引集合。

+   有关定义索引的 JSON 文档的更多信息，请参见定义索引。

+   有关完整语法定义，请参见集合索引管理函数。
