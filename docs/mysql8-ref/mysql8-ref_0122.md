# 5.3 创建和使用数据库

> 原文：[`dev.mysql.com/doc/refman/8.0/en/database-use.html`](https://dev.mysql.com/doc/refman/8.0/en/database-use.html)

5.3.1 创建和选择数据库

5.3.2 创建表

5.3.3 将数据加载到表中

5.3.4 从表中检索信息

一旦你知道如何输入 SQL 语句，你就可以准备访问数据库了。

假设你家里有几只宠物（你的动物园），你想要跟踪它们的各种信息。你可以通过创建表来保存你的数据，并加载所需的信息。然后，通过从表中检索数据，你可以回答关于你的动物的不同问题。本节将向你展示如何执行以下操作：

+   创建一个数据库

+   创建一个表

+   将数据加载到表中

+   以各种方式从表中检索数据

+   使用多个表

动物园数据库很简单（故意的），但很容易想象在现实世界中可能使用类似类型的数据库的情况。例如，这样的数据库可以被农民用来跟踪牲畜，或者被兽医用来跟踪患者记录。包含以下部分中使用的一些查询和示例数据的动物园分发可以从 MySQL 网站获取。它以压缩的**tar**文件和 Zip 格式提供在`dev.mysql.com/doc/`。

使用`SHOW`语句查找服务器上当前存在的数据库：

```sql
mysql> SHOW DATABASES;
+----------+
| Database |
+----------+
| mysql    |
| test     |
| tmp      |
+----------+
```

`mysql`数据库描述了用户访问权限。`test`数据库通常作为用户尝试事物的工作空间可用。

由该语句显示的数据库列表在您的机器上可能不同；如果您没有`SHOW DATABASES`权限，则`SHOW DATABASES`不会显示您没有权限的数据库。参见 Section 15.7.7.14, “SHOW DATABASES Statement”。

如果`test`数据库存在，尝试访问它：

```sql
mysql> USE test
Database changed
```

`USE`，像`QUIT`一样，不需要分号。（如果你愿意，可以用分号终止这样的语句；这并不会造成任何伤害。）`USE`语句在另一个方面也很特殊：它必须在一行上给出。

你可以在接下来的示例中使用`test`数据库（如果你有权限），但是任何你在该数据库中创建的内容都可以被其他人删除。因此，你应该向你的 MySQL 管理员请求使用自己的数据库的权限。假设你想要称之为`menagerie`。管理员需要执行类似这样的语句：

```sql
mysql> GRANT ALL ON menagerie.* TO 'your_mysql_name'@'your_client_host';
```

其中`your_mysql_name`是分配给您的 MySQL 用户名，`your_client_host`是您连接到服务器的主机。
