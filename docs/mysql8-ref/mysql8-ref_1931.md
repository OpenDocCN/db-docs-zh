# 28.3.36 INFORMATION_SCHEMA ST_SPATIAL_REFERENCE_SYSTEMS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-st-spatial-reference-systems-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-st-spatial-reference-systems-table.html)

`ST_SPATIAL_REFERENCE_SYSTEMS` 表提供有关可用空间数据的空间参考系统（SRS）的信息。此表基于 SQL/MM（ISO/IEC 13249-3）标准。

`ST_SPATIAL_REFERENCE_SYSTEMS` 表中的条目基于 [European Petroleum Survey Group](http://epsg.org)（EPSG）数据集，除了对应于 MySQL 中使用的特殊 SRS 的 SRID 0，该 SRS 表示一个无限的平坦笛卡尔平面，其轴没有分配单位。有关 SRS 的其他信息，请参见 Section 13.4.5, “Spatial Reference System Support”。

`ST_SPATIAL_REFERENCE_SYSTEMS` 表具有以下列：

+   `SRS_NAME`

    空间参考系统名称。此值是唯一的。

+   `SRS_ID`

    空间参考系统数值 ID。此值是唯一的。

    `SRS_ID` 值代表与几何值的 SRID 相同类型的值，或作为空间函数的 SRID 参数传递。SRID 0（无单位的笛卡尔平面）是特殊的。它始终是合法的空间参考系统 ID，并可用于依赖于 SRID 值的空间数据的任何计算中。

+   `ORGANIZATION`

    定义了空间参考系统基础坐标系的组织名称。

+   `ORGANIZATION_COORDSYS_ID`

    组织定义的空间参考系统的数值 ID。

+   `DEFINITION`

    空间参考系统定义。`DEFINITION` 值是 WKT 值，表示如 [Open Geospatial Consortium](http://www.opengeospatial.org) 文档 [OGC 12-063r5](http://docs.opengeospatial.org/is/12-063r5/12-063r5.html) 中指定的。

    当 GIS 函数需要定义时，会按需解析 SRS 定义。解析的定义存储在数据字典缓存中，以便重用并避免为每个需要 SRS 信息的语句产生解析开销。

+   `DESCRIPTION`

    空间参考系统描述。

#### 注意

+   `SRS_NAME`、`ORGANIZATION`、`ORGANIZATION_COORDSYS_ID` 和 `DESCRIPTION` 列包含可能对用户感兴趣的信息，但它们不被 MySQL 使用。

#### 示例

```sql
mysql> SELECT * FROM ST_SPATIAL_REFERENCE_SYSTEMS
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

本条目描述了用于 GPS 系统的 SRS。它具有名称（`SRS_NAME`）为 WGS 84 和 ID（`SRS_ID`）为 4326，这是[欧洲石油调查组](http://epsg.org)（EPSG）使用的 ID。

投影和地理 SRS 的`DEFINITION`值分别以`PROJCS`和`GEOGCS`开头。SRID 0 的定义是特殊的，并且具有空的`DEFINITION`值。以下查询根据`DEFINITION`值确定`ST_SPATIAL_REFERENCE_SYSTEMS`表中有多少条目对应于投影、地理和其他 SRS：

```sql
mysql> SELECT
         COUNT(*),
         CASE LEFT(DEFINITION, 6)
           WHEN 'PROJCS' THEN 'Projected'
           WHEN 'GEOGCS' THEN 'Geographic'
           ELSE 'Other'
         END AS SRS_TYPE
       FROM INFORMATION_SCHEMA.ST_SPATIAL_REFERENCE_SYSTEMS
       GROUP BY SRS_TYPE;
+----------+------------+
| COUNT(*) | SRS_TYPE   |
+----------+------------+
|        1 | Other      |
|     4668 | Projected  |
|      483 | Geographic |
+----------+------------+
```

为了使存储在数据字典中的 SRS 条目可以进行操作，MySQL 提供了以下 SQL 语句：

+   `创建空间参考系统`：参见第 15.1.19 节，“创建空间参考系统语句”。该语句的描述包括有关 SRS 组件的附加信息。

+   `删除空间参考系统`：参见第 15.1.31 节，“删除空间参考系统语句”。
