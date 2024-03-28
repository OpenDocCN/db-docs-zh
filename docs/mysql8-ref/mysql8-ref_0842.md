# 14.16.13 空间便利函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html)

本节中的函数提供对几何值的便利操作。

除非另有说明，本节中的函数处理它们的几何参数如下：

+   如果任何参数为`NULL`，返回值为`NULL`。

+   如果任何几何参数不是语法上良好的几何形状，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义的空间参考系统（SRS）中的语法上良好的几何形状，则会发生`ER_SRS_NOT_FOUND`错误。

+   对于接受多个几何参数的函数，如果这些参数不在相同的 SRS 中，则会发生`ER_GIS_DIFFERENT_SRIDS`错误。

+   否则，返回值为非`NULL`。

这些便利函数可用：

+   [`ST_Distance_Sphere(*`g1`*, *`g2`* [, *`radius`*])`](spatial-convenience-functions.html#function_st-distance-sphere)

    返回`Point`或`MultiPoint`参数在球面上的最小距离，单位为米。（对于通用距离计算，请参见`ST_Distance()`函数。）可选的*`radius`*参数应以米为单位给出。

    如果两个几何参数都是有效的笛卡尔`Point`或`MultiPoint`值在 SRID 0 中，返回值是提供的半径上两个几何之间的最短距离。如果省略， 默认半径为 6,370,986 米，点 X 和 Y 坐标分别解释为经度和纬度，单位为度。

    如果两个几何参数都是在地理空间参考系统（SRS）中的有效`Point`或`MultiPoint`值，则返回值是提供的半径上两个几何之间的最短距离。如果省略，默认半径等于平均半径，定义为（2a+b）/3，其中 a 是 SRS 的半长轴，b 是半短轴。

    `ST_Distance_Sphere()`处理其参数如本节介绍的那样，但有以下例外：

    +   支持的几何参数组合为`Point`和`Point`，或`Point`和`MultiPoint`（任意参数顺序）。如果至少一个几何图形既不是`Point`也不是`MultiPoint`，且其 SRID 为 0，则会出现`ER_NOT_IMPLEMENTED_FOR_CARTESIAN_SRS`错误。如果至少一个几何图形既不是`Point`也不是`MultiPoint`，且其 SRID 指向地理 SRS，则会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。如果任何几何图形指向投影 SRS，则会出现`ER_NOT_IMPLEMENTED_FOR_PROJECTED_SRS`错误。

    +   如果任何参数的经度或纬度超出范围，将会出现错误：

        +   如果经度值不在范围（−180, 180]内，将会出现`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

        +   如果纬度值不在范围[−90, 90]内，则会出现`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

        显示的范围以度为单位。如果一个 SRS 使用其他单位，范围将使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

    +   如果*`radius`*参数存在但不是正数，则会出现`ER_NONPOSITIVE_RADIUS`错误。

    +   如果距离超出双精度数的范围，则会出现`ER_STD_OVERFLOW_ERROR`错误。

    ```sql
    mysql> SET @pt1 = ST_GeomFromText('POINT(0 0)');
    mysql> SET @pt2 = ST_GeomFromText('POINT(180 0)');
    mysql> SELECT ST_Distance_Sphere(@pt1, @pt2);
    +--------------------------------+
    | ST_Distance_Sphere(@pt1, @pt2) |
    +--------------------------------+
    |             20015042.813723423 |
    +--------------------------------+
    ```

+   `ST_IsValid(*`g`*)`

    如果参数在几何上有效，则返回 1，如果参数在几何上无效，则返回 0。几何有效性由 OGC 规范定义。

    唯一有效的空几何以空几何集合值的形式表示。在这种情况下，`ST_IsValid()`返回 1。MySQL 不支持 GIS `EMPTY` 值，如 `POINT EMPTY`。

    `ST_IsValid()`处理其参数如本节介绍的那样，但有一个例外：

    +   如果几何图形具有经度或纬度超出范围的地理 SRS，则会出现错误：

        +   如果经度值不在范围(−180, 180]内，则会发生`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误��在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

        +   如果纬度值不在范围[−90, 90]内，则会发生`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

        显示的范围以度为单位。如果 SRS 使用另一个单位，则范围使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

    ```sql
    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0,-0.00 0,0.0 0)');
    mysql> SET @ls2 = ST_GeomFromText('LINESTRING(0 0, 1 1)');
    mysql> SELECT ST_IsValid(@ls1);
    +------------------+
    | ST_IsValid(@ls1) |
    +------------------+
    |                0 |
    +------------------+
    mysql> SELECT ST_IsValid(@ls2);
    +------------------+
    | ST_IsValid(@ls2) |
    +------------------+
    |                1 |
    +------------------+
    ```

