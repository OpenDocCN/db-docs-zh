> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-documents-find.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-documents-find.html)

#### 22.3.3.3 查找文档

您可以使用`find()`方法查询并返回模式中集合中的文档。MySQL Shell 提供额外的方法与`find()`方法一起使用，以过滤和排序返回的文档。

MySQL 提供以下运算符来指定搜索条件：`OR`（`||`）、`AND`（`&&`）、`XOR`、`IS`、`NOT`、`BETWEEN`、`IN`、`LIKE`、`!=`、`<>`、`>`、`>=`、`<`、`<=`、`&`、`|`、`<<`、`>>`、`+`、`-`、`*`、`/`、`~`和`%`。

##### 查找集合中的所有文档

要返回集合中的所有文档，请使用不指定搜索条件的`find()`方法。例如，以下操作返回`countryinfo`集合中的所有文档。

```sql
mysql-js> db.countryinfo.find()
[
     {
          "GNP": 828,
          "Code:": "ABW",
          "Name": "Aruba",
          "IndepYear": null,
          "geography": {
              "Continent": "North America",
              "Region": "Caribbean",
              "SurfaceArea": 193
          },
          "government": {
              "GovernmentForm": "Nonmetropolitan Territory of The Netherlands",
              "HeadOfState": "Beatrix"
          }
          "demographics": {
              "LifeExpectancy": 78.4000015258789,
              "Population": 103000
          },
          ...
      }
 ]
240 documents in set (0.00 sec)
```

该方法生成包含操作信息以及集合中所有文档的结果。

空集（无匹配文档）返回以下信息：

```sql
Empty set (0.00 sec)

```

##### 过滤搜索

您可以使用`find()`方法包含搜索条件。形成搜索条件的表达式语法与传统 MySQL 第十四章，*函数和运算符*相同。您必须将所有表达式括在引号中。为简洁起见，一些示例未显示输出。

一个简单的搜索条件可能包括`Name`字段和我们知道在文档中的值。以下示例返回单个文档：

```sql
mysql-js> db.countryinfo.find("Name = 'Australia'")
[
    {
        "GNP": 351182,
        "Code:": "AUS",
        "Name": "Australia",
        "IndepYear": 1901,
        "geography": {
            "Continent": "Oceania",
            "Region": "Australia and New Zealand",
            "SurfaceArea": 7741220
        },
        "government": {
            "GovernmentForm": "Constitutional Monarchy, Federation",
            "HeadOfState": "Elisabeth II"
        }
        "demographics": {
            "LifeExpectancy": 79.80000305175781,
            "Population": 18886000
        },
    }
]
```

以下示例搜索所有人均 GNP 高于 5000 亿美元的国家。`countryinfo`集合以百万为单位衡量 GNP。

```sql
mysql-js> db.countryinfo.find("GNP > 500000")
...[*output removed*]
10 documents in set (0.00 sec)
```

下面查询中的人口字段嵌入在 demographics 对象中。要访问嵌入字段，请在 demographics 和 Population 之间使用句点来标识关系。文档和字段名称区分大小写。

```sql
mysql-js> db.countryinfo.find("GNP > 500000 and demographics.Population < 100000000")
...[*output removed*]
6 documents in set (0.00 sec)
```

下面表达式中的算术运算符用于查询人均 GNP 高于$30000 的国家。搜索条件可以包括算术运算符和大多数 MySQL 函数。

注意

`countryinfo`集合中有七个文档的人口值为零。因此，在输出末尾会出现警告消息。

```sql
mysql-js> db.countryinfo.find("GNP*1000000/demographics.Population > 30000")
...[*output removed*]
9 documents in set, 7 warnings (0.00 sec)
Warning (Code 1365): Division by 0
Warning (Code 1365): Division by 0
Warning (Code 1365): Division by 0
Warning (Code 1365): Division by 0
Warning (Code 1365): Division by 0
Warning (Code 1365): Division by 0
Warning (Code 1365): Division by 0
```

您可以使用`bind()`方法将值与搜索条件分离。例如，不要将硬编码的国家名称指定为条件，而是用以字母开头的名称后跟冒号组成的命名占位符替换。然后使用`bind(*`占位符`*, *`值`*)`方法如下：

