> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html)

#### 14.16.7.4 多边形和多多边形属性函数

本节中的函数返回`Polygon`或`MultiPolygon`值的属性。

除非另有说明，本节中的函数处理其几何体参数如下：

+   如果任何参数是`NULL`或任何几何体参数是空几何体，则返回值为`NULL`。

+   如果任何几何体参数不是语法正确的几何体，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何体参数是在未定义的空间参考系统（SRS）中语法正确的几何体，则会发生`ER_SRS_NOT_FOUND`错误。

+   对于接受多个几何体参数的函数，如果这些参数不在相同的 SRS 中，则会发生`ER_GIS_DIFFERENT_SRIDS`错误。

+   否则，返回值为非`NULL`。

可用于获取多边形属性的这些函数：

+   `ST_Area({*`poly`*|*`mpoly`*})`

    返回一个双精度数字，指示`Polygon`或`MultiPolygon`参数的面积，以其空间参考系统中的度量为准。

    截至 MySQL 8.0.13，`ST_Area()`处理其参数如本节介绍中所述，但有以下例外：

    +   如果几何体在几何上无效，则结果要么是未定义的面积（即可以是任何数字），要么会发生错误。

    +   如果几何体有效但不是`Polygon`或`MultiPolygon`对象，则会发生`ER_UNEXPECTED_GEOMETRY_TYPE`错误。

    +   如果几何体在笛卡尔 SRS 中是有效的`Polygon`，则结果是多边形的笛卡尔面积。

    +   如果几何体在笛卡尔 SRS 中是有效的`MultiPolygon`，则结果是多边形的笛卡尔面积之和。

    +   如果几何体在地理 SRS 中是有效的`Polygon`，则结果是该 SRS 中多边形的大地面积，单位为平方米。

    +   如果几何体在地理空间参考系统（SRS）中是有效的`MultiPolygon`，则结果是该 SRS 中多边形的大地面积之和，单位为平方米。

    +   如果面积计算结果为`+inf`，则会发生`ER_DATA_OUT_OF_RANGE`错误。

    +   如果几何体具有经度或纬度超出范围的地理 SRS，则会��生错误：

        +   如果经度值不在(-180, 180]范围内，则会出现`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

        +   如果纬度值不在[-90, 90]范围内，则会出现`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

        显示的范围为度数。由于浮点运算，确切的范围限制略有偏差。

    在 MySQL 8.0.13 之前，`ST_Area()` 处理其参数如本节介绍中所述，但有以下例外：

    +   对于维度为 0 或 1 的参数，结果为 0。

    +   如果几何体为空，则返��值为 0 而不是`NULL`。

    +   对于几何集合，结果是所有组件面积值的总和。如果几何集合为空，则其面积返回为 0。

    +   如果几何体具有地理空间参考系统（SRS）的 SRID 值，则会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    ```sql
    mysql> SET @poly =
           'Polygon((0 0,0 3,3 0,0 0),(1 1,1 2,2 1,1 1))';
    mysql> SELECT ST_Area(ST_GeomFromText(@poly));
    +---------------------------------+
    | ST_Area(ST_GeomFromText(@poly)) |
    +---------------------------------+
    |                               4 |
    +---------------------------------+

    mysql> SET @mpoly =
           'MultiPolygon(((0 0,0 3,3 3,3 0,0 0),(1 1,1 2,2 2,2 1,1 1)))';
    mysql> SELECT ST_Area(ST_GeomFromText(@mpoly));
    +----------------------------------+
    | ST_Area(ST_GeomFromText(@mpoly)) |
    +----------------------------------+
    |                                8 |
    +----------------------------------+
    ```

+   `ST_Centroid({*`poly`*|*`mpoly`*})`

    将`Polygon`或`MultiPolygon`参数的数学质心作为`Point`返回。结果不能保证在`MultiPolygon`上。

    此函数通过计算集合中维度最高的组件的质心点来处理几何集合。这些组件被提取并组成一个单独的`MultiPolygon`、`MultiLineString`或`MultiPoint`以进行质心计算。

    `ST_Centroid()` 处理其参数如本节介绍中所述，但有以下例外：

    +   当参数为一个空的几何集合时，返回值为`NULL`。

    +   如果几何体具有地理空间参考系统（SRS）的 SRID 值，则会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    ```sql
    mysql> SET @poly =
           ST_GeomFromText('POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7,5 5))');
    mysql> SELECT ST_GeometryType(@poly),ST_AsText(ST_Centroid(@poly));
    +------------------------+--------------------------------------------+
    | ST_GeometryType(@poly) | ST_AsText(ST_Centroid(@poly))              |
    +------------------------+--------------------------------------------+
    | POLYGON                | POINT(4.958333333333333 4.958333333333333) |
    +------------------------+--------------------------------------------+
    ```

+   `ST_ExteriorRing(*`poly`*)`

    将`Polygon`值 *`poly`* 的外环作为`LineString`返回。

    `ST_ExteriorRing()` 处理其参数如本节介绍中所述。

    ```sql
    mysql> SET @poly =
           'Polygon((0 0,0 3,3 3,3 0,0 0),(1 1,1 2,2 2,2 1,1 1))';
    mysql> SELECT ST_AsText(ST_ExteriorRing(ST_GeomFromText(@poly)));
    +----------------------------------------------------+
    | ST_AsText(ST_ExteriorRing(ST_GeomFromText(@poly))) |
    +----------------------------------------------------+
    | LINESTRING(0 0,0 3,3 3,3 0,0 0)                    |
    +----------------------------------------------------+
    ```

+   `ST_InteriorRingN(*`poly`*, *`N`*)`

    返回`Polygon`值*`poly`*中第*`N`*个内部环作为`LineString`。环从 1 开始编号。

    `ST_InteriorRingN()` 处理其参数，如本节介绍所述。

    ```sql
    mysql> SET @poly =
           'Polygon((0 0,0 3,3 3,3 0,0 0),(1 1,1 2,2 2,2 1,1 1))';
    mysql> SELECT ST_AsText(ST_InteriorRingN(ST_GeomFromText(@poly),1));
    +-------------------------------------------------------+
    | ST_AsText(ST_InteriorRingN(ST_GeomFromText(@poly),1)) |
    +-------------------------------------------------------+
    | LINESTRING(1 1,1 2,2 2,2 1,1 1)                       |
    +-------------------------------------------------------+
    ```

+   `ST_NumInteriorRing(*`poly`*)`, `ST_NumInteriorRings(*`poly`*)`

    返回`Polygon`值*`poly`*中内部环的数量。

    `ST_NumInteriorRing()` 和 `ST_NuminteriorRings()` 处理其参数，如本节介绍所述。

    ```sql
    mysql> SET @poly =
           'Polygon((0 0,0 3,3 3,3 0,0 0),(1 1,1 2,2 2,2 1,1 1))';
    mysql> SELECT ST_NumInteriorRings(ST_GeomFromText(@poly));
    +---------------------------------------------+
    | ST_NumInteriorRings(ST_GeomFromText(@poly)) |
    +---------------------------------------------+
    |                                           1 |
    +---------------------------------------------+
    ```
