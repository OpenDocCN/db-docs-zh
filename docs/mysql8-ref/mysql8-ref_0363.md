# 8.2.3 授权表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/grant-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html)

`mysql`系统数据库包含几个授权表，其中包含有关用户账户和其持有的权限的信息。本节描述了这些表。有关系统数据库中其他表的信息，请参阅第 7.3 节，“mysql 系统模式”。

此处讨论了授权表的基本结构以及服务器在与客户端交互时如何使用其内容。然而，通常情况下不直接修改授权表。当您使用诸如`CREATE USER`, `GRANT`, 和 `REVOKE`等账户管理语句设置账户和控制每个账户可用权限时，修改是间接发生的。请参阅第 15.7.1 节，“账户管理语句”。当您使用这些语句执行账户操作时，服务器会代表您修改授权表。

注意

不鼓励直接使用诸如`INSERT`, `UPDATE`, 或 `DELETE`等语句直接修改授权表，这样做风险自负。服务器可以忽略由于这些修改导致的格式错误的行。

对于任何修改授权表的操作，服务器会检查表是否具有预期的结构，如果不是，则会产生错误。要将表更新为预期的结构，请执行 MySQL 升级过程。请参阅第三章，“*升级 MySQL*”。

+   授权表概述

+   用户和数据库授权表

+   tables_priv 和 columns_priv 授权表

+   procs_priv 授权表

+   proxies_priv 授权表

+   全局授权表

+   默认角色授权表

+   角色边缘授权表

+   密码历史授权表

+   授予表范围列属性

+   授予表权限列属性

+   授予表并发性

#### 授予表概述

这些`mysql`数据库表包含授予信息：

+   `user`: 用户账户，静态全局权限和其他非权限列。

+   `global_grants`: 动态全局权限。

+   `db`: 数据库级权限。

+   `tables_priv`: 表级权限。

+   `columns_priv`: 列级权限。

+   `procs_priv`: 存储过程和函数权限。

+   `proxies_priv`: 代理用户权限。

+   `default_roles`: 默认用户角色。

+   `role_edges`: 角色子图的边缘。

+   `password_history`: 密码更改历史。

有关静态和动态全局权限之间的区别，请参阅静态与动态权限。

在 MySQL 8.0 中，授予表使用`InnoDB`存储引擎并且是事务性的。在 MySQL 8.0 之前，授予表使用`MyISAM`存储引擎并且是非事务性的。授予表存储引擎的更改使得伴随的账户管理语句的行为也发生了变化，例如`CREATE USER`或`GRANT`。以前，命名多个用户的账户管理语句可能对某些用户成功，对其他用户失败。现在，每个语句都是事务性的，要么对所有命名用户成功，要么回滚并且如果出现任何错误则不起作用。

每个授予表包含范围列和权限列：

+   范围列确定表中每行的范围；也就是说，行适用的上下文。例如，具有`Host`和`User`值为`'h1.example.net'`和`'bob'`的`user`表行适用于由指定用户名称为`bob`的客户端从主机`h1.example.net`进行的身份验证连接。类似地，具有`Host`，`User`和`Db`列值为`'h1.example.net'`，`'bob'`和`'reports'`的`db`表行适用于`bob`从主机`h1.example.net`连接以访问`reports`数据库。`tables_priv`和`columns_priv`表包含指示每行适用于哪些表或表/列组合的范围列。`procs_priv`范围列指示每行适用于的存储过程。

+   特权列指示表行授予哪些特权；也就是说，它允许执行哪些操作。服务器将各种授权表中的信息组合起来，形成用户特权的完整描述。第 8.2.7 节，“访问控制，第 2 阶段：请求验证”描述了这方面的规则。

此外，授权表可能包含用于除范围或特权评估之外的其他目的的列。

服务器使用授权表的方式如下：

