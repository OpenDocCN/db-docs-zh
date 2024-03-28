# 15.1.19 CREATE SPATIAL REFERENCE SYSTEM 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-spatial-reference-system.html`](https://dev.mysql.com/doc/refman/8.0/en/create-spatial-reference-system.html)

```sql
CREATE OR REPLACE SPATIAL REFERENCE SYSTEM
    *srid* *srs_attribute* ...

CREATE SPATIAL REFERENCE SYSTEM
    [IF NOT EXISTS]
    *srid* *srs_attribute* ...

*srs_attribute*: {
    NAME '*srs_name*'
  | DEFINITION '*definition*'
  | ORGANIZATION '*org_name*' IDENTIFIED BY *org_id*
  | DESCRIPTION '*description*'
}

*srid*, *org_id*: *32-bit unsigned integer*
```

此语句创建一个空间参考系统（SRS）定义，并将其存储在数据字典中。它需要`SUPER`权限。生成的数据字典条目可以使用`INFORMATION_SCHEMA` `ST_SPATIAL_REFERENCE_SYSTEMS`表进行检查。

SRID 值必须是唯一的，因此如果未指定`OR REPLACE`或`IF NOT EXISTS`，则如果具有给定*`srid`*值的 SRS 定义已经存在，则会发生错误。

使用`CREATE OR REPLACE`语法，任何具有相同 SRID 值的现有 SRS 定义都将被替换，除非该 SRID 值被现有表中的某列使用。在这种情况下，将会发生错误。例如：

```sql
mysql> CREATE OR REPLACE SPATIAL REFERENCE SYSTEM 4326 ...;
ERROR 3716 (SR005): Can't modify SRID 4326\. There is at
least one column depending on it.
```

要确定哪些列使用了 SRID，请使用以下查询，将 4326 替换为您要创建的定义的 SRID：

```sql
SELECT * FROM INFORMATION_SCHEMA.ST_GEOMETRY_COLUMNS WHERE SRS_ID=4326;
```

使用`CREATE ... IF NOT EXISTS`语法，任何具有相同 SRID 值的现有 SRS 定义都会导致新定义被忽略，并发出警告。

SRID 值必须在 32 位无符号整数范围内，具有以下限制：

+   SRID 0 是一个有效的 SRID，但不能与`CREATE SPATIAL REFERENCE SYSTEM`一起使用。

