# 第三十章 MySQL sys 模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema.html)

**目录**

30.1 使用 sys 模式的先决条件

30.2 使用 sys 模式

30.3 sys 模式的进度报告

30.4 sys 模式对象参考

30.4.1 sys 模式对象索引

30.4.2 sys 模式的表和触发器

30.4.3 sys 模式的视图

30.4.4 sys 模式的存储过程

30.4.5 sys 模式的存储函数

MySQL 8.0 包括`sys`模式，这是一组对象，帮助 DBA 和开发人员解释 Performance Schema 收集的数据。`sys`模式对象可用于典型的调优和诊断用例。此模式中的对象包括：

+   将 Performance Schema 数据汇总为更易理解形式的视图。

+   执行操作，如 Performance Schema 配置和生成诊断报告的存储过程。

+   查询 Performance Schema 配置并提供格式化服务的存储函数。

对于新安装，如果您使用带有`--initialize`或`--initialize-insecure`选项的**mysqld**进行数据目录初始化，默认情况下会安装`sys`模式。如果不需要这个模式，您可以在初始化后手动删除`sys`模式。

如果存在一个没有`version`视图的`sys`模式，则 MySQL 升级过程会产生错误，假设缺少此视图表示用户创建了`sys`模式。在这种情况下，要升级，请先删除或重命名现有的`sys`模式。

`sys`模式对象的`DEFINER`为`'mysql.sys'@'localhost'`。使用专用的`mysql.sys`帐户可以避免如果 DBA 重命名或删除`root`帐户而导致的问题。
