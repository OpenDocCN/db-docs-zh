> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-multipolygon.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-multipolygon.html)

#### 13.4.2.13 MultiPolygon 类

一个`MultiPolygon`是由`Polygon`元素组成的`MultiSurface`对象。

**`MultiPolygon`示例**

+   在区域地图上，一个`MultiPolygon`可以代表一个湖泊系统。

**`MultiPolygon`断言**

+   一个`MultiPolygon`没有两个内部相交的`Polygon`元素。

+   一个`MultiPolygon`没有两个相交的`Polygon`元素（相交也被前一个断言禁止），或者在无限多点处接触。

+   一个`MultiPolygon`不能有切线、尖点或穿孔。一个`MultiPolygon`是一个规则的、闭合的点集。

+   一个具有多个`Polygon`的`MultiPolygon`具有不连通的内部。`MultiPolygon`内部的连通分量数量等于`MultiPolygon`中的`Polygon`值数量。

**`MultiPolygon`属性**

+   一个`MultiPolygon`是一个二维几何体。

+   一个`MultiPolygon`的边界是一组闭合曲线（`LineString`值），对应于其`Polygon`元素的边界。

+   `MultiPolygon`边界中的每个`Curve`都在一个`Polygon`元素的边界中。

+   每个`Polygon`元素边界中的每个`Curve`都在`MultiPolygon`的边界中。
