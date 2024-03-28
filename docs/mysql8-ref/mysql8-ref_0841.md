# 14.16.12 空间聚合函数

> [`dev.mysql.com/doc/refman/8.0/en/spatial-aggregate-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-aggregate-functions.html)

MySQL 支持对一组值执行计算的聚合函数。有关这些函数的一般信息，请参见第 14.19.1 节，“聚合函数描述”。本节描述了`ST_Collect()`空间聚合函数。

`ST_Collect()`可以作为窗口函数使用，如其语法描述中的`[*`over_clause`*]`所示，表示可选的`OVER`子句。*`over_clause`*在第 14.20.2 节，“窗口函数概念和语法”中描述，该节还包括有关窗口函数使用的其他信息。

+   [`ST_Collect([DISTINCT] *`g`*) [*`over_clause`*]`](spatial-aggregate-functions.html#function_st-collect)

    聚合几何值并返回单个几何集合值。使用`DISTINCT`选项，返回不同几何参数的聚合。

    与其他聚合函数一样，可以使用`GROUP BY`将参数分组为子集。`ST_Collect()`为每个子集返回一个聚合值。

    如果*`over_clause`*存在，则此函数将作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中描述的那样。与大多数支持窗口化的聚合函数不同，`ST_Collect()`允许将*`over_clause`*与`DISTINCT`一起使用。

    `ST_Collect()`处理其参数如下：

    +   `NULL`参数将被忽略。

    +   如果所有参数都为`NULL`或聚合结果为空，则返回值为`NULL`。

    +   如果任何几何参数不是语法上良好形式的几何图形，则会发生`ER_GIS_INVALID_DATA`错误。

    +   如果任何几何参数是在未定义的空间参考系统（SRS）中为语法上良好形式的几何图形，则会发生`ER_SRS_NOT_FOUND`错误。

    +   如果存在多个几何参数且这些参数在相同的 SRS 中，则返回值也将在该 SRS 中。如果这些参数不在相同的 SRS 中，则会发生`ER_GIS_DIFFERENT_SRIDS_AGGREGATION`错误。

    +   结果是可能的最窄的`Multi*`Xxx`*`或`GeometryCollection`值，结果类型根据非`NULL`几何参数确定如下：

        +   如果所有参数都是`Point`值，则结果是一个`MultiPoint`值。

        +   如果所有参数都是`LineString`值，则结果是一个`MultiLineString`值。

        +   如果所有参数都是`Polygon`值，则结果是一个`MultiPolygon`值。

        +   否则，参数是几何类型的混合，结果是一个`GeometryCollection`值。

    这个示例数据集展示了假设产品按年份和制造地点的情况：

    ```sql
    CREATE TABLE product (
      year INTEGER,
      product VARCHAR(256),
      location Geometry
    );

    INSERT INTO product
    (year,  product,     location) VALUES
    (2000, "Calculator", ST_GeomFromText('point(60 -24)',4326)),
    (2000, "Computer"  , ST_GeomFromText('point(28 -77)',4326)),
    (2000, "Abacus"    , ST_GeomFromText('point(28 -77)',4326)),
    (2000, "TV"        , ST_GeomFromText('point(38  60)',4326)),
    (2001, "Calculator", ST_GeomFromText('point(60 -24)',4326)),
    (2001, "Computer"  , ST_GeomFromText('point(28 -77)',4326));
    ```

    一些使用`ST_Collect()`在数据集上的示例查询：

    ```sql
    mysql> SELECT ST_AsText(ST_Collect(location)) AS result
           FROM product;
    +------------------------------------------------------------------+
    | result                                                           |
    +------------------------------------------------------------------+
    | MULTIPOINT((60 -24),(28 -77),(28 -77),(38 60),(60 -24),(28 -77)) |
    +------------------------------------------------------------------+

    mysql> SELECT ST_AsText(ST_Collect(DISTINCT location)) AS result
           FROM product;
    +---------------------------------------+
    | result                                |
    +---------------------------------------+
    | MULTIPOINT((60 -24),(28 -77),(38 60)) |
    +---------------------------------------+

    mysql> SELECT year, ST_AsText(ST_Collect(location)) AS result
           FROM product GROUP BY year;
    +------+------------------------------------------------+
    | year | result                                         |
    +------+------------------------------------------------+
    | 2000 | MULTIPOINT((60 -24),(28 -77),(28 -77),(38 60)) |
    | 2001 | MULTIPOINT((60 -24),(28 -77))                  |
    +------+------------------------------------------------+

    mysql> SELECT year, ST_AsText(ST_Collect(DISTINCT location)) AS result
           FROM product GROUP BY year;
    +------+---------------------------------------+
    | year | result                                |
    +------+---------------------------------------+
    | 2000 | MULTIPOINT((60 -24),(28 -77),(38 60)) |
    | 2001 | MULTIPOINT((60 -24),(28 -77))         |
    +------+---------------------------------------+

    # selects nothing
    mysql> SELECT ST_Collect(location) AS result
           FROM product WHERE year = 1999;
    +--------+
    | result |
    +--------+
    | NULL   |
    +--------+

    mysql> SELECT ST_AsText(ST_Collect(location)
             OVER (ORDER BY year, product ROWS BETWEEN 1 PRECEDING AND CURRENT ROW))
             AS result
           FROM product;
    +-------------------------------+
    | result                        |
    +-------------------------------+
    | MULTIPOINT((28 -77))          |
    | MULTIPOINT((28 -77),(60 -24)) |
    | MULTIPOINT((60 -24),(28 -77)) |
    | MULTIPOINT((28 -77),(38 60))  |
    | MULTIPOINT((38 60),(60 -24))  |
    | MULTIPOINT((60 -24),(28 -77)) |
    +-------------------------------+
    ```

    此函数在 MySQL 8.0.24 中添加。
