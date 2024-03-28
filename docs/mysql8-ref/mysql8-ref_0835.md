# 14.16.8 空间运算符函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html)

OpenGIS 提出了许多可以生成几何的函数。它们旨在实现空间运算符。这些函数支持除根据[Open Geospatial Consortium](http://www.opengeospatial.org)规范不适用的参数类型组合之外的所有参数类型组合。

MySQL 还实现了一些作为 OpenGIS 扩展的特定函数，如函数描述中所述。此外，第 14.16.7 节，“几何属性函数”讨论了几个从现有几何构造新几何的函数。请参阅该部分以了解这些函数的描述：

+   `ST_Envelope(*`g`*)`

+   `ST_StartPoint(*`ls`*)`

+   `ST_EndPoint(*`ls`*)`

+   `ST_PointN(*`ls`*, *`N`*)`

+   `ST_ExteriorRing(*`poly`*)`

+   `ST_InteriorRingN(*`poly`*, *`N`*)`

+   `ST_GeometryN(*`gc`*, *`N`*)`

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何参数为`NULL`，则返回值为`NULL`。

+   如果任何几何参数不是语法上良好形式的几何，则会出现`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义空间参考系统（SRS）中的语法上良好形式的几何，则会出现`ER_SRS_NOT_FOUND`错误。

+   对于需要多个几何参数的函数，如果这些参数不在相同的 SRS 中，则会出现`ER_GIS_DIFFERENT_SRIDS`错误。

+   如果任何几何参数具有地理 SRS 的 SRID 值，且函数不处理地理几何，则会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

+   对于地理 SRS 几何参数，如果任何参数的经度或纬度超出范围，将会出现错误：

    +   如果经度值不在范围 (−180, 180] 内，会出现一个`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前是 `ER_LONGITUDE_OUT_OF_RANGE`）。

    +   如果纬度值不在范围 [−90, 90] 内，会出现一个`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前是 `ER_LATITUDE_OUT_OF_RANGE`）。

    显示的范围是以度为单位的。如果一个 SRS 使用另一个单位，范围将使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

+   否则，返回值为非`NULL`。

这些空间操作函数可用：

+   [`ST_Buffer(*`g`*, *`d`* [, *`strategy1`* [, *`strategy2`* [, *`strategy3`*]]])`](spatial-operator-functions.html#function_st-buffer)

    返回一个表示所有距离几何值 *`g`* 距离小于或等于距离 *`d`* 的点的几何体。结果与几何参数相同的 SRS 中。

    如果几何参数为空，`ST_Buffer()` 返回一个空几何体。

    如果距离为 0，`ST_Buffer()` 返回不变的几何参数：

    ```sql
    mysql> SET @pt = ST_GeomFromText('POINT(0 0)');
    mysql> SELECT ST_AsText(ST_Buffer(@pt, 0));
    +------------------------------+
    | ST_AsText(ST_Buffer(@pt, 0)) |
    +------------------------------+
    | POINT(0 0)                   |
    +------------------------------+
    ```

    如果几何参���在笛卡尔 SRS 中：

    +   `ST_Buffer()` 支持`Polygon`和`MultiPolygon`值的负距离，以及包含`Polygon`或`MultiPolygon`值的几何集合。

    +   如果结果减少到消失，结果是一个空几何体。

    +   对于具有负距离的`Point`、`MultiPoint`、`LineString`和`MultiLineString`值以及不包含任何`Polygon`或`MultiPolygon`值的几何集合的`ST_Buffer()`会出现一个`ER_WRONG_ARGUMENTS`错误。

    如果几何参数在地理 SRS 中：

    +   在 MySQL 8.0.26 之前，会出现一个`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    +   截至 MySQL 8.0.26，允许在地理 SRS 中使用`Point`几何体。对于非`Point`几何体，仍会出现一个`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    对于允许地理`Point`几何体的 MySQL 版本：

    +   如果距离不为负数且未指定任何策略，则函数返回其 SRS 中点的地理缓冲区。距离参数必须以 SRS 距离单位（目前始终为米）表示。

    +   如果距离为负数或指定了任何策略（除了`NULL`），则会发生`ER_WRONG_ARGUMENTS`错误。

    `ST_Buffer()`允许在距离参数后跟随最多三个可选策略参数。策略影响缓冲计算。这些参数是由`ST_Buffer_Strategy()`函数生成的字节字符串值，用于点、连接和端点策略：

    +   点策略适用于`Point`和`MultiPoint`几何体。如果未指定点策略，则默认为`ST_Buffer_Strategy('point_circle', 32)`。

    +   连接策略适用于`LineString`、`MultiLineString`、`Polygon`和`MultiPolygon`几何体。如果未指定连接策略，则默认为`ST_Buffer_Strategy('join_round', 32)`。

    +   端点策略适用于`LineString`和`MultiLineString`几何体。如果未指定端点策略，则默认为`ST_Buffer_Strategy('end_round', 32)`。

    每种类型最多可以指定一个策略，并且可以以任何顺序给出。

    如果缓冲策略无效，则会发生`ER_WRONG_ARGUMENTS`错误。在以下任何情况下，策略都是无效的：

    +   指定了给定类型（点、连接或端点）的多个策略。

    +   作为策略传递的不是策略的值（例如任意二进制字符串或数字）。

    +   传递了一个`Point`策略，几何体不包含`Point`或`MultiPoint`值。

    +   传递了端点或连接策略，几何体不包含`LineString`、`Polygon`、`MultiLinestring`或`MultiPolygon`值。

    ```sql
    mysql> SET @pt = ST_GeomFromText('POINT(0 0)');
    mysql> SET @pt_strategy = ST_Buffer_Strategy('point_square');
    mysql> SELECT ST_AsText(ST_Buffer(@pt, 2, @pt_strategy));
    +--------------------------------------------+
    | ST_AsText(ST_Buffer(@pt, 2, @pt_strategy)) |
    +--------------------------------------------+
    | POLYGON((-2 -2,2 -2,2 2,-2 2,-2 -2))       |
    +--------------------------------------------+
    ```

    ```sql
    mysql> SET @ls = ST_GeomFromText('LINESTRING(0 0,0 5,5 5)');
    mysql> SET @end_strategy = ST_Buffer_Strategy('end_flat');
    mysql> SET @join_strategy = ST_Buffer_Strategy('join_round', 10);
    mysql> SELECT ST_AsText(ST_Buffer(@ls, 5, @end_strategy, @join_strategy))
    +---------------------------------------------------------------+
    | ST_AsText(ST_Buffer(@ls, 5, @end_strategy, @join_strategy))   |
    +---------------------------------------------------------------+
    | POLYGON((5 5,5 10,0 10,-3.5355339059327373 8.535533905932738, |
    | -5 5,-5 0,0 0,5 0,5 5))                                       |
    +---------------------------------------------------------------+
    ```

+   [`ST_Buffer_Strategy(*`strategy`* [, *`points_per_circle`*])`](spatial-operator-functions.html#function_st-buffer-strategy)

    此函数返回一个策略字节字符串，用于影响`ST_Buffer()`的缓冲计算。

    关于策略的信息可在[Boost.org](http://www.boost.org)上找到。

    第一个参数必须是指示策略选项的字符串：

    +   对于点策略，允许的值为`'point_circle'`和`'point_square'`。

    +   对于连接策略，允许的值为`'join_round'`和`'join_miter'`。

    +   对于端点策略，允许的值为`'end_round'`和`'end_flat'`。

    如果第一个参数是`'point_circle'`、`'join_round'`、`'join_miter'`或`'end_round'`，则*`points_per_circle`*参数必须作为正数值给出。最大*`points_per_circle`*值是`max_points_in_geometry`系统变量的值。

    有关示例，请参阅`ST_Buffer()`的描述。

    `ST_Buffer_Strategy()`处理其参数如本节介绍的那样，但有以下例外：

    +   如果任何参数无效，则会出现`ER_WRONG_ARGUMENTS`错误。

    +   如果第一个参数是`'point_square'`或`'end_flat'`，则*`points_per_circle`*参数不能给出，否则会出现`ER_WRONG_ARGUMENTS`错误。

+   `ST_ConvexHull(*`g`*)`

    返回一个表示几何值*`g`*的凸包的几何体。

    此函数通过首先检查几何体的顶点是否共线来计算几何的凸包。如果是，则函数返回线性凸包，否则返回多边形凸包。此函数通过提取集合的所有组件的所有顶点来处理几何集合，从中创建一个`MultiPoint`值，并计算其凸包。

    `ST_ConvexHull()`处理其参数如本节介绍的那样，但有一个例外：

    +   对于额外条件，如果参数是空几何集合，则返回值为`NULL`。

    ```sql
    mysql> SET @g = 'MULTIPOINT(5 0,25 0,15 10,15 25)';
    mysql> SELECT ST_AsText(ST_ConvexHull(ST_GeomFromText(@g)));
    +-----------------------------------------------+
    | ST_AsText(ST_ConvexHull(ST_GeomFromText(@g))) |
    +-----------------------------------------------+
    | POLYGON((5 0,25 0,15 25,5 0))                 |
    +-----------------------------------------------+
    ```

+   `ST_Difference(*`g1`*, *`g2`*)`

    返回一个几何体，表示几何值*`g1`*和*`g2`*的点集差集。结果与几何参数在相同的 SRS 中。

    截至 MySQL 8.0.26，`ST_Difference()`允许在笛卡尔或地理 SRS 中使用参数。在 MySQL 8.0.26 之前，`ST_Difference()`仅允许在笛卡尔 SRS 中使用参数；对于地理 SRS 中的参数，会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    `ST_Difference()`处理其参数如本节介绍的那样。

    ```sql
    mysql> SET @g1 = Point(1,1), @g2 = Point(2,2);
    mysql> SELECT ST_AsText(ST_Difference(@g1, @g2));
    +------------------------------------+
    | ST_AsText(ST_Difference(@g1, @g2)) |
    +------------------------------------+
    | POINT(1 1)                         |
    +------------------------------------+
    ```

+   `ST_Intersection(*`g1`*, *`g2`*)`

    返回一个几何体，表示几何值*`g1`*和*`g2`*的点集交集。结果与几何参数在相同的 SRS 中。

    截至 MySQL 8.0.27，`ST_Intersection()`允许在笛卡尔或地理 SRS 中使用参数。在 MySQL 8.0.27 之前，`ST_Intersection()`仅允许在笛卡尔 SRS 中使用参数；对于地理 SRS 中的参数，会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    `ST_Intersection()`处理其参数的方式如本节介绍的那样。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('LineString(1 1, 3 3)');
    mysql> SET @g2 = ST_GeomFromText('LineString(1 3, 3 1)');
    mysql> SELECT ST_AsText(ST_Intersection(@g1, @g2));
    +--------------------------------------+
    | ST_AsText(ST_Intersection(@g1, @g2)) |
    +--------------------------------------+
    | POINT(2 2)                           |
    +--------------------------------------+
    ```

+   `ST_LineInterpolatePoint(*`ls`*, *`fractional_distance`*)`

    此函数接受一个`LineString`几何和一个范围在[0.0, 1.0]内的分数距离，并返回`LineString`上给定分数距离从起点到终点的距离的`Point`。它可用于回答诸如哪个`Point`位于由几何参数描述的道路中点的问题。

    该函数在所有空间参考系统中实现了对`LineString`几何的支持，包括笛卡尔和地理坐标系。

    如果*`fractional_distance`*参数为 1.0，则由于近似值计算中的数值不准确性，结果可能并非`LineString`参数的确切最后一个点，而是接近它的点。

    一个相关函数，`ST_LineInterpolatePoints()`，接受类似参数，但返回由`LineString`上每个距离从起点到终点的分数的`Point`值组成的`MultiPoint`。有关这两个函数的示例，请参阅`ST_LineInterpolatePoints()`描述。

    `ST_LineInterpolatePoint()`处理其参数的方式如本节介绍的那样，但有以下例外：

    +   如果几何参数不是`LineString`，则会出现`ER_UNEXPECTED_GEOMETRY_TYPE`错误。

    +   如果分数距离参数超出范围[0.0, 1.0]，则会出现`ER_DATA_OUT_OF_RANGE`错误。

    `ST_LineInterpolatePoint()`是 MySQL 对 OpenGIS 的扩展。此函数在 MySQL 8.0.24 中添加。

+   `ST_LineInterpolatePoints(*`ls`*, *`fractional_distance`*)`

    此函数接受一个`LineString`几何体和一个范围在(0.0, 1.0]的分数距离，并返回由`LineString`起始点加上沿着`LineString`的每个分数距离处的`Point`值组成的`MultiPoint`。它可以用来回答诸如描述几何参数的道路每隔 10%的距离处是哪些`Point`值的问题。

    该函数适用于所有空间参考系统中的`LineString`几何体，包括笛卡尔和地理坐标系。

    如果*`fractional_distance`*参数将 1.0 除以零余数，则由于近似值计算中的数值不准确性，结果可能不包含`LineString`参数的最后一个点，而是接近它的点。

    一个相关的函数，`ST_LineInterpolatePoint()`，接受类似的参数，但返回`LineString`中距离起始点到终点给定分数距离处的`Point`。

    `ST_LineInterpolatePoints()`处理其参数的方式如本节介绍所述，但有以下例外：

    +   如果几何参数不是`LineString`，则会发生`ER_UNEXPECTED_GEOMETRY_TYPE`错误。

    +   如果分数距离参数超出范围[0.0, 1.0]，则会发生`ER_DATA_OUT_OF_RANGE`错误。

    ```sql
    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0,0 5,5 5)');
    mysql> SELECT ST_AsText(ST_LineInterpolatePoint(@ls1, .5));
    +----------------------------------------------+
    | ST_AsText(ST_LineInterpolatePoint(@ls1, .5)) |
    +----------------------------------------------+
    | POINT(0 5)                                   |
    +----------------------------------------------+
    mysql> SELECT ST_AsText(ST_LineInterpolatePoint(@ls1, .75));
    +-----------------------------------------------+
    | ST_AsText(ST_LineInterpolatePoint(@ls1, .75)) |
    +-----------------------------------------------+
    | POINT(2.5 5)                                  |
    +-----------------------------------------------+
    mysql> SELECT ST_AsText(ST_LineInterpolatePoint(@ls1, 1));
    +---------------------------------------------+
    | ST_AsText(ST_LineInterpolatePoint(@ls1, 1)) |
    +---------------------------------------------+
    | POINT(5 5)                                  |
    +---------------------------------------------+
    mysql> SELECT ST_AsText(ST_LineInterpolatePoints(@ls1, .25));
    +------------------------------------------------+
    | ST_AsText(ST_LineInterpolatePoints(@ls1, .25)) |
    +------------------------------------------------+
    | MULTIPOINT((0 2.5),(0 5),(2.5 5),(5 5))        |
    +------------------------------------------------+
    ```

    `ST_LineInterpolatePoints()`是 MySQL 对 OpenGIS 的扩展。此函数在 MySQL 8.0.24 中添加。

+   `ST_PointAtDistance(*`ls`*, *`distance`*)`

    此函数接受一个`LineString`几何体和一个距离范围在 0.0, [`ST_Length(*`ls`*)`]之间的距离，以`LineString`的空间参考系统（SRS）单位测量，并返回沿着`LineString`在该距离处于起始点的`Point`。它可以用来回答诸如描述几何参数的道路起点 400 米处是哪个`Point`值的问题。

    该函数适用于所有空间参考系统中的`LineString`几何体，包括笛卡尔和地理坐标系。

    `ST_PointAtDistance()`处理其参数的方式如本节介绍所述，但有以下例外：

    +   如果几何参数不是`LineString`，则会发生`ER_UNEXPECTED_GEOMETRY_TYPE`错误。

    +   如果分数距离参数在范围 0.0, [`ST_Length(*`ls`*)`] 之外，则会发生 `ER_DATA_OUT_OF_RANGE` 错误。

    `ST_PointAtDistance()` 是 MySQL 对 OpenGIS 的扩展。此函数在 MySQL 8.0.24 中添加。

+   `ST_SymDifference(*`g1`*, *`g2`*)`

    返回表示几何值 *`g1`* 和 *`g2`* 的点集对称差异的几何体，定义如下：

    ```sql
    *g1* symdifference *g2* := (*g1* union *g2*) difference (*g1* intersection *g2*)
    ```

    或者，在函数调用符号中：

    ```sql
    ST_SymDifference(*g1*, *g2*) = ST_Difference(ST_Union(*g1*, *g2*), ST_Intersection(*g1*, *g2*))
    ```

    结果与几何参数相同的空间参考系统（SRS）中。

    截至 MySQL 8.0.27，`ST_SymDifference()` 允许在笛卡尔或地理 SRS 中使用参数。在 MySQL 8.0.27 之前，`ST_SymDifference()` 仅允许在笛卡尔 SRS 中使用参数；对于地理 SRS 中的参数，会出现 `ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS` 错误。

    `ST_SymDifference()` 处理其参数如本节介绍的那样。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('MULTIPOINT(5 0,15 10,15 25)');
    mysql> SET @g2 = ST_GeomFromText('MULTIPOINT(1 1,15 10,15 25)');
    mysql> SELECT ST_AsText(ST_SymDifference(@g1, @g2));
    +---------------------------------------+
    | ST_AsText(ST_SymDifference(@g1, @g2)) |
    +---------------------------------------+
    | MULTIPOINT((1 1),(5 0))               |
    +---------------------------------------+
    ```

+   `ST_Transform(*`g`*, *`target_srid`*)`

    将几何从一个空间参考系统（SRS）转换到另一个。返回值是与输入几何相同类型的几何，所有坐标都转换为目标 SRID，*`target_srid`*。在 MySQL 8.0.30 之前，转换支持仅限于地理 SRS（除非几何参数的 SRID 与目标 SRID 值相同，在这种情况下，对于任何有效的 SRS，返回值是输入几何），此函数不支持笛卡尔 SRS。从 MySQL 8.0.30 开始，支持 Popular Visualisation Pseudo Mercator（EPSG 1024）投影方法，用于 WGS 84 Pseudo-Mercator（SRID 3857）。在 MySQL 8.0.32 及更高版本中，支持扩展到除以下列出的 SRS 外的所有由 EPSG 定义的 SRS：

    +   EPSG 1042 Krovak Modified

    +   EPSG 1043 Krovak Modified（北向）

    +   EPSG 9816 突尼斯矿业网格

    +   EPSG 9826 Lambert Conic Conformal（西向）

    `ST_Transform()` 处理其参数如本节介绍的那样，但有以下例外：

    +   具有地理空间参考系统（SRS）的 SRID 值的几何参数不会产生错误。

    +   如果几何或目标 SRID 参数具有指向未定义空间参考系统（SRS）的 SRID 值，则会发生 `ER_SRS_NOT_FOUND` 错误。

    +   如果几何体在一个`ST_Transform()`无法从中转换的 SRS 中，将会发生`ER_TRANSFORM_SOURCE_SRS_NOT_SUPPORTED`错误。

    +   如果目标 SRID 在一个`ST_Transform()`无法转换到的 SRS 中，将会发生`ER_TRANSFORM_TARGET_SRS_NOT_SUPPORTED`错误。

    +   如果几何体在一个没有 TOWGS84 子句的不是 WGS 84 的 SRS 中，将会发生`ER_TRANSFORM_SOURCE_SRS_MISSING_TOWGS84`错误。

    +   如果目标 SRID 在一个没有 TOWGS84 子句的不是 WGS 84 的 SRS 中，将会发生`ER_TRANSFORM_TARGET_SRS_MISSING_TOWGS84`错误。

    `ST_SRID(*`g`*, *`target_srid`*)`和`ST_Transform(*`g`*, *`target_srid`*)`有以下不同：

    +   `ST_SRID()`改变几何 SRID 值，而不转换其坐标。

    +   `ST_Transform()`会转换几何坐标，同时改变其 SRID 值。

    ```sql
    mysql> SET @p = ST_GeomFromText('POINT(52.381389 13.064444)', 4326);
    mysql> SELECT ST_AsText(@p);
    +----------------------------+
    | ST_AsText(@p)              |
    +----------------------------+
    | POINT(52.381389 13.064444) |
    +----------------------------+
    mysql> SET @p = ST_Transform(@p, 4230);
    mysql> SELECT ST_AsText(@p);
    +---------------------------------------------+
    | ST_AsText(@p)                               |
    +---------------------------------------------+
    | POINT(52.38208611407426 13.065520672345304) |
    +---------------------------------------------+
    ```

+   `ST_Union(*`g1`*, *`g2`*)`

    返回一个几何体，表示几何值*`g1`*和*`g2`*的点集并。结果与几何参数在相同的 SRS 中。

    截至 MySQL 8.0.26，`ST_Union()`允许参数在笛卡尔或地理 SRS 中。在 MySQL 8.0.26 之前，`ST_Union()`仅允许参数在笛卡尔 SRS 中；对于地理 SRS 中的参数，将会发生`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    `ST_Union()`处理其参数如本节介绍的那样。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('LineString(1 1, 3 3)');
    mysql> SET @g2 = ST_GeomFromText('LineString(1 3, 3 1)');
    mysql> SELECT ST_AsText(ST_Union(@g1, @g2));
    +--------------------------------------+
    | ST_AsText(ST_Union(@g1, @g2))        |
    +--------------------------------------+
    | MULTILINESTRING((1 1,3 3),(1 3,3 1)) |
    +--------------------------------------+
    ```
