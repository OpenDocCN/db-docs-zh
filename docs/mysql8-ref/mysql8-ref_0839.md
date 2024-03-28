# 14.16.10 空间 Geohash 函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-geohash-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-geohash-functions.html)

Geohash 是一种将任意精度的纬度和经度坐标编码为文本字符串的系统。Geohash 值是仅包含字符`"0123456789bcdefghjkmnpqrstuvwxyz"`的字符串。

本节中的函数使得可以操作 geohash 值，从而为应用程序提供了导入和导出 geohash 数据、以及索引和搜索 geohash 值的功能。

除非另有规定，本节中的函数将按照以下方式处理它们的几何参数：

+   如果任何参数为`NULL`，则返回值为`NULL`。

+   如果任何参数无效，则会发生错误。

+   如果任何参数的经度或纬度超出范围，则会发生错误：

    +   如果经度值不在范围（−180, 180]内，则会发生`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

    +   如果纬度值不在范围[−90, 90]内，则会发生`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`��。

    显示的范围以度为单位。由于浮点运算，确切的范围限制略有偏差。

+   如果任何点参数的 SRID 不是 0 或 4326，则会发生`ER_SRS_NOT_FOUND`错误。*`point`*参数的 SRID 有效性不会被检查。

+   如果任何 SRID 参数引用未定义的空间参考系统（SRS），则会发生`ER_SRS_NOT_FOUND`错误。

+   如果任何 SRID 参数不在 32 位无符号整数的范围内，则会发生`ER_DATA_OUT_OF_RANGE`错误。

+   否则，返回值为非`NULL`。

这些 geohash 函数可用：

+   `ST_GeoHash(*`longitude`*, *`latitude`*, *`max_length`*)`, `ST_GeoHash(*`point`*, *`max_length`*)`

    返回一个连接字符集和排序规则中的 geohash 字符串。

    对于第一个语法，*`longitude`*必须是范围[−180, 180]内的数字，*`latitude`*必须是范围[−90, 90]内的数字。对于第二个语法，需要一个`POINT`值，其中 X 和 Y 坐标分别在经度和纬度的有效范围内。

    结果字符串的长度不超过*`max_length`*个字符，上限为 100。字符串可能短于*`max_length`*个字符，因为创建地理哈希值的算法会一直持续，直到生成一个精确表示位置或*`max_length`*个字符的字符串，以先出现者为准。

    `ST_GeoHash()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SELECT ST_GeoHash(180,0,10), ST_GeoHash(-180,-90,15);
    +----------------------+-------------------------+
    | ST_GeoHash(180,0,10) | ST_GeoHash(-180,-90,15) |
    +----------------------+-------------------------+
    | xbpbpbpbpb           | 000000000000000         |
    +----------------------+-------------------------+
    ```

+   `ST_LatFromGeoHash(*`geohash_str`*)`

    返回地理哈希字符串值的纬度，作为双精度数在范围[−90, 90]内。

    `ST_LatFromGeoHash()`解码函数从*`geohash_str`*参数中最多读取 433 个字符。这代表了坐标值内部表示中信息的上限。超过第 433 个字符的字符将被忽略，即使它们在其他情况下是非法的并产生错误。

    `ST_LatFromGeoHash()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SELECT ST_LatFromGeoHash(ST_GeoHash(45,-20,10));
    +------------------------------------------+
    | ST_LatFromGeoHash(ST_GeoHash(45,-20,10)) |
    +------------------------------------------+
    |                                      -20 |
    +------------------------------------------+
    ```

+   `ST_LongFromGeoHash(*`geohash_str`*)`

    返回地理哈希字符串值的经度，作为双精度数在范围[−180, 180]内。

    关于从*`geohash_str`*参数中处理的最大字符数的描述在`ST_LatFromGeoHash()`中的备注也适用于`ST_LongFromGeoHash()`。

    `ST_LongFromGeoHash()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SELECT ST_LongFromGeoHash(ST_GeoHash(45,-20,10));
    +-------------------------------------------+
    | ST_LongFromGeoHash(ST_GeoHash(45,-20,10)) |
    +-------------------------------------------+
    |                                        45 |
    +-------------------------------------------+
    ```

+   `ST_PointFromGeoHash(*`geohash_str`*, *`srid`*)`

    返回包含解码地理哈希值的`POINT`值，给定地理哈希字符串值。

    点的 X 和 Y 坐标分别是经度范围为[−180, 180]和纬度范围为[−90, 90]。

    *`srid`*参数是一个 32 位无符号整数。

    关于从*`geohash_str`*参数中处理的最大字符数的描述在`ST_LatFromGeoHash()`中的备注也适用于`ST_PointFromGeoHash()`。

    `ST_PointFromGeoHash()`处理其参数的方式如本节介绍所述。

    ```sql
    mysql> SET @gh = ST_GeoHash(45,-20,10);
    mysql> SELECT ST_AsText(ST_PointFromGeoHash(@gh,0));
    +---------------------------------------+
    | ST_AsText(ST_PointFromGeoHash(@gh,0)) |
    +---------------------------------------+
    | POINT(45 -20)                         |
    +---------------------------------------+
    ```
