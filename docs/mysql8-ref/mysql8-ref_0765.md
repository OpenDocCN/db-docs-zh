> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-linestring.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-linestring.html)

#### 13.4.2.5 LineString 类

一个`LineString`是一个具有点之间线性插值的`Curve`。

**`LineString` 示例**

+   在世界地图上，`LineString`对象可以表示河流。

+   在城市地图上，`LineString`对象可以表示街道。

**`LineString` 属性**

+   一个`LineString`由每对连续点定义的段的坐标组成。

+   如果一个`LineString`由恰好两个点组成，则它是一条`Line`。

+   一个`LineString`如果既是封闭的又是简单的，就是一个`LinearRing`。
