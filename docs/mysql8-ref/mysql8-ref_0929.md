# 15.1.31 删除空间参考系统语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-spatial-reference-system.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-spatial-reference-system.html)

```sql
DROP SPATIAL REFERENCE SYSTEM
    [IF EXISTS]
    *srid*

*srid*: *32-bit unsigned integer*
```

此语句从数据字典中删除一个空间参考系统（SRS）定义。它需要`SUPER`权限。

示例：

```sql
DROP SPATIAL REFERENCE SYSTEM 4120;
```

如果不存在具有 SRID 值的 SRS 定义，则会发生错误，除非指定了 `IF EXISTS`。在这种情况下，会发出警告而不是错误。

如果某个现有表中的某列使用了 SRID 值，则会发生错误。例如：

```sql
mysql> DROP SPATIAL REFERENCE SYSTEM 4326;
ERROR 3716 (SR005): Can't modify SRID 4326\. There is at
least one column depending on it.
```

要确定哪些列使用了 SRID，请使用以下查询：

```sql
SELECT * FROM INFORMATION_SCHEMA.ST_GEOMETRY_COLUMNS WHERE SRS_ID=4326;
```

SRID 值必须在 32 位无符号整数范围内，有以下限制：

+   SRID 0 是一个有效的 SRID，但不能与`删除空间参考系统`一起使用。

+   如果值在保留的 SRID 范围内，会发出警告。保留范围为 [0, 32767]（由 EPSG 保留）、[60,000,000, 69,999,999]（由 EPSG 保留）和 [2,000,000,000, 2,147,483,647]（由 MySQL 保留）。EPSG 代表[欧洲石油调查组](http://epsg.org)。

+   用户不应删除具有保留范围内 SRID 的 SRS。如果删除了系统安装的 SRS，则可能会在 MySQL 升级时重新创建 SRS 定义。
