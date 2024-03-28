> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-add.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-add.html)

#### 22.4.3.2 处理集合

要处理模式中的集合，请使用`db`全局对象访问当前模式。在此示例中，我们使用之前导入的`world_x`模式和`countryinfo`集合。因此，您发出的操作的格式是`db.*`collection_name`*.operation`，其中*`collection_name`*是执行操作的集合的名称。在以下示例中，操作是针对`countryinfo`集合执行的。

##### 添加文档

使用`add()`方法将一个文档或文档列表插入到现有集合中。将以下文档插入到`countryinfo`集合中。由于这是多行内容，请按两次**Enter**键以插入文档。

```sql
mysql-py> db.countryinfo.add(
 {
    "GNP": .6,
    "IndepYear": 1967,
    "Name": "Sealand",
    "Code:": "SEA",
    "demographics": {
        "LifeExpectancy": 79,
        "Population": 27
    },
    "geography": {
        "Continent": "Europe",
        "Region": "British Islands",
        "SurfaceArea": 193
    },
    "government": {
        "GovernmentForm": "Monarchy",
        "HeadOfState": "Michael Bates"
    }
  }
)
```

该方法返回操作的状态。您可以通过搜索文档来验证操作。例如：

```sql
mysql-py> db.countryinfo.find("Name = 'Sealand'")
{
    "GNP": 0.6,
    "_id": "00005e2ff4af00000000000000f4",
    "Name": "Sealand",
    "Code:": "SEA",
    "IndepYear": 1967,
    "geography": {
        "Region": "British Islands",
        "Continent": "Europe",
        "SurfaceArea": 193
    },
    "government": {
        "HeadOfState": "Michael Bates",
        "GovernmentForm": "Monarchy"
    },
    "demographics": {
        "Population": 27,
        "LifeExpectancy": 79
    }
}
```

请注意，除了在添加文档时指定的字段之外，还有一个字段，即`_id`。每个文档都需要一个名为`_id`的标识符字段。`_id`字段的值在同一集合中的所有文档中必须是唯一的。在 MySQL 8.0.11 及更高版本中，文档 ID 是由服务器生成的，而不是客户端，因此 MySQL Shell 不会自动设置`_id`值。如果文档不包含`_id`字段，MySQL 8.0.11 或更高版本的服务器会设置`_id`值。在较早的 8.0 版本或 5.7 版本的 MySQL 服务器中，在这种情况下不会设置`_id`值，因此您必须明确指定。如果不指定，MySQL Shell 将返回错误 5115 文档缺少必需字段。更多信息请参见理解文档 ID。

##### 相关信息

+   请查看 CollectionAddFunction 以获取完整的语法定义。

+   请查看理解文档 ID。
