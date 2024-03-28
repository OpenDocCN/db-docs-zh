# 13.4.5 空间参考系统支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-reference-systems.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-reference-systems.html)

空间数据的空间参考系统（SRS）是用于地理位置的基于坐标的系统。

有不同类型的空间参考系统：

+   投影 SRS 是将地球投影到平面表面的投影；也就是说，是一个平面地图。例如，一个位于地球仪内部的灯泡照射到包围地球的纸筒上，将地图投影到纸上。结果是地理参考：每个点映射到地球上的一个位置。该平面上的坐标系统是笛卡尔坐标系，使用长度单位（米、英尺等），而不是经度和纬度的度数。

    在这种情况下，地球是椭球体；也就是说，是扁平的球体。地球在南北轴上比东西轴短一点，所以稍微扁平的球体更正确，但完美的球体可以进行更快的计算。

+   地理 SRS 是表示椭球上经度-纬度（或纬度-经度）坐标的非投影 SRS，使用任何角度单位。

+   MySQL 中由 SRID 0 表示的 SRS 代表一个无限的平面笛卡尔平面，其轴没有分配单位。与投影 SRS 不同，它没有地理参考，也不一定代表地球。它是一个可以用于任何事物的抽象平面。SRID 0 是 MySQL 中空间数据的默认 SRID。

MySQL 在数据字典`mysql.st_spatial_reference_systems`表中维护有关空间数据可用空间参考系统的信息，该表可以存储投影和地理 SRS 的条目。这个数据字典表是不可见的，但 SRS 条目内容可以通过`INFORMATION_SCHEMA` `ST_SPATIAL_REFERENCE_SYSTEMS`表获得，该表作为`mysql.st_spatial_reference_systems`上的视图实现（参见 Section 28.3.36, “INFORMATION_SCHEMA ST_SPATIAL_REFERENCE_SYSTEMS 表”）。

以下示例展示了 SRS 条目的外观：

```sql
mysql> SELECT *
       FROM INFORMATION_SCHEMA.ST_SPATIAL_REFERENCE_SYSTEMS
       WHERE SRS_ID = 4326\G
*************************** 1\. row ***************************
                SRS_NAME: WGS 84
                  SRS_ID: 4326
            ORGANIZATION: EPSG
ORGANIZATION_COORDSYS_ID: 4326
              DEFINITION: GEOGCS["WGS 84",DATUM["World Geodetic System 1984",
                          SPHEROID["WGS 84",6378137,298.257223563,
                          AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],
                          PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],
                          UNIT["degree",0.017453292519943278,
                          AUTHORITY["EPSG","9122"]],
                          AXIS["Lat",NORTH],AXIS["Long",EAST],
                          AUTHORITY["EPSG","4326"]]
             DESCRIPTION:
```

本条目描述了用于 GPS 系统的 SRS。它的名称（`SRS_NAME`）是 WGS 84，ID（`SRS_ID`）是 4326，这是[欧洲石油勘探集团](http://epsg.org)（EPSG）使用的 ID。

`DEFINITION`列中的 SRS 定义是 WKT 值，表示为[开放地理空间联盟](http://www.opengeospatial.org)文档[OGC 12-063r5](http://docs.opengeospatial.org/is/12-063r5/12-063r5.html)中指定的形式。

`SRS_ID`值代表与几何值的 SRID 相同类型的值，或作为空间函数的 SRID 参数传递。SRID 0（无单位的笛卡尔平面）是特殊的。它始终是合法的空间参考系统 ID，并且可以在依赖于 SRID 值的空间数据的任何计算中使用。

对于多个几何值的计算，所有值必须具有相同的 SRID，否则会出错。

当 GIS 函数需要定义时，SRS 定义会按需解析。解析后的定义存储在数据字典缓存中，以便重复使用，并避免为每个需要 SRS 信息的语句产生解析开销。

为了使数据字典中存储的 SRS 条目可以进行操作，MySQL 提供了以下 SQL 语句：

+   `CREATE SPATIAL REFERENCE SYSTEM`: 查看第 15.1.19 节，“CREATE SPATIAL REFERENCE SYSTEM Statement”。该语句的描述包括有关 SRS 组件的附加信息。

+   `DROP SPATIAL REFERENCE SYSTEM`: 查看第 15.1.31 节，“DROP SPATIAL REFERENCE SYSTEM Statement”。
