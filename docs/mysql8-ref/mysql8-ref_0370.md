# 8.2.10 使用角色

> 原文：[`dev.mysql.com/doc/refman/8.0/en/roles.html`](https://dev.mysql.com/doc/refman/8.0/en/roles.html)

MySQL 角色是一组命名的权限集合。与用户帐户一样，角色可以被授予和撤销权限。

用户帐户可以被授予角色，从而向帐户授予与每个角色相关联的权限。这使得可以将一组权限分配给帐户，并为所需的权限分配提供了一个方便的替代方法，用于概念化所需的权限分配并实施它们。

以下列表总结了 MySQL 提供的角色管理功能：

+   `CREATE ROLE` 和 `DROP ROLE` 创建和移除角色。

+   `GRANT` 和 `REVOKE` 分配权限以撤销用户帐户和角色的权限。

+   `SHOW GRANTS` 显示用户帐户和角色的权限和角色分配。

+   `SET DEFAULT ROLE` 指定默认情况下活动的帐户角色。

+   `SET ROLE` 更改当前会话中的活动角色。

+   `CURRENT_ROLE()` 函数显示当前会话中活动的角色。

+   `mandatory_roles` 和 `activate_all_roles_on_login` 系统变量允许定义强制角色和用户登录到服务器时自动激活授予的角色。

有关单个角色操作语句的描述（包括使用它们所需的权限），请参阅第 15.7.1 节，“帐户管理语句”。以下讨论提供了角色使用的示例。除非另有说明，此处显示的 SQL 语句应使用具有足够管理权限的 MySQL 帐户（如 `root` 帐户）执行。

+   创建角色并授予权限

+   定义强制角色

+   检查角色权限

+   激活角色

+   撤销角色或角色权限

+   删除角色

+   用户和角色的可互换性

#### 创建角色并授予权限

考虑以下情景：

+   一个应用程序使用名为`app_db`的数据库。

+   与应用程序关联，可以为创建和维护应用程序的开发人员以及与之交互的用户创建帐户。

+   开发人员需要对数据库拥有完全访问权限。一些用户只需要读取访问权限，其他用户需要读取/写入访问权限。

为了避免向可能有许多用户帐户单独授予权限，将角色创建为所需权限集的名称。这样可以轻松地向用户帐户授予所需的权限，方法是授予适当的角色。

要创建角色，请使用`CREATE ROLE`语句：

```sql
CREATE ROLE 'app_developer', 'app_read', 'app_write';
```

角色名称与用户帐户名称非常相似，由`'*`user_name`*'@'*`host_name`*'`格式中的用户部分和主机部分组成。如果省略主机部分，则默认为`'%'`。用户和主机部分可以不带引号，除非它们包含特殊字符，如`-`或`%`。与帐户名称不同，角色名称的用户部分不能为空。有关更多信息，请参见第 8.2.5 节“指定角色名称”。

要为角色分配权限，执行与为用户帐户分配权限相同的语法的`GRANT`语句：

```sql
GRANT ALL ON app_db.* TO 'app_developer';
GRANT SELECT ON app_db.* TO 'app_read';
GRANT INSERT, UPDATE, DELETE ON app_db.* TO 'app_write';
```

现在假设最初您需要一个开发人员帐户，两个需要只读访问权限的用户帐户和一个需要读取/写入访问权限的用户帐户。使用`CREATE USER`创建这些帐户：

```sql
CREATE USER 'dev1'@'localhost' IDENTIFIED BY 'dev1pass';
CREATE USER 'read_user1'@'localhost' IDENTIFIED BY 'read_user1pass';
CREATE USER 'read_user2'@'localhost' IDENTIFIED BY 'read_user2pass';
CREATE USER 'rw_user1'@'localhost' IDENTIFIED BY 'rw_user1pass';
```

为每个用户帐户分配所需的权限，您可以使用与刚刚显示的相同形式的`GRANT`语句，但这需要为每个用户枚举单独的权限。相反，使用另一种允许授予角色而不是权限的`GRANT`语法：

```sql
GRANT 'app_developer' TO 'dev1'@'localhost';
GRANT 'app_read' TO 'read_user1'@'localhost', 'read_user2'@'localhost';
GRANT 'app_read', 'app_write' TO 'rw_user1'@'localhost';
```

对于`rw_user1`帐户的`GRANT`语句授予读取和写入角色，这些角色结合起来提供所需的读取和写入权限。

将角色授予帐户的`GRANT`语法与授予权限的语法不同：有一个`ON`子句用于分配权限，而没有`ON`子句用于分配角色。因为语法不同，您不能在同一语句中混合分配权限和角色。（允许向帐户分配权限和角色，但必须使用适用于要授予的内容的语法的单独的`GRANT`语句。）截至 MySQL 8.0.16，无法向匿名用户授予角色。

创建角色时，角色被锁定，没有密码，并分配默认的身份验证插件。（这些角色属性可以稍后由具有全局`CREATE USER`权限的用户使用`ALTER USER`语句更改。）

当角色被锁定时，无法用于服务器身份验证。如果解锁，则可以用于身份验证。这是因为角色和用户都是授权标识符，有很多共同之处，很少有区别。另请参阅用户和角色的可互换性。

#### 定义强制角色

可以通过在`mandatory_roles`系统变量的值中命名角色来指定角色为强制角色。服务器将强制角色视为授予所有用户的角色，因此不需要显式授予任何帐户。

要在服务器启动时指定强制角色，请在服务器的`my.cnf`文件中定义`mandatory_roles`：

```sql
[mysqld]
mandatory_roles='role1,role2@localhost,r3@%.example.com'
```

要在运行时设置和持久化`mandatory_roles`，请使用以下语句：

```sql
SET PERSIST mandatory_roles = 'role1,role2@localhost,r3@%.example.com';
```

`SET PERSIST`为运行中的 MySQL 实例设置一个值。它还保存该值，导致其在后续服务器重启时保留。要更改运行中的 MySQL 实例的值，而不使其在后续重启时保留，使用`GLOBAL`关键字而不是`PERSIST`。请参阅第 15.7.6.1 节，“变量赋值的 SET 语法”。

设置`mandatory_roles`需要`ROLE_ADMIN`权限，除了通常需要设置全局系统变量的`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。

强制角色，就像显式授予的角色一样，在激活之前不会生效（请参阅激活角色）。在登录时，如果启用了`activate_all_roles_on_login`系统变量，则为所有授予的角色激活，否则为设置为默认角色的角色激活。在运行时，`SET ROLE`激活角色。

在`mandatory_roles`值中命名的角色不能通过`REVOKE`或`DROP ROLE`或`DROP USER`来撤销。

为防止会话默认成为系统会话，具有`SYSTEM_USER`权限的角色不能列在`mandatory_roles`系统变量的值中：

+   如果在启动时将`mandatory_roles`分配给具有`SYSTEM_USER`权限的角色，则服务器会向错误日志写入消息并退出。

+   如果在运行时将`mandatory_roles`分配给具有`SYSTEM_USER`权限的角色，则会发生错误，并且`mandatory_roles`值保持不变。

即使有此保障，最好避免通过角色授予`SYSTEM_USER`权限，以防止权限升级的可能性。

如果在`mysql.user`系统表中不存在`mandatory_roles`中命名的角色，则不会将该角色授予用户。当服务器尝试为用户激活角色时，它不会将不存在的角色视为强制角色，并将警告写入错误日志。如果稍后创建了角色并因此变为有效，则可能需要使用`FLUSH PRIVILEGES`使服务器将其视为强制角色。

`SHOW GRANTS` 根据第 15.7.7.21 节，“SHOW GRANTS Statement”中描述的规则显示强制角色。

#### 检查角色权限

要验证分配给账户的权限，请使用`SHOW GRANTS`。例如：

```sql
mysql> SHOW GRANTS FOR 'dev1'@'localhost';
+-------------------------------------------------+
| Grants for dev1@localhost                       |
+-------------------------------------------------+
| GRANT USAGE ON *.* TO `dev1`@`localhost`        |
| GRANT `app_developer`@`%` TO `dev1`@`localhost` |
+-------------------------------------------------+
```

然而，这显示了每个授予的角色，而没有将其“展开”为角色代表的权限。要显示角色权限，还需添加一个`USING`子句，命名要显示权限的授予角色：

```sql
mysql> SHOW GRANTS FOR 'dev1'@'localhost' USING 'app_developer';
+----------------------------------------------------------+
| Grants for dev1@localhost                                |
+----------------------------------------------------------+
| GRANT USAGE ON *.* TO `dev1`@`localhost`                 |
| GRANT ALL PRIVILEGES ON `app_db`.* TO `dev1`@`localhost` |
| GRANT `app_developer`@`%` TO `dev1`@`localhost`          |
+----------------------------------------------------------+
```

类似地验证每种类型的用户：

```sql
mysql> SHOW GRANTS FOR 'read_user1'@'localhost' USING 'app_read';
+--------------------------------------------------------+
| Grants for read_user1@localhost                        |
+--------------------------------------------------------+
| GRANT USAGE ON *.* TO `read_user1`@`localhost`         |
| GRANT SELECT ON `app_db`.* TO `read_user1`@`localhost` |
| GRANT `app_read`@`%` TO `read_user1`@`localhost`       |
+--------------------------------------------------------+
mysql> SHOW GRANTS FOR 'rw_user1'@'localhost' USING 'app_read', 'app_write';
+------------------------------------------------------------------------------+
| Grants for rw_user1@localhost                                                |
+------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `rw_user1`@`localhost`                                 |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `app_db`.* TO `rw_user1`@`localhost` |
| GRANT `app_read`@`%`,`app_write`@`%` TO `rw_user1`@`localhost`               |
+------------------------------------------------------------------------------+
```

`SHOW GRANTS` 根据第 15.7.7.21 节，“SHOW GRANTS Statement”中描述的规则显示强制角色。

#### 激活角色

授予用户账户的角色可以在账户会话中处于活动或非活动状态。如果授予的角色在会话中处于活动状态，则其权限生效；否则，不生效。要确定当前会话中哪些角色处于活动状态，请使用`CURRENT_ROLE()`函数。

默认情况下，将角色授予给一个账户或在`mandatory_roles`系统变量值中命名它不会自动导致角色在账户会话中变为活动状态。例如，因为在前面的讨论中到目前为止没有激活任何`rw_user1`角色，如果您以`rw_user1`身份连接到服务器并调用`CURRENT_ROLE()`函数，结果是`NONE`（没有活动角色）：

```sql
mysql> SELECT CURRENT_ROLE();
+----------------+
| CURRENT_ROLE() |
+----------------+
| NONE           |
+----------------+
```

要指定每次用户连接到服务器并进行身份验证时应激活哪些角色，请使用`SET DEFAULT ROLE`。要将默认设置为早期创建的每个账户的所有分配角色，请使用此语句：

```sql
SET DEFAULT ROLE ALL TO
  'dev1'@'localhost',
  'read_user1'@'localhost',
  'read_user2'@'localhost',
  'rw_user1'@'localhost';
```

现在，如果您以`rw_user1`身份连接，`CURRENT_ROLE()`的初始值反映了新的默认角色分配：

```sql
mysql> SELECT CURRENT_ROLE();
+--------------------------------+
| CURRENT_ROLE()                 |
+--------------------------------+
| `app_read`@`%`,`app_write`@`%` |
+--------------------------------+
```

要在用户连接到服务器时自动激活所有明确授予和强制性角色，请启用`activate_all_roles_on_login`系统变量。默认情况下，自动角色激活被禁用。

在一个会话中，用户可以执行`SET ROLE`来改变活动角色集。例如，对于`rw_user1`：

```sql
mysql> SET ROLE NONE; SELECT CURRENT_ROLE();
+----------------+
| CURRENT_ROLE() |
+----------------+
| NONE           |
+----------------+
mysql> SET ROLE ALL EXCEPT 'app_write'; SELECT CURRENT_ROLE();
+----------------+
| CURRENT_ROLE() |
+----------------+
| `app_read`@`%` |
+----------------+
mysql> SET ROLE DEFAULT; SELECT CURRENT_ROLE();
+--------------------------------+
| CURRENT_ROLE()                 |
+--------------------------------+
| `app_read`@`%`,`app_write`@`%` |
+--------------------------------+
```

第一个`SET ROLE`语句取消所有角色。第二个使`rw_user1`有效地只读。第三个恢复默认角色。

存储程序和视图对象的有效用户受`DEFINER`和`SQL SECURITY`属性的影响，这些属性确定执行是在调用者还是定义者上下文中发生（参见第 27.6 节，“存储对象访问控制”）：

+   在调用者上下文中执行的存储程序和视图对象将使用当前会话中处于活动状态的角色执行。

+   在定义者上下文中执行的存储程序和视图对象将使用其`DEFINER`属性中命名的用户的默认角色执行。如果启用了`activate_all_roles_on_login`，这样的对象将使用授予`DEFINER`用户的所有角色，包括强制性角色。对于存储程序，如果执行应该使用与默认不同的角色，则程序体可以执行`SET ROLE`来激活所需的角色。这必须谨慎进行，因为分配给角色的权限可以更改。

#### 撤销角色或角色权限

就像角色可以授予给一个账户一样，它们也可以从一个账户中撤销：

```sql
REVOKE *role* FROM *user*;
```

在`mandatory_roles`系统变量值中命名的角色不能被撤销。

`REVOKE`也可以应用于角色以修改授予其的权限。这不仅影响角色本身，还影响分配了该角色的任何帐户。假设您想暂时使所有应用程序用户只读。为此，请使用`REVOKE`来从`app_write`角色中撤销修改权限：

```sql
REVOKE INSERT, UPDATE, DELETE ON app_db.* FROM 'app_write';
```

正如所发生的那样，这将使角色完全没有任何权限，可以使用`SHOW GRANTS`来查看（这表明此语句可以与角色一起使用，而不仅仅是用户）：

```sql
mysql> SHOW GRANTS FOR 'app_write';
+---------------------------------------+
| Grants for app_write@%                |
+---------------------------------------+
| GRANT USAGE ON *.* TO `app_write`@`%` |
+---------------------------------------+
```

因为从角色中撤销权限会影响分配了修改后角色的任何用户的权限，`rw_user1`现在没有表修改权限（`INSERT`、`UPDATE`和`DELETE`不再存在）：

```sql
mysql> SHOW GRANTS FOR 'rw_user1'@'localhost'
       USING 'app_read', 'app_write';
+----------------------------------------------------------------+
| Grants for rw_user1@localhost                                  |
+----------------------------------------------------------------+
| GRANT USAGE ON *.* TO `rw_user1`@`localhost`                   |
| GRANT SELECT ON `app_db`.* TO `rw_user1`@`localhost`           |
| GRANT `app_read`@`%`,`app_write`@`%` TO `rw_user1`@`localhost` |
+----------------------------------------------------------------+
```

实际上，`rw_user1`读/写用户已成为只读用户。对于被授予`app_write`角色的任何其他帐户也是如此，说明使用角色使得不必为单个帐户修改权限。

要恢复角色的修改权限，只需重新授予它们：

```sql
GRANT INSERT, UPDATE, DELETE ON app_db.* TO 'app_write';
```

现在`rw_user1`再次具有修改权限，任何其他被授予`app_write`角色的帐户也是如此。

#### 删除角色

要删除角色，请使用`DROP ROLE`：

```sql
DROP ROLE 'app_read', 'app_write';
```

删除一个角色会从授予它的每个帐户中撤销该角色。

在`mandatory_roles`系统变量值中命名的角色不能被删除。

#### 用户和角色的互换性

正如早些时候暗示的那样，对于`SHOW GRANTS`，它显示用户帐户或角色的授权，帐户和角色可以互换使用。

角色和用户之间的一个区别是，`CREATE ROLE`默认创建一个被锁定的授权标识符，而`CREATE USER`默认创建一个未锁定的授权标识符。您应该记住这个区别不是不可改变的；具有适当权限的用户可以在创建后锁定或解锁角色或（其他）用户。

如果数据库管理员有一个偏好，即特定的授权标识符必须是一个角色，那么可以使用命名方案来传达这一意图。例如，您可以为所有您打算作为角色而不是其他内容的授权标识符使用`r_`前缀。

角色和用户之间的另一个区别在于用于管理它们的权限的可用性：

+   `CREATE ROLE` 和 `DROP ROLE` 权限仅允许使用 `CREATE ROLE` 和 `DROP ROLE` 语句。

+   `CREATE USER` 权限允许使用 `ALTER USER`、`CREATE ROLE`、`CREATE USER`、`DROP ROLE`、`DROP USER`、`RENAME USER` 和 `REVOKE ALL PRIVILEGES` 语句。

因此，`CREATE ROLE` 和 `DROP ROLE` 权限不如 `CREATE USER` 强大，可能授予那些只允许创建和删除角色而不执行更一般帐户操作的用户。

关于权限和用户与角色的可互换性，您可以将用户帐户视为角色并将该帐户授予另一个用户或角色。效果是将该帐户的权限和角色授予另一个用户或角色。

这组语句表明您可以将用户授予用户，将角色授予用户，将用户授予角色，或将角色授予角色：

```sql
CREATE USER 'u1';
CREATE ROLE 'r1';
GRANT SELECT ON db1.* TO 'u1';
GRANT SELECT ON db2.* TO 'r1';
CREATE USER 'u2';
CREATE ROLE 'r2';
GRANT 'u1', 'r1' TO 'u2';
GRANT 'u1', 'r1' TO 'r2';
```

每种情况的结果是将授予对象的权限与授予对象相关联的权限授予给受让对象。执行这些语句后，`u2` 和 `r2` 分别从用户 (`u1`) 和角色 (`r1`) 获得了权限：

```sql
mysql> SHOW GRANTS FOR 'u2' USING 'u1', 'r1';
+-------------------------------------+
| Grants for u2@%                     |
+-------------------------------------+
| GRANT USAGE ON *.* TO `u2`@`%`      |
| GRANT SELECT ON `db1`.* TO `u2`@`%` |
| GRANT SELECT ON `db2`.* TO `u2`@`%` |
| GRANT `u1`@`%`,`r1`@`%` TO `u2`@`%` |
+-------------------------------------+
mysql> SHOW GRANTS FOR 'r2' USING 'u1', 'r1';
+-------------------------------------+
| Grants for r2@%                     |
+-------------------------------------+
| GRANT USAGE ON *.* TO `r2`@`%`      |
| GRANT SELECT ON `db1`.* TO `r2`@`%` |
| GRANT SELECT ON `db2`.* TO `r2`@`%` |
| GRANT `u1`@`%`,`r1`@`%` TO `r2`@`%` |
+-------------------------------------+
```

前面的例子仅供参考，但用户帐户和角色的可互换性具有实际应用，例如在以下情况下：假设一个传统的应用开发项目在 MySQL 的角色出现之前就开始了，因此与该项目相关的所有用户帐户都直接被授予权限（而不是通过授予角色而获得权限）。其中一个帐户是最初被授予权限的开发人员帐户如下：

```sql
CREATE USER 'old_app_dev'@'localhost' IDENTIFIED BY 'old_app_devpass';
GRANT ALL ON old_app.* TO 'old_app_dev'@'localhost';
```

如果该开发人员离开项目，则有必要将权限分配给另一个用户，或者如果开发活动已扩展，则可能需要分配给多个用户。以下是处理此问题的一些方法：

+   不使用角色：更改帐户密码，以防止原始开发人员使用它，并让新开发人员使用该帐户：

    ```sql
    ALTER USER 'old_app_dev'@'localhost' IDENTIFIED BY '*new_password*';
    ```

+   使用角色：锁定帐户以防止任何人使用它连接到服务器：

    ```sql
    ALTER USER 'old_app_dev'@'localhost' ACCOUNT LOCK;
    ```

    然后将该帐户视为一个角色。对于项目中的每个新开发人员，创建一个新帐户并授予其原始开发人员帐户：

    ```sql
    CREATE USER 'new_app_dev1'@'localhost' IDENTIFIED BY '*new_password*';
    GRANT 'old_app_dev'@'localhost' TO 'new_app_dev1'@'localhost';
    ```

    该效果是将原始开发者账户的权限分配给新账户。
