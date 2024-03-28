> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-multicurve.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-multicurve.html)

#### 13.4.2.10 MultiCurve 类

`MultiCurve` 是由 `Curve` 元素组成的几何集合。`MultiCurve` 是一个不可实例化的类。

**`MultiCurve` 属性**

+   `MultiCurve` 是一维几何体。

+   当且仅当 `MultiCurve` 的所有元素都是简单的时，`MultiCurve` 才是简单的；任何两个元素之间的唯一交点发生在这两个元素的边界上。

+   通过应用“模 2 并集规则”（也称为“奇偶规则”）可以获得 `MultiCurve` 边界：如果一个点在奇数个 `Curve` 元素的边界上，则它在 `MultiCurve` 的边界上。

+   如果 `MultiCurve` 的所有元素都是闭合的，则 `MultiCurve` 是闭合的。

+   闭合 `MultiCurve` 的边界始终为空。
