> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-version.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-version.html)

#### 30.4.3.47 版本视图

这个视图提供了当前的`sys`模式和 MySQL 服务器版本。

注意

截至 MySQL 8.0.18 版本，此视图已被弃用，并可能在未来的 MySQL 版本中被移除。使用它的应用程序应迁移到使用其他替代方法。例如，使用`VERSION()`函数来检索 MySQL 服务器版本。

`version`视图包含以下列：

+   `sys_version`

    `sys`模式版本。

+   `mysql_version`

    MySQL 服务器版本。
