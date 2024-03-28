> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html)

#### 14.16.9.1 使用对象形状的空间关系函数

OpenGIS 规范定义了以下函数，用于测试两个几何值 *`g1`* 和 *`g2`* 之间的关系，使用精确的对象形状。返回值 1 和 0 分别表示真和假，除了距离函数返回距离值。

此部分中的函数检测笛卡尔或地理空间参考系统（SRS）中的参数，并返回适合该 SRS 的结果。

除非另有说明，此部分中的函数处理其几何参数如下：

+   如果任何参数为`NULL`或任何几何参数为空几何体，则返回值为`NULL`。

+   如果任何几何参数不是语法上良好形成的几何体，则会发生`ER_GIS_INVALID_DATA`错误。

+   如果任何几何参数是在未定义空间参考系统（SRS）中的语法上良好形成的几何体，则会发生`ER_SRS_NOT_FOUND`错误。

+   对于接受多个几何参数的函数，如果这些参数不在同一个 SRS 中，则会发生`ER_GIS_DIFFERENT_SRIDS`错误。

+   如果任何几何参数几何上无效，则结果为真或假（未定义哪个），或者会发生错误。

+   对于地理 SRS 几何参数，如果任何参数的经度或��度超出范围，则会发生错误：

    +   如果经度值不在范围（−180, 180]内，则会发生`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LONGITUDE_OUT_OF_RANGE`）。

    +   如果纬度值不在范围 [−90, 90] 内，则会发生`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误（在 MySQL 8.0.12 之前为`ER_LATITUDE_OUT_OF_RANGE`）。

    显示的范围以度为单位。如果一个 SRS 使用另一个单位，则范围使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

+   否则，返回值为非`NULL`。

此部分中的一些函数允许一个单位参数，指定返回值的长度单位。除非另有说明，函数处理其单位参数如下：

+   如果在`INFORMATION_SCHEMA` `ST_UNITS_OF_MEASURE`表中找到单位，则支持该单位。请参见 Section 28.3.37, “The INFORMATION_SCHEMA ST_UNITS_OF_MEASURE Table”。

+   如果指定了单位但 MySQL 不支持，将出现`ER_UNIT_NOT_FOUND`错误。

+   如果指定了支持的线性单位且 SRID 为 0，则会出现`ER_GEOMETRY_IN_UNKNOWN_LENGTH_UNIT`错误。

+   如果指定了支持的线性单位且 SRID 不为 0，则结果以该单位表示。

+   如果未指定单位，则结果以几何图形的 SRS 单位表示，无论是笛卡尔还是地理。目前，所有 MySQL SRS 都以米表示。

这些对象形状函数可用于测试几何关系：

+   `ST_Contains(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示*`g1`*是否完全包含*`g2`*。这测试与`ST_Within()`相反的关系。

    `ST_Contains()` 处理其参数如本节介绍的那样。

+   `ST_Crosses(*`g1`*, *`g2`*)`

    如果它们的空间关系具有以下属性，则两个几何体*空间交叉*。

    +   除非*`g1`*和*`g2`*都是 1 维的：*`g1`*与*`g2`*相交，如果*`g2`*的内部与*`g1`*的内部有共同点，但*`g2`*不覆盖*`g1`*的整个内部。

    +   如果*`g1`*和*`g2`*都是 1 维的：如果线在有限数量的点上相交（即没有共同的线段，只有共同的单个点）。

    此函数返回 1 或 0 以指示*`g1`*是否在空间上与*`g2`*相交。

    `ST_Crosses()` 处理其参数如本节介绍的那样，除了对于这些额外条件，返回值为`NULL`。

    +   *`g1`*是 2 维的（`Polygon`或`MultiPolygon`）。

    +   *`g2`*是 1 维的（`Point`或`MultiPoint`）。

+   `ST_Disjoint(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示*`g1`*是否在空间上与*`g2`*不相交（不相交）。

    `ST_Disjoint()` 处理其参数如本节介绍的那样。

+   [`ST_Distance(*`g1`*, *`g2`* [, *`unit`*])`](spatial-relation-functions-object-shapes.html#function_st-distance)

    返回*`g1`*和*`g2`*之间的距离，以几何参数的空间参考系统（SRS）的长度单位或可选的*`unit`*参数指定的单位进行测量。

    该函数通过返回两个几何参数的所有组件之间的最短距离来处理几何集合。

    `ST_Distance()` 处理其几何参数的方式如本节介绍所述，但有以下例外：

    +   `ST_Distance()` 检测地理（椭球体）空间参考系统中的参数，并返回椭球体上的大地距离。从 MySQL 8.0.18 开始，`ST_Distance()` 支持所有几何类型的地理 SRS 参数的距离计算。在 MySQL 8.0.18 之前，唯一允许的地理参数类型是`Point`和`Point`，或`Point`和`MultiPoint`（任意参数顺序）。如果在地理 SRS 中使用其他几何类型参数组合调用，将会出现`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误。

    +   如果任何参数在几何上无效，则结果要么是未定义的距离（即可以是任何数字），要么会出现错误。

    +   如果中间或最终结果产生`NaN`或负数，则会出现`ER_GIS_INVALID_DATA`错误。

    从 MySQL 8.0.14 开始，`ST_Distance()` 允许一个可选的*`unit`*参数，用于指定返回距离值的线性单位。`ST_Distance()` 处理其*`unit`*参数的方式如本节介绍所述。

    ```sql
    mysql> SET @g1 = ST_GeomFromText('POINT(1 1)');
    mysql> SET @g2 = ST_GeomFromText('POINT(2 2)');
    mysql> SELECT ST_Distance(@g1, @g2);
    +-----------------------+
    | ST_Distance(@g1, @g2) |
    +-----------------------+
    |    1.4142135623730951 |
    +-----------------------+

    mysql> SET @g1 = ST_GeomFromText('POINT(1 1)', 4326);
    mysql> SET @g2 = ST_GeomFromText('POINT(2 2)', 4326);
    mysql> SELECT ST_Distance(@g1, @g2);
    +-----------------------+
    | ST_Distance(@g1, @g2) |
    +-----------------------+
    |     156874.3859490455 |
    +-----------------------+
    mysql> SELECT ST_Distance(@g1, @g2, 'metre');
    +--------------------------------+
    | ST_Distance(@g1, @g2, 'metre') |
    +--------------------------------+
    |              156874.3859490455 |
    +--------------------------------+
    mysql> SELECT ST_Distance(@g1, @g2, 'foot');
    +-------------------------------+
    | ST_Distance(@g1, @g2, 'foot') |
    +-------------------------------+
    |             514679.7439273146 |
    +-------------------------------+
    ```

    对于在球体上进行距离计算的特殊情况，请参阅`ST_Distance_Sphere()`函数。

+   `ST_Equals(*`g1`*, *`g2`*)`

    返回 1 或 0 以指示*`g1`*是否空间上等于*`g2`*。

    `ST_Equals()` 处理其参数的方式如本节介绍所述，但对于空几何参数不返回`NULL`。

    ```sql
    mysql> SET @g1 = Point(1,1), @g2 = Point(2,2);
    mysql> SELECT ST_Equals(@g1, @g1), ST_Equals(@g1, @g2);
    +---------------------+---------------------+
    | ST_Equals(@g1, @g1) | ST_Equals(@g1, @g2) |
    +---------------------+---------------------+
    |                   1 |                   0 |
    +---------------------+---------------------+
    ```

+   [`ST_FrechetDistance(*`g1`*, *`g2`* [, *`unit`*])`](spatial-relation-functions-object-shapes.html#function_st-frechetdistance)

    返回两个几何图形之间的离散弗雷歇距离，反映了几何图形的相似程度。结果是一个双精度数字，以几何参数的空间参考系统（SRS）的长度单位或*`unit`*参数的长度单位来衡量，如果给定了该参数。

    此函数实现了离散弗雷歇距离，这意味着它仅限于几何图形上的点之间的距离。例如，给定两个`LineString`参数，只考虑几何图形中明确提到的点。这些点之间的线段上的点不被考虑。

    `ST_FrechetDistance()` 处理其几何参数，如本节介绍中所述，但有以下例外：

    +   几何图形可以具有笛卡尔或地理 SRS，但仅支持`LineString`值。如果参数在相同的笛卡尔或地理 SRS 中，但任一参数不是`LineString`，则会出现`ER_NOT_IMPLEMENTED_FOR_CARTESIAN_SRS`或`ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS`错误，取决于 SRS 类型。

    `ST_FrechetDistance()` 处理其可选的*`unit`*参数，如本节介绍中所述。

    ```sql
    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0,0 5,5 5)');
    mysql> SET @ls2 = ST_GeomFromText('LINESTRING(0 1,0 6,3 3,5 6)');
    mysql> SELECT ST_FrechetDistance(@ls1, @ls2);
    +--------------------------------+
    | ST_FrechetDistance(@ls1, @ls2) |
    +--------------------------------+
    |             2.8284271247461903 |
    +--------------------------------+

    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0,0 5,5 5)', 4326);
    mysql> SET @ls2 = ST_GeomFromText('LINESTRING(0 1,0 6,3 3,5 6)', 4326);
    mysql> SELECT ST_FrechetDistance(@ls1, @ls2);
    +--------------------------------+
    | ST_FrechetDistance(@ls1, @ls2) |
    +--------------------------------+
    |              313421.1999416798 |
    +--------------------------------+
    mysql> SELECT ST_FrechetDistance(@ls1, @ls2, 'foot');
    +----------------------------------------+
    | ST_FrechetDistance(@ls1, @ls2, 'foot') |
    +----------------------------------------+
    |                     1028284.7767115477 |
    +----------------------------------------+
    ```

    此函数在 MySQL 8.0.23 中添加。

+   [`ST_HausdorffDistance(*`g1`*, *`g2`* [, *`unit`*])`](spatial-relation-functions-object-shapes.html#function_st-hausdorffdistance)

    返回两个几何图形之间的离散豪斯多夫距离，反映了几何图形的相似程度。结果是一个双精度数字，以几何参数的空间参考系统（SRS）的长度单位或*`unit`*参数的长度单位来衡量，如果给定了该参数。

    此函数实现了离散豪斯多夫距离，这意味着它仅限于几何图形上的点之间的距离。例如，给定两个`LineString`参数，只考虑几何图形中明确提到的点。这些点之间的线段上的点不被考虑。

    `ST_HausdorffDistance()` 处理其几何参数，如本节介绍中所述，但有以下例外：

    +   如果几何体参数在相同的笛卡尔或地理 SRS 中，但不是支持的组合，则会出现 `ER_NOT_IMPLEMENTED_FOR_CARTESIAN_SRS` 或 `ER_NOT_IMPLEMENTED_FOR_GEOGRAPHIC_SRS` 错误，取决于 SRS 类型。支持以下组合：

        +   `LineString` 和 `LineString`

        +   `Point` 和 `MultiPoint`

        +   `LineString` 和 `MultiLineString`

        +   `MultiPoint` 和 `MultiPoint`

        +   `MultiLineString` 和 `MultiLineString`

    `ST_HausdorffDistance()` 处理其可选的 *`unit`* 参数如本节介绍所述。

    ```sql
    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0,0 5,5 5)');
    mysql> SET @ls2 = ST_GeomFromText('LINESTRING(0 1,0 6,3 3,5 6)');
    mysql> SELECT ST_HausdorffDistance(@ls1, @ls2);
    +----------------------------------+
    | ST_HausdorffDistance(@ls1, @ls2) |
    +----------------------------------+
    |                                1 |
    +----------------------------------+

    mysql> SET @ls1 = ST_GeomFromText('LINESTRING(0 0,0 5,5 5)', 4326);
    mysql> SET @ls2 = ST_GeomFromText('LINESTRING(0 1,0 6,3 3,5 6)', 4326);
    mysql> SELECT ST_HausdorffDistance(@ls1, @ls2);
    +----------------------------------+
    | ST_HausdorffDistance(@ls1, @ls2) |
    +----------------------------------+
    |               111319.49079326246 |
    +----------------------------------+
    mysql> SELECT ST_HausdorffDistance(@ls1, @ls2, 'foot');
    +------------------------------------------+
    | ST_HausdorffDistance(@ls1, @ls2, 'foot') |
    +------------------------------------------+
    |                        365221.4264870815 |
    +------------------------------------------+
    ```

    此函数在 MySQL 8.0.23 中添加。

+   `ST_Intersects(*`g1`*, *`g2`*)`

    返回 1 或 0 表示 *`g1`* 在空间上是否与 *`g2`* 相交。

    `ST_Intersects()` 处理其参数的方式如本节介绍所述。

+   `ST_Overlaps(*`g1`*, *`g2`*)`

    如果两个几何体相交并且它们的交集结果是相同维度但不等于给定几何体之一，则它们在空间上重叠。

    此函数返回 1 或 0 表示 *`g1`* 在空间上是否与 *`g2`* 重叠。

    `ST_Overlaps()` 处理其参数的方式如本节介绍所述，除了两个几何体的维度不相等的额外条件，返回值为 `NULL`。

+   `ST_Touches(*`g1`*, *`g2`*)`

    如果两个几何体的内部不相交，但其中一个几何体的边界与另一个的边界或内部相交，则它们在空间上*接触*。

    此函数返回 1 或 0 表示 *`g1`* 在空间上是否与 *`g2`* 接触。

    `ST_Touches()` 处理其参数的方式如本节介绍所述，除了对于两个几何体都是 0 维（`Point` 或 `MultiPoint`）的额外条件，返回值为 `NULL`。

+   `ST_Within(*`g1`*, *`g2`*)`

    返回 1 或 0 表示 *`g1`* 是否在空间上在 *`g2`* 内部。这测试与 `ST_Contains()` 相反的关系。

    `ST_Within()` 处理其参数的方式如本节介绍所述。
