> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-multipoint.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-multipoint.html)

#### 13.4.2.9 MultiPoint 类

一个`MultiPoint`是由`Point`元素组成的几何集合。这些点没有以任何方式连接或排序。

**`MultiPoint` 示例**

+   在世界地图上，一个`MultiPoint`可以代表一串小岛。

+   在城市地图上，一个`MultiPoint`可以代表售票处的出口。

**`MultiPoint` 属性**

+   一个`MultiPoint`是一个零维几何体。

+   如果一个`MultiPoint`的两个`Point`值不相等（具有相同的坐标值），那么它就是简单的。

+   一个`MultiPoint`的边界是空集。