+   `user`表的范围列确定是否拒绝或允许传入连接。对于允许的连接，在`user`表中授予的任何特权表示用户的静态全局特权。在此表中授予的任何特权都适用于*所有*服务器上的数据库。

    注意

    因为任何静态全局特权都被视为所有数据库的特权，任何静态全局特权都使用户能够通过`SHOW DATABASES`或通过检查`INFORMATION_SCHEMA`的`SCHEMATA`表来查看所有数据库名称，除了在数据库级别通过部分撤销限制的数据库。

+   `global_grants`表列出了对用户帐户分配的动态全局特权。对于每行，范围列确定具有特权列中命名的特权的用户。

+   `db`表的范围列确定哪些用户可以从哪些主机访问哪些数据库。特权列确定允许的操作。在数据库级别授予的特权适用于数据库及其所有对象，如表和存储程序。

+   `tables_priv`和`columns_priv`表类似于`db`表，但更加精细：它们适用于表和列级别，而不是数据库级别。在表级别授予的特权适用于表及其所有列。在列级别授予的特权仅适用于特定列。

+   `procs_priv` 表适用于存储例程（存储过程和函数）。在例程级别授予的权限仅适用于单个过程或函数。

+   `proxies_priv` 表指示哪些用户可以代表其他用户行事，以及用户是否可以向其他用户授予 `PROXY` 权限。

+   `default_roles` 和 `role_edges` 表包含有关角色关系的信息。

+   `password_history` 表保留先前选择的密码，以启用对密码重用的限制。请参阅 第 8.2.15 节，“密码管理”。

服务器在启动时将授予表的内容读入内存。您可以通过发出 `FLUSH PRIVILEGES` 语句或执行 **mysqladmin flush-privileges** 或 **mysqladmin reload** 命令来告诉它重新加载表。对授权表的更改将按照 第 8.2.13 节，“权限更改生效时间” 中指示的方式生效。

当您修改帐户时，最好验证您的更改是否产生预期效果。要检查给定帐户的权限，请使用 `SHOW GRANTS` 语句。例如，要确定授予具有用户名和主机名值为 `bob` 和 `pc84.example.com` 的帐户的权限，请使用此语句：

```sql
SHOW GRANTS FOR 'bob'@'pc84.example.com';
```

要显示帐户的非权限属性，请使用 `SHOW CREATE USER`：

```sql
SHOW CREATE USER 'bob'@'pc84.example.com';
```

#### 用户和数据库授权表

服务器在访问控制的第一和第二阶段都使用 `mysql` 数据库中的 `user` 和 `db` 表（参见 第 8.2 节，“访问控制和账户管理”）。这里显示了 `user` 和 `db` 表中的列。

**表 8.4 user 和 db 表列**

| 表名 | `user` | `db` |
| --- | --- | --- |
| **范围列** | `Host` | `Host` |
|  | `User` | `Db` |
|  |  | `User` |
| **权限列** | `Select_priv` | `Select_priv` |
|  | `Insert_priv` | `Insert_priv` |
|  | `Update_priv` | `Update_priv` |
|  | `Delete_priv` | `Delete_priv` |
|  | `Index_priv` | `Index_priv` |
|  | `Alter_priv` | `Alter_priv` |
|  | `Create_priv` | `Create_priv` |
|  | `Drop_priv` | `Drop_priv` |
|  | `Grant_priv` | `Grant_priv` |
|  | `Create_view_priv` | `Create_view_priv` |
|  | `Show_view_priv` | `Show_view_priv` |
|  | `Create_routine_priv` | `Create_routine_priv` |
|  | `Alter_routine_priv` | `Alter_routine_priv` |
|  | `Execute_priv` | `Execute_priv` |
|  | `Trigger_priv` | `Trigger_priv` |
|  | `Event_priv` | `Event_priv` |
|  | `Create_tmp_table_priv` | `Create_tmp_table_priv` |
|  | `Lock_tables_priv` | `Lock_tables_priv` |
|  | `References_priv` | `References_priv` |
|  | `Reload_priv` |  |
|  | `Shutdown_priv` |  |
|  | `Process_priv` |  |
|  | `File_priv` |  |
|  | `Show_db_priv` |  |
|  | `Super_priv` |  |
|  | `Repl_slave_priv` |  |
|  | `Repl_client_priv` |  |
|  | `Create_user_priv` |  |
|  | `Create_tablespace_priv` |  |
|  | `Create_role_priv` |  |
|  | `Drop_role_priv` |  |
| **安全列** | `ssl_type` |  |
|  | `ssl_cipher` |  |
|  | `x509_issuer` |  |
|  | `x509_subject` |  |
|  | `plugin` |  |
|  | `authentication_string` |  |
|  | `password_expired` |  |
|  | `password_last_changed` |  |
|  | `password_lifetime` |  |
|  | `account_locked` |  |
|  | `Password_reuse_history` |  |
|  | `Password_reuse_time` |  |
|  | `Password_require_current` |  |
|  | `User_attributes` |  |
| **资源控制列** | `max_questions` |  |
|  | `max_updates` |  |
|  | `max_connections` |  |
|  | `max_user_connections` |  |
| 表名 | `user` | `db` |

