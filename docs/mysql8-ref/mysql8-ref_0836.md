# 14.16.9 测试几何对象之间空间关系的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions.html)

14.16.9.1 使用对象形状的空间关系函数

14.16.9.2 使用最小外接矩形的空间关系函数

本节描述的函数接受两个几何对象作为参数，并返回它们之间的定性或定量关系。

MySQL 实现了两组函数，使用由 OpenGIS 规范定义的函数名称。一组函数使用精确的对象形状测试两个几何值之间的关系，另一组函数使用对象的最小外接矩形（MBRs）。
