> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-modify.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-modify.html)

#### 22.4.3.4 修改文档

您可以使用`modify()`方法来更新集合中的一个或多个文档。X DevAPI 提供了与`modify()`方法一起使用的附加方法，以：

+   设置和取消文档中的字段。

+   追加、插入和删除数组

+   绑定、限制和排序要修改的文档。

##### 设置和取消文档字段

`modify()`方法通过过滤集合以仅包括要修改的文档，然后对这些文档应用您指定的操作来工作。

在以下示例中，`modify()`方法使用搜索条件标识要更改的文档，然后`set()`方法替换嵌套的 demographics 对象中的两个值。

```sql
mysql-py> db.countryinfo.modify("Code = 'SEA'").set(
"demographics", {"LifeExpectancy": 78, "Population": 28})
```

修改文档后，请使用`find()`方法验证更改。

要从文档中删除内容，请使用`modify()`和`unset()`方法。例如，以下查询从符合搜索条件的文档中删除 GNP。

```sql
mysql-py> db.countryinfo.modify("Name = 'Sealand'").unset("GNP")
```

使用`find()`方法验证更改。

```sql
mysql-py> db.countryinfo.find("Name = 'Sealand'")
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

要向数组字段追加元素，或者插入、删除数组中的元素，请使用`array_append()`、`array_insert()`或`array_delete()`方法。以下示例修改`countryinfo`集合以启用对国际机场的跟踪。

第一个示例使用`modify()`和`set()`方法在所有文档中创建一个新的 Airports 字段。

注意

在修改文档时没有指定搜索条件时要小心；这样会修改集合中的所有文档。

```sql
mysql-py> db.countryinfo.modify("true").set("Airports", [])
```

添加了 Airports 字段后，下一个示例使用`array_append()`方法向其中一个文档添加一个新机场。在以下示例中，*$.Airports*代表当前文档的 Airports 字段。

```sql
mysql-py> db.countryinfo.modify("Name = 'France'").array_append("$.Airports", "ORY")
```

使用`find()`查看更改。

```sql
mysql-py> db.countryinfo.find("Name = 'France'")
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

要在数组中的不同位置插入元素，请使用`array_insert()`方法来指定要在路径表达式中插入的索引。在这种情况下，索引为 0，或者数组中的第一个元素。

```sql
mysql-py> db.countryinfo.modify("Name = 'France'").array_insert("$.Airports[0]", "CDG")
```

要从数组中删除元素，必须将要删除的元素的索引传递给`array_delete()`方法。

```sql
mysql-py> db.countryinfo.modify("Name = 'France'").array_delete("$.Airports[1]")
```

##### 相关信息

+   MySQL 参考手册提供了帮助您搜索和修改 JSON 值的说明。

+   请参阅 CollectionModifyFunction 获取完整的语法定义。
