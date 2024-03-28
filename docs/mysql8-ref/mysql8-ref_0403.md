> 原文：[`dev.mysql.com/doc/refman/8.0/en/no-login-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/no-login-pluggable-authentication.html)

#### 8.4.1.9 无登录可插拔认证

`mysql_no_login` 服务器端认证插件阻止所有使用它的账户的客户端连接。此插件的用例包括：

+   必须能够执行存储过程和视图并具有提升权限，而不会将这些权限暴露给普通用户的账户。

+   代理账户永远不应允许直接登录，而是仅通过代理账户访问的账户。

以下表显示了插件和库文件名。文件名后缀可能因您的系统而异。该文件必须位于由 `plugin_dir` 系统变量命名的目录中。

**表 8.25 无登录认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `mysql_no_login` |
| 客户端插件 | 无 |
| 库文件 | `mysql_no_login.so` |

以下各节提供了特定于无登录可插拔认证的安装和使用信息：

+   安装无登录可插拔认证

+   卸载无登录可插拔认证

+   使用无登录可插拔认证

有关 MySQL 中可插拔认证的一般信息，请参阅第 8.2.17 节，“可插拔认证”。有关代理用户信息，请参阅第 8.2.19 节，“代理用户”。

##### 安装无登录可插拔认证

本节描述了如何安装无登录认证插件。有关安装插件的一般信息，请参阅第 7.6.1 节，“安装和卸载插件”。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由 `plugin_dir` 系统变量命名的目录）。如有必要，请通过在服务器启动时设置 `plugin_dir` 的值来配置插件目录位置。

插件库文件基本名称为 `mysql_no_login`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为 `.so`，对于 Windows 为 `.dll`）。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，选项必须在每次服务器启动时给出。例如，将这些行放入服务器的`my.cnf`文件中，根据需要调整您平台的`.so`后缀：

```sql
[mysqld]
plugin-load-add=mysql_no_login.so
```

修改`my.cnf`后，请重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用此语句，根据需要调整您平台的`.so`后缀：

```sql
INSTALL PLUGIN mysql_no_login SONAME 'mysql_no_login.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装，请检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%login%';
+----------------+---------------+
| PLUGIN_NAME    | PLUGIN_STATUS |
+----------------+---------------+
| mysql_no_login | ACTIVE        |
+----------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

要将 MySQL 帐户与无登录插件关联，请参阅使用无登录可插拔认证。

##### 卸载无登录可插拔认证

卸载无登录认证插件的方法取决于您安装它的方式：

+   如果您在服务器启动时使用`--plugin-load-add`选项安装了插件，请在不带该选项的情况下重新启动服务器。

+   如果您在运行时使用`INSTALL PLUGIN`语句安装了插件，则在服务器重新启动时仍然保留安装。要卸载它，请使用`UNINSTALL PLUGIN`：

    ```sql
    UNINSTALL PLUGIN mysql_no_login;
    ```

##### 使用无登录可插拔认证

本节描述如何使用无登录认证插件防止帐户被用于从 MySQL 客户端程序连接到服务器。假定服务器正在运行，并启用了无登录插件，如安装无登录可插拔认证中所述。

在`CREATE USER`语句的`IDENTIFIED WITH`子句中引用无登录认证插件时，请使用名称`mysql_no_login`。

使用`mysql_no_login`进行身份验证的账户可用作存储过程和视图对象的`DEFINER`。如果此类对象定义还包括`SQL SECURITY DEFINER`，则将以该账户的权限执行。数据库管理员可以利用此行为提供仅通过受控接口公开的机密或敏感数据的访问。

以下示例说明了这些原则。它定义了一个不允许客户端连接的账户，并将其与一个仅公开`mysql.user`系统表的某些列的视图关联起来：

```sql
CREATE DATABASE nologindb;
CREATE USER 'nologin'@'localhost'
  IDENTIFIED WITH mysql_no_login;
GRANT ALL ON nologindb.*
  TO 'nologin'@'localhost';
GRANT SELECT ON mysql.user
  TO 'nologin'@'localhost';
CREATE DEFINER = 'nologin'@'localhost'
  SQL SECURITY DEFINER
  VIEW nologindb.myview
  AS SELECT User, Host FROM mysql.user;
```

要为普通用户提供对视图的受保护访问，请执行以下操作：

```sql
GRANT SELECT ON nologindb.myview
  TO 'ordinaryuser'@'localhost';
```

现在普通用户可以使用视图访问它呈现的有限信息：

```sql
SELECT * FROM nologindb.myview;
```

用户尝试访问视图中未公开的列或未被授权访问的用户尝试从视图中选择将导致错误。

注意

由于`nologin`账户不能直接使用，因此必须由具有创建对象和设置`DEFINER`值所需特权的`root`或类似账户执行设置对象的操作。

`mysql_no_login`插件在代理场景中也很有用。（有关代理涉及的概念讨论，请参阅第 8.2.19 节，“代理用户”。）使用`mysql_no_login`进行身份验证的账户可用作代理账户的被代理用户：

```sql
-- create proxied account
CREATE USER 'proxied_user'@'localhost'
  IDENTIFIED WITH mysql_no_login;
-- grant privileges to proxied account
GRANT ...
  ON ...
  TO 'proxied_user'@'localhost';
-- permit proxy_user to be a proxy account for proxied account
GRANT PROXY
  ON 'proxied_user'@'localhost'
  TO 'proxy_user'@'localhost';
```

这使客户端可以通过代理账户(`proxy_user`)访问 MySQL，但不能通过直接连接作为被代理用户(`proxied_user`)绕过代理机制。使用`proxy_user`账户连接的客户端具有`proxied_user`账户的权限，但`proxied_user`本身不能用于连接。

若要了解有关保护代理账户免受直接使用的替代方法，请参阅防止直接登录到代理账户。
