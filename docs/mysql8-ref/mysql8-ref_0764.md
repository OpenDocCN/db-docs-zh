> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-curve.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-curve.html)

#### 13.4.2.4 Curve 类

一个`Curve`是一维几何体，通常由一系列点表示。特定的`Curve`子类定义了点之间的插值类型。`Curve`是一个不可实例化的类。

**`Curve`属性**

+   一个`Curve`具有其点的坐标。

+   一个`Curve`被定义为一维几何体。

+   如果一个`Curve`不通过同一点两次，则它是简单的，但如果起点和终点相同，则曲线仍然可以是简单的。

+   如果一个`Curve`的起点等于终点，则该`Curve`是闭合的。

+   闭合`Curve`的边界为空。

+   非闭合`Curve`的边界由其两个端点组成。

+   一个简单且闭合的`Curve`是`LinearRing`。
