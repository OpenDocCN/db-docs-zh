> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html)

#### 14.16.7.3 LineString 和 MultiLineString 属性函数

`LineString`由`Point`值组成。您可以提取`LineString`的特定点，计算其包含的点数，或获取其长度。

本节中的一些函数也适用于`MultiLineString`值。

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何参数为`NULL`或任何几何参数为空几何体，则返回值为`NULL`。

+   如果任何几何参数不是语法上格式良好的几何体，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义空间参考系统（SRS）中语法上格式良好的几何体，则会发生`ER_SRS_NOT_FOUND`错误。

+   否则，返回值为非`NULL`。

可用于获取线串属性的这些函数：

+   `ST_EndPoint(*`ls`*)`

    返回`LineString`值*`ls`*的端点的`Point`。

    `ST_EndPoint()`处理其参数如本节介绍的那样。

    ```sql
    mysql> SET @ls = 'LineString(1 1,2 2,3 3)';
    mysql> SELECT ST_AsText(ST_EndPoint(ST_GeomFromText(@ls)));
    +----------------------------------------------+
    | ST_AsText(ST_EndPoint(ST_GeomFromText(@ls))) |
    +----------------------------------------------+
    | POINT(3 3)                                   |
    +----------------------------------------------+
    ```

+   `ST_IsClosed(*`ls`*)`

    对于`LineString`值*`ls`*，`ST_IsClosed()`在*`ls`*闭合时返回 1（即，其`ST_StartPoint()`和`ST_EndPoint()`的值相同）。

    对于`MultiLineString`值*`ls`*，`ST_IsClosed()`在*`ls`*闭合时返回 1（即，对于*`ls`*中的每个`LineString`，`ST_StartPoint()`和`ST_EndPoint()`的值相同）。

    `ST_IsClosed()`如果*`ls`*未闭合，则返回 0，如果*`ls`*为`NULL`，则返回`NULL`。

    `ST_IsClosed()`处理其参数如本节介绍的那样，但有一个例外：

    +   如果几何体具有地理空间参考系统（SRS）的 SRID 值，则会发生`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    ```sql
    mysql> SET @ls1 = 'LineString(1 1,2 2,3 3,2 2)';
    mysql> SET @ls2 = 'LineString(1 1,2 2,3 3,1 1)';

    mysql> SELECT ST_IsClosed(ST_GeomFromText(@ls1));
    +------------------------------------+
    | ST_IsClosed(ST_GeomFromText(@ls1)) |
    +------------------------------------+
    |                                  0 |
    +------------------------------------+

    mysql> SELECT ST_IsClosed(ST_GeomFromText(@ls2));
    +------------------------------------+
    | ST_IsClosed(ST_GeomFromText(@ls2)) |
    +------------------------------------+
    |                                  1 |
    +------------------------------------+

    mysql> SET @ls3 = 'MultiLineString((1 1,2 2,3 3),(4 4,5 5))';

    mysql> SELECT ST_IsClosed(ST_GeomFromText(@ls3));
    +------------------------------------+
    | ST_IsClosed(ST_GeomFromText(@ls3)) |
    +------------------------------------+
    |                                  0 |
    +------------------------------------+
    ```

+   [`ST_Length(*`ls`* [, *`unit`*])`](gis-linestring-property-functions.html#function_st-length)

    返回一个双精度数字，���示`LineString`或`MultiLineString`值*`ls`*在其关联的空间参考系统中的长度。`MultiLineString`值的长度等于其元素的长度之和。

    `ST_Length()`计算结果如下：

    +   如果几何图形在笛卡尔空间参考系统中是有效的`LineString`，则返回值为几何图形的笛卡尔长度。

    +   如果几何图形在笛卡尔空间参考系统中是有效的`MultiLineString`，则返回值为其元素的笛卡尔长度之和。

    +   如果几何图形是地理空间参考系统中有效的`LineString`，则返回值为该 SRS 中几何图形的大地长度，单位为米。

    +   如果几何图形在地理空间参考系统中是有效的`MultiLineString`，则返回值为该 SRS 中其元素的大地长度之和，单位为米。

    `ST_Length()`处理其参数的方式如本节介绍所述，但有以下例外：

    +   如果几何图形不是`LineString`或`MultiLineString`，则返回值为`NULL`。

    +   如果几何图形在几何上无效，则结果要么是未定义长度（即可以是任何数字），要么会发生错误。

    +   如果长度计算结果为`+inf`，则会发生`ER_DATA_OUT_OF_RANGE`错误。

    +   如果几何图形具有地理空间参考系统（SRS），其中经度或纬度超出范围，则会发生错误：

        +   如果经度值不在范围（−180，180]内，则会发生`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

        +   如果纬度值不在范围[−90，90]内，则会发生`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

        显示的范围以度为单位。由于浮点运算，确切的范围限制略有偏差。

    从 MySQL 8.0.16 开始，`ST_Length()`允许一个可选的*`unit`*参数，用于指定返回长度值的线性单位。以下规则适用：

    +   如果指定了一个 MySQL 不支持的单位，将会发生`ER_UNIT_NOT_FOUND`错误。

    +   如果指定了一个受支持的线性单位且 SRID 为 0，则会发生`ER_GEOMETRY_IN_UNKNOWN_LENGTH_UNIT`错误。

    +   如果指定了支持的线性单位且 SRID 不为 0，则结果将以该单位表示。

    +   如果未指定单位，则结果将以几何图形的 SRS 单位（笛卡尔或地理）表示。目前，所有 MySQL SRS 均以米表示。

    如果在`INFORMATION_SCHEMA` `ST_UNITS_OF_MEASURE`表中找到支持的单位，则支持该单位。请参阅 Section 28.3.37, “The INFORMATION_SCHEMA ST_UNITS_OF_MEASURE Table”。

    ```sql
    mysql> SET @ls = ST_GeomFromText('LineString(1 1,2 2,3 3)');
    mysql> SELECT ST_Length(@ls);
    +--------------------+
    | ST_Length(@ls)     |
    +--------------------+
    | 2.8284271247461903 |
    +--------------------+

    mysql> SET @mls = ST_GeomFromText('MultiLineString((1 1,2 2,3 3),(4 4,5 5))');
    mysql> SELECT ST_Length(@mls);
    +-------------------+
    | ST_Length(@mls)   |
    +-------------------+
    | 4.242640687119286 |
    +-------------------+

    mysql> SET @ls = ST_GeomFromText('LineString(1 1,2 2,3 3)', 4326);
    mysql> SELECT ST_Length(@ls);
    +-------------------+
    | ST_Length(@ls)    |
    +-------------------+
    | 313701.9623204328 |
    +-------------------+
    mysql> SELECT ST_Length(@ls, 'metre');
    +-------------------------+
    | ST_Length(@ls, 'metre') |
    +-------------------------+
    |       313701.9623204328 |
    +-------------------------+
    mysql> SELECT ST_Length(@ls, 'foot');
    +------------------------+
    | ST_Length(@ls, 'foot') |
    +------------------------+
    |     1029205.9131247795 |
    +------------------------+
    ```

+   `ST_NumPoints(*`ls`*)`

    返回`LineString`值*`ls`*中的`Point`对象数量。

    `ST_NumPoints()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SET @ls = 'LineString(1 1,2 2,3 3)';
    mysql> SELECT ST_NumPoints(ST_GeomFromText(@ls));
    +------------------------------------+
    | ST_NumPoints(ST_GeomFromText(@ls)) |
    +------------------------------------+
    |                                  3 |
    +------------------------------------+
    ```

+   `ST_PointN(*`ls`*, *`N`*)`

    返回`LineString`值*`ls`*中第*`N`*个`Point`。点从 1 开始编号。

    `ST_PointN()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SET @ls = 'LineString(1 1,2 2,3 3)';
    mysql> SELECT ST_AsText(ST_PointN(ST_GeomFromText(@ls),2));
    +----------------------------------------------+
    | ST_AsText(ST_PointN(ST_GeomFromText(@ls),2)) |
    +----------------------------------------------+
    | POINT(2 2)                                   |
    +----------------------------------------------+
    ```

+   `ST_StartPoint(*`ls`*)`

    返回`LineString`值*`ls`*的起始点`Point`。

    `ST_StartPoint()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SET @ls = 'LineString(1 1,2 2,3 3)';
    mysql> SELECT ST_AsText(ST_StartPoint(ST_GeomFromText(@ls)));
    +------------------------------------------------+
    | ST_AsText(ST_StartPoint(ST_GeomFromText(@ls))) |
    +------------------------------------------------+
    | POINT(1 1)                                     |
    +------------------------------------------------+
    ```
