# 16.8 数据字典限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-dictionary-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-limitations.html)

本节描述了 MySQL 数据字典引入的临时限制。

+   在数据目录下手动创建数据库目录（例如，使用**mkdir**）是不受支持的。MySQL 服务器不会识别手动创建的数据库目录。

+   由于写入存储、撤销日志和重做日志而不是`.frm`文件，DDL 操作需要更长时间。
