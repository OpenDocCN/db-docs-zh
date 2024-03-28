> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html)

#### 14.16.9.2 使用最小外接矩形的空间关系函数

MySQL 提供了几个 MySQL 特定的函数，用于测试两个几何图形*`g1`*和*`g2`*的最小外接矩形（MBRs）之间的关系。返回值 1 和 0 分别表示真和假。

点的边界框被解释为既是边界又是内部的点。

直线的边界框被解释为一条线，其中线的内部也是边界。端点是边界点。

如果任何参数是几何集合，则这些参数的内部、边界和外部是集合中所有元素的并集。

本节中的函数检测笛卡尔或地理空间参考系统（SRS）中的参数，并返回适合 SRS 的结果。

除非另有说明，本节中的函数处理其几何参数如下：

+   如果任何参数为`NULL`或为空几何，则返回值为`NULL`。

+   如果任何几何参数不是语法上格式良好的几何图形，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义的空间参考系统（SRS）中语法上格式良好的几何图形，则会发生`ER_SRS_NOT_FOUND`错误。

+   对于接受多个几何参数的函数，如果这些参数不在相同的 SRS 中，则会发生`ER_GIS_DIFFERENT_SRIDS`错误。

+   如果任何参数在几何上无效，则结果为真或假（未定义），或者会发生错误���

+   对于地理空间参考系统（SRS）的几何参数，如果任何参数的经度或纬度超出范围，则会发生错误：

    +   如果经度值不在范围（−180, 180]内，则会发生`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

    +   如果纬度值不在范围[−90, 90]内，则会发生`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

    显示的范围以度为单位。如果一个 SRS 使用另一个单位，范围将使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

+   否则，返回值为非`NULL`。

这些 MBR 函数可用于测试几何关系：

+   `MBRContains(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示 *`g1`* 的最小边界矩形是否包含 *`g2`* 的最小边界矩形。这测试与 `MBRWithin()` 相反的关系。

    `MBRContains()` 处理其参数如本节介绍的那样。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('Polygon((0 0,0 3,3 3,3 0,0 0))');
    mysql> SET @g2 = ST_GeomFromText('Point(1 1)');
    mysql> SELECT MBRContains(@g1,@g2), MBRWithin(@g2,@g1);
    +----------------------+--------------------+
    | MBRContains(@g1,@g2) | MBRWithin(@g2,@g1) |
    +----------------------+--------------------+
    |                    1 |                  1 |
    +----------------------+--------------------+
    ```

+   `MBRCoveredBy(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示 *`g1`* 的最小边界矩形是否被 *`g2`* 的最小边界矩形覆盖。这测试与 `MBRCovers()` 相反的关系。

    `MBRCoveredBy()` 处理其参数如本节介绍的那样。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('Polygon((0 0,0 3,3 3,3 0,0 0))');
    mysql> SET @g2 = ST_GeomFromText('Point(1 1)');
    mysql> SELECT MBRCovers(@g1,@g2), MBRCoveredby(@g1,@g2);
    +--------------------+-----------------------+
    | MBRCovers(@g1,@g2) | MBRCoveredby(@g1,@g2) |
    +--------------------+-----------------------+
    |                  1 |                     0 |
    +--------------------+-----------------------+
    mysql> SELECT MBRCovers(@g2,@g1), MBRCoveredby(@g2,@g1);
    +--------------------+-----------------------+
    | MBRCovers(@g2,@g1) | MBRCoveredby(@g2,@g1) |
    +--------------------+-----------------------+
    |                  0 |                     1 |
    +--------------------+-----------------------+
    ```

+   `MBRCovers(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示 *`g1`* 的最小边界矩形是否覆盖 *`g2`* 的最小边界矩形。这测试与 `MBRCoveredBy()` 相反的关系。参见 `MBRCoveredBy()` 的描述以获取示例。

    `MBRCovers()` 处理其参数如本节介绍的那样。

+   `MBRDisjoint(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示两个几何体 *`g1`* 和 *`g2`* 的最小边界矩形是否不相交。

    `MBRDisjoint()` 处理其参数如本节介绍的那样。

+   `MBREquals(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示两个几何体 *`g1`* 和 *`g2`* 的最小边界矩形是否相同。

    `MBREquals()` 处理其参数如本节介绍的那样，除了对空几何体参数不返回`NULL`。

+   `MBRIntersects(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示两个几何体 *`g1`* 和 *`g2`* 的最小边界矩形是否相交。

    `MBRIntersects()` 处理其参数如本节介绍的那样。

+   `MBROverlaps(*`g1`*, *`g2`*)`

    如果两个几何体相交且它们的交集结果是与给定几何体的维度相同但不等于任何一个给定几何体的几何体，则两个几何体*空间重叠*。

    此函数返回 1 或 0，指示两个几何体 *`g1`* 和 *`g2`* 的最小外接矩形是否重叠。

    `MBROverlaps()` 处理其参数，如本节介绍的那样。

+   `MBRTouches(*`g1`*, *`g2`*)`

    如果两个几何体的内部不相交，但其中一个几何体的边界与另一个的边界或内部相交，则两个几何体*空间接触*。

    此函数返回 1 或 0，指示两个几何体 *`g1`* 和 *`g2`* 的最小外接矩形是否相接。

    `MBRTouches()` 处理其参数，如本节介绍的那样。

+   `MBRWithin(*`g1`*, *`g2`*)`

    返回 1 或 0，指示*`g1`* 的最小外接矩形是否在*`g2`* 的最小外接矩形内。这测试与 `MBRContains()` 相反的关系。

    `MBRWithin()` 处理其参数，如本节介绍的那样。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('Polygon((0 0,0 3,3 3,3 0,0 0))');
    mysql> SET @g2 = ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))');
    mysql> SELECT MBRWithin(@g1,@g2), MBRWithin(@g2,@g1);
    +--------------------+--------------------+
    | MBRWithin(@g1,@g2) | MBRWithin(@g2,@g1) |
    +--------------------+--------------------+
    |                  1 |                  0 |
    +--------------------+--------------------+
    ```
