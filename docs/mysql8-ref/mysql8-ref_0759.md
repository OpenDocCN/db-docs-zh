# 13.4.1 空间数据类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-type-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-type-overview.html)

MySQL 具有与 OpenGIS 类对应的空间数据类型。这些类型的基础在第 13.4.2 节，“OpenGIS 几何模型”中描述。

一些空间数据类型保存单个几何值：

+   `GEOMETRY`

+   `POINT`

+   `LINESTRING`

+   `POLYGON`

`GEOMETRY`可以存储任何类型的几何值。其他单值类型（`POINT`，`LINESTRING`和`POLYGON`）将其值限制为特定的几何类型。

其他空间数据类型保存值的集合：

+   `MULTIPOINT`

+   `MULTILINESTRING`

+   `MULTIPOLYGON`

+   `GEOMETRYCOLLECTION`

`GEOMETRYCOLLECTION`可以存储任何类型的对象集合。其他集合类型（`MULTIPOINT`，`MULTILINESTRING`和`MULTIPOLYGON`）将集合成员限制为具有特定几何类型的成员。

示例：要创建一个名为`geom`的表，其中包含一个名为`g`的列，可以存储任何几何类型的值，请使用以下语句：

```sql
CREATE TABLE geom (g GEOMETRY);
```

具有空间数据类型的列可以具有`SRID`属性，以明确指示存储在列中的值的空间参考系统（SRS）。例如：

```sql
CREATE TABLE geom (
    p POINT SRID 0,
    g GEOMETRY NOT NULL SRID 4326
);
```

如果计划在列上创建`SPATIAL`索引，则可以在具有特定 SRID 的列上创建`SPATIAL`索引，因此，如果计划对列进行索引，请声明具有`NOT NULL`和`SRID`属性：

```sql
CREATE TABLE geom (g GEOMETRY NOT NULL SRID 4326);
```

`InnoDB`表允许笛卡尔和地理 SRS 的`SRID`值。`MyISAM`表允许笛卡尔 SRS 的`SRID`值。

`SRID`属性使空间列受 SRID 限制，这具有以下影响：

+   该列只能包含具有给定 SRID 的值。尝试插入具有不同 SRID 的值会产生错误。

+   优化器可以在列上使用`SPATIAL`索引。请参见第 10.3.3 节，“SPATIAL 索引优化”。

没有`SRID`属性的空间列不受 SRID 限制，并接受任何 SRID 的值。但是，在将列定义修改为包含`SRID`属性之前，优化器无法在其上使用`SPATIAL`索引，这可能需要首先修改列内容，以使所有值具有相同的 SRID。

有关如何在 MySQL 中使用空间数据类型的其他示例，请参见第 13.4.6 节，“创建空间列”。有关空间参考系统的信息，请参见第 13.4.5 节，“空间参考系统支持”。