+   `ST_MakeEnvelope(*`pt1`*, *`pt2`*)`

    返回围绕两点形成的矩形作为`Point`、`LineString`或`Polygon`。

    计算是在笛卡尔坐标系上进行的，而不是在球体、椭球体或地球上进行的。

    给定两个点*`pt1`*和*`pt2`*，`ST_MakeEnvelope()`在一个类似这样的抽象平面上创建结果几何体：

    +   如果*`pt1`*和*`pt2`*相等，则结果是点*`pt1`*。

    +   否则，如果`(*`pt1`*, *`pt2`*)`是垂直或水平线段，则结果是线段`(*`pt1`*, *`pt2`*)`。

    +   否则，结果是使用*`pt1`*和*`pt2`*作为对角点的多边形。

    结果几何体的 SRID 为 0。

    `ST_MakeEnvelope()`处理其参数如本节介绍中所述，但有以下例外：

    +   如果参数不是`Point`值，则会发生`ER_WRONG_ARGUMENTS`错误。

    +   对于两点的任何坐标值为无穷大或`NaN`的额外条件，会发生`ER_GIS_INVALID_DATA`错误。

    +   如果任何几何体具有地理空间参考系统（SRS）的 SRID 值，则会发生`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    ```sql
    mysql> SET @pt1 = ST_GeomFromText('POINT(0 0)');
    mysql> SET @pt2 = ST_GeomFromText('POINT(1 1)');
    mysql> SELECT ST_AsText(ST_MakeEnvelope(@pt1, @pt2));
    +----------------------------------------+
    | ST_AsText(ST_MakeEnvelope(@pt1, @pt2)) |
    +----------------------------------------+
    | POLYGON((0 0,1 0,1 1,0 1,0 0))         |
    +----------------------------------------+
    ```

+   `ST_Simplify(*`g`*, *`max_distance`*)`

    使用 Douglas-Peucker 算法简化几何体，并返回相同类型的简化值。

    几何体可以是任何几何类型，尽管 Douglas-Peucker 算法可能实际上并未处理每种类型。几何体集合通过逐个将其组件传递给简化算法来处理，并将返回的几何体放入几何体集合中作为结果。

    *`max_distance`*参数是要删除的顶点到其他线段的距离（以输入坐标单位表示）。在简化折线时，距离内的顶点将被移除。

    根据 Boost.Geometry，几何图形可能由于简化过程而变得无效，并且该过程可能会创建自相交。要检查结果的有效性，请将其传递给`ST_IsValid()`。

    `ST_Simplify()`处理其参数的方式如本节介绍所述，但有以下例外：

    +   如果*`max_distance`*参数不是正数，或为`NaN`，则会发生`ER_WRONG_ARGUMENTS`错误。

    ```sql
    mysql> SET @g = ST_GeomFromText('LINESTRING(0 0,0 1,1 1,1 2,2 2,2 3,3 3)');
    mysql> SELECT ST_AsText(ST_Simplify(@g, 0.5));
    +---------------------------------+
    | ST_AsText(ST_Simplify(@g, 0.5)) |
    +---------------------------------+
    | LINESTRING(0 0,0 1,1 1,2 3,3 3) |
    +---------------------------------+
    mysql> SELECT ST_AsText(ST_Simplify(@g, 1.0));
    +---------------------------------+
    | ST_AsText(ST_Simplify(@g, 1.0)) |
    +---------------------------------+
    | LINESTRING(0 0,3 3)             |
    +---------------------------------+
    ```

+   `ST_Validate(*`g`*)`

    根据 OGC 规范验证几何图形。几何图形可以在语法上良好（WKB 值加 SRID），但在几何上无效。例如，此多边形在几何上无效：`POLYGON((0 0, 0 0, 0 0, 0 0, 0 0))`

    如果几何图形在语法上良好且几何上有效，则`ST_Validate()`返回该几何图形，如果参数在语法上不良好或几何上无效或为`NULL`，则返回`NULL`。

    `ST_Validate()`可用于过滤无效的几何数据，尽管会有一定代价。对于需要更精确结果且不受无效数据影响的应用程序，这种惩罚可能是值得的。

    如果几何参数有效，则原样返回，但在检查有效性之前，如果输入的`Polygon`或`MultiPolygon`具有顺时针环，则将这些环反转。如果几何有效，则返回具有反转环的值。

    唯一有效的空几何以空几何集合值的形式表示。在这种情况下，`ST_Validate()`直接返回它，无需进一步检查。

    截至 MySQL 8.0.13，`ST_Validate()`处理其参数的方式如本节介绍所述，但有以下例外：

    +   如果几何图形具有地理 SRS，并且经度或纬度超出范围，则会发生错误：

        +   如果经度值不在范围（−180, 180]内，则会发生`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

        +   如果纬度值不在范围[-90, 90]内，则会发生`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

        显示的范围单位为度。由于浮点运算，确切的范围限制略有偏差。

    在 MySQL 8.0.13 之前，`ST_Validate()`处理其参数的方式如本节介绍中所述，但有以下例外：

    +   如果几何图形不符合语法规范，则返回值为`NULL`。不会发生`ER_GIS_INVALID_DATA`错误。

    +   如果几何图形具有地理空间参考系统（SRS）的 SRID 值，则会发生`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    ```sql
    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0)');
    mysql> SET @ls2 = ST_GeomFromText('LINESTRING(0 0, 1 1)');
    mysql> SELECT ST_AsText(ST_Validate(@ls1));
    +------------------------------+
    | ST_AsText(ST_Validate(@ls1)) |
    +------------------------------+
    | NULL                         |
    +------------------------------+
    mysql> SELECT ST_AsText(ST_Validate(@ls2));
    +------------------------------+
    | ST_AsText(ST_Validate(@ls2)) |
    +------------------------------+
    | LINESTRING(0 0,1 1)          |
    +------------------------------+
    ```