`user`表的`plugin`和`authentication_string`列存储身份验证插件和凭据信息。

服务器使用帐户行的`plugin`列中命名的插件来验证帐户的连接尝试。

`plugin`列必须非空。在启动时，以及在执行`FLUSH PRIVILEGES`时，服务器会检查`user`表行。对于任何`plugin`列为空的行，服务器会向错误日志写入以下警告：

```sql
[Warning] User entry '*user_name*'@'*host_name*' has an empty plugin
value. The user will be ignored and no one can login with this user
anymore.
```

要为缺少插件的帐户分配插件，请使用`ALTER USER`语句。

`password_expired`列允许 DBA 过期帐户密码并要求用户重置密码。默认的`password_expired`值为`'N'`，但可以使用`ALTER USER`语句设置为`'Y'`。帐户密码过期后，在后续连接到服务器时，帐户执行的所有操作都会导致错误，直到用户发出`ALTER USER`语句以建立新的帐户密码。

注意

尽管可以通过将过期密码设置为当前值来“重置”过期密码，但最好根据良好的政策选择不同的密码。DBA 可以通过建立适当的密码重用策略来强制不重用。请参阅密码重用策略。

`password_last_changed`是一个`TIMESTAMP`列，指示密码上次更改的时间。该值仅对使用 MySQL 内置身份验证插件（`mysql_native_password`、`sha256_password`或`caching_sha2_password`）的帐户为非`NULL`。对于其他帐户，如使用外部身份验证系统进行身份验证的帐户，该值为`NULL`。

`password_last_changed` 由 `CREATE USER`、`ALTER USER` 和 `SET PASSWORD` 语句以及创建帐户或更改帐户密码的 `GRANT` 语句更新。

`password_lifetime` 指示帐户密码的生命周期，以天为单位。如果密码已超过其生命周期（使用 `password_last_changed` 列进行评估），当客户端使用该帐户连接时，服务器将视密码为已过期。大于零的 *`N`* 值表示密码必须每 *`N`* 天更改一次。值为 0 禁用自动密码过期。如果值为 `NULL`（默认值），则全局过期策略适用，由 `default_password_lifetime` 系统变量定义。

`account_locked` 指示帐户是否被锁定（参见第 8.2.20 节，“帐户锁定”）。

`Password_reuse_history` 是帐户的 `PASSWORD HISTORY` 选项的值，或者对于默认历史记录为 `NULL`。

`Password_reuse_time` 是帐户的 `PASSWORD REUSE INTERVAL` 选项的值，或者对于默认间隔为 `NULL`。

`Password_require_current`（MySQL 8.0.13 中新增）对应于帐户的 `PASSWORD REQUIRE` 选项的值，如下表所示。

**表 8.5 允许的 Password_require_current 值**

| Password_require_current 值 | 对应的 PASSWORD REQUIRE 选项 |
| --- | --- |
| `'Y'` | `PASSWORD REQUIRE CURRENT` |
| `'N'` | `PASSWORD REQUIRE CURRENT OPTIONAL` |
| `NULL` | `PASSWORD REQUIRE CURRENT DEFAULT` |

