# 14.16.6 几何格式转换函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-format-conversion-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-format-conversion-functions.html)

MySQL 支持本节列出的函数，用于将几何值从内部几何格式转换为 WKT 或 WKB 格式，或交换 X 和 Y 坐标的顺序。

还有函数用于将字符串从 WKT 或 WKB 格式转换为内部几何格式。参见第 14.16.3 节，“从 WKT 值创建几何值的函数”和第 14.16.4 节，“从 WKB 值创建几何值的函数”。

函数，比如`ST_GeomFromText()`，接受 WKT 几何集合参数，理解 OpenGIS 标准语法`'GEOMETRYCOLLECTION EMPTY'`和 MySQL 非标准语法`'GEOMETRYCOLLECTION()'`。另一种生成空几何集合的方法是调用不带参数的`GeometryCollection()`。生成 WKT 值的函数，比如`ST_AsWKT()`，生成`'GEOMETRYCOLLECTION EMPTY'`标准语法：

```sql
mysql> SET @s1 = ST_GeomFromText('GEOMETRYCOLLECTION()');
mysql> SET @s2 = ST_GeomFromText('GEOMETRYCOLLECTION EMPTY');
mysql> SELECT ST_AsWKT(@s1), ST_AsWKT(@s2);
+--------------------------+--------------------------+
| ST_AsWKT(@s1)            | ST_AsWKT(@s2)            |
+--------------------------+--------------------------+
| GEOMETRYCOLLECTION EMPTY | GEOMETRYCOLLECTION EMPTY |
+--------------------------+--------------------------+
mysql> SELECT ST_AsWKT(GeomCollection());
+----------------------------+
| ST_AsWKT(GeomCollection()) |
+----------------------------+
| GEOMETRYCOLLECTION EMPTY   |
+----------------------------+
```

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何参数为`NULL`，返回值为`NULL`。

+   如果任何几何参数不是语法上良好形式的几何，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数处于未定义的空间参考系统中，轴将按照几何中出现的顺序输出，并发生`ER_WARN_SRS_NOT_FOUND_AXIS_ORDER`警告。

+   默认情况下，地理坐标（纬度、经度）按照几何参数的空间参考系统指定的顺序解释。可提供一个可选的*`options`*参数来覆盖默认轴顺序。`options`由逗号分隔的`*`key`*=*`value`*`列表组成。唯一允许的*`key`*值是`axis-order`，允许的值为`lat-long`、`long-lat`和`srid-defined`（默认值）。

    如果*`options`*参数为`NULL`，返回值为`NULL`。如果*`options`*参数无效，将出现错误指示原因。

+   否则，返回值为非`NULL`。

这些函数可用于格式转换或坐标交换：

+   [`ST_AsBinary(*`g`* [, *`options`*])`](gis-format-conversion-functions.html#function_st-asbinary)，[`ST_AsWKB(*`g`* [, *`options`*])`](gis-format-conversion-functions.html#function_st-asbinary)

    将内部几何格式的值转换为其 WKB 表示形式，并返回二进制结果。

    函数返回值的地理坐标（纬度、经度）顺序由适用于几何参数的空间参考系统指定。可提供一个可选的*`options`*参数来覆盖默认轴顺序。

    `ST_AsBinary()` 和 `ST_AsWKB()` 处理其参数的方式如本节介绍的那样。

    ```sql
    mysql> SET @g = ST_LineFromText('LINESTRING(0 5,5 10,10 15)', 4326);
    mysql> SELECT ST_AsText(ST_GeomFromWKB(ST_AsWKB(@g)));
    +-----------------------------------------+
    | ST_AsText(ST_GeomFromWKB(ST_AsWKB(@g))) |
    +-----------------------------------------+
    | LINESTRING(5 0,10 5,15 10)              |
    +-----------------------------------------+
    mysql> SELECT ST_AsText(ST_GeomFromWKB(ST_AsWKB(@g, 'axis-order=long-lat')));
    +----------------------------------------------------------------+
    | ST_AsText(ST_GeomFromWKB(ST_AsWKB(@g, 'axis-order=long-lat'))) |
    +----------------------------------------------------------------+
    | LINESTRING(0 5,5 10,10 15)                                     |
    +----------------------------------------------------------------+
    mysql> SELECT ST_AsText(ST_GeomFromWKB(ST_AsWKB(@g, 'axis-order=lat-long')));
    +----------------------------------------------------------------+
    | ST_AsText(ST_GeomFromWKB(ST_AsWKB(@g, 'axis-order=lat-long'))) |
    +----------------------------------------------------------------+
    | LINESTRING(5 0,10 5,15 10)                                     |
    +----------------------------------------------------------------+
    ```

+   [`ST_AsText(*`g`* [, *`options`*])`](gis-format-conversion-functions.html#function_st-astext), [`ST_AsWKT(*`g`* [, *`options`*])`](gis-format-conversion-functions.html#function_st-astext)

    将内部几何格式的值转换为其 WKT 表示形式，并返回字符串结果。

    函数返回值的地理坐标（纬度、经度）顺序由适用于几何参数的空间参考系统指定。可提供一个可选的*`options`*参数来覆盖默认轴顺序。

    `ST_AsText()` 和 `ST_AsWKT()` 处理其参数的方式如本节介绍的那样。

    ```sql
    mysql> SET @g = 'LineString(1 1,2 2,3 3)';
    mysql> SELECT ST_AsText(ST_GeomFromText(@g));
    +--------------------------------+
    | ST_AsText(ST_GeomFromText(@g)) |
    +--------------------------------+
    | LINESTRING(1 1,2 2,3 3)        |
    +--------------------------------+
    ```

    `MultiPoint` 值的输出包括每个点周围的括号。例如：

    ```sql
    mysql> SELECT ST_AsText(ST_GeomFromText(@mp));
    +---------------------------------+
    | ST_AsText(ST_GeomFromText(@mp)) |
    +---------------------------------+
    | MULTIPOINT((1 1),(2 2),(3 3))   |
    +---------------------------------+
    ```

+   `ST_SwapXY(*`g`*)`

    接受内部几何格式的参数，交换几何内每个坐标对的 X 和 Y 值，并返回结果。

    `ST_SwapXY()` 处理其参数的方式如本节介绍的那样。

    ```sql
    mysql> SET @g = ST_LineFromText('LINESTRING(0 5,5 10,10 15)');
    mysql> SELECT ST_AsText(@g);
    +----------------------------+
    | ST_AsText(@g)              |
    +----------------------------+
    | LINESTRING(0 5,5 10,10 15) |
    +----------------------------+
    mysql> SELECT ST_AsText(ST_SwapXY(@g));
    +----------------------------+
    | ST_AsText(ST_SwapXY(@g))   |
    +----------------------------+
    | LINESTRING(5 0,10 5,15 10) |
    +----------------------------+
    ```
