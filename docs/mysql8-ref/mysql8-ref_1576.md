> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-documents-modify.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-documents-modify.html)

#### 22.3.3.4 修改文档

您可以使用`modify()`方法更新集合中的一个或多个文档。X DevAPI 提供了与`modify()`方法一起使用的其他方法，以：

+   在文档内设置和取消设置字段。

+   追加、插入和删除数组。

+   绑定、限制和排序要修改的文档。

##### 设置和取消设置文档字段

`modify()`方法通过过滤集合以仅包括要修改的文档，然后将您指定的操作应用于这些文档来工作。

在下面的示例中，`modify()`方法使用搜索条件标识要更改的文档，然后`set()`方法替换了嵌套的 demographics 对象中的两个值。

```sql
mysql-js> db.countryinfo.modify("Code = 'SEA'").set(
"demographics", {"LifeExpectancy": 78, "Population": 28})
```

修改文档后，请使用`find()`方法验证更改。

要从文档中删除内容，请使用`modify()`和`unset()`方法。例如，以下查询从符合搜索条件的文档中删除了 GNP。

```sql
mysql-js> db.countryinfo.modify("Name = 'Sealand'").unset("GNP")
```

使用`find()`方法验证更改。

```sql
mysql-js> db.countryinfo.find("Name = 'Sealand'")
{
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

##### 追加、插入和删除数组

要向数组字段追加元素，或在数组中插入、删除元素，请使用`arrayAppend()`、`arrayInsert()`或`arrayDelete()`方法。以下示例修改了`countryinfo`集合以启用对国际机场的跟踪。

第一个示例使用`modify()`和`set()`方法在所有文档中创建一个新的 Airports 字段。

注意

在修改文档时要小心，如果没有指定搜索条件，会修改集合中的所有文档。

```sql
mysql-js> db.countryinfo.modify("true").set("Airports", [])
```

添加了 Airports 字段后，下一个示例使用`arrayAppend()`方法向其中一个文档添加新机场。在下面的示例中，*$.Airports*代表当前文档的 Airports 字段。

```sql
mysql-js> db.countryinfo.modify("Name = 'France'").arrayAppend("$.Airports", "ORY")
```

使用`find()`查看更改。

```sql
mysql-js> db.countryinfo.find("Name = 'France'")
{
    "GNP": 1424285,
    "_id": "00005de917d80000000000000048",
    "Code": "FRA",
    "Name": "France",
    "Airports": [
        "ORY"
    ],
    "IndepYear": 843,
    "geography": {
        "Region": "Western Europe",
        "Continent": "Europe",
        "SurfaceArea": 551500
    },
    "government": {
        "HeadOfState": "Jacques Chirac",
        "GovernmentForm": "Republic"
    },
    "demographics": {
        "Population": 59225700,
        "LifeExpectancy": 78.80000305175781
    }
}
```

要在数组中的不同位置插入元素，请使用`arrayInsert()`方法指定要插入的索引路径表达式。在这种情况下，索引为 0，即数组中的第一个元素。

```sql
mysql-js> db.countryinfo.modify("Name = 'France'").arrayInsert("$.Airports[0]", "CDG")
```

要从数组中删除元素，必须向`arrayDelete()`方法传递要删除的元素的索引。

```sql
mysql-js> db.countryinfo.modify("Name = 'France'").arrayDelete("$.Airports[1]")
```

##### 相关信息

+   MySQL 参考手册提供了帮助您搜索和修改 JSON 值的说明。

+   查看 CollectionModifyFunction 以获取完整的语法定义。