`User_attributes`（MySQL 8.0.14 中新增）是一个以 JSON 格式存储帐户属性的列，这些属性未存储在其他列中。截至 MySQL 8.0.21，`INFORMATION_SCHEMA` 通过 `USER_ATTRIBUTES` 表公开这些属性。

`User_attributes` 列可能包含这些属性：

+   `additional_password`：次要密码，如果有的话。请参阅双密码支持。

+   `Restrictions`：限制列表，如果有的话。限制是通过部分撤销操作添加的。属性值是一个元素数组，每个元素都有 `Database` 和 `Restrictions` 键，指示受限数据库的名称和适用于其上的限制（参见第 8.2.12 节，“使用部分撤销进行权限限制”）。

+   `Password_locking`: 如果有的话，是关于失败登录跟踪和临时账户锁定的条件（参见失败登录跟踪和临时账户锁定）。`Password_locking`属性根据`CREATE USER`和`ALTER USER`语句的`FAILED_LOGIN_ATTEMPTS`和`PASSWORD_LOCK_TIME`选项进行更新。该属性值是一个哈希，其中包含`failed_login_attempts`和`password_lock_time_days`键，指示为该账户指定的选项值。如果某个键缺失，则其值隐式为 0。如果键值隐式或显式为 0，则相应的功能被禁用。此属性在 MySQL 8.0.19 中添加。

+   `multi_factor_authentication`: `mysql.user`系统表中的行具有一个`plugin`列，指示认证插件。对于单因素认证，该插件是唯一的认证因素。对于多因素认证的两因素或三因素形式，该插件对应于第一个认证因素，但必须为第二和第三因素存储额外信息。`multi_factor_authentication`属性保存了这些信息。此属性在 MySQL 8.0.27 中添加。

    `multi_factor_authentication`值是一个数组，其中每个数组元素都是一个哈希，描述了使用以下属性描述的认证因素：

    +   `plugin`: 认证插件的名称。

    +   `authentication_string`: 认证字符串的值。

    +   `passwordless`: 一个标志，表示用户是否可以在没有密码的情况下使用（仅使用安全令牌作为唯一的认证方法）。

    +   `requires_registration`: 一个定义用户账户是否已注册安全令牌的标志。

    第一个和第二个数组元素描述了多因素认证因素 2 和 3。

如果没有属性适用，则`User_attributes`为`NULL`。

例子：一个具有次要密码和部分撤销数据库权限的账户在列值中具有`additional_password`和`Restrictions`属性：

```sql
mysql> SELECT User_attributes FROM mysql.User WHERE User = 'u'\G
*************************** 1\. row ***************************
User_attributes: {"Restrictions":
                   [{"Database": "mysql", "Privileges": ["SELECT"]}],
                  "additional_password": "*hashed_credentials*"}
```

要确定存在哪些属性，请使用`JSON_KEYS()`函数：

```sql
SELECT User, Host, JSON_KEYS(User_attributes)
FROM mysql.user WHERE User_attributes IS NOT NULL;
```

要提取特定属性，例如`Restrictions`，请执行以下操作：

```sql
SELECT User, Host, User_attributes->>'$.Restrictions'
FROM mysql.user WHERE User_attributes->>'$.Restrictions' <> '';
```

这是存储在`multi_factor_authentication`中的信息示例：

```sql
{
  "multi_factor_authentication": [
    {
      "plugin": "authentication_ldap_simple",
      "passwordless": 0,
      "authentication_string": "ldap auth string",
      "requires_registration": 0
    },
    {
      "plugin": "authentication_fido",
      "passwordless": 0,
      "authentication_string": "",
      "requires_registration": 1
    }
  ]
}
```

#### tables_priv 和 columns_priv 授权表

