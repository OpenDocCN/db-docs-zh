> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-multisurface.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-multisurface.html)

#### 13.4.2.12 多表面类

一个`MultiSurface`是由表面元素组成的几何集合。`MultiSurface`是一个不可实例化的类。它唯一可实例化的子类是`MultiPolygon`。

**`MultiSurface`断言**

+   `MultiSurface`内的表面没有相交的内部。

+   `MultiSurface`内的表面的边界最多在有限数量的点处相交。
