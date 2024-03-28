# 14.16.7 几何属性函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-property-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-property-functions.html)

14.16.7.1 通用几何属性函数

14.16.7.2 点属性函数

14.16.7.3 线串和多线串属性函数

14.16.7.4 多边形和多多边形属性函数

14.16.7.5 几何集合属性函数

每个属于这个组的函数都以几何值作为其参数，并返回几何的一些定量或定性属性。一些函数限制其参数类型。如果参数的几何类型不正确，则这些函数返回`NULL`。例如，`ST_Area()` 多边形函数在对象类型既不是`Polygon`也不是`MultiPolygon`时返回`NULL`。