在访问控制的第二阶段，服务器执行请求验证，以确保每个客户端对其发出的每个请求都具有足够的权限。除了`user`和`db`授权表外，服务器还可能在涉及表的请求中查阅`tables_priv`和`columns_priv`表。后者在表和列级别提供更精细的权限控制。它们具有以下表中显示的列。

**表 8.6 tables_priv 和 columns_priv 表列**

| 表名 | `tables_priv` | `columns_priv` |
| --- | --- | --- |
| **范围列** | `主机` | `主机` |
|  | `数据库` | `数据库` |
|  | `用户` | `用户` |
|  | `表名` | `表名` |
|  |  | `列名` |
| **权限列** | `表权限` | `列权限` |
|  | `列权限` |  |
| **其他列** | `时间戳` | `时间戳` |
|  | `授权者` |  |

`时间戳` 和 `授权者` 列分别设置为当前时间戳和`CURRENT_USER`值，但其他情况下未使用。

#### procs_priv 授权表

为了验证涉及存储例程的请求，服务器可能会查阅`procs_priv`表，该表具有以下表中显示的列。

**表 8.7 procs_priv 表列**

| 表名 | `procs_priv` |
| --- | --- |
| **范围列** | `主机` |
|  | `数据库` |
|  | `用户` |
|  | `例程名` |
|  | `例程类型` |
| **权限列** | `Proc 权限` |
| **其他列** | `时间戳` |
|  | `授权者` |

`例程类型` 列是一个具有值 `'FUNCTION'` 或 `'PROCEDURE'` 的`ENUM`列，用于指示行所指的例程类型。该列使得可以分别为具有相同名称的函数和过程授予权限。

`时间戳` 和 `授权者` 列未使用。

#### proxies_priv 授权表

`proxies_priv` 表记录有关代理账户的信息。它具有以下列：

+   `主机`，`用户`：代理账户；即具有被代理账户的`PROXY`权限的账户。

+   `被代理主机`，`被代理用户`：被代理账户。

+   `授权者`，`时间戳`：未使用。

+   `With_grant`：代理账户是否可以将`PROXY`权限授予其他账户。

要使账户能够向其他账户授予`PROXY`特权，必须在`proxies_priv`表中具有一行，其中`With_grant`设置为 1，`Proxied_host`和`Proxied_user`设置为指示可以授予特权的账户或账户。例如，在 MySQL 安装期间创建的`'root'@'localhost'`账户在`proxies_priv`表中有一行，该行允许为`''@''`，即所有用户和所有主机，授予`PROXY`特权。这使得`root`能够设置代理用户，以及委派给其他账户权限设置代理用户的权限。参见 Section 8.2.19, “Proxy Users”。

#### 全局授权授权表

`global_grants`表列出了动态全局特权当前分配给用户账户的情况。该表包含以下列：

+   `用户`，`主机`：被授予权限的账户的用户名和主机名。

+   `权限`：权限名称。

+   `WITH_GRANT_OPTION`：账户是��可以向其他账户授予特权。

#### 默认角色授权表

`default_roles`表列出了默认用户角色。它包含以下列：

+   `主机`，`用户`：应用默认角色的账户或角色。

+   `DEFAULT_ROLE_HOST`，`DEFAULT_ROLE_USER`：默认角色。

#### 角色边缘授权表

`role_edges`表列出了角色子图的边缘。它包含以下列：

+   `FROM_HOST`，`FROM_USER`：被授予角色的账户。

+   `TO_HOST`，`TO_USER`：授予给账户的角色。

+   `WITH_ADMIN_OPTION`：账户是否可以通过使用`WITH ADMIN OPTION`向其他账户授予角色和撤销角色。

#### 密码历史授权表

`password_history`表包含有关密码更改的信息。它包含以下列：

+   `主机`，`用户`：密码更改发生的账户。

+   `密码时间戳`：密码更改发生的时间。

+   `密码`：新密码哈希值。

`password_history`表累积每个账户的足够数量的非空密码，以使 MySQL 能够执行针对账户密码历史长度和重用间隔的检查。当密码更改尝试发生时，会自动修剪超出这两个限制的条目。

注意

