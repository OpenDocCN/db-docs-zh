# A.7 MySQL 8.0 FAQ: INFORMATION_SCHEMA

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-information-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-information-schema.html)

A.7.1\. 我在哪里可以找到 MySQL INFORMATION_SCHEMA 数据库的文档？

A.7.2\. 是否有关于 INFORMATION_SCHEMA 的讨论论坛？

A.7.3\. 我在哪里可以找到 ANSI SQL 2003 关于 INFORMATION_SCHEMA 的规范？

A.7.4\. Oracle 数据字典和 MySQL INFORMATION_SCHEMA 有什么区别？

A.7.5\. 我可以添加或修改在 INFORMATION_SCHEMA 数据库中找到的表吗？

| **A.7.1.** | 我在哪里可以找到 MySQL `INFORMATION_SCHEMA`数据库的文档？ |
| --- | --- |
|  | 请参阅第二十八章，*INFORMATION_SCHEMA Tables*。您也可以在[MySQL 用户论坛](https://forums.mysql.com/list.php?20)找到帮助。 |
| **A.7.2.** | 是否有关于`INFORMATION_SCHEMA`的讨论论坛？ |
|  | 请参阅[MySQL 用户论坛](https://forums.mysql.com/list.php?20)。 |
| **A.7.3.** | 我在哪里可以找到 ANSI SQL 2003 关于`INFORMATION_SCHEMA`的规范？ |
|  | 不幸的是，官方规范并非免费提供。（ANSI 提供购买）。但是，有一些书籍可供参考，比如 Peter Gulutzan 和 Trudy Pelzer 的*SQL-99 Complete, Really*，提供了关于标准的全面概述，包括`INFORMATION_SCHEMA`。 |
| **A.7.4.** | Oracle 数据字典和 MySQL `INFORMATION_SCHEMA`有什么区别？ |
|  | Oracle 和 MySQL 都在表中提供元数据。但是，Oracle 和 MySQL 使用不同的表名和列名。MySQL 的实现更类似于 DB2 和 SQL Server 中找到的实现，它们也支持 SQL 标准中定义的`INFORMATION_SCHEMA`。 |
| **A.7.5.** | 我可以添加或修改在`INFORMATION_SCHEMA`数据库中找到的表吗？ |
|  | 没有。由于应用程序可能依赖于某种标准结构，因此不应进行修改。因此，*我们无法支持由于修改`INFORMATION_SCHEMA`表或数据而导致的错误或其他问题*。 |
