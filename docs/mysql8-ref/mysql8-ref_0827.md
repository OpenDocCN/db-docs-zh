# 14.16.5 创建几何值的 MySQL 特定函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html)

MySQL 提供了一组有用的非标准函数来创建几何值。本节中描述的函数是 MySQL 对 OpenGIS 规范的扩展。

这些函数从 WKB 值或几何对象作为参数生成几何对象。如果任何参数不是适当的 WKB 或几何表示适当对象类型的表示，则返回值为`NULL`。

例如，您可以直接将`Point()`的几何返回值插入到`POINT`列中：

```sql
INSERT INTO t1 (pt_col) VALUES(Point(1,2));
```

+   [`GeomCollection(*`g`* [, *`g`*] ...)`](gis-mysql-specific-functions.html#function_geomcollection)

    从几何参数构造`GeomCollection`值。

    `GeomCollection()`返回所有包含在参数中的适当几何体，即使存在不受支持的几何体也是如此。

    `GeomCollection()`允许不带参数作为创建空几何体的一种方式。此外，接受 WKT 几何集参数的函数，如`ST_GeomFromText()`，理解 OpenGIS 的`'GEOMETRYCOLLECTION EMPTY'`标准语法和 MySQL 的`'GEOMETRYCOLLECTION()'`非标准语法。

    `GeomCollection()`和`GeometryCollection()`是同义词���首选函数为`GeomCollection()`。

+   [`GeometryCollection(*`g`* [, *`g`*] ...)`](gis-mysql-specific-functions.html#function_geometrycollection)

    从几何参数构造`GeomCollection`值。

    `GeometryCollection()`返回所有包含在参数中的适当几何体，即使存在不受支持的几何体也是如此。

    `GeometryCollection()`允许不带参数作为创建空几何体的一种方式。此外，接受 WKT 几何集参数的函数，如`ST_GeomFromText()`，理解 OpenGIS 的`'GEOMETRYCOLLECTION EMPTY'`标准语法和 MySQL 的`'GEOMETRYCOLLECTION()'`非标准语法。

    `GeomCollection()`和`GeometryCollection()`是同义词，首选函数为`GeomCollection()`。

+   [`LineString(*`pt`* [, *`pt`*] ...)`](gis-mysql-specific-functions.html#function_linestring)

    从一些`Point`或 WKB`Point`参数构建一个`LineString`值。如果参数数量少于两个，返回值为`NULL`。

+   [`MultiLineString(*`ls`* [, *`ls`*] ...)`](gis-mysql-specific-functions.html#function_multilinestring)

    使用`LineString`或 WKB`LineString`参数构建一个`MultiLineString`值。

+   [`MultiPoint(*`pt`* [, *`pt2`*] ...)`](gis-mysql-specific-functions.html#function_multipoint)

    使用`Point`或 WKB`Point`参数构建一个`MultiPoint`值。

+   [`MultiPolygon(*`poly`* [, *`poly`*] ...)`](gis-mysql-specific-functions.html#function_multipolygon)

    从一组`Polygon`或 WKB`Polygon`参数构建一个`MultiPolygon`值。

+   `Point(*`x`*, *`y`*)`

    使用其坐标构建一个`Point`。

+   [`Polygon(*`ls`* [, *`ls`*] ...)`](gis-mysql-specific-functions.html#function_polygon)

    从一些`LineString`或 WKB`LineString`参数构建一个`Polygon`值。如果任何参数不代表一个`LinearRing`（即不是一个封闭且简单的`LineString`），返回值为`NULL`。
