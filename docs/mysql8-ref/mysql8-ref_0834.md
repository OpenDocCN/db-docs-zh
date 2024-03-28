> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-geometrycollection-property-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-geometrycollection-property-functions.html)

#### 14.16.7.5 几何集合属性函数

这些函数返回 `GeometryCollection` 值的属性。

除非另有说明，本节中的函数将其几何参数处理如下：

+   如果任何参数是 `NULL` 或任何几何参数是空几何体，则返回值为 `NULL`。

+   如果任何几何参数不是语法上良好形成的几何体，则会发生 `ER_GIS_INVALID_DATA` 错误。

+   如果任何几何参数是在未定义的空间参考系统（SRS）中是语法上良好形成的几何体，则会发生 `ER_SRS_NOT_FOUND` 错误。

+   否则，返回值为非 `NULL`。

这些函数可用于获取几何集合属性：

+   `ST_GeometryN(*`gc`*, *`N`*)`

    返回 `GeometryCollection` 值 *`gc`* 中第 *`N`* 个几何体。几何体从 1 开始编号。

    `ST_GeometryN()` 处理其参数，如本节介绍的那样。

    ```sql
    mysql> SET @gc = 'GeometryCollection(Point(1 1),LineString(2 2, 3 3))';
    mysql> SELECT ST_AsText(ST_GeometryN(ST_GeomFromText(@gc),1));
    +-------------------------------------------------+
    | ST_AsText(ST_GeometryN(ST_GeomFromText(@gc),1)) |
    +-------------------------------------------------+
    | POINT(1 1)                                      |
    +-------------------------------------------------+
    ```

+   `ST_NumGeometries(*`gc`*)`

    返回 `GeometryCollection` 值 *`gc`* 中的几何体数量。

    `ST_NumGeometries()` 处理其参数，如本节介绍的那样。

    ```sql
    mysql> SET @gc = 'GeometryCollection(Point(1 1),LineString(2 2, 3 3))';
    mysql> SELECT ST_NumGeometries(ST_GeomFromText(@gc));
    +----------------------------------------+
    | ST_NumGeometries(ST_GeomFromText(@gc)) |
    +----------------------------------------+
    |                                      2 |
    +----------------------------------------+
    ```
