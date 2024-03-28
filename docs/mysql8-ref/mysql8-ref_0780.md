# 13.4.9 优化空间分析

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-spatial-analysis.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-spatial-analysis.html)

对于`MyISAM`和`InnoDB`表，包含空间数据的列中的搜索操作可以使用`SPATIAL`索引进行优化。最典型的操作包括：

+   点查询，搜索包含给定点的所有对象

+   区域查询，搜索与给定区域重叠的所有对象

MySQL 在空间列上使用**具有二次分裂的 R-Tree**来构建`SPATIAL`索引。`SPATIAL`索引是使用几何图形的最小外接矩形（MBR）构建的。对于大多数几何图形，MBR 是一个围绕几何图形的最小矩形。对于水平或垂直线串，MBR 是一个退化为线串的矩形。对于点，MBR 是一个退化为点的矩形。

也可以在空间列上创建普通索引。在非`SPATIAL`索引中，必须为除`POINT`列之外的任何空间列声明前缀。

`MyISAM`和`InnoDB`都支持`SPATIAL`和非`SPATIAL`索引。其他存储引擎支持非`SPATIAL`索引，如第 15.1.15 节，“CREATE INDEX 语句”中所述。
