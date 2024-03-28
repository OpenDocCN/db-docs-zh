# 5.3.1 创建和选择数据库

> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-database.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-database.html)

如果管理员在设置权限时为您创建数据库，则可以开始使用它。否则，您需要自行创建：

```sql
mysql> CREATE DATABASE menagerie;
```

在 Unix 下，数据库名称区分大小写（不像 SQL 关键字），因此您必须始终将数据库称为`menagerie`，而不是`Menagerie`、`MENAGERIE`或其他变体。表名也是如此。（在 Windows 下，此限制不适用，尽管您必须在给定查询中始终使用相同的大小写来引用数据库和表。但是，出于各种原因，推荐的最佳实践始终是使用创建数据库时使用的相同大小写。）

注意

如果在尝试创建数据库时出现诸如 ERROR 1044 (42000): Access denied for user 'micah'@'localhost' to database 'menagerie'的错误，这意味着您的用户帐户没有必要的权限来执行此操作。请与管理员讨论此问题或参见第 8.2 节，“访问控制和帐户管理”。

创建数据库不会自动选择它以供使用；您必须明确执行此操作。要使`menagerie`成为当前数据库，请使用以下语句：

```sql
mysql> USE menagerie
Database changed
```

您只需创建数据库一次，但每次开始**mysql**会话时，必须选择它以供使用。您可以通过发出如示例中所示的`USE`语句来执行此操作。或者，您可以在调用**mysql**时在命令行上选择数据库。只需在可能需要提供的任何连接参数之后指定其名称。例如：

```sql
$> mysql -h *host* -u *user* -p menagerie
Enter password: ********
```

重要

在刚刚显示的命令中，`menagerie`**不是**您的密码。如果您想在`-p`选项后的命令行上提供密码，必须在没有空格的情况下这样做（例如，作为`-p*`password`*`，而不是作为`-p *`password`*`）。但是，将密码放在命令行上并不推荐，因为这样做会使其暴露给其他登录到您的计算机上的用户。

注意

您可以随时使用`SELECT` `DATABASE()`查看当前选择的数据库。
