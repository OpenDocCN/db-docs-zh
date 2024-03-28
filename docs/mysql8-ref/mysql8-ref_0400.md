> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-pluggable-authentication.html)

#### 8.4.1.6 Windows 可插拔认证

注意

Windows 可插拔认证是 MySQL 企业版中包含的扩展，这是一个商业产品。要了解更多关于商业产品的信息，请查看[`www.mysql.com/products/`](https://www.mysql.com/products/)。

Windows 版 MySQL 企业版支持一种在 Windows 上执行外部认证的认证方法，使 MySQL 服务器能够使用本机 Windows 服务对客户端连接进行认证。已登录到 Windows 的用户可以根据其环境中的信息从 MySQL 客户端程序连接到服务器，而无需指定额外的密码。

客户端和服务器在认证握手中交换数据包。由于这种交换，服务器创建一个代表客户端在 Windows OS 中身份的安全上下文对象。这个身份包括客户端账户的名称。Windows 可插拔认证使用客户端的身份来检查它是否是给定账户或组的成员。默认情况下，协商使用 Kerberos 进行认证，如果 Kerberos 不可用，则使用 NTLM。

Windows 可插拔认证提供了以下功能：

+   外部认证：Windows 认证使 MySQL 服务器能够接受来自在 Windows 登录的 MySQL 授权表之外定义的用户的连接。

+   代理用户支持：Windows 认证可以将一个与客户端程序传递的外部用户名不同的用户名返回给 MySQL。这意味着插件可以返回定义外部 Windows 认证用户应具有的权限的 MySQL 用户。例如，一个名为`joe`的 Windows 用户可以连接并具有名为`developer`的 MySQL 用户的权限。

以下表格显示了插件和库文件的名称。文件必须位于由`plugin_dir`系统变量命名的目录中。

**表 8.21 Windows 认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `authentication_windows` |
| 客户端插件 | `authentication_windows_client` |
| 库文件 | `authentication_windows.dll` |

库文件仅包含服务器端插件。客户端插件内置于`libmysqlclient`客户端库中。

服务器端 Windows 认证插件仅包含在 MySQL 企业版中。它不包含在 MySQL 社区发行版中。客户端插件包含在所有发行版中，包括社区发行版。这使得来自任何发行版的客户端都能连接到加载了服务器端插件的服务器。

以下部分提供了特定于 Windows 可插拔认证的安装和使用信息：

+   安装 Windows 可插拔认证

+   卸载 Windows 可插拔认证

+   使用 Windows 可插拔认证

有关 MySQL 中可插拔认证的一般信息，请参阅 Section 8.2.17, “Pluggable Authentication”。有关代理用户信息，请参阅 Section 8.2.19, “Proxy Users”。

##### 安装 Windows 可插拔认证

本节描述了如何安装服务器端 Windows 认证插件。有关安装插件的一般信息，请参阅 Section 7.6.1, “Installing and Uninstalling Plugins”。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如果需要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，每次服务器启动时都必须提供该选项。例如，将以下行放入服务器`my.cnf`文件中：

```sql
[mysqld]
plugin-load-add=authentication_windows.dll
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用以下语句：

```sql
INSTALL PLUGIN authentication_windows SONAME 'authentication_windows.dll';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%windows%';
+------------------------+---------------+
| PLUGIN_NAME            | PLUGIN_STATUS |
+------------------------+---------------+
| authentication_windows | ACTIVE        |
+------------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

要将 MySQL 帐户与 Windows 认证插件关联，请参阅使用 Windows 可插拔认证。 通过`authentication_windows_use_principal_name`和`authentication_windows_log_level`系统变量提供了额外的插件控制。 请参阅第 7.1.8 节，“服务器系统变量”。

##### 卸载 Windows 可插拔认证

卸载 Windows 认证插件的方法取决于您安装它的方式：

+   如果您在服务器启动时使用`--plugin-load-add`选项安装插件，请在不带该选项的情况下重新启动服务器。

+   如果您使用`INSTALL PLUGIN`语句在运行时安装插件，则插件将在服务器重新启动时保持安装状态。 要卸载它，请使用`UNINSTALL PLUGIN`：

    ```sql
    UNINSTALL PLUGIN authentication_windows;
    ```

另外，删除任何设置 Windows 插件相关系统变量的启动选项。

##### 使用 Windows 可插拔认证

Windows 认证插件支持使用 MySQL 帐户，使得已登录 Windows 的用户可以连接到 MySQL 服务器，而无需指定额外的密码。 假定服务器正在运行并启用了服务器端插件，如安装 Windows 可插拔认证中所述。 一旦 DBA 启用了服务器端插件并设置了要使用它的帐户，客户端就可以使用这些帐户连接，而无需进行其他设置。

在`CREATE USER`语句的`IDENTIFIED WITH`子句中引用 Windows 认证插件，请使用名称`authentication_windows`。 假设 Windows 用户`Rafal`和`Tasha`应被允许连接到 MySQL，以及`Administrators`或`Power Users`组中的任何用户。 要设置这一点，请创建一个名为`sql_admin`的 MySQL 帐户，该帐户使用 Windows 插件进行身份验证：

```sql
CREATE USER sql_admin
  IDENTIFIED WITH authentication_windows
  AS 'Rafal, Tasha, Administrators, "Power Users"';
```

插件名称为`authentication_windows`。 `AS`关键字后面的字符串是认证字符串。 它指定了名为`Rafal`或`Tasha`的 Windows 用户被允许作为 MySQL 用户`sql_admin`进行服务器身份验证，以及`Administrators`或`Power Users`组中的任何 Windows 用户。 后一个组名包含一个空格，因此必须用双引号括起来。

创建`sql_admin`账户后，已登录到 Windows 的用户可以尝试使用该账户连接到服务器：

```sql
C:\> mysql --user=sql_admin
```

这里不需要密码。`authentication_windows`插件使用 Windows 安全 API 来检查连接的是哪个 Windows 用户。如果该用户名为`Rafal`或`Tasha`，或者是`Administrators`或`Power Users`组的成员，则服务器授予访问权限，并将客户端验证为`sql_admin`，并具有授予`sql_admin`账户的任何权限。否则，服务器将拒绝访问。

Windows 认证插件的认证字符串语法遵循以下规则：

+   该字符串由一个或多个以逗号分隔的用户映射组成。

+   每个用户映射将 Windows 用户或组名与 MySQL 用户名称关联起来：

    ```sql
    *win_user_or_group_name=mysql_user_name*
    *win_user_or_group_name*
    ```

    对于后一种语法，如果没有给出*`mysql_user_name`*值，则隐式值是由`CREATE USER`语句创建的 MySQL 用户。因此，以下语句是等效的：

    ```sql
    CREATE USER sql_admin
      IDENTIFIED WITH authentication_windows
      AS 'Rafal, Tasha, Administrators, "Power Users"';

    CREATE USER sql_admin
      IDENTIFIED WITH authentication_windows
      AS 'Rafal=sql_admin, Tasha=sql_admin, Administrators=sql_admin,
          "Power Users"=sql_admin';
    ```

+   值中的每个反斜杠字符（`\`）都必须加倍，因为反斜杠是 MySQL 字符串中的转义字符。

+   在双引号内部的前导和尾随空格将被忽略。

+   未引用的*`win_user_or_group_name`*和*`mysql_user_name`*值可以包含除等号、逗号或空格之外的任何内容。

+   如果*`win_user_or_group_name`*和/或*`mysql_user_name`*值用双引号引起来，那么引号之间的所有内容都是值的一部分。例如，如果名称包含空格字符，则这是必要的。双引号内的所有字符都是合法的，除了双引号和反斜杠。要包含这两个字符，需要用反斜杠进行转义。

+   *`win_user_or_group_name`*值使用 Windows 主体的传统语法，可以是本地的或域中的。示例（注意反斜杠的加倍）：

    ```sql
    domain\\user
    .\\user
    domain\\group
    .\\group
    BUILTIN\\WellKnownGroup
    ```

当服务器调用插件来验证客户端时，插件从左到右扫描认证字符串，以查找与 Windows 用户匹配的用户或组。如果有匹配，插件将相应的*`mysql_user_name`*返回给 MySQL 服务器。如果没有匹配，则认证失败。

用户名称匹配优先于组名称匹配。假设名为`win_user`的 Windows 用户是`win_group`的成员，并且认证字符串如下所示：

```sql
'win_group = sql_user1, win_user = sql_user2'
```

当`win_user`连接到 MySQL 服务器时，既与`win_group`匹配，也与`win_user`匹配。插件将用户验证为`sql_user2`，因为更具体的用户匹配优先于组匹配，即使组在认证字符串中首先列出。

Windows 身份验证始终适用于从运行服务器的同一台计算机发起的连接。对于跨计算机连接，两台计算机必须在 Microsoft Active Directory 中注册。如果它们在同一个 Windows 域中，则无需指定域名。也可以允许来自不同域的连接，就像这个例子中一样：

```sql
CREATE USER sql_accounting
  IDENTIFIED WITH authentication_windows
  AS 'SomeDomain\\Accounting';
```

这里 `SomeDomain` 是另一个域的名称。反斜杠字符被加倍，因为它是字符串中的 MySQL 转义字符。

MySQL 支持代理用户的概念，即客户端可以使用一个账户连接和认证到 MySQL 服务器，但在连接时具有另一个账户的权限（参见 Section 8.2.19, “Proxy Users”）。假设您希望 Windows 用户使用单个用户名连接，但根据其 Windows 用户和组名称映射到特定的 MySQL 账户，如下所示：

+   `local_user` 和 `MyDomain\domain_user` 本地和域 Windows 用户应映射到 `local_wlad` MySQL 账户。

+   `MyDomain\Developers` 域组中的用户应映射到 `local_dev` MySQL 账户。

+   本地机器管理员应映射到 `local_admin` MySQL 账户。

要设置这个，为 Windows 用户创建一个代理账户进行连接，并配置此账户，使用户和组映射到适当的 MySQL 账户（`local_wlad`、`local_dev`、`local_admin`）。此外，授予 MySQL 账户执行所需操作的适当权限。以下说明使用 `win_proxy` 作为代理账户，`local_wlad`、`local_dev` 和 `local_admin` 作为代理账户。

1.  创建代理 MySQL 账户：

    ```sql
    CREATE USER win_proxy
      IDENTIFIED WITH  authentication_windows
      AS 'local_user = local_wlad,
          MyDomain\\domain_user = local_wlad,
          MyDomain\\Developers = local_dev,
          BUILTIN\\Administrators = local_admin';
    ```

1.  为了使代理工作，代理账户必须存在，因此创建它们：

    ```sql
    CREATE USER local_wlad
      IDENTIFIED WITH mysql_no_login;
    CREATE USER local_dev
      IDENTIFIED WITH mysql_no_login;
    CREATE USER local_admin
      IDENTIFIED WITH mysql_no_login;
    ```

    代理账户使用 `mysql_no_login` 认证插件，以防止客户端直接使用这些账户登录到 MySQL 服务器。相反，使用 Windows 进行身份验证的用户应该使用 `win_proxy` 代理账户。（这假定插件已安装。有关说明，请参见 Section 8.4.1.9, “No-Login Pluggable Authentication”。）有关保护代理账户免受直接使用的替代方法，请参见 Preventing Direct Login to Proxied Accounts。

    您还应该执行 `GRANT` 语句（未显示），为每个代理账户授予所需的 MySQL 访问权限。

1.  为代理账户授予 `PROXY` 权限，以代表每个代理账户：

    ```sql
    GRANT PROXY ON local_wlad TO win_proxy;
    GRANT PROXY ON local_dev TO win_proxy;
    GRANT PROXY ON local_admin TO win_proxy;
    ```

现在，Windows 用户`local_user`和`MyDomain\domain_user`可以作为`win_proxy`连接到 MySQL 服务器，并在经过身份验证后具有身份验证字符串中指定帐户（在本例中为`local_wlad`）的权限。以`win_proxy`身份连接的`MyDomain\Developers`组中的用户具有`local_dev`帐户的权限。`BUILTIN\Administrators`组中的用户具有`local_admin`帐户的权限。

要配置身份验证，使所有没有自己的 MySQL 帐户的 Windows 用户通过代理帐户进行身份验证，请在上述说明中将默认代理帐户（`''@''`）替换为`win_proxy`。有关默认代理帐户的信息，请参阅 Section 8.2.19, “Proxy Users”。

注意

如果您的 MySQL 安装中存在匿名用户，则它们可能与默认代理用户发生冲突。有关此问题的更多信息以及处理方法，请参阅 Default Proxy User and Anonymous User Conflicts。

要在 Connector/NET 8.0 及更高版本中使用 Windows 身份验证插件与 Connector/NET 连接字符串，请参阅 Connector/NET Authentication。
