# 13.4.3 支持的空间数据格式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-data-formats.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-data-formats.html)

用于在查询中表示几何对象的两种标准空间数据格式：

+   Well-Known Text (WKT)格式

+   Well-Known Binary (WKB)格式

在内部，MySQL 以一种与 WKT 或 WKB 格式不完全相同的格式存储几何值（内部格式类似于 WKB，但具有用于指示 SRID 的初始 4 个字节）。

有函数可用于在不同数据格式之间转换；参见第 14.16.6 节，“几何格式转换函数”。

以下各节描述了 MySQL 使用的空间数据格式：

+   Well-Known Text (WKT) Format Format")

+   Well-Known Binary (WKB) Format Format")

+   Internal Geometry Storage Format

#### Well-Known Text (WKT)格式

几何值的 Well-Known Text (WKT)表示设计用于以 ASCII 形式交换几何数据。OpenGIS 规范提供了一个 Backus-Naur 语法，指定了编写 WKT 值的正式生成规则（参见第 13.4 节，“空间数据类型”）。

几何对象的 WKT 表示示例：

+   一个`Point`：

    ```sql
    POINT(15 20)
    ```

    点坐标未用逗号分隔。这与 SQL `Point()`函数的语法不同，后者要求坐标之间有逗号。请注意使用适合给定空间操作上下文的语法。例如，以下语句都使用`ST_X()`从`Point`对象中提取 X 坐标。第一个直接使用`Point()`函数生成对象。第二个使用转换为`Point`的 WKT 表示，使用`ST_GeomFromText()`。

    ```sql
    mysql> SELECT ST_X(Point(15, 20));
    +---------------------+
    | ST_X(POINT(15, 20)) |
    +---------------------+
    |                  15 |
    +---------------------+

    mysql> SELECT ST_X(ST_GeomFromText('POINT(15 20)'));
    +---------------------------------------+
    | ST_X(ST_GeomFromText('POINT(15 20)')) |
    +---------------------------------------+
    |                                    15 |
    +---------------------------------------+
    ```

+   具有四个点的`LineString`：

    ```sql
    LINESTRING(0 0, 10 10, 20 25, 50 60)
    ```

    点坐标对由逗号分隔。

+   具有一个外环和一个内环的`Polygon`：

    ```sql
    POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5))
    ```

+   具有三个`Point`值的`MultiPoint`：

    ```sql
    MULTIPOINT(0 0, 20 20, 60 60)
    ```

    接受`MultiPoint`值的 WKT 格式表示的空间函数，如`ST_MPointFromText()`和`ST_GeomFromText()`，允许值内的单个点被括号括起来。例如，以下两个函数调用都是有效的：

    ```sql
    ST_MPointFromText('MULTIPOINT (1 1, 2 2, 3 3)')
    ST_MPointFromText('MULTIPOINT ((1 1), (2 2), (3 3))')
    ```

+   一个具有两个`LineString`值的`MultiLineString`：

    ```sql
    MULTILINESTRING((10 10, 20 20), (15 15, 30 15))
    ```

+   具有��个`Polygon`值的`MultiPolygon`：

    ```sql
    MULTIPOLYGON(((0 0,10 0,10 10,0 10,0 0)),((5 5,7 5,7 7,5 7, 5 5)))
    ```

+   由两个`Point`值和一个`LineString`组成的`GeometryCollection`：

    ```sql
    GEOMETRYCOLLECTION(POINT(10 10), POINT(30 30), LINESTRING(15 15, 20 20))
    ```

#### 熟知二进制（WKB）格式

几何值的熟知二进制（WKB）表示用于以二进制流交换几何数据，表示为包含几何 WKB 信息的 `BLOB` 值。该格式由 OpenGIS 规范定义（请参阅第 13.4 节，“空间数据类型”）。它还在 ISO *SQL/MM 第 3 部分：空间*标准中定义。

WKB 使用 1 字节无符号整数、4 字节无符号整数和 8 字节双精度数（IEEE 754 格式）。一个字节是八位。

例如，与 `POINT(1 -1)` 对应的 WKB 值由以下 21 个字节序列组成，每个字节由两个十六进制数字表示：

```sql
0101000000000000000000F03F000000000000F0BF
```

该序列由下表中显示的组件组成。

**表 13.2 WKB 组件示例**

| 组件 | 大小 | 值 |
| --- | --- | --- |
| 字节顺序 | 1 字节 | `01` |
| WKB 类型 | 4 字节 | `01000000` |
| X 坐标 | 8 字节 | `000000000000F03F` |
| Y 坐标 | 8 字节 | `000000000000F0BF` |

组件表示如下：

+   字节顺序指示符为 1 或 0，表示小端或大端存储。小端和大端字节顺序也被称为网络数据表示（NDR）和外部数据表示（XDR）。

+   WKB 类型是指示几何类型的代码。MySQL 使用值从 1 到 7 来表示 `Point`、`LineString`、`Polygon`、`MultiPoint`、`MultiLineString`、`MultiPolygon` 和 `GeometryCollection`。

+   `Point` 值具有 X 和 Y 坐标，每个坐标表示为双精度值。

更复杂的几何值的 WKB 值具有更复杂的数据结构，详细信息请参阅 OpenGIS 规范。

#### 内部几何存储格式

MySQL 使用 4 个字节来指示 SRID，然后是值的 WKB 表示。有关 WKB 格式的描述，请参阅熟知二进制（WKB）格式。

对于 WKB 部分，这些是适用于 MySQL 的特定考虑因素：

+   字节顺序指示符字节为 1，因为 MySQL 将几何值存储为小端值。

+   MySQL 支持 `Point`、`LineString`、`Polygon`、`MultiPoint`、`MultiLineString`、`MultiPolygon` 和 `GeometryCollection` 几何类型。不支持其他几何类型。

+   只有 `GeometryCollection` 可以为空。这样的值存储为 0 元素。

+   多边形环可以指定顺时针和逆时针。MySQL 在读取数据时会自动翻转环。

笛卡尔坐标以空间参考系统的长度单位存储，X 值在 X 坐标中，Y 值在 Y 坐标中。轴方向由空间参考系统指定。

地理坐标以空间参考系统的角度单位存储，经度在 X 坐标中，纬度在 Y 坐标中。轴方向和子午线由空间参考系统指定。

`LENGTH()` 函数返回存储值所需的字节空间。例如：

```sql
mysql> SET @g = ST_GeomFromText('POINT(1 -1)');
mysql> SELECT LENGTH(@g);
+------------+
| LENGTH(@g) |
+------------+
|         25 |
+------------+
mysql> SELECT HEX(@g);
+----------------------------------------------------+
| HEX(@g)                                            |
+----------------------------------------------------+
| 000000000101000000000000000000F03F000000000000F0BF |
+----------------------------------------------------+
```

值长度为 25 个字节，由以下组件组成（如从十六进制值中可以看出）：

+   4 个字节用于整数 SRID（0）

+   1 个字节用于整数字节顺序（1 = 小端）

+   4 个字节用于整数类型信息（1 = `Point`）

+   8 个字节用于双精度 X 坐标（1）

+   8 个字节用于双精度 Y 坐标（−1）
