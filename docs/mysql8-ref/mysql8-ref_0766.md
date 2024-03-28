> 原文：[`dev.mysql.com/doc/refman/8.0/en/gis-class-surface.html`](https://dev.mysql.com/doc/refman/8.0/en/gis-class-surface.html)

#### 13.4.2.6 Surface 类

一个`Surface`是一个二维几何体。它是一个不可实例化的类。它唯一可实例化的子类是`Polygon`。

**`Surface`属性**

+   一个`Surface`被定义为一个二维几何体。

+   OpenGIS 规范将简单的`Surface`定义为一个几何体，由一个与单个外部边界和零个或多个内部边界相关联的“补丁”组成。

+   一个简单`Surface`的边界是与其外部和内部边界对应的一组闭合曲线。
