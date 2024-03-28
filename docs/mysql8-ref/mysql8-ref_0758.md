# 13.4 空间数据类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-types.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html)

13.4.1 空间数据类型

13.4.2 OpenGIS 几何模型

13.4.3 支持的空间数据格式

13.4.4 几何形态和有效性

13.4.5 空间参考系统支持

13.4.6 创建空间列

13.4.7 填充空间列

13.4.8 获取空间数据

13.4.9 优化空间分析

13.4.10 创建空间索引

13.4.11 使用空间索引

[开放地理空间联盟](http://www.opengeospatial.org)（OGC）是一个由 250 多家公司、机构和大学参与开发公开可用的概念解决方案的国际联盟，这些解决方案可用于管理各种处理空间数据的应用程序。

开放地理空间联盟发布了*OpenGIS®实施标准地理信息 - 简单要素访问 - 第 2 部分：SQL 选项*，这是一份提出了几种概念性扩展 SQL RDBMS 以支持空间数据的文件。此规范可从 OGC 网站[`www.opengeospatial.org/standards/sfs`](http://www.opengeospatial.org/standards/sfs)获取。

遵循 OGC 规范，MySQL 将空间扩展实现为**带有几何类型的 SQL**环境的子集。该术语指的是已扩展为一组几何类型的 SQL 环境。实现为具有几何类型的列的几何值的 SQL 列。规范描述了一组 SQL 几何类型，以及在这些类型上创建和分析几何值的函数。

MySQL 空间扩展使得可以生成、存储和分析地理要素：

+   用于表示空间值的数据类型

+   用于操作空间值的函数

+   空间索引以提高对空间列的访问时间

空间数据类型和函数可用于`MyISAM`, `InnoDB`, `NDB`, 和 `ARCHIVE` 表。对于索引空间列，`MyISAM` 和 `InnoDB` 支持 `SPATIAL` 和非 `SPATIAL` 索引。其他存储引擎支持非 `SPATIAL` 索引，如 第 15.1.15 节，“CREATE INDEX 语句” 中所述。

**地理要素** 是世界上任何具有位置的事物。一个要素可以是：

+   一个实体。例如，一座山，一个池塘，一个城市。

+   一个空间。例如，城镇区，热带地区。

+   可定义的位置。例如，十字路口，作为两条街道交汇的特定地点。

一些文档使用术语**地理空间特征**来指代地理特征。

**几何**是指地理特征的另一个词。最初，**几何**一词指地球的测量。另一个含义来自制图学，指制图师用来绘制世界地图的几何特征。

这里讨论的术语视为同义词：**地理特征**，**地理空间特征**，**特征**或**几何**。最常用的术语是**几何**，定义为*代表世界上任何具有位置的点或点的集合*。

以下材料涵盖了这些主题：

+   MySQL 模型中实现的空间数据类型

+   OpenGIS 几何模型中空间扩展的基础

+   用于表示空间数据的数据格式

+   如何在 MySQL 中使用空间数据

+   用于空间数据的索引使用

+   MySQL 与 OpenGIS 规范的差异

有关操作空间数据的函数信息，请参见第 14.16 节，“空间分析函数”。

### 其他资源

这些标准对于 MySQL 实现空间操作很重要：

+   SQL/MM 第 3 部分：空间。

+   [开放地理空间联盟](http://www.opengeospatial.org)发布了*地理信息的 OpenGIS®实施标准*，该文件提出了几种扩展 SQL RDBMS 以支持空间数据的概念方法。特别参见 Simple Feature Access - Part 1: Common Architecture 和 Simple Feature Access - Part 2: SQL Option。开放地理空间联盟（OGC）在[`www.opengeospatial.org/`](http://www.opengeospatial.org/)维护一个网站。规范可在[`www.opengeospatial.org/standards/sfs`](http://www.opengeospatial.org/standards/sfs)上找到。其中包含与此材料相关的其他信息。

+   空间参考系统（SRS）定义的语法基于*OpenGIS 实施规范：坐标转换服务*，修订版 1.00，OGC 01-009，2001 年 1 月 12 日，第 7.2 节中定义的语法。该规范可在[`www.opengeospatial.org/standards/ct`](http://www.opengeospatial.org/standards/ct)上找到。有关 MySQL 中 SRS 定义与该规范的差异，请参见第 15.1.19 节，“CREATE SPATIAL REFERENCE SYSTEM Statement”。

如果您对 MySQL 的空间扩展使用有疑问或担忧，可以在 GIS 论坛中讨论：[`forums.mysql.com/list.php?23`](https://forums.mysql.com/list.php?23)。