空密码不计入密码历史记录，并可随时重新使用。

如果账户被重命名，其条目也将被重命名以匹配。如果账户被删除或其认证插件被更改，其条目将被移除。

#### 授权表范围列属性

授权表中的范围列包含字符串。每个的默认值为空字符串。以下表显示了每个列中允许的字符数。

**表 8.8 授权表范围列长度**

| 列名 | 最大允许字符数 |
| --- | --- |
| `主机`，`代理主机` | 255（在 MySQL 8.0.17 之前为 60） |
| `用户`，`代理用户` | 32 |
| `数据库` | 64 |
| `表名` | 64 |
| `Column_name` | 64 |
| `Routine_name` | 64 |

`Host` 和 `Proxied_host` 的值在存储在授权表中之前会被转换为小写。

为了访问检查目的，`User`、`Proxied_user`、`authentication_string`、`Db` 和 `Table_name` 的值的比较是区分大小写的。`Host`、`Proxied_host`、`Column_name` 和 `Routine_name` 的值的比较是不区分大小写的。

#### 授权表权限列属性

`user` 和 `db` 表中列出了每个权限在一个单独的列中声明为 `ENUM('N','Y') DEFAULT 'N'`。换句话说，每个权限可以被禁用或启用，默认情况下为禁用。

`tables_priv`、`columns_priv` 和 `procs_priv` 表将权限列声明为 `SET` 列。这些列中的值可以包含由表控制的任何权限的任意组合。只有列值中列出的权限才会被启用。

**表 8.9 Set-Type Privilege Column Values**

| 表名 | 列名 | 可能的集合元素 |
| --- | --- | --- |
| `tables_priv` | `Table_priv` | `'Select', 'Insert', 'Update', 'Delete', 'Create', 'Drop', 'Grant', 'References', 'Index', 'Alter', 'Create View', 'Show view', 'Trigger'` |
| `tables_priv` | `Column_priv` | `'Select', 'Insert', 'Update', 'References'` |
| `columns_priv` | `Column_priv` | `'Select', 'Insert', 'Update', 'References'` |
| `procs_priv` | `Proc_priv` | `'Execute', 'Alter Routine', 'Grant'` |

只有 `user` 和 `global_grants` 表指定了管理权限，例如 `RELOAD`、`SHUTDOWN` 和 `SYSTEM_VARIABLES_ADMIN`。管理操作是对服务器本身的操作，不是特定于数据库的，因此没有理由在其他授权表中列出这些权限。因此，服务器只需要查看 `user` 和 `global_grants` 表来确定用户是否可以执行管理操作。

`FILE` 权限也仅在 `user` 表中指定。它不是一种管理权限，而是用户在服务器主机上读取或写入文件的能力与所访问的数据库无关。

#### 授权表并发性

从 MySQL 8.0.22 开始，为了允许在 MySQL 授权表上进行并发的 DML 和 DDL 操作，以前在 MySQL 授权表上获取行锁的读操作将作为非锁定读取执行。在 MySQL 授权表上执行为非锁定读取的操作包括：

+   通过联接列表和子查询从授权表中读取数据的 `SELECT` 语句和其他只读语句，包括使用任何事务隔离级别的 `SELECT ... FOR SHARE` 语句。

+   从授权表中读取数据的 DML 操作（通过连接列表或子查询）但不修改它们，在任何事务隔离级别下使用。

从授权表中读取数据时不再获取行锁的语句，在使用基于语句的复制时执行会报告警告。

当使用 -`binlog_format=mixed` 时，从授权表中读取数据的 DML 操作会被写入二进制日志作为行事件，以使操作对混合模式复制安全。

`SELECT ... FOR SHARE` 语句从授权表中读取数据时会报告警告。使用`FOR SHARE`子句时，不支持在授权表上进行读取锁定。

从授权表中读取数据并使用`SERIALIZABLE`隔离级别执行的 DML 操作会报告警告。在使用`SERIALIZABLE`隔离级别时通常会获取的读取锁在授权表上不受支持。
