# 14.16.3 从 WKT 值创建几何值的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html)

这些函数的参数是一个 Well-Known Text（WKT）表示，可选地，一个空间参考系统标识符（SRID）。它们返回相应的几何。有关 WKT 格式的描述，请参阅 Well-Known Text (WKT) Format Format")。

本节中的函数检测笛卡尔或地理空间参考系统（SRS）中的参数，并返回适合 SRS 的结果。

`ST_GeomFromText()`接受任何几何类型的 WKT 值作为其第一个参数。其他函数提供了针对每种几何类型构造几何值的特定类型构造函数。

诸如`ST_MPointFromText()`和`ST_GeomFromText()`这样接受`MultiPoint`值的 WKT 格式表示的函数允许值内的各个点被括在括号中。例如，以下两个函数调用都是有效的：

```sql
ST_MPointFromText('MULTIPOINT (1 1, 2 2, 3 3)')
ST_MPointFromText('MULTIPOINT ((1 1), (2 2), (3 3))')
```

诸如`ST_GeomFromText()`这样接受 WKT 几何集合参数的函数理解 OpenGIS `'GEOMETRYCOLLECTION EMPTY'`标准语法和 MySQL `'GEOMETRYCOLLECTION()'`非标准语法。诸如`ST_AsWKT()`这样产生 WKT 值的函数产生`'GEOMETRYCOLLECTION EMPTY'`标准语法：

```sql
mysql> SET @s1 = ST_GeomFromText('GEOMETRYCOLLECTION()');
mysql> SET @s2 = ST_GeomFromText('GEOMETRYCOLLECTION EMPTY');
mysql> SELECT ST_AsWKT(@s1), ST_AsWKT(@s2);
+--------------------------+--------------------------+
| ST_AsWKT(@s1)            | ST_AsWKT(@s2)            |
+--------------------------+--------------------------+
| GEOMETRYCOLLECTION EMPTY | GEOMETRYCOLLECTION EMPTY |
+--------------------------+--------------------------+
```

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何几何参数为`NULL`或不是语法上良好形式的几何，或者 SRID 参数为`NULL`，则返回值为`NULL`。

+   默认情况下，地理坐标（纬度、经度）按照几何参数的空间参考系统指定的顺序进行解释。可以提供一个可选的*`options`*参数来覆盖默认的轴顺序。`options`由逗号分隔的`*`key`*=*`value`*列表组成。唯一允许的*`key`*值是`axis-order`，允许的值为`lat-long`、`long-lat`和`srid-defined`（默认值）。

    如果*`options`*参数为`NULL`，则返回值为`NULL`。如果*`options`*参数无效，则会发生错误以指示原因。

+   如果 SRID 参数引用未定义的空间参考系统（SRS），则会发生`ER_SRS_NOT_FOUND`错误。

+   对于地理 SRS 几何参数，如果任何参数的经度或纬度超出范围，则会发生错误：

    +   如果经度值不在范围（−180, 180]内，则会发生`ER_LONGITUDE_OUT_OF_RANGE`错误。

    +   如果纬度值不在范围[−90, 90]内，则会发生`ER_LATITUDE_OUT_OF_RANGE`错误。

    显示的范围以度为单位。如果 SRS 使用另一个单位，则范围使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

可用于从 WKT 值创建几何体的这些函数：

+   [`ST_GeomCollFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-geomcollfromtext), [`ST_GeometryCollectionFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-geomcollfromtext), [`ST_GeomCollFromTxt(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-geomcollfromtext)

    使用其 WKT 表示和 SRID 构造`GeometryCollection`值。

    这些函数处理它们的参数如本节介绍的那样。

    ```sql
    mysql> SET @g = "MULTILINESTRING((10 10, 11 11), (9 9, 10 10))";
    mysql> SELECT ST_AsText(ST_GeomCollFromText(@g));
    +--------------------------------------------+
    | ST_AsText(ST_GeomCollFromText(@g))         |
    +--------------------------------------------+
    | MULTILINESTRING((10 10,11 11),(9 9,10 10)) |
    +--------------------------------------------+
    ```

+   [`ST_GeomFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-geomfromtext), [`ST_GeometryFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-geomfromtext)

    使用其 WKT 表示和 SRID 构造任何类型的几何值。

    这些函数处理它们的参数如本节介绍的那样。

+   [`ST_LineFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-linefromtext), [`ST_LineStringFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-linefromtext)

    使用其 WKT 表示和 SRID 构造`LineString`值。

    这些函数处理它们的参数如本节介绍的那样。

+   [`ST_MLineFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-mlinefromtext), [`ST_MultiLineStringFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-mlinefromtext)

    使用其 WKT 表示和 SRID 构造`MultiLineString`值。

    这些函数处理它们的参数如本节介绍的那样。

+   [`ST_MPointFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-mpointfromtext), [`ST_MultiPointFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-mpointfromtext)

    使用其 WKT 表示和 SRID 构造`MultiPoint`值。

    这些函数处理它们的参数如本节介绍的那样。

+   [`ST_MPolyFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-mpolyfromtext), [`ST_MultiPolygonFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-mpolyfromtext)

    使用其 WKT 表示和 SRID 构造`MultiPolygon`值。

    这些函数处理它们的参数，如本节介绍中所述。

+   [`ST_PointFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-pointfromtext)

    使用其 WKT 表示和 SRID 构建一个`Point`值。

    `ST_PointFromText()`处理它的参数，如本节介绍中所述。

+   [`ST_PolyFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-polyfromtext), [`ST_PolygonFromText(*`wkt`* [, *`srid`* [, *`options`*]])`](gis-wkt-functions.html#function_st-polyfromtext)

    使用其 WKT 表示和 SRID 构建一个`Polygon`值。

    这些函数处理它们的参数，如本节介绍中所述。
