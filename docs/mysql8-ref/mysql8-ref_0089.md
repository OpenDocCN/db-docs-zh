# 2.8.9 MySQL 配置和第三方工具

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-configuration-third-party.html`](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-third-party.html)

需要从 MySQL 源代码中确定 MySQL 版本的第三方工具可以读取顶级源目录中的 `MYSQL_VERSION` 文件。该文件单独列出了版本的各个部分。例如，如果版本是 MySQL 8.0.36，则文件如下所示：

```sql
MYSQL_VERSION_MAJOR=8
MYSQL_VERSION_MINOR=0
MYSQL_VERSION_PATCH=36
MYSQL_VERSION_EXTRA=
MYSQL_VERSION_STABILITY="LTS"
```

注意

在 MySQL 5.7 及更早版本中，此文件名为 `VERSION`。

要从版本组件构建一个五位数，使用以下公式：

```sql
MYSQL_VERSION_MAJOR*10000 + MYSQL_VERSION_MINOR*100 + MYSQL_VERSION_PATCH
```