+   如果值在保留的 SRID 范围内，将会发出警告。保留范围为[0, 32767]（由 EPSG 保留），[60,000,000, 69,999,999]（由 EPSG 保留）和[2,000,000,000, 2,147,483,647]（由 MySQL 保留）。EPSG 代表[欧洲石油测量组](http://epsg.org)。

+   用户不应该使用保留范围内的 SRID 创建 SRS。这样做会导致 SRID 与 MySQL 分发的未来 SRS 定义发生冲突，结果是新的系统提供的 SRS 未安装用于 MySQL 升级，或者用户定义的 SRS 被覆盖。

语句的属性必须满足以下条件：

+   属性可以以任何顺序给出，但不能重复给出任何属性。

+   `NAME`和`DEFINITION`属性是必需的。

+   `NAME` *`srs_name`*属性值必须是唯一的。`ORGANIZATION` *`org_name`*和*`org_id`*属性值的组合必须是唯一的。

+   `NAME` *`srs_name`*属性值和`ORGANIZATION` *`org_name`*属性值不能为空，也不能以空格开始或结束。

+   属性规范中的字符串值不能包含控制字符，包括换行符。

+   以下表显示了字符串属性值的最大长度。

    **表 15.6 CREATE SPATIAL REFERENCE SYSTEM 属性长度**

    | 属性 | 最大长度（字符） |
    | --- | --- |
    | `NAME` | 80 |
    | `DEFINITION` | 4096 |
    | `ORGANIZATION` | 256 |
    | `DESCRIPTION` | 2048 |

这里是一个 `CREATE SPATIAL REFERENCE SYSTEM` 语句的示例。`DEFINITION` 值被重新格式化为多行以提高可读性。（为了语句合法，实际上值必须在单行上给出。）

```sql
CREATE SPATIAL REFERENCE SYSTEM 4120
NAME 'Greek'
ORGANIZATION 'EPSG' IDENTIFIED BY 4120
DEFINITION
  'GEOGCS["Greek",DATUM["Greek",SPHEROID["Bessel 1841",
  6377397.155,299.1528128,AUTHORITY["EPSG","7004"]],
  AUTHORITY["EPSG","6120"]],PRIMEM["Greenwich",0,
  AUTHORITY["EPSG","8901"]],UNIT["degree",0.017453292519943278,
  AUTHORITY["EPSG","9122"]],AXIS["Lat",NORTH],AXIS["Lon",EAST],
  AUTHORITY["EPSG","4120"]]';
```

SRS 定义的语法基于 *OpenGIS Implementation Specification: Coordinate Transformation Services*，修订版 1.00，OGC 01-009，2001 年 1 月 12 日，第 7.2 节中定义的语法。该规范可在 [`www.opengeospatial.org/standards/ct`](http://www.opengeospatial.org/standards/ct) 上找到。

MySQL 将这些更改纳入规范中：

+   只有`<horz cs>`产生规则被实现（即地理和投影 SRSs）。

+   `<authority>` 子句对于 `<parameter>` 是可选的，这样可以通过权威而不是名称来识别投影参数。

+   规范中并未要求在`GEOGCS`空间参考系统定义中使`AXIS`子句成为必需的。然而，如果没有`AXIS`子句，MySQL 无法确定一个定义中的坐标轴是按纬度-经度顺序还是经度-纬度顺序。MySQL 强制要求每个`GEOGCS`定义必须包括两个`AXIS`子句。一个必须是`NORTH`或`SOUTH`，另一个是`EAST`或`WEST`。`AXIS`子句的顺序决定了定义中的坐标轴是按纬度-经度顺序还是经度-纬度顺序。

+   SRS 定义不得包含换行符。

如果 SRS 定义为投影指定了权威代码（建议这样做），则如果定义缺少必需参数，将会发生错误。在这种情况下，错误消息会指出问题所在。MySQL 支持的投影方法和必需参数显示在 Table 15.7, “Supported Spatial Reference System Projection Methods” 和 Table 15.8, “Spatial Reference System Projection Parameters” 中。

有关在 MySQL 中编写 SRS 定义的更多信息，请参阅 [MySQL 8.0 中的地理空间参考系统](https://mysqlserverteam.com/geographic-spatial-reference-systems-in-mysql-8-0/) 和 [MySQL 8.0 中的投影空间参考系统](https://mysqlserverteam.com/projected-spatial-reference-systems-in-mysql-8-0/)

下表显示了 MySQL 支持的投影方法。MySQL 允许未知的投影方法，但无法检查必需参数的定义，也无法将空间数据转换为未知的投影或从未知的投影转换。有关每种投影如何工作的详细解释，包括公式，请参阅[EPSG 指南 7-2](http://www.epsg.org/Portals/0/373-07-2.pdf)。

**表 15.7 支持的空间参考系统投影方法**

| EPSG 代码 | 投影名称 | 必需参数（EPSG 代码） |
| --- | --- | --- |
| 1024 | 流行的可视化伪墨卡托 | 8801, 8802, 8806, 8807 |
| 1027 | 兰伯特等面积圆锥（球形） | 8801, 8802, 8806, 8807 |
| 1028 | 等距圆柱 | 8823, 8802, 8806, 8807 |
| 1029 | 等距圆柱（球形） | 8823, 8802, 8806, 8807 |
| 1041 | 克罗瓦克（北向） | 8811, 8833, 1036, 8818, 8819, 8806, 8807 |
| 1042 | 修改后的克罗瓦克 | 8811, 8833, 1036, 8818, 8819, 8806, 8807, 8617, 8618, 1026, 1027, 1028, 1029, 1030, 1031, 1032, 1033, 1034, 1035 |
| 1043 | 修改后的克罗瓦克（北向） | 8811, 8833, 1036, 8818, 8819, 8806, 8807, 8617, 8618, 1026, 1027, 1028, 1029, 1030, 1031, 1032, 1033, 1034, 1035 |
| 1051 | 兰伯特圆锥等积（2SP 密歇根） | 8821, 8822, 8823, 8824, 8826, 8827, 1038 |
| 1052 | 哥伦比亚城市 | 8801, 8802, 8806, 8807, 1039 |
| 9801 | 兰伯特圆锥等积（1SP） | 8801, 8802, 8805, 8806, 8807 |
| 9802 | 兰伯特圆锥等积（2SP） | 8821, 8822, 8823, 8824, 8826, 8827 |
| 9803 | 兰伯特圆锥等积（2SP 比利时） | 8821, 8822, 8823, 8824, 8826, 8827 |
| 9804 | 墨卡托（变种 A） | 8801, 8802, 8805, 8806, 8807 |
| 9805 | 墨卡托（变种 B） | 8823, 8802, 8806, 8807 |
| 9806 | 卡西尼-索德纳 | 8801, 8802, 8806, 8807 |
| 9807 | 横轴墨卡托 | 8801, 8802, 8805, 8806, 8807 |
| 9808 | 横轴墨卡托（南向） | 8801, 8802, 8805, 8806, 8807 |
| 9809 | 斜方位立体投影 | 8801, 8802, 8805, 8806, 8807 |
| 9810 | 极地立体投影（变种 A） | 8801, 8802, 8805, 8806, 8807 |
| 9811 | 新西兰地图网格 | 8801, 8802, 8806, 8807 |
| 9812 | 侧面斜方位墨卡托（变种 A） | 8811, 8812, 8813, 8814, 8815, 8806, 8807 |
| 9813 | 拉博德斜方位墨卡托 | 8811, 8812, 8813, 8815, 8806, 8807 |
| 9815 | 侧面斜方位墨卡托（变种 B） | 8811, 8812, 8813, 8814, 8815, 8816, 8817 |
| 9816 | 突尼斯矿业网格 | 8821, 8822, 8826, 8827 |
| 9817 | 兰伯特圆锥近等积 | 8801, 8802, 8805, 8806, 8807 |
| 9818 | 美国多锥形 | 8801, 8802, 8806, 8807 |
| 9819 | 克罗瓦克 | 8811, 8833, 1036, 8818, 8819, 8806, 8807 |
| 9820 | 兰伯特等面积圆锥 | 8801, 8802, 8806, 8807 |
| 9822 | 阿尔伯斯等面积 | 8821, 8822, 8823, 8824, 8826, 8827 |
| 9824 | 横轴墨卡托分带网格系统 | 8801, 8830, 8831, 8805, 8806, 8807 |
| 9826 | 兰伯特圆锥等积（西向） | 8801, 8802, 8805, 8806, 8807 |
| 9828 | 博恩（南向） | 8801, 8802, 8806, 8807 |
| 9829 | 极射赤面投影（变种 B） | 8832, 8833, 8806, 8807 |
| 9830 | 极射赤面投影（变种 C） | 8832, 8833, 8826, 8827 |
| 9831 | 关岛投影 | 8801, 8802, 8806, 8807 |
| 9832 | 修改的等距方位投影 | 8801, 8802, 8806, 8807 |
| 9833 | 双曲卡西尼-索尔德投影 | 8801, 8802, 8806, 8807 |
| 9834 | 兰伯特圆柱等面积投影（球面） | 8823, 8802, 8806, 8807 |
| 9835 | 兰伯特圆柱等面积投影 | 8823, 8802, 8806, 8807 |
| EPSG 代码 | 投影名称 | 强制参数（EPSG 代码） |

下表显示了 MySQL 可识别的投影参数。识别主要通过授权代码进行。如果没有授权代码，MySQL 将回退到对参数名称的不区分大小写的字符串匹配。有关每个参数的详细信息，请通��� [EPSG 在线注册表](https://www.epsg-registry.org) 的代码查找。

**表 15.8 空间参考系统投影参数**

| EPSG 代码 | 回退名称（MySQL 可识别） | EPSG 名称 |
| --- | --- | --- |
| 1026 | c1 | C1 |
| 1027 | c2 | C2 |
| 1028 | c3 | C3 |
| 1029 | c4 | C4 |
| 1030 | c5 | C5 |
| 1031 | c6 | C6 |
| 1032 | c7 | C7 |
| 1033 | c8 | C8 |
| 1034 | c9 | C9 |
| 1035 | c10 | C10 |
| 1036 | 方位角 | 锥轴的共纬度 |
| 1038 | 椭球体比例因子 | 椭球体比例因子 |
| 1039 | 投影平面原点高度 | 投影平面原点高度 |
| 8617 | evaluation_point_ordinate_1 | 评估点纵坐标 1 |
| 8618 | evaluation_point_ordinate_2 | 评估点纵坐标 2 |
| 8801 | 原点纬度 | 自然原点纬度 |
| 8802 | 中央经线 | 自然原点经度 |
| 8805 | 比例因子 | 自然原点的比例因子 |
| 8806 | 东偏移 | 伪东移距离 |
| 8807 | 北偏移 | 伪北移距离 |
| 8811 | 中心纬度 | 投影中心纬度 |
| 8812 | 中心经度 | 投影中心经度 |
| 8813 | 方位角 | 初始线方位角 |
| 8814 | 矫正网格角度 | 从矫正到斜网格的角度 |
| 8815 | 比例因子 | 初始线上的比例因子 |
| 8816 | 东偏移 | 投影中心的东移距离 |
| 8817 | 北偏移 | 投影中心的北移距离 |
| 8818 | 伪标准纬线 1 | 伪标准纬线纬度 |
| 8819 | 比例因子 | 伪标准纬线上的比例因子 |
| 8821 | 原点纬度 | 伪原点纬度 |
| 8822 | central_meridian | 伪原点经度 |
| 8823 | 第 1 标准纬线 | 第 1 标准纬线纬度 |
| 8824 | 第 2 标准纬线 | 第 2 标准纬线纬度 |
| 8826 | 东偏移 | 伪原点的东移距离 |
| 8827 | 北偏移 | 伪原点的北移距离 |
| 8830 | 初始经度 | 初始经度 |
| 8831 | 分带宽度 | 分带宽度 |
| 8832 | 标准纬线 | 标准纬线纬度 |
| 8833 | 中央经度 | 起始经度 |
| EPSG 代码 | 回退名称（MySQL 可识别） | EPSG 名称 |
