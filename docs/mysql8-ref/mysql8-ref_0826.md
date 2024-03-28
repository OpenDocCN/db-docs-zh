# 14.16.4 从 WKB 值创建几何值的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html)

这些函数的参数是包含 Well-Known Binary (WKB) 表示的 `BLOB`，可选地还有空间参考系统标识符（SRID）。它们返回相应的几何图形。有关 WKB 格式的描述，请参阅 Well-Known Binary (WKB) 格式 格式")。

本节中的函数检测笛卡尔或地理空间参考系统（SRS）中的参数，并返回适合 SRS 的结果。

`ST_GeomFromWKB()` 接受任何几何类型的 WKB 值作为其第一个参数。其他函数为每种几何类型的几何值构造提供类型特定的构造函数。

在 MySQL 8.0 之前，这些函数还接受由 第 14.16.5 节，“MySQL 特定函数创建几何值” 中的函数返回的几何对象。不再允许几何参数，并产生错误。要将调用从使用几何参数迁移到使用 WKB 参数，请遵循以下准则：

+   重写类似 `ST_GeomFromWKB(Point(0, 0))` 的结构为 `Point(0, 0)`。

+   重写类似 `ST_GeomFromWKB(Point(0, 0), 4326)` 的结构为 `ST_SRID(Point(0, 0), 4326)` 或 `ST_GeomFromWKB(ST_AsWKB(Point(0, 0)), 4326)`。

除非另有说明，本节中的函数处理其几何参数如下：

+   如果 WKB 或 SRID 参数为 `NULL`，则返回值为 `NULL`。

+   默认情况下，地理坐标（纬度、经度）按照几何参数的空间参考系统指定的顺序解释。可以提供一个可选的 *`options`* 参数来覆盖默认轴顺序。`options` 包括一个逗号分隔的 `*`key`*=*`value`*` 列表。唯一允许的 *`key`* 值是 `axis-order`，其允许的值为 `lat-long`、`long-lat` 和 `srid-defined`（默认值）。

    如果 *`options`* 参数为 `NULL`，则返回值为 `NULL`。如果 *`options`* 参数无效，则会发生错误以指示原因。

+   如果 SRID 参数引用未定义的空间参考系统（SRS），则会发生 `ER_SRS_NOT_FOUND` 错误。

+   对于地理 SRS 几何参数，如果任何参数的经度或纬度超出范围，则会发生错误：

    +   如果经度值不在范围 (−180, 180] 内，则会发生 `ER_LONGITUDE_OUT_OF_RANGE` 错误。

    +   如果纬度值不在范围[−90, 90]内，则会发生`ER_LATITUDE_OUT_OF_RANGE`错误。

    显示的范围是以度为单位。如果 SRS 使用另一个单位，则范围使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

这些函数可用于从 WKB 值创建几何图形：

+   [`ST_GeomCollFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-geomcollfromwkb), [`ST_GeometryCollectionFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-geomcollfromwkb)

    使用其 WKB 表示和 SRID 构造一个`GeometryCollection`值。

    这些函数处理它们的参数，就像本节介绍的那样。

+   [`ST_GeomFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-geomfromwkb), [`ST_GeometryFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-geomfromwkb)

    使用其 WKB 表示和 SRID 构造任何类型的几何值。

    这些函数处理它们的参数，就像本节介绍的那样。

+   [`ST_LineFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-linefromwkb), [`ST_LineStringFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-linefromwkb)

    使用其 WKB 表示和 SRID 构造一个`LineString`值。

    这些函数处理它们的参数，就像本节介绍的那样。

+   [`ST_MLineFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-mlinefromwkb), [`ST_MultiLineStringFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-mlinefromwkb)

    使用其 WKB 表示和 SRID 构造一个`MultiLineString`值。

    这些函数处理它们的参数，就像本节介绍的那样。

+   [`ST_MPointFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-mpointfromwkb), [`ST_MultiPointFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-mpointfromwkb)

    使用其 WKB 表示和 SRID 构造一个`MultiPoint`值。

    这些函数处理它们的参数，就像本节介绍的那样。

+   [`ST_MPolyFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-mpolyfromwkb), [`ST_MultiPolygonFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-mpolyfromwkb)

    使用其 WKB 表示和 SRID 构造一个`MultiPolygon`值。

    这些函数处理它们的参数，就像本节介绍的那样。

+   [`ST_PointFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-pointfromwkb)

    使用其 WKB 表示和 SRID 构造一个`Point`值。

    `ST_PointFromWKB()`按照本节介绍的方式处理它的参数。

+   [`ST_PolyFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-polyfromwkb), [`ST_PolygonFromWKB(*`wkb`* [, *`srid`* [, *`options`*]])`](gis-wkb-functions.html#function_st-polyfromwkb)

    使用其 WKB 表示和 SRID 构造一个`Polygon`值。

    这些函数按照本节介绍的方式处理它们的参数。
