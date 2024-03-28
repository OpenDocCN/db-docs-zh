> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-geometrycollection.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-geometrycollection.html)

#### 13.4.2.8 GeometryCollection 类

`GeomCollection` 是一个包含零个或多个任意类别几何体的集合几何体。

`GeomCollection` 和 `GeometryCollection` 是同义词，`GeomCollection` 是首选类型名称。

几何集合中的所有元素必须在相同的空间参考系统中（即在相同的坐标系统中）。对于几何集合的元素没有其他约束，尽管下面描述的 `GeomCollection` 的子类可能会限制成员资格。限制可能基于：

+   元素类型（例如，`MultiPoint` 可能只包含 `Point` 元素）

+   维度

+   元素之间空间重叠程度的约束
