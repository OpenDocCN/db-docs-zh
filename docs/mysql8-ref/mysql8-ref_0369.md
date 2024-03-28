# 8.2.9 保留帐户

> 原文：[`dev.mysql.com/doc/refman/8.0/en/reserved-accounts.html`](https://dev.mysql.com/doc/refman/8.0/en/reserved-accounts.html)

MySQL 安装过程的一部分是数据目录初始化（请参阅 Section 2.9.1，“初始化数据目录”）。在数据目录初始化期间，MySQL 创建应被视为保留的用户帐户：

+   `'root'@'localhost`: 用于管理目的。此帐户具有所有特权，是系统帐户，并且可以执行任何操作。

    严格来说，此帐户名称并非保留，因为某些安装将`root`帐户重命名为其他名称，以避免暴露具有众所周知名称的高度特权帐户。

+   `'mysql.sys'@'localhost'`: 用作`sys`模式对象的`DEFINER`。使用`mysql.sys`帐户可以避免如果 DBA 重命名或删除`root`帐户而导致的问题。此帐户已锁定，因此不能用于客户端连接。

+   `'mysql.session'@'localhost'`: 内部使用的帐户，用于插件访问服务器。此帐户已锁定，因此不能用于客户端连接。该帐户是系统帐户。

+   `'mysql.infoschema'@'localhost'`: 用作`INFORMATION_SCHEMA`视图的`DEFINER`。使用`mysql.infoschema`帐户可以避免如果 DBA 重命名或删除 root 帐户而导致的问题。此帐户已锁定，因此不能用于客户端连接。
