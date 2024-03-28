> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-polygon.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-polygon.html)

#### 13.4.2.7 Polygon 类

一个`Polygon`是表示多边几何形状的平面`Surface`。它由单个外部边界和零个或多个内部边界定义，其中每个内部边界定义了`Polygon`中的一个孔。

**`Polygon`示例**

+   在区域地图上，`Polygon`对象可以表示森林、区域等。

**`Polygon`断言**

+   `Polygon`的边界由一组`LinearRing`对象（即，既简单又闭合的`LineString`对象）组成，构成其外部和内部边界。

+   一个`Polygon`没有交叉的环。`Polygon`边界中的环可能在一个`Point`处相交，但只能作为切线。

+   一个`Polygon`没有线条、尖角或穿孔。

+   一个`Polygon`具有连通的内部点集。

+   一个`Polygon`可能有孔。带有孔的`Polygon`的外部不是连通的。每个孔定义了外部的一个连通组件。

上述断言使`Polygon`成为一个简单的几何体。
