> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-geometry-class-hierarchy.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-geometry-class-hierarchy.html)

#### 几何类层次结构

几何类定义如下层次结构：

+   `Geometry`（不可实例化）

    +   `Point`（可实例化）

    +   `Curve`（不可实例化）

        +   `LineString`（可实例化）

            +   `Line`

            +   `LinearRing`

    +   `Surface`（不可实例化）

        +   `Polygon`（可实例化）

    +   `GeometryCollection`（可实例化）

        +   `MultiPoint`（可实例化）

        +   `MultiCurve`（不可实例化）

            +   `MultiLineString`（可实例化）

        +   `MultiSurface`（不可实例化）

            +   `MultiPolygon`（可实例化）

不可能在不可实例化的类中创建对象。可以在可实例化的类中创建对象。所有类都有属性，可实例化的类也可能有断言（定义有效类实例的规则）。

`Geometry`是基类。它是一个抽象类。`Geometry`的可实例化子类限制为存在于二维坐标空间中的零、一和二维几何对象。所有可实例化的几何类都被定义为有效实例是拓扑闭合的（即，所有定义的几何包括其边界）。

基类`Geometry`具有`Point`、`Curve`、`Surface`和`GeometryCollection`的子类：

+   `Point`代表零维对象。

+   `Curve`代表一维对象，具有子类`LineString`，子子类`Line`和`LinearRing`。

+   `Surface`设计用于二维对象，具有子类`Polygon`。

+   `GeometryCollection`具有专门的零、一和二维集合类，分别命名为`MultiPoint`、`MultiLineString`和`MultiPolygon`，用于建模对应于`Points`、`LineStrings`和`Polygons`的几何。`MultiCurve`和`MultiSurface`被引入为概括集合接口以处理`Curves`和`Surfaces`的抽象超类。

`Geometry`、`Curve`、`Surface`、`MultiCurve`和`MultiSurface`被定义为不可实例化的类。它们为其子类定义了一组共同的方法，并且包含用于可扩展性的方法。

`Point`、`LineString`、`Polygon`、`GeometryCollection`、`MultiPoint`、`MultiLineString`和`MultiPolygon`是可实例化的类。
