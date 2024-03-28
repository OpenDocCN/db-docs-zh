# 14.16.11 空间 GeoJSON 函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-geojson-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-geojson-functions.html)

本节描述了在 GeoJSON 文档和空间值之间转换的函数。GeoJSON 是一种用于编码几何/地理要素的开放标准。有关更多信息，请参见[`geojson.org`](http://geojson.org)。这里讨论的函数遵循 GeoJSON 规范修订版 1.0。

GeoJSON 支持与 MySQL 支持的相同的几何/地理数据类型。除了从中提取几何对象外，不支持 Feature 和 FeatureCollection 对象。CRS 支持仅限于标识 SRID 的值。

MySQL 还支持本机`JSON`数据类型和一组 SQL 函数，以便对 JSON 值进行操作。有关更多信息，请参见第 13.5 节，“JSON 数据类型”和第 14.17 节，“JSON 函数”。

+   [`ST_AsGeoJSON(*`g`* [, *`max_dec_digits`* [, *`options`*]])`](spatial-geojson-functions.html#function_st-asgeojson)

    从几何体*`g`*生成一个 GeoJSON 对象。对象字符串具有连接字符集和排序规则。

    如果任何参数为`NULL`，则返回值为`NULL`。如果任何非`NULL`参数无效，则会发生错误。

    *`max_dec_digits`*，如果指定，限制了坐标的小数位数并导致输出四舍五入。如果未指定，则此参数默认为其最大值 2³² − 1。最小值为 0。

    *`options`*，如果指定，是一个位掩码。下表显示了允许的标志值。如果几何参数的 SRID 为 0，则即使请求了 CRS 对象的标志值也不会产生 CRS 对象。

    | 标志值 | 含义 |
    | --- | --- |
    | 0 | 无选项。如果未指定*`options`*，则为默认值。 |
    | 1 | 将边界框添加到输出中。 |
    | 2 | 将短格式 CRS URN 添加到输出中。默认格式是短格式（`EPSG:*`srid`*`）。 |
    | 4 | 添加长格式 CRS URN（`urn:ogc:def:crs:EPSG:*`srid`*`）。此标志覆盖标志 2。例如，选项值为 5 和 7 表示相同的含义（添加边界框和长格式 CRS URN）。 |

    ```sql
    mysql> SELECT ST_AsGeoJSON(ST_GeomFromText('POINT(11.11111 12.22222)'),2);
    +-------------------------------------------------------------+
    | ST_AsGeoJSON(ST_GeomFromText('POINT(11.11111 12.22222)'),2) |
    +-------------------------------------------------------------+
    | {"type": "Point", "coordinates": [11.11, 12.22]}            |
    +-------------------------------------------------------------+
    ```

+   [`ST_GeomFromGeoJSON(*`str`* [, *`options`* [, *`srid`*]])`](spatial-geojson-functions.html#function_st-geomfromgeojson)

    解析表示 GeoJSON 对象的字符串*`str`*并返回几何体。

    如果任何参数为`NULL`，则返回值为`NULL`。如果任何非`NULL`参数无效，则会发生错误。

    *`options`*，如果给定，描述如何处理包含高于 2 维坐标的几何体的 GeoJSON 文档。下表显示了允许的*`options`*值。

    | 选项值 | 含义 |
    | --- | --- |
    | 1 | 拒绝文档并生成错误。如果未指定*`options`*，则为默认值。 |
    | 2, 3, 4 | 接受文档并去除高维坐标的坐标。 |

    *`options`* 的值为 2、3 和 4 目前产生相同的效果。如果将来支持坐标维度高于 2 的几何体，可以预期这些值会产生不同的效果。

    如果给定 *`srid`* 参数，必须是一个 32 位无符号整数。如果未给定，几何返回值的 SRID 为 4326。

    如果 *`srid`* 指的是未定义的空间参考系统 (SRS)，将会出现 `ER_SRS_NOT_FOUND` 错误。

    对于地理 SRS 几何参数，如果任何参数的经度或纬度超出范围，将会出现错误：

    +   如果经度值不在范围 (−180, 180] 内，将会出现 `ER_LONGITUDE_OUT_OF_RANGE` 错误。

    +   如果纬度值不在范围 [−90, 90] 内，将会出现 `ER_LATITUDE_OUT_OF_RANGE` 错误。

    显示的范围以度为单位。如果一个 SRS 使用其他单位，范围将使用其单位的相应值。由于浮点运算，确切的范围限制略有偏差。

    GeoJSON 几何体、要素和要素集合对象可能具有一个 `crs` 属性。解析函数解析 `urn:ogc:def:crs:EPSG::*`srid`*` 和 `EPSG:*`srid`*` 命名的 CRS URNs，但不解析作为链接对象给出的 CRS。此外，`urn:ogc:def:crs:OGC:1.3:CRS84` 被识别为 SRID 4326。如果对象具有无法理解的 CRS，将会出现错误，但如果给定了可选的 *`srid`* 参数，则会忽略任何 CRS，即使它是无效的。

    如果在 GeoJSON 文档的较低级别找到一个指定与顶级对象 SRID 不同的 SRID 的 `crs` 成员，将会出现 `ER_INVALID_GEOJSON_CRS_NOT_TOP_LEVEL` 错误。

    如 GeoJSON 规范所述，对于 GeoJSON 输入的 `type` 成员（`Point`、`LineString` 等），解析是区分大小写的。规范对于其他解析的大小写敏感性保持沉默，在 MySQL 中不区分大小写。

    这个例子展示了一个简单 GeoJSON 对象的解析结果。请注意，坐标的顺序取决于所使用的 SRID。

    ```sql
    mysql> SET @json = '{ "type": "Point", "coordinates": [102.0, 0.0]}';
    mysql> SELECT ST_AsText(ST_GeomFromGeoJSON(@json));
    +--------------------------------------+
    | ST_AsText(ST_GeomFromGeoJSON(@json)) |
    +--------------------------------------+
    | POINT(0 102)                         |
    +--------------------------------------+
    mysql> SELECT ST_SRID(ST_GeomFromGeoJSON(@json));
    +------------------------------------+
    | ST_SRID(ST_GeomFromGeoJSON(@json)) |
    +------------------------------------+
    |                               4326 |
    +------------------------------------+
    mysql> SELECT ST_AsText(ST_SRID(ST_GeomFromGeoJSON(@json),0));
    +-------------------------------------------------+
    | ST_AsText(ST_SRID(ST_GeomFromGeoJSON(@json),0)) |
    +-------------------------------------------------+
    | POINT(102 0)                                    |
    +-------------------------------------------------+
    ```
