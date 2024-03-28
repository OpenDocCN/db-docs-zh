> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html)

#### 14.16.7.1 通用几何属性函数

本节列出的函数不限制其参数，并接受任何类型的几何值。

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何参数为`NULL`，则返回值为`NULL`。

+   如果任何几何参数不是语法上良好形式的几何图形，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义的空间参考系统（SRS）中语法上良好形式的几何，则会发生`ER_SRS_NOT_FOUND`错误。

+   如果任何 SRID 参数不在 32 位无符号整数的范围内，则会发生`ER_DATA_OUT_OF_RANGE`错误。

+   如果任何 SRID 参数引用未定义的 SRS，则会发生`ER_SRS_NOT_FOUND`错误。

+   否则，返回值为非`NULL`。

这些函数可用于获取几何属性：

+   `ST_Dimension(*`g`*)`

    返回几何值*g*的固有维度。维度可以是−1、0、1 或 2。这些值的含义在第 13.4.2.2 节“几何类”中给出。

    `ST_Dimension()`处理其参数如本节介绍的那样。

    ```sql
    mysql> SELECT ST_Dimension(ST_GeomFromText('LineString(1 1,2 2)'));
    +------------------------------------------------------+
    | ST_Dimension(ST_GeomFromText('LineString(1 1,2 2)')) |
    +------------------------------------------------------+
    |                                                    1 |
    +------------------------------------------------------+
    ```

+   `ST_Envelope(*`g`*)`

    返回几何值*g*的最小边界矩形（MBR）。结果以由边界框的角点定义的`Polygon`值返回：

    ```sql
    POLYGON((MINX MINY, MAXX MINY, MAXX MAXY, MINX MAXY, MINX MINY))
    ```

    ```sql
    mysql> SELECT ST_AsText(ST_Envelope(ST_GeomFromText('LineString(1 1,2 2)')));
    +----------------------------------------------------------------+
    | ST_AsText(ST_Envelope(ST_GeomFromText('LineString(1 1,2 2)'))) |
    +----------------------------------------------------------------+
    | POLYGON((1 1,2 1,2 2,1 2,1 1))                                 |
    +----------------------------------------------------------------+
    ```

    如果参数是点或垂直或水平线段，则`ST_Envelope()`返回点或线段作为其 MBR，而不是返回无效多边形：

    ```sql
    mysql> SELECT ST_AsText(ST_Envelope(ST_GeomFromText('LineString(1 1,1 2)')));
    +----------------------------------------------------------------+
    | ST_AsText(ST_Envelope(ST_GeomFromText('LineString(1 1,1 2)'))) |
    +----------------------------------------------------------------+
    | LINESTRING(1 1,1 2)                                            |
    +----------------------------------------------------------------+
    ```

    `ST_Envelope()`处理其参数如本节介绍的那样，但有一个例外：

    +   如果几何图形具有地理空间参考系统（SRS）的 SRID 值，则会发生`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

+   `ST_GeometryType(*`g`*)`

    返回一个二进制字符串，指示几何实例*g*是其成员的几何类型的名称。该名称对应于可实例化的`Geometry`子类之一。

    `ST_GeometryType()` 处理其参数如本节介绍的那样。

    ```sql
    mysql> SELECT ST_GeometryType(ST_GeomFromText('POINT(1 1)'));
    +------------------------------------------------+
    | ST_GeometryType(ST_GeomFromText('POINT(1 1)')) |
    +------------------------------------------------+
    | POINT                                          |
    +------------------------------------------------+
    ```

+   `ST_IsEmpty(*`g`*)`

    此函数是一个占位符，对于空几何体集合值返回 1，否则返回 0。

    唯一有效的空几何体以空几何体集合值的形式表示。MySQL 不支持 GIS `EMPTY` 值，如 `POINT EMPTY`。

    `ST_IsEmpty()` 处理其参数如本节介绍的那样。

+   `ST_IsSimple(*`g`*)`

    如果几何值 *`g`* 根据 ISO *SQL/MM Part 3: Spatial* 标准是简单的，则返回 1。如果参数不简单，则 `ST_IsSimple()` 返回 0。

    在 Section 13.4.2, “The OpenGIS Geometry Model” 下给出的可实例化几何类的描述包括导致类实例被分类为非简单的特定条件。

    `ST_IsSimple()` 处理其参数如本节介绍的那样，除了：

    +   如果几何体具有地理 SRS，其经度或纬度超出范围，则会出现错误：

        +   如果经度值不在范围内（−180, 180]，则会出现 `ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE` 错误（在 MySQL 8.0.12 之前为 `ER_LONGITUDE_OUT_OF_RANGE`）。

        +   如果纬度值不在范围内 [−90, 90]，则会出现 `ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE` 错误（在 MySQL 8.0.12 之前为 `ER_LATITUDE_OUT_OF_RANGE`）。

        显示的范围以度为单位。由于浮点运算，确切的范围限制略有偏差。

+   [`ST_SRID(*`g`* [, *`srid`*])`](gis-general-property-functions.html#function_st-srid)

    使用表示有效几何对象 *`g`* 的单个参数，`ST_SRID()` 返回一个整数，指示与 *`g`* 关联的空间参考系统（SRS）的 ID。

    带有可选的第二个参数表示有效的 SRID 值，`ST_SRID()` 返回一个与其第一个参数类型相同的对象，其 SRID 值等于第二个参数。这仅设置对象的 SRID 值；不执行任何坐标值的转换。

    `ST_SRID()`处理其参数的方式如本节介绍中所述，但有一个例外：

    +   对于单参数语法，`ST_SRID()`即使引用未定义的 SRS 也会返回几何图形的 SRID。不会发生`ER_SRS_NOT_FOUND`错误。

    `ST_SRID(*`g`*, *`target_srid`*)`和`ST_Transform(*`g`*, *`target_srid`*)`的区别如下：

    +   `ST_SRID()`改变几何图形的 SRID 值，而不改变其坐标。

    +   `ST_Transform()`会在改变 SRID 值的同时转换几何图形的坐标。

    ```sql
    mysql> SET @g = ST_GeomFromText('LineString(1 1,2 2)', 0);
    mysql> SELECT ST_SRID(@g);
    +-------------+
    | ST_SRID(@g) |
    +-------------+
    |           0 |
    +-------------+
    mysql> SET @g = ST_SRID(@g, 4326);
    mysql> SELECT ST_SRID(@g);
    +-------------+
    | ST_SRID(@g) |
    +-------------+
    |        4326 |
    +-------------+
    ```

    通过将一个 MySQL 特定函数创建空间值的结果和一个 SRID 值传递给`ST_SRID()`，可以在特定的 SRID 中创建几何图形。例如：

    ```sql
    SET @g1 = ST_SRID(Point(1, 1), 4326);
    ```

    然而，该方法会在 SRID 0 中创建几何图形，然后将其转换为 SRID 4326（WGS 84）。更好的选择是一开始就使用正确的空间参考系统创建几何图形。例如：

    ```sql
    SET @g1 = ST_PointFromText('POINT(1 1)', 4326);
    SET @g1 = ST_GeomFromText('POINT(1 1)', 4326);
    ```

    `ST_SRID()`的两个参数形式对于纠正或更改具有不正确 SRID 的几何图形非常有用。
