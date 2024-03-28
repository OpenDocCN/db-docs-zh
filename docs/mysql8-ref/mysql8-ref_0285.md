# 7.5.4 查询属性组件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/query-attribute-components.html`](https://dev.mysql.com/doc/refman/8.0/en/query-attribute-components.html)

从 MySQL 8.0.23 开始，一个组件服务提供了对查询属性的访问（参见第 11.6 节，“查询属性”）。`query_attributes`组件使用此服务在 SQL 语句中提供对查询属性的访问。

+   目的：实现`mysql_query_attribute_string()`函数，该函数接受属性名称参数并将属性值作为字符串返回，如果属性不存在则返回`NULL`。

+   URN：`file://component_query_attributes`

希望使用`query_attributes`所使用的相同查询属性组件服务的开发人员应查阅 MySQL 源代码分发中的`mysql_query_attributes.h`文件。
