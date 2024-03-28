> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-point-property-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-point-property-functions.html)

#### 14.16.7.2 点属性函数

一个`Point`由 X 和 Y 坐标组成，可以分别使用`ST_X()`和`ST_Y()`函数获取。这些函数还允许一个可选的第二个参数，指定 X 或 Y 坐标值，此时函数结果是第一个参数的`Point`对象，相应坐标被修改为等于第二个参数。

对于具有地理空间参考系统（SRS）的`Point`对象，可以分别使用`ST_Longitude()`和`ST_Latitude()`函数获取经度和纬度。这些函数还允许一个可选的第二个参数，指定经度或纬度值，此时函数结果是第一个参数的`Point`对象，经度或纬度被修改为等于第二个参数。

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何参数为`NULL`，则返回值为`NULL`。

+   如果任何几何参数是有效几何但不是`Point`对象，则会发生`ER_UNEXPECTED_GEOMETRY_TYPE`错误。

+   如果任何几何参数不是语法上良好形式的几何图形，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义的空间参考系统（SRS）中语法上良好形式的几何图形，则会发生`ER_SRS_NOT_FOUND`错误。

+   如果提供了 X 或 Y 坐标参数且值为`-inf`、`+inf`或`NaN`，则会发生`ER_DATA_OUT_OF_RANGE`错误。

+   如果经度或纬度值超出范围，则会发生错误：

    +   如果经度值不在范围（−180, 180]内，则会发生`ER_LONGITUDE_OUT_OF_RANGE`错误。

    +   如果纬度值不在范围[-90, 90]内，则会发生`ER_LATITUDE_OUT_OF_RANGE`错误。

    显示的范围为度数。由于浮点运算，确切的范围限制略有偏差。

+   否则，返回值为非`NULL`。

这些函数可用于获取点属性：

+   [`ST_Latitude(*`p`* [, *`new_latitude_val`*])`](gis-point-property-functions.html#function_st-latitude)

    以一个代表具有地理空间参考系统（SRS）的有效`Point`对象*`p`*的单个参数为参数，`ST_Latitude()`返回*`p`*的纬度值作为双精度数。

    通过可选的第二个参数代表有效的纬度值，`ST_Latitude()`返回一个`Point`对象，其纬度等于第二个参数。

    `ST_Latitude()`处理其参数如本节介绍的那样，另外如果`Point`对象有效但没有地理 SRS，则会发生`ER_SRS_NOT_GEOGRAPHIC`错误。

    ```sql
    mysql> SET @pt = ST_GeomFromText('POINT(45 90)', 4326);
    mysql> SELECT ST_Latitude(@pt);
    +------------------+
    | ST_Latitude(@pt) |
    +------------------+
    |               45 |
    +------------------+
    mysql> SELECT ST_AsText(ST_Latitude(@pt, 10));
    +---------------------------------+
    | ST_AsText(ST_Latitude(@pt, 10)) |
    +---------------------------------+
    | POINT(10 90)                    |
    +---------------------------------+
    ```

    此函数在 MySQL 8.0.12 中添加。

+   [`ST_Longitude(*`p`* [, *`new_longitude_val`*])`](gis-point-property-functions.html#function_st-longitude)

    以一个代表具有地理空间参考系统（SRS）的有效`Point`对象*`p`*的单个参数为参数，`ST_Longitude()`返回*`p`*的经度值作为双精度数。

    通过可选的第二个参数代表有效的经度值，`ST_Longitude()`返回一个`Point`对象，其经度等于第二个参数。

    `ST_Longitude()`处理其参数如本节介绍的那样，另外如果`Point`对象有效但没有地理 SRS，则会发生`ER_SRS_NOT_GEOGRAPHIC`错误。

    ```sql
    mysql> SET @pt = ST_GeomFromText('POINT(45 90)', 4326);
    mysql> SELECT ST_Longitude(@pt);
    +-------------------+
    | ST_Longitude(@pt) |
    +-------------------+
    |                90 |
    +-------------------+
    mysql> SELECT ST_AsText(ST_Longitude(@pt, 10));
    +----------------------------------+
    | ST_AsText(ST_Longitude(@pt, 10)) |
    +----------------------------------+
    | POINT(45 10)                     |
    +----------------------------------+
    ```

    此函数在 MySQL 8.0.12 中添加。

+   [`ST_X(*`p`* [, *`new_x_val`*])`](gis-point-property-functions.html#function_st-x)

    以一个代表有效的`Point`对象*`p`*的单个参数为参数，`ST_X()`返回*`p`*的 X 坐标值作为双精度数。从 MySQL 8.0.12 开始，X 坐标被认为是指`Point`空间参考系统（SRS）定义中首先出现的轴。

    通过可选的第二个参数，`ST_X()`返回一个`Point`对象，其 X 坐标等于第二个参数。从 MySQL 8.0.12 开始，如果`Point`对象具有地理 SRS，则第二个参数必须在经度或纬度值的适当范围内。

    `ST_X()`处理其参数如本节介绍的那样。

    ```sql
    mysql> SELECT ST_X(Point(56.7, 53.34));
    +--------------------------+
    | ST_X(Point(56.7, 53.34)) |
    +--------------------------+
    |                     56.7 |
    +--------------------------+
    mysql> SELECT ST_AsText(ST_X(Point(56.7, 53.34), 10.5));
    +-------------------------------------------+
    | ST_AsText(ST_X(Point(56.7, 53.34), 10.5)) |
    +-------------------------------------------+
    | POINT(10.5 53.34)                         |
    +-------------------------------------------+
    ```

+   [`ST_Y(*`p`* [, *`new_y_val`*])`](gis-point-property-functions.html#function_st-y)

    带有一个表示有效`Point`对象*`p`*的单个参数，`ST_Y()`以双精度数值返回*`p`*的 Y 坐标值。从 MySQL 8.0.12 开始，Y 坐标被认为是指`Point`空间参考系统（SRS）定义中出现的第二个轴。

    使用可选的第二个参数，`ST_Y()`返回一个`Point`对象，其 Y 坐标等于第二个参数。从 MySQL 8.0.12 开始，如果`Point`对象具有地理 SRS，则第二个参数必须在经度或纬度值的适当范围内。

    `ST_Y()`处理其参数的方式如本节介绍的那样。

    ```sql
    mysql> SELECT ST_Y(Point(56.7, 53.34));
    +--------------------------+
    | ST_Y(Point(56.7, 53.34)) |
    +--------------------------+
    |                    53.34 |
    +--------------------------+
    mysql> SELECT ST_AsText(ST_Y(Point(56.7, 53.34), 10.5));
    +-------------------------------------------+
    | ST_AsText(ST_Y(Point(56.7, 53.34), 10.5)) |
    +-------------------------------------------+
    | POINT(56.7 10.5)                          |
    +-------------------------------------------+
    ```
