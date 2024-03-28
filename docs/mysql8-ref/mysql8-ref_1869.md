# 27.2.2 存储例程和 MySQL 权限

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-routines-privileges.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-routines-privileges.html)

MySQL 授权系统如下考虑存储例程：

+   需要`CREATE ROUTINE` 权限来创建存储例程。

+   需要`ALTER ROUTINE` 权限来修改或删除存储例程。如果需要，此权限将自动授予例程的创建者，并在删除例程时从创建者那里删除。

+   执行存储例程需要`EXECUTE` 权限。但是，如果需要，此权限将自动授予例程的创建者（在删除例程时从创建者那里删除）。此外，例程的默认`SQL SECURITY`特性为`DEFINER`，这使得具有与例程关联的数据库访问权限的用户可以执行该例程。

+   如果`automatic_sp_privileges` 系统变量为 0，则不会自动授予和删除例程创建者的`EXECUTE` 和 `ALTER ROUTINE` 权限。

+   例程的创建者是用于执行其`CREATE`语句的帐户。这可能与例程定义中命名为`DEFINER`的帐户不同。

+   例程`DEFINER`命名的帐户可以查看所有例程属性，包括其定义。因此，该帐户完全可以访问由以下产生的例程输出：

    +   信息模式`ROUTINES` 表的内容。

    +   `SHOW CREATE FUNCTION` 和 `SHOW CREATE PROCEDURE` 语句。

    +   `SHOW FUNCTION CODE` 和 `SHOW PROCEDURE CODE` 语句。

    +   `SHOW FUNCTION STATUS` 和 `SHOW PROCEDURE STATUS` 语句。

+   对于非例程`DEFINER`命名的帐户，访问例程属性取决于授予帐户的权限：

    +   使用`SHOW_ROUTINE` 权限或全局`SELECT` 权限，帐户可以查看所有例程属性，包括其定义。

    +   通过在包含例程的范围内授予`CREATE ROUTINE`、`ALTER ROUTINE`或`EXECUTE`权限，帐户可以查看所有例程属性，除了其定义。
