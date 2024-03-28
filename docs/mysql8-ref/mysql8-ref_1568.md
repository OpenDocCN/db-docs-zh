# 22.2 文档存储概念

> 原文：[`dev.mysql.com/doc/refman/8.0/en/document-store-concepts.html`](https://dev.mysql.com/doc/refman/8.0/en/document-store-concepts.html)

本节解释了在使用 MySQL 作为文档存储时引入的概念。

+   JSON 文档

+   集合

+   CRUD 操作

### JSON 文档

JSON 文档是由键值对组成的数据结构，是使用 MySQL 作为文档存储的基本结构。例如，world_x 模式（本章后面安装）包含这个文档：

```sql
{
    "GNP": 4834,
    "_id": "00005de917d80000000000000023",
    "Code": "BWA",
    "Name": "Botswana",
    "IndepYear": 1966,
    "geography": {
        "Region": "Southern Africa",
        "Continent": "Africa",
        "SurfaceArea": 581730
    },
    "government": {
        "HeadOfState": "Festus G. Mogae",
        "GovernmentForm": "Republic"
    },
    "demographics": {
        "Population": 1622000,
        "LifeExpectancy": 39.29999923706055
    }
}
```

这份文档显示，键的值可以是简单的数据类型，比如整数或字符串，也可以包含其他文档、数组和文档列表。例如，`geography` 键的值由多个键值对组成。JSON 文档在 MySQL 中内部表示为二进制 JSON 对象，通过`JSON` MySQL 数据类型。

文档与传统关系数据库中的表格之间最重要的区别在于，文档的结构不需要提前定义，并且一个集合可以包含具有不同结构的多个文档。另一方面，关系表要求定义其结构，并且表中的所有行必须包含相同的列。

### 集合

集合是用于在 MySQL 数据库中存储 JSON 文档的容器。应用程序通常针对文档集合运行操作，例如查找特定文档。

### CRUD 操作

可以针对集合执行的四个基本操作是创建（Create）、读取（Read）、更新（Update）和删除（Delete）（CRUD）。在 MySQL 中，这意味着：

+   创建一个新文档（插入或添加）

+   读取一个或多个文档（查询）

+   更新一个或多个文档

+   删除一个或多个文档
