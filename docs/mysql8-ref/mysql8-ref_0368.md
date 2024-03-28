# 8.2.8 添加账户、分配权限和删除账户

> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-accounts.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-accounts.html)

要管理 MySQL 账户，请使用为此目的而设计的 SQL 语句：

+   `CREATE USER` 和 `DROP USER` 创建和移除账户。

+   `GRANT` 和 `REVOKE` 分配权限给账户并从账户中撤销权限。

+   `SHOW GRANTS` 显示账户权限分配。

账户管理语句会导致服务器对底层授权表进行适当的修改，这些表在第 8.2.3 节，“授权表”中讨论。

注意

不鼓励直接使用诸如`INSERT`、`UPDATE`或`DELETE`等语句直接修改授权表，并且自担风险。服务器可以忽略由于这些修改而变得畸形的行。

对于任何修改授权表的操作，服务器会检查表是否具有预期的结构，如果不是，则会产生错误。要将表更新为预期的结构，请执行 MySQL 升级过程。参见第三章，“升级 MySQL”。

创建账户的另一种选择是使用 GUI 工具 MySQL Workbench。此外，一些第三方程序提供了用于 MySQL 账户管理的功能。`phpMyAdmin` 就是这样一个程序。

本节讨论以下主题：

+   创建账户和授予权限

+   检查账户权限和属性

+   撤销账户权限

+   删除账户

有关本文讨论的语句的更多信息，请参阅第 15.7.1 节，“账户管理语句”。

#### 创建账户和授予权限

以下示例展示如何使用**mysql**客户端程序设置新账户。这些示例假定 MySQL `root` 账户具有`CREATE USER`权限以及授予其他账户的所有权限。

在命令行中，以 MySQL `root` 用户连接到服务器，并在密码提示符处输入适当的密码：

```sql
$> mysql -u root -p
Enter password: *(enter root password here)*
```

连接到服务器后，您可以添加新账户。以下示例使用 `CREATE USER` 和 `GRANT` 语句设置四个账户（在看到 `'*`password`*'` 时，请替换为适当的密码）：

```sql
CREATE USER 'finley'@'localhost'
  IDENTIFIED BY '*password*';
GRANT ALL
  ON *.*
  TO 'finley'@'localhost'
  WITH GRANT OPTION;

CREATE USER 'finley'@'%.example.com'
  IDENTIFIED BY '*password*';
GRANT ALL
  ON *.*
  TO 'finley'@'%.example.com'
  WITH GRANT OPTION;

CREATE USER 'admin'@'localhost'
  IDENTIFIED BY '*password*';
GRANT RELOAD,PROCESS
  ON *.*
  TO 'admin'@'localhost';

CREATE USER 'dummy'@'localhost';
```

由这些语句创建的账户具有以下属性：

+   有两个用户名为 `finley` 的账户。两者都是具有完全全局权限可以执行任何操作的超级用户账户。`'finley'@'localhost'` 账户仅可在从本地主机连接时使用。`'finley'@'%.example.com'` 账户在主机部分使用 `'%'` 通配符，因此可用于从 `example.com` 域中的任何主机连接。

    如果存在 `localhost` 的匿名用户账户，则 `'finley'@'localhost'` 账户是必需的。如果没有 `'finley'@'localhost'` 账户，则当 `finley` 从本地主机连接时，匿名用户账户优先，`finley` 将被视为匿名用户。原因是匿名用户账户的 `Host` 列值比 `'finley'@'%'` 账户更具体，因此在 `user` 表排序顺序中排在前面。（有关 `user` 表排序的信息，请参见 Section 8.2.6, “Access Control, Stage 1: Connection Verification”。）

+   `'admin'@'localhost'` 账户仅可由 `admin` 用于从本地主机连接。它被授予全局 `RELOAD` 和 `PROCESS` 管理权限。这些权限使 `admin` 用户能够执行 **mysqladmin reload**、**mysqladmin refresh** 和 **mysqladmin flush-*`xxx`*** 命令，以及 **mysqladmin processlist**。未授予访问任何数据库的权限。您可以使用 `GRANT` 语句添加此类权限。

