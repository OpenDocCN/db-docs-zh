# 8.2.19 代理用户

> 原文：[`dev.mysql.com/doc/refman/8.0/en/proxy-users.html`](https://dev.mysql.com/doc/refman/8.0/en/proxy-users.html)

MySQL 服务器使用认证插件对客户端连接进行身份验证。认证给定连接的插件可能要求将连接的（外部）用户视为不同用户以进行特权检查。这使得外部用户可以成为第二用户的代理；也就是说，承担第二用户的权限：

+   外部用户是“代理用户”（可以冒充或成为另一个用户的用户）。

+   第二个用户是“被代理用户”（其身份和特权可以被代理用户所假定）。

本节描述了代理用户功能的工作原理。有关认证插件的一般信息，请参阅第 8.2.17 节“可插拔认证”。有关特定插件的信息，请参阅第 8.4.1 节“认证插件”。有关编写支持代理用户的认证插件的信息，请参阅在认证插件中实现代理用户支持。

+   代理用户支持要求

+   简单代理用户示例

+   防止直接登录代理帐户

+   授予和撤销 PROXY 特权

+   默认代理用户

+   默认代理用户和匿名用户冲突

+   服务器支持代理用户映射

+   代理用户系统变量

注意

代理的一个管理优势是，DBA 可以设置一个具有一组特权的单个帐户，然后启用多个代理用户具有这些特权，而无需为每个用户单独分配特权。作为代理用户的替代方案，DBA 可能会发现角色提供了一种适合将用户映射到特定命名特权集的方法。每个用户可以被授予一个给定的单一角色，以实际上被授予适当的特权集。请参阅第 8.2.10 节“使用角色”。

#### 代理用户支持要求

要对给定认证插件进行代理，必须满足以下条件：

+   必须支持代理，可以是插件本身支持，也可以是 MySQL 服务器代表插件支持。在后一种情况下，可能需要显式启用服务器支持；参见服务器支持代理用户映射。

+   外部代理用户的账户必须设置为由插件进行身份验证。使用`CREATE USER`语句将账户与身份验证插件关联，或使用`ALTER USER`更改其插件。

+   被代理用户的账户必须存在并被授予代理用户所需的权限。使用`CREATE USER`和`GRANT`语句进行此操作。

+   通常，代理用户被配置为仅在代理场景中使用，而不用于直接登录。

+   代理用户账户必须具有`PROXY`权限以供被代理账户使用。使用`GRANT`语句进行此操作。

+   对于连接到代理账户的客户端要被视为代理用户，身份验证插件必须返回与客户端用户名不同的用户名，以指示由代理用户承担的权限定义的被代理账户的用户名。

    或者，对于由服务器提供代理映射的插件，被代理用户是由代理用户持有的`PROXY`权限确定的。

代理机制仅允许将外部客户端用户名映射到被代理用户名称。没有提供主机名映射的规定：

+   当客户端连接到服务器时，服务器根据客户端程序传递的用户名和客户端连接的主机确定适当的账户。

+   如果该账户是代理账户，则服务器尝试通过找到与身份验证插件返回的用户名和代理账户的主机名匹配的被代理账户来确定适当的被代理账户。被代理账户中的主机名将被忽略。

#### 简单代理用户示例

考虑以下账户定义：

```sql
-- create proxy account
CREATE USER 'employee_ext'@'localhost'
  IDENTIFIED WITH my_auth_plugin
  AS '*my_auth_string*';

-- create proxied account and grant its privileges;
-- use mysql_no_login plugin to prevent direct login
CREATE USER 'employee'@'localhost'
  IDENTIFIED WITH mysql_no_login;
GRANT ALL
  ON employees.*
  TO 'employee'@'localhost';

-- grant to proxy account the
-- PROXY privilege for proxied account
GRANT PROXY
  ON 'employee'@'localhost'
  TO 'employee_ext'@'localhost';
```

当客户端以`employee_ext`从本地主机连接时，MySQL 使用名为`my_auth_plugin`的插件执行身份验证。假设`my_auth_plugin`根据`'*`my_auth_string`*'`的内容并可能通过查询某些外部身份验证系统返回一个名为`employee`的用户名给服务器。名称`employee`与`employee_ext`不同，因此返回`employee`作为请求服务器将`employee_ext`外部用户视为`employee`本地用户进行权限检查。

在这种情况下，`employee_ext`是代理用户，`employee`是被代理用户。

服务器通过检查`employee_ext`（代理用户）是否对`employee`（被代理用户）具有`PROXY`权限来验证`employee`的代理认证是否对`employee_ext`用户可行。如果未授予此权限，则会发生错误。否则，`employee_ext`假定`employee`的权限。服务器通过检查由`employee_ext`在客户端会话期间执行的语句来针对授予`employee`的权限。在这种情况下，`employee_ext`可以访问`employees`数据库中的表。

代理账户`employee`使用`mysql_no_login`认证插件来防止客户端直接使用该账户登录。（假设插件已安装。有关说明，请参见 Section 8.4.1.9, “无登录可插入式认证”。）要了解保护代理账户免受直接使用的替代方法，请参见防止直接登录到代理账户。

当进行代理时，`USER()`和`CURRENT_USER()`函数可用于查看连接用户（代理用户）与当前会话期间适用的账户（被代理用户）之间的区别。对于刚才描述的示例，这些函数返回以下值：

```sql
mysql> SELECT USER(), CURRENT_USER();
+------------------------+--------------------+
| USER()                 | CURRENT_USER()     |
+------------------------+--------------------+
| employee_ext@localhost | employee@localhost |
+------------------------+--------------------+
```

在创建代理用户账户的`CREATE USER`语句中，指定支持代理的认证插件的`IDENTIFIED WITH`子句后面可以选择跟随一个`AS '*`auth_string`*'`子句，该子句指定服务器在用户连接时传递给插件的字符串。如果存在，该字符串提供帮助插件确定如何将代理（外部）客户端用户名映射到被代理用户名称的信息。每个插件是否需要`AS`子句取决于插件打算如何使用它。如果需要，认证字符串的格式取决于插件打算如何使用它。请查阅特定插件的文档，了解其接受的认证字符串值的信息。

#### 防止直接登录到代理账户

代理账户通常只能通过代理账户使用。也就是说，客户端使用代理账户连接，然后映射到并假定适当被代理用户的权限。

有多种方法可以确保代理账户不能直接使用：

+   将账户与 `mysql_no_login` 认证插件关联。在这种情况下，账户无论如何都不能用于直接登录。这假定插件已安装。有关说明，请参见 Section 8.4.1.9, “No-Login Pluggable Authentication”。

+   创建账户时包括 `ACCOUNT LOCK` 选项。参见 Section 15.7.1.3, “CREATE USER Statement”。使用这种方法时，还要包括一个密码，这样如果账户稍后解锁，就不能在没有密码的情况下访问它。（如果启用了 `validate_password` 组件，则不允许创建没有密码的账户，即使账户被锁定。参见 Section 8.4.3, “The Password Validation Component”。）

+   创建带有密码的账户，但不要告诉其他人密码。如果不让任何人知道账户的密码，客户端就无法使用它直接连接到 MySQL 服务器。

#### 授予和撤销 PROXY 权限

需要 `PROXY` 权限以使外部用户能够连接并具有另一个用户的权限。要授予此权限，请使用 `GRANT` 语句。例如：

```sql
GRANT PROXY ON '*proxied_user*' TO '*proxy_user*';
```

该语句在 `mysql.proxies_priv` 授权表中创建一行。

在连接时，*`proxy_user`* 必须代表一个有效的外部认证的 MySQL 用户，而 *`proxied_user`* 必须代表一个有效的本地认证用户。否则，连接尝试将失败。

相应的 `REVOKE` 语法为：

```sql
REVOKE PROXY ON '*proxied_user*' FROM '*proxy_user*';
```

MySQL `GRANT` 和 `REVOKE` 语法扩展与往常一样工作。示例：

```sql
-- grant PROXY to multiple accounts
GRANT PROXY ON 'a' TO 'b', 'c', 'd';

-- revoke PROXY from multiple accounts
REVOKE PROXY ON 'a' FROM 'b', 'c', 'd';

-- grant PROXY to an account and enable the account to grant
-- PROXY to the proxied account
GRANT PROXY ON 'a' TO 'd' WITH GRANT OPTION;

-- grant PROXY to default proxy account
GRANT PROXY ON 'a' TO ''@'';
```

可以在以下情况下授予 `PROXY` 权限：

+   由具有 *`proxied_user`* 的 `GRANT PROXY ... WITH GRANT OPTION` 权限的用户。

+   由 *`proxied_user`* 为自身：`USER()` 的值必须与 `CURRENT_USER()` 和 *`proxied_user`* 完全匹配，包括账户名和主机名部分。

在 MySQL 安装期间创建的初始 `root` 账户具有 `PROXY ... WITH GRANT OPTION` 权限，适用于 `''@''`，即所有用户和所有主机。这使得 `root` 能够设置代理用户，并授权其他账户设置代理用户。例如，`root` 可以这样做：

```sql
CREATE USER 'admin'@'localhost'
  IDENTIFIED BY '*admin_password*';
GRANT PROXY
  ON ''@''
  TO 'admin'@'localhost'
  WITH GRANT OPTION;
```

这些语句创建一个可以管理所有 `GRANT PROXY` 映射的 `admin` 用��。例如，`admin` 可以这样做：

```sql
GRANT PROXY ON sally TO joe;
```

#### 默认代理用户

要指定某些或所有用户应使用特定的身份验证插件连接，请创建一个“空白”MySQL 帐户，具有空用户名和主机名（`''@''`），将其与该插件关联，并让插件返回真实的经过身份验证的用户名（如果与空白用户不同）。假设存在一个名为`ldap_auth`的插件，实现 LDAP 身份验证并将连接用户映射到开发者或经理帐户。要设置用户代理到这些帐户，使用以下语句：

```sql
-- create default proxy account
CREATE USER ''@''
  IDENTIFIED WITH ldap_auth
  AS 'O=Oracle, OU=MySQL';

-- create proxied accounts; use
-- mysql_no_login plugin to prevent direct login
CREATE USER 'developer'@'localhost'
  IDENTIFIED WITH mysql_no_login;
CREATE USER 'manager'@'localhost'
  IDENTIFIED WITH mysql_no_login;

-- grant to default proxy account the
-- PROXY privilege for proxied accounts
GRANT PROXY
  ON 'manager'@'localhost'
  TO ''@'';
GRANT PROXY
  ON 'developer'@'localhost'
  TO ''@'';
```

现在假设一个客户端连接如下：

```sql
$> mysql --user=myuser --password ...
Enter password: *myuser_password*
```

服务器未找到定义为 MySQL 用户的`myuser`，但是因为存在一个空用户帐户（`''@''`）与客户端用户名和主机名匹配，服务器会根据该帐户对客户端进行身份验证。服务器调用`ldap_auth`身份验证插件，并将`myuser`和*`myuser_password`*作为用户名和密码传递给它。

如果`ldap_auth`插件在 LDAP 目录中发现*`myuser_password`*不是`myuser`的正确密码，身份验证失败，服务器拒绝连接。

如果密码正确，并且`ldap_auth`发现`myuser`是一个开发者，它会将用户名`developer`返回给 MySQL 服务器，而不是`myuser`。返回与客户端用户名`myuser`不同的用户名向服务器发出信号，表明它应该将`myuser`视为代理。服务器验证`''@''`是否可以作为`developer`进行身份验证（因为`''@''`具有`PROXY`权限），并接受连接。会话继续进行，`myuser`具有`developer`代理用户的权限。（这些权限应由 DBA 使用`GRANT`语句设置，未显示。）`USER()`和`CURRENT_USER()`函数返回这些值：

```sql
mysql> SELECT USER(), CURRENT_USER();
+------------------+---------------------+
| USER()           | CURRENT_USER()      |
+------------------+---------------------+
| myuser@localhost | developer@localhost |
+------------------+---------------------+
```

如果插件在 LDAP 目录中发现`myuser`是一个经理，它会将`manager`作为用户名返回，并且会话将继续进行，`myuser`具有`manager`代理用户的权限。

```sql
mysql> SELECT USER(), CURRENT_USER();
+------------------+-------------------+
| USER()           | CURRENT_USER()    |
+------------------+-------------------+
| myuser@localhost | manager@localhost |
+------------------+-------------------+
```

为简单起见，外部身份验证不能是多级的：在上述示例中，`developer`或`manager`的凭据都不会被考虑。但是，如果客户端尝试直接连接并作为`developer`或`manager`帐户进行身份验证，仍然会使用这些帐户，因此应该保护这些代理帐户免受直接登录的影响（请参阅防止直接登录到代理帐户）。

#### 默认代理用户和匿名用户冲突

如果您打算创建一个默认代理用户，请检查其他现有的“匹配任何用户”帐户，因为它们可能优先于默认代理用户，从而阻止该用户按预期工作。

在前面的讨论中，默认代理用户账户在主机部分有`''`，匹配任何主机。如果设置了默认代理用户，请务必检查是否存在具有相同用户部分和`'%'`主机部分的非代理账户，因为`'%'`也匹配任何主机，但根据服务器用于内部排序账户行的规则，`'%'`优先于`''`（请参阅第 8.2.6 节，“访问控制，阶段 1：连接验证”）。

假设 MySQL 安装包括这两个账户：

```sql
-- create default proxy account
CREATE USER ''@''
  IDENTIFIED WITH some_plugin
  AS '*some_auth_string*';
-- create anonymous account
CREATE USER ''@'%'
  IDENTIFIED BY '*anon_user_password*';
```

第一个账户（`''@''`）旨在作为默认代理用户，用于验证那些不匹配更具体账户的用户的连接。第二个账户（`''@'%'`）是一个匿名用户账户，例如可能已创建，以便使没有自己账户的用户能够匿名连接。

两个账户的用户部分（`''`）相同，匹配任何用户。每个账户都有一个匹配任何主机的主机部分。然而，在连接尝试的账户匹配中存在优先级，因为匹配规则将`'%'`的主机排在`''`之前。对于不再匹配任何更具体账户的账户，服务器会尝试对其进行`''@'%'`（匿名用户）的身份验证，而不是`''@''`（默认代理用户）。因此，默认代理账户永远不会被使用。

为避免此问题，请使用以下策略之一：

+   删除匿名账户，以避免与默认代理用户冲突。

+   使用匹配在匿名用户之前的更具体默认代理用户。例如，为了仅允许`localhost`代理连接，使用`''@'localhost'`：

    ```sql
    CREATE USER ''@'localhost'
      IDENTIFIED WITH some_plugin
      AS '*some_auth_string*';
    ```

    此外，修改任何`GRANT PROXY`语句，将`''@'localhost'`命名为代理用户，而不是`''@''`。

    请注意，此策略会阻止来自`localhost`的匿名用户连接。

+   使用具名的默认账户而不是匿名的默认账户。有关此技术的示例，请参考使用`authentication_windows`插件的说明。请参阅第 8.4.1.6 节，“Windows 可插拔认证”。

+   创建多个代理用户，一个用于本地连接，另一个用于“其他所有”（远程连接）。这在本地用户应该具有不同权限于远程用户时特别有用。

    创建代理用户：

    ```sql
    -- create proxy user for local connections
    CREATE USER ''@'localhost'
      IDENTIFIED WITH some_plugin
      AS '*some_auth_string*';
    -- create proxy user for remote connections
    CREATE USER ''@'%'
      IDENTIFIED WITH some_plugin
      AS '*some_auth_string*';
    ```

    创建被代理用户：

    ```sql
    -- create proxied user for local connections
    CREATE USER 'developer'@'localhost'
      IDENTIFIED WITH mysql_no_login;
    -- create proxied user for remote connections
    CREATE USER 'developer'@'%'
      IDENTIFIED WITH mysql_no_login;
    ```

    为每个代理账户授予对应被代理账户的`PROXY`权限：

    ```sql
    GRANT PROXY
      ON 'developer'@'localhost'
      TO ''@'localhost';
    GRANT PROXY
      ON 'developer'@'%'
      TO ''@'%';
    ```

    最后，为本地和远程被代理用户授予适当的权限（未显示）。

    假设 `some_plugin`/`'*`some_auth_string`*'` 组合导致 `some_plugin` 将客户端用户名映射到 `developer`。本地连接匹配 `''@'localhost'` 代理用户，映射到 `'developer'@'localhost'` 被代理用户。远程连接匹配 `''@'%'` 代理用户，映射到 `'developer'@'%'` 被代理用户。

#### 服务器对代理用户映射的支持

一些认证插件为自身实现了代理用户映射（例如，PAM 和 Windows 认证插件）。其他认证插件默认不支持代理用户。其中一些可以请求 MySQL 服务器根据授予的代理权限映射代理用户：`mysql_native_password`，`sha256_password`。如果启用了 `check_proxy_users` 系统变量，服务器将为任何请求进行代理用户映射的认证插件执行代理用户映射：

+   默认情况下，`check_proxy_users` 处于禁用状态，因此服务器即使对请求服务器支持代理用户的认证插件也不执行代理用户映射。

+   如果启用了 `check_proxy_users`，可能还需要启用特定于插件的系统变量以利用服务器代理用户映射支持：

    +   对于 `mysql_native_password` 插件，请启用 `mysql_native_password_proxy_users`。

    +   对于 `sha256_password` 插件，请启用 `sha256_password_proxy_users`。

例如，要启用所有上述功能，请在 `my.cnf` 文件中添加以下行启动服务器：

```sql
[mysqld]
check_proxy_users=ON
mysql_native_password_proxy_users=ON
sha256_password_proxy_users=ON
```

假设相关系统变量已启用，请像往常一样使用 `CREATE USER` 创建代理用户，然后授予它对一个其他账户的 `PROXY` 权限，以将其视为被代理用户。当服务器接收到代理用户的成功连接请求时，发现用户具有 `PROXY` 权限，并使用它确定适当的被代理用户。

```sql
-- create proxy account
CREATE USER 'proxy_user'@'localhost'
  IDENTIFIED WITH mysql_native_password
  BY '*password*';

-- create proxied account and grant its privileges;
-- use mysql_no_login plugin to prevent direct login
CREATE USER 'proxied_user'@'localhost'
  IDENTIFIED WITH mysql_no_login;
-- grant privileges to proxied account
GRANT ...
  ON ...
  TO 'proxied_user'@'localhost';

-- grant to proxy account the
-- PROXY privilege for proxied account
GRANT PROXY
  ON 'proxied_user'@'localhost'
  TO 'proxy_user'@'localhost';
```

要使用代理账户，请使用其名称和密码连接到服务器：

```sql
$> mysql -u proxy_user -p
Enter password: *(enter proxy_user password here)*
```

认证成功后，服务器发现 `proxy_user` 对 `proxied_user` 有 `PROXY` 权限，会话将继续进行，`proxy_user` 具有 `proxied_user` 的权限。

服务器执行的代理用户映射受到以下限制：

+   服务器不会代理匿名用户，即使授予了相关的 `PROXY` 权限。

+   当一个单一账户被授予多个被代理账户的代理权限时，服务器代理用户映射是不确定的。因此，不建议为单一账户授予多个被代理账户的代理权限。

#### 代理用户系统变量

有两个系统变量帮助跟踪代理登录过程：

+   `proxy_user`：如果未使用代理，则该值为`NULL`。否则，它表示代理用户账户。例如，如果客户端通过`''@''`代理账户进行身份验证，那么该变量将设置如下：

    ```sql
    mysql> SELECT @@proxy_user;
    +--------------+
    | @@proxy_user |
    +--------------+
    | ''@''        |
    +--------------+
    ```

+   `external_user`：有时认证插件可能使用外部用户来认证到 MySQL 服务器。例如，当使用 Windows 本地认证时，一个使用 Windows API 进行认证的插件不需要传递登录 ID。然而，它仍然使用 Windows 用户 ID 进行认证。插件可能会将这个外部用户 ID（或其前 512 个 UTF-8 字节）返回给服务器，使用`external_user`只读会话变量。如果插件没有设置这个变量，其值为`NULL`。
