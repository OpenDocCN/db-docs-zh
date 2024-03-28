# 14.16.1 空间函数参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-function-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-function-reference.html)

以下表格列出了每个空间函数并提供了每个函数的简短描述。

**表格 14.21 空间函数**

| 名称 | 描述 | 引入版本 |
| --- | --- | --- |
| `GeomCollection()` | 从几何图形构造几何集合 |  |
| `GeometryCollection()` | 从几何图形构造几何集合 |  |
| `LineString()` | 从 Point 值构造 LineString |  |
| `MBRContains()` | 判断一个几何图形的 MBR 是否包含另一个几何图形的 MBR |  |
| `MBRCoveredBy()` | 判断一个 MBR 是否被另一个 MBR 覆盖 |  |
| `MBRCovers()` | 判断一个 MBR 是否覆盖另一个 MBR |  |
| `MBRDisjoint()` | 判断两个几何图形的 MBR 是否不相交 |  |
| `MBREquals()` | 判断两个几何图形的 MBR 是否相等 |  |
| `MBRIntersects()` | 判断两个几何图形的 MBR 是否相交 |  |
| `MBROverlaps()` | 判断两个几何图形的 MBR 是否重叠 |  |
| `MBRTouches()` | 判断两个几何图形的 MBR 是否相接触 |  |
| `MBRWithin()` | 判断一个几何图形的 MBR 是否在另一个几何图形的 MBR 内部 |  |
| `MultiLineString()` | 从 LineString 值构造 MultiLineString |  |
| `MultiPoint()` | 从 Point 值构造 MultiPoint |  |
| `MultiPolygon()` | 从多边形值构造 MultiPolygon |  |
| `Point()` | 从坐标构造点 |  |
| `Polygon()` | 从 LineString 参数构造多边形 |  |
| `ST_Area()` | 返回多边形或 MultiPolygon 的面积 |  |
| `ST_AsBinary()`, `ST_AsWKB()` | 将内部几何格式转换为 WKB |  |
| `ST_AsGeoJSON()` | 从几何图形生成 GeoJSON 对象 |  |
| `ST_AsText()`, `ST_AsWKT()` | 从内部几何格式转换为 WKT |  |
| `ST_Buffer()` | 返回距离给定距离内的几何体的几何体 |  |
| `ST_Buffer_Strategy()` | 为 ST_Buffer()生成策略选项 |  |
| `ST_Centroid()` | 返回几何体的质心点 |  |
| `ST_Collect()` | 将空间值聚合成集合 | 8.0.24 |
| `ST_Contains()` | 一个几何体是否包含另一个 |  |
| `ST_ConvexHull()` | 返回几何体的凸包 |  |
| `ST_Crosses()` | 一个几何体是否与另一个相交 |  |
| `ST_Difference()` | 两个几何体的点集差 |  |
| `ST_Dimension()` | 几何体的维度 |  |
| `ST_Disjoint()` | 一个几何体是否与另一个不相交 |  |
| `ST_Distance()` | 一个几何体到另一个的距离 |  |
| `ST_Distance_Sphere()` | 两个几何体在地球上的最小距离 |  |
| `ST_EndPoint()` | 线串的终点 |  |
| `ST_Envelope()` | 返回几何体的最小边界矩形 |  |
| `ST_Equals()` | 一个几何体是否等于另一个 |  |
| `ST_ExteriorRing()` | 返回多边形的外环 |  |
| `ST_FrechetDistance()` | 一个几何体到另一个的离散弗雷歇距离 | 8.0.23 |
| `ST_GeoHash()` | 生成地理哈希值 |  |
| `ST_GeomCollFromText()`, `ST_GeometryCollectionFromText()`, `ST_GeomCollFromTxt()` | 从 WKT 返回几何体集合 |  |
| `ST_GeomCollFromWKB()`, `ST_GeometryCollectionFromWKB()` | 从 WKB 返回几何体集合 |  |
| `ST_GeometryN()` | 从几何体集合返回第 N 个几何体 |  |
| `ST_GeometryType()` | 返回几何类型的名称 |  |
| `ST_GeomFromGeoJSON()` | 从 GeoJSON 对象生成几何图形 |  |
| `ST_GeomFromText()`, `ST_GeometryFromText()` | 从 WKT 返回几何图形 |  |
| `ST_GeomFromWKB()`, `ST_GeometryFromWKB()` | 从 WKB 返回几何图形 |  |
| `ST_HausdorffDistance()` | 一个几何图形到另一个的离散豪斯多夫距离 | 8.0.23 |
| `ST_InteriorRingN()` | 返回多边形的第 N 个内环 |  |
| `ST_Intersection()` | 返回两个几何图形的点集交集 |  |
| `ST_Intersects()` | 一个几何图形是否与另一个相交 |  |
| `ST_IsClosed()` | 几何图形是否闭合且简单 |  |
| `ST_IsEmpty()` | 几何图形是否为空 |  |
| `ST_IsSimple()` | 几何图形是否简单 |  |
| `ST_IsValid()` | 几何图形是否有效 |  |
| `ST_LatFromGeoHash()` | 从 geohash 值返回纬度 |  |
| `ST_Latitude()` | 返回 Point 的纬度 | 8.0.12 |
| `ST_Length()` | 返回 LineString 的长度 |  |
| `ST_LineFromText()`, `ST_LineStringFromText()` | 从 WKT 构造 LineString |  |
| `ST_LineFromWKB()`, `ST_LineStringFromWKB()` | 从 WKB 构造 LineString |  |
| `ST_LineInterpolatePoint()` | 沿着 LineString 给定百分比的点 | 8.0.24 |
| `ST_LineInterpolatePoints()` | 沿着 LineString 给定百分比的点 | 8.0.24 |
| `ST_LongFromGeoHash()` | 从 geohash 值返回经度 |  |
| `ST_Longitude()` | 返回 Point 的经度 | 8.0.12 |
| `ST_MakeEnvelope()` | 两点周围的矩形 |  |
| `ST_MLineFromText()`, `ST_MultiLineStringFromText()` | 从 WKT 构建 MultiLineString |  |
| `ST_MLineFromWKB()`, `ST_MultiLineStringFromWKB()` | 从 WKB 构建 MultiLineString |  |
| `ST_MPointFromText()`, `ST_MultiPointFromText()` | 从 WKT 构建 MultiPoint |  |
| `ST_MPointFromWKB()`, `ST_MultiPointFromWKB()` | 从 WKB 构建 MultiPoint |  |
| `ST_MPolyFromText()`, `ST_MultiPolygonFromText()` | 从 WKT 构建 MultiPolygon |  |
| `ST_MPolyFromWKB()`, `ST_MultiPolygonFromWKB()` | 从 WKB 构建 MultiPolygon |  |
| `ST_NumGeometries()` | 返回几何集合中的几何对象数量 |  |
| `ST_NumInteriorRing()`, `ST_NumInteriorRings()` | 返回多边形中内部环的数量 |  |
| `ST_NumPoints()` | 返回 LineString 中的点数 |  |
| `ST_Overlaps()` | 一个几何对象是否与另一个重叠 |  |
| `ST_PointAtDistance()` | 沿着 LineString 给定距离的点 | 8.0.24 |
| `ST_PointFromGeoHash()` | 将 geohash 值转换为 POINT 值 |  |
| `ST_PointFromText()` | 从 WKT 构建 Point |  |
| `ST_PointFromWKB()` | 从 WKB 构建 Point |  |
| `ST_PointN()` | 返回 LineString 中的第 N 个点 |  |
| `ST_PolyFromText()`, `ST_PolygonFromText()` | 从 WKT 构建 Polygon |  |
| `ST_PolyFromWKB()`, `ST_PolygonFromWKB()` | 从 WKB 构建 Polygon |  |
| `ST_Simplify()` | 返回简化的几何对象 |  |
| `ST_SRID()` | 返回几何对象的空间参考系统 ID |  |
| `ST_StartPoint()` | LineString 的起始点 |  |
| `ST_SwapXY()` | 返回交换 X/Y 坐标的参数 |  |
| `ST_SymDifference()` | 返回两个几何对象的点集对称差 |  |
| `ST_Touches()` | 一个几何体是否与另一个几何体相接触 |  |
| `ST_Transform()` | 转换几何体的坐标 | 8.0.13 |
| `ST_Union()` | 返回两个几何体的点集并集 |  |
| `ST_Validate()` | 返回经过验证的几何体 |  |
| `ST_Within()` | 一个几何体是否在另一个几何体内部 |  |
| `ST_X()` | 返回点的 X 坐标 |  |
| `ST_Y()` | 返回点的 Y 坐标 |  |
| 名称 | 描述 | 引入版本 |