+   `'dummy'@'localhost'` 账户没有密码（这是不安全且不推荐的）。此账户仅可用于从本地主机连接。未授予任何权限。假定您使用 `GRANT` 语句为该账户授予特定权限。

前面的示例授予了全局级别的权限。下一个示例创建了三个账户，并在较低级别授予了访问权限；即，对特定数据库或数据库中的对象。每个账户的用户名都是`custom`，但主机名部分不同：

```sql
CREATE USER 'custom'@'localhost'
  IDENTIFIED BY '*password*';
GRANT ALL
  ON bankaccount.*
  TO 'custom'@'localhost';

CREATE USER 'custom'@'host47.example.com'
  IDENTIFIED BY '*password*';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
  ON expenses.*
  TO 'custom'@'host47.example.com';

CREATE USER 'custom'@'%.example.com'
  IDENTIFIED BY '*password*';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
  ON customer.addresses
  TO 'custom'@'%.example.com';
```

这三个账户可以如下使用：

+   `'custom'@'localhost'`账户具有访问`bankaccount`数据库的所有数据库级别权限。该账户只能从本地主机连接到服务器。

+   `'custom'@'host47.example.com'`账户具有特定数据库级别权限，可以访问`expenses`数据库。该账户只能从主机`host47.example.com`连接到服务器。

+   `'custom'@'%.example.com'`账户具有特定表级别权限，可以访问`customer`数据库中的`addresses`表，来自`example.com`域中的任何主机。由于在账户名的主机部分使用了`%`通配符字符，该账户可以从该域中的所有机器连接到服务器。

#### 检查账户权限和属性

要查看账户的权限，请使用`SHOW GRANTS`：

```sql
mysql> SHOW GRANTS FOR 'admin'@'localhost';
+-----------------------------------------------------+
| Grants for admin@localhost                          |
+-----------------------------------------------------+
| GRANT RELOAD, PROCESS ON *.* TO `admin`@`localhost` |
+-----------------------------------------------------+
```

要查看账户的非权限属性，请使用`SHOW CREATE USER`：

```sql
mysql> SET print_identified_with_as_hex = ON;
mysql> SHOW CREATE USER 'admin'@'localhost'\G
*************************** 1\. row ***************************
CREATE USER for admin@localhost: CREATE USER `admin`@`localhost`
IDENTIFIED WITH 'caching_sha2_password'
AS 0x24412430303524301D0E17054E2241362B1419313C3E44326F294133734B30792F436E77764270373039612E32445250786D43594F45354532324B6169794F47457852796E32
REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK
PASSWORD HISTORY DEFAULT
PASSWORD REUSE INTERVAL DEFAULT
PASSWORD REQUIRE CURRENT DEFAULT
```

启用`print_identified_with_as_hex`系统变量（自 MySQL 8.0.17 起可用）会导致`SHOW CREATE USER`显示包含不可打印字符的哈希值为十六进制字符串，而不是常规字符串文字。

#### 撤销账户权限

要撤销账户权限，请使用`REVOKE`语句。权限可以在不同级别撤销，就像它们可以在不同级别授予一样。

撤销全局权限：

```sql
REVOKE ALL
  ON *.*
  FROM 'finley'@'%.example.com';

REVOKE RELOAD
  ON *.*
  FROM 'admin'@'localhost';
```

撤销数据库级别权限：

```sql
REVOKE CREATE,DROP
  ON expenses.*
  FROM 'custom'@'host47.example.com';
```

撤销表级别权限：

```sql
REVOKE INSERT,UPDATE,DELETE
  ON customer.addresses
  FROM 'custom'@'%.example.com';
```

要检查权限撤销的效果，请使用`SHOW GRANTS`：

```sql
mysql> SHOW GRANTS FOR 'admin'@'localhost';
+---------------------------------------------+
| Grants for admin@localhost                  |
+---------------------------------------------+
| GRANT PROCESS ON *.* TO `admin`@`localhost` |
+---------------------------------------------+
```

#### 删除账户

要移除一个账户，请使用`DROP USER`语句。例如，要删除之前创建的一些账户：

```sql
DROP USER 'finley'@'localhost';
DROP USER 'finley'@'%.example.com';
DROP USER 'admin'@'localhost';
DROP USER 'dummy'@'localhost';
```