```sql
mysql-js> db.countryinfo.find("Name = :country").bind("country", "Italy")
{
    "GNP": 1161755,
    "_id": "00005de917d8000000000000006a",
    "Code": "ITA",
    "Name": "Italy",
    "Airports": [],
    "IndepYear": 1861,
    "geography": {
        "Region": "Southern Europe",
        "Continent": "Europe",
        "SurfaceArea": 301316
    },
    "government": {
        "HeadOfState": "Carlo Azeglio Ciampi",
        "GovernmentForm": "Republic"
    },
    "demographics": {
        "Population": 57680000,
        "LifeExpectancy": 79
    }
}
1 document in set (0.01 sec)
```

提示

在程序内，绑定使您能够在表达式中指定占位符，在执行之前用值填充，并且可以从适当的自动转义中受益。

始终使用绑定来清理输入。避免使用字符串拼接在查询中引入值，这可能会产生无效输入，并且在某些情况下可能会导致安全问题。

您可以使用占位符和`bind()`方法创建保存的搜索，然后可以使用不同的值调用它们。例如，为一个国家创建一个保存的搜索：

```sql
mysql-js> var myFind = db.countryinfo.find("Name = :country")
mysql-js> myFind.bind('country', 'France')
{
    "GNP": 1424285,
    "_id": "00005de917d80000000000000048",
    "Code": "FRA",
    "Name": "France",
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
1 document in set (0.0028 sec)

mysql-js> myFind.bind('country', 'Germany')
{
    "GNP": 2133367,
    "_id": "00005de917d80000000000000038",
    "Code": "DEU",
    "Name": "Germany",
    "IndepYear": 1955,
    "geography": {
        "Region": "Western Europe",
        "Continent": "Europe",
        "SurfaceArea": 357022
    },
    "government": {
        "HeadOfState": "Johannes Rau",
        "GovernmentForm": "Federal Republic"
    },
    "demographics": {
        "Population": 82164700,
        "LifeExpectancy": 77.4000015258789
    }
}

1 document in set (0.0026 sec)
```

##### 项目结果

您可以返回文档的特定字段，而不是返回所有字段。以下示例返回符合搜索条件的`countryinfo`集合中所有文档的 GNP 和 Name 字段。

使用`fields()`方法传递要返回的字段列表。

```sql
mysql-js> db.countryinfo.find("GNP > 5000000").fields(["GNP", "Name"])
[
    {
        "GNP": 8510700,
        "Name": "United States"
    }
]
1 document in set (0.00 sec)
```

此外，您可以修改返回的文档——添加、重命名、嵌套甚至计算新字段值——使用描述要返回的文档的表达式。例如，使用以下表达式更改字段的名称以仅返回两个文档。

```sql
mysql-js> db.countryinfo.find().fields(
mysqlx.expr('{"Name": upper(Name), "GNPPerCapita": GNP*1000000/demographics.Population}')).limit(2)
{
    "Name": "ARUBA",
    "GNPPerCapita": 8038.834951456311
}
{
    "Name": "AFGHANISTAN",
    "GNPPerCapita": 263.0281690140845
}
```

##### 限制、排序和跳过结果

您可以应用`limit()`、`sort()`和`skip()`方法来管理`find()`方法返回的文档数量和顺序。

要指定结果集中包含的文档数量，请将`limit()`方法附加到`find()`方法，并指定一个值。以下查询返回`countryinfo`集合中的前五个文档。

```sql
mysql-js> db.countryinfo.find().limit(5)
... [*output removed*]
5 documents in set (0.00 sec)
```

要为结果指定顺序，请将`sort()`方法附加到`find()`方法。将一个或多个要按其排序的字段列表传递给`sort()`方法，并根据需要选择降序（`desc`）或升序（`asc`）属性。升序顺序是默认的顺序类型。

例如，以下查询按独立年份字段对所有文档进行排序，然后按降序返回前八个文档。

```sql
mysql-js> db.countryinfo.find().sort(["IndepYear desc"]).limit(8)
... [*output removed*]
8 documents in set (0.00 sec)
```

默认情况下，`limit()`方法从集合中的第一个文档开始。您可以使用`skip()`方法更改起始文档。例如，要忽略第一个文档并返回符合条件的下一个八个文档，请将值 1 传递给`skip()`方法。

```sql
mysql-js> db.countryinfo.find().sort(["IndepYear desc"]).limit(8).skip(1)
... [*output removed*]
8 documents in set (0.00 sec)
```

##### 相关信息

+   MySQL 参考手册提供了有关函数和运算符的详细文档。

+   请参阅 CollectionFindFunction 获取完整的语法定义。
