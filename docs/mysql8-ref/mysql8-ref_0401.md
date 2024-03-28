> 原文：[`dev.mysql.com/doc/refman/8.0/en/ldap-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/ldap-pluggable-authentication.html)

#### 8.4.1.7 LDAP 可插拔身份验证

注意

LDAP 可插拔身份验证是 MySQL 企业版中包含的扩展，是一种商业产品。要了解更多关于商业产品的信息，请参见[`www.mysql.com/products/`](https://www.mysql.com/products/)。

MySQL 企业版支持一种身份验证方法，使 MySQL 服务器能够使用 LDAP（轻量级目录访问协议）通过访问目录服务（如 X.500）对 MySQL 用户进行身份验证。MySQL 使用 LDAP 获取用户、凭据和组信息。

LDAP 可插拔身份验证提供以下功能：

+   外部身份验证：LDAP 身份验证使 MySQL 服务器能够接受来自 LDAP 目录中定义的用户的连接，而不是在 MySQL 授权表中定义的用户。

+   代理用户支持：LDAP 身份验证可以根据外部用户所属的 LDAP 组返回一个与客户端程序传递的外部用户名不同的 MySQL 用户名给 MySQL，这意味着 LDAP 插件可以返回定义外部 LDAP 身份验证用户应具有的特权的 MySQL 用户。例如，名为`joe`的 LDAP 用户可以连接并具有名为`developer`的 MySQL 用户的特权，如果`joe`的 LDAP 组是`developer`。

+   安全性：使用 TLS，连接到 LDAP 服务器可以是安全的。

服务器端和客户端插件可用于简单和基于 SASL 的 LDAP 身份验证。在 Microsoft Windows 上，不支持基于 SASL 的 LDAP 身份验证的服务器端插件，但支持客户端插件。

以下表格显示了简单和基于 SASL 的 LDAP 身份验证的插件和库文件名称。文件名后缀可能在您的系统上有所不同。这些文件必须位于由`plugin_dir`系统变量指定的目录中。

**表 8.22 简单 LDAP 身份验证的插件和库名称**

| 插件或文件 | 插件或文件名称 |
| --- | --- |
| 服务器端插件名称 | `authentication_ldap_simple` |
| 客户端插件名称 | `mysql_clear_password` |
| 库文件名称 | `authentication_ldap_simple.so` |

**表 8.23 基于 SASL 的 LDAP 身份验证的插件和库名称**

| 插件或文件 | 插件或文件名称 |
| --- | --- |
| 服务器端插件名称 | `authentication_ldap_sasl` |
| 客户端插件名称 | `authentication_ldap_sasl_client` |
| 库文件名称 | `authentication_ldap_sasl.so`，`authentication_ldap_sasl_client.so` |

库文件仅包含`authentication_ldap_*`XXX`*`身份验证插件。客户端`mysql_clear_password`插件内置于`libmysqlclient`客户端库中。

每个服务器端 LDAP 插件与特定的客户端插件配合使用：

+   服务器端的`authentication_ldap_simple`插件执行简单的 LDAP 认证。对于使用此插件的帐户连接，客户端程序使用客户端的`mysql_clear_password`插件，该插件将密码以明文形式发送到服务器。不使用密码哈希或加密，因此建议在 MySQL 客户端和服务器之间建立安全连接，以防止密码泄露。

+   服务器端的`authentication_ldap_sasl`插件执行基于 SASL 的 LDAP 认证。对于使用此插件的帐户连接，客户端程序使用客户端的`authentication_ldap_sasl_client`插件。客户端端和服务器端的 SASL LDAP 插件使用 SASL 消息来在 LDAP 协议中安全传输凭据，以避免在 MySQL 客户端和服务器之间发送明文密码。

    注意

    在 Microsoft Windows 上，不支持基于 SASL 的 LDAP 认证的服务器插件，但支持客户端插件。在其他平台上，服务器和客户端插件都受支持。

服务器端的 LDAP 认证插件仅包含在 MySQL 企业版中。它们不包含在 MySQL 社区发行版中。客户端端的 SASL LDAP 插件包含在所有发行版中，包括社区发行版，并且如前所述，客户端的`mysql_clear_password`插件内置于`libmysqlclient`客户端库中，该库也包含在所有发行版中。这使得来自任何发行版的客户端都可以连接到加载了适当服务器端插件的服务器。

以下各节提供了特定于 LDAP 可插拔认证的安装和使用信息：

+   LDAP 可插拔认证的先决条件

+   MySQL 用户的 LDAP 认证工作原理

+   安装 LDAP 可插拔认证

+   卸载 LDAP 可插拔认证

+   LDAP 可插拔认证和 ldap.conf

+   使用 LDAP 可插拔认证

+   简单 LDAP 认证

+   基于 SASL 的 LDAP 认证

+   代理 LDAP 认证

+   LDAP 认证组偏好和映射规范

+   LDAP 认证用户 DN 后缀

+   LDAP 认证方法

+   GSSAPI/Kerberos 认证方法

+   LDAP 搜索引荐

有关 MySQL 中可插拔认证的一般信息，请参见第 8.2.17 节，“可插拔认证”。有关`mysql_clear_password`插件的信息，请参见第 8.4.1.4 节，“客户端明文可插拔认证”。有关代理用户信息，请参见第 8.2.19 节，“代理用户”。

注意

如果您的系统支持 PAM 并允许 LDAP 作为 PAM 认证方法，另一种使用 LDAP 进行 MySQL 用户认证的方法是使用服务器端`authentication_pam`插件。请参见第 8.4.1.5 节，“PAM 可插拔认证”。

##### LDAP 可插拔认证的先决条件

要为 MySQL 使用 LDAP 可插拔认证，必须满足以下先决条件：

+   必须有 LDAP 服务器可用，以便 LDAP 认证插件与之通信。

+   要由 MySQL 进行认证的 LDAP 用户必须存在于 LDAP 服务器管理的目录中。

+   在使用服务器端`authentication_ldap_sasl`或`authentication_ldap_simple`插件的系统上必须有 LDAP 客户端库可用。目前支持的库是 Windows 本机 LDAP 库，或非 Windows 系统上的 OpenLDAP 库。

+   要使用基于 SASL 的 LDAP 认证：

    +   LDAP 服务器必须配置为与 SASL 服务器通信。

    +   在使用客户端端`authentication_ldap_sasl_client`插件的系统上必须有 SASL 客户端库可用。目前，唯一支持的库是 Cyrus SASL 库。

    +   要使用特定的 SASL 认证方法，必须提供该方法所需的任何其他服务。例如，要使用 GSSAPI/Kerberos，必须提供 GSSAPI 库和 Kerberos 服务。

##### MySQL 用户的 LDAP 认证工作原理

本节概述了 MySQL 和 LDAP 如何共同工作以对 MySQL 用户进行认证。有关如何设置 MySQL 账户以使用特定 LDAP 认证插件的示例，请参见使用 LDAP 可插拔认证。有关 LDAP 插件可用的认证方法的信息，请参见 LDAP 认证方法。

客户端连接到 MySQL 服务器，提供 MySQL 客户端用户名和密码：

+   对于简单的 LDAP 认证，客户端和服务器端插件以明文形式传输密码。建议在 MySQL 客户端和服务器之间建立安全连接，以防止密码泄露。

+   对于基于 SASL 的 LDAP 认证，客户端和服务器端插件避免在 MySQL 客户端和服务器之间发送明文密码。例如，插件可能使用 SASL 消息来在 LDAP 协议内安全传输凭据。对于 GSSAPI 认证方法，客户端和服务器端插件使用 Kerberos 进行安全通信，而不直接使用 LDAP 消息。

如果客户端用户名和主机名与任何 MySQL 账户不匹配，则连接将被拒绝。

如果存在匹配的 MySQL 账户，则进行 LDAP 认证。LDAP 服务器查找与用户匹配的条目，并根据 LDAP 密码对该条目进行认证：

+   如果 MySQL 账户指定了 LDAP 用户的区分名称（DN），则 LDAP 认证将使用该值和客户端提供的 LDAP 密码。（要将 LDAP 用户 DN 与 MySQL 账户关联，包括在创建账户的`CREATE USER`语句中指定认证字符串的`BY`子句。）

+   如果 MySQL 账户名称不是 LDAP 用户 DN，则 LDAP 认证使用客户端提供的用户名和 LDAP 密码。在这种情况下，认证插件首先使用根 DN 和密码作为凭据绑定到 LDAP 服务器，以根据客户端用户名找到用户 DN，然后根据 LDAP 密码对该用户 DN 进行认证。如果根凭据的绑定失败，说明根 DN 和密码设置为不正确的值，或为空（未设置），且 LDAP 服务器不允许匿名连接。

如果 LDAP 服务器找不到匹配项或找到多个匹配项，则认证失败，客户端连接将被拒绝。

如果 LDAP 服务器找到单个匹配项，则 LDAP 认证成功（假设密码正确），LDAP 服务器返回 LDAP 条目，认证插件根据该条目确定经过认证的用户名称：

+   如果 LDAP 条目具有组属性（默认为`cn`属性），插件将其值作为经过认证的用户名返回。

+   如果 LDAP 条目没有组属性，认证插件将返回客户端用户名作为经过认证的用户名。

MySQL 服务器将客户端用户名与经过认证的用户名进行比较，以确定客户端会话是否发生代理：

+   如果名称相同，则不会发生代理：将用于权限检查的与客户端用户名匹配的 MySQL 帐户。

+   如果名称不同，将发生代理：MySQL 将寻找与经过认证的用户名匹配的帐户。该帐户将成为代理用户，用于权限检查。与客户端用户名匹配的 MySQL 帐户将被视为外部代理用户。

##### 安装 LDAP 可插入认证

本节描述了如何安装服务器端 LDAP 认证插件。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

服务器端插件库文件的基本名称为`authentication_ldap_simple`和`authentication_ldap_sasl`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为`.so`，对于 Windows 为`.dll`）。

注意

在 Microsoft Windows 上，不支持基于 SASL 的 LDAP 认证的服务器插件，但支持客户端插件。在其他平台上，服务器和客户端插件都受支持。

要在服务器启动时加载插件，使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，选项必须在每次服务器启动时给出。还要为您希望配置的任何插件提供的系统变量指定值。

每个服务器端 LDAP 插件都公开一组系统变量，以使其操作可以配置。设置大多数这些变量是可选的，但您必须设置指定 LDAP 服务器主机（以便插件知道在哪里连接）和 LDAP 绑定操作的基本区分名称（以限制搜索范围并获得更快的搜索）的变量。有关所有 LDAP 系统变量的详细信息，请参见第 8.4.1.13 节，“可插入认证系统变量”。

要加载插件并设置 LDAP 服务器主机和 LDAP 绑定操作的基本区分名称，需要在您的`my.cnf`文件中添加类似以下的行，根据需要调整`.so`后缀以适应您的平台：

```sql
[mysqld]
plugin-load-add=authentication_ldap_simple.so
authentication_ldap_simple_server_host=127.0.0.1
authentication_ldap_simple_bind_base_dn="dc=example,dc=com"
plugin-load-add=authentication_ldap_sasl.so
authentication_ldap_sasl_server_host=127.0.0.1
authentication_ldap_sasl_bind_base_dn="dc=example,dc=com"
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用以下语句，根据需要调整您平台的`.so`后缀：

```sql
INSTALL PLUGIN authentication_ldap_simple
  SONAME 'authentication_ldap_simple.so';
INSTALL PLUGIN authentication_ldap_sasl
  SONAME 'authentication_ldap_sasl.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

在运行时安装插件后，它们公开的系统变量变得可用，您可以将其设置添加到您的`my.cnf`文件中，以配置插件以供后续重新启动。例如：

```sql
[mysqld]
authentication_ldap_simple_server_host=127.0.0.1
authentication_ldap_simple_bind_base_dn="dc=example,dc=com"
authentication_ldap_sasl_server_host=127.0.0.1
authentication_ldap_sasl_bind_base_dn="dc=example,dc=com"
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

要在运行时设置和持久化每个值，而不是在启动时，请使用以下语句：

```sql
SET PERSIST authentication_ldap_simple_server_host='127.0.0.1';
SET PERSIST authentication_ldap_simple_bind_base_dn='dc=example,dc=com';
SET PERSIST authentication_ldap_sasl_server_host='127.0.0.1';
SET PERSIST authentication_ldap_sasl_bind_base_dn='dc=example,dc=com';
```

`SET PERSIST`为运行中的 MySQL 实例设置一个值。它还保存该值，导致其在后续服务器重新启动时保留。要更改运行中的 MySQL 实例的值，而不使其在后续重新启动时保留，请使用`GLOBAL`关键字而不是`PERSIST`。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。

要验证插件安装，请检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%ldap%';
+----------------------------+---------------+
| PLUGIN_NAME                | PLUGIN_STATUS |
+----------------------------+---------------+
| authentication_ldap_sasl   | ACTIVE        |
| authentication_ldap_simple | ACTIVE        |
+----------------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

要将 MySQL 帐户与 LDAP 插件关联，请参阅使用 LDAP 可插拔认证。

SELinux 的附加说明

在运行 EL6 或启用 SELinux 的 EL 系统上，需要更改 SELinux 策略以启用 MySQL LDAP 插件与 LDAP 服务通信：

1.  创建一个包含以下内容的文件`mysqlldap.te`：

    ```sql
    module mysqlldap 1.0;

    require {
            type ldap_port_t;
            type mysqld_t;
            class tcp_socket name_connect;
    }

    #============= mysqld_t ==============

    allow mysqld_t ldap_port_t:tcp_socket name_connect;
    ```

1.  将安全策略模块编译为二进制表示：

    ```sql
    checkmodule -M -m mysqlldap.te -o mysqlldap.mod
    ```

1.  创建一个 SELinux 策略模块包：

    ```sql
    semodule_package -m mysqlldap.mod  -o mysqlldap.pp
    ```

1.  安装模块包：

    ```sql
    semodule -i mysqlldap.pp
    ```

1.  当 SELinux 策略更改完成后，重新启动 MySQL 服务器：

    ```sql
    service mysqld restart
    ```

##### 卸载 LDAP 可插拔认证

卸载 LDAP 认证插件的方法取决于您如何安装它们：

+   如果您在服务器启动时使用`--plugin-load-add`选项安装了插件，请在没有这些选项的情况下重新启动服务器。

+   如果您在运行时使用 `INSTALL PLUGIN` 安装了插件，则它们将在服务器重新启动时保持安装状态。要卸载它们，请使用 `UNINSTALL PLUGIN`：

    ```sql
    UNINSTALL PLUGIN authentication_ldap_simple;
    UNINSTALL PLUGIN authentication_ldap_sasl;
    ```

另外，从您的 `my.cnf` 文件中删除任何设置 LDAP 插件相关系统变量的启动选项。如果您使用 `SET PERSIST` 来持久化 LDAP 系统变量，请使用 `RESET PERSIST` 来删除这些设置。

##### LDAP 可插拔认证和 ldap.conf

对于使用 OpenLDAP 的安装，`ldap.conf` 文件为 LDAP 客户端提供全局默认值。可以在此文件中设置选项以影响 LDAP 客户端，包括 LDAP 认证插件。OpenLDAP 使用以下优先顺序的配置选项：

+   由 LDAP 客户端指定的配置。

+   在 `ldap.conf` 文件中指定的配置。要禁用此文件的使用，请设置 `LDAPNOINIT` 环境变量。

+   OpenLDAP 库内置默认值。

如果库默认值或 `ldap.conf` 值不能产生适当的选项值，则 LDAP 认证插件可能能够设置相关变量以直接影响 LDAP 配置。例如，LDAP 插件可以覆盖 `ldap.conf` 中的参数，如：

+   TLS 配置：系统变量可用于启用 TLS 并控制 CA 配置，例如 `authentication_ldap_simple_tls` 和 `authentication_ldap_simple_ca_path` 用于简单 LDAP 认证，以及 `authentication_ldap_sasl_tls` 和 `authentication_ldap_sasl_ca_path` 用于 SASL LDAP 认证。

+   LDAP 引荐。请参阅 LDAP 搜索引荐。

有关 `ldap.conf` 的更多信息，请参阅 `ldap.conf(5)` 手册页。

##### 使用 LDAP 可插拔认证

本节描述了如何启用 MySQL 帐户使用 LDAP 可插拔认证连接到 MySQL 服务器。假设服务器正在运行，并启用了适当的服务器端插件，如安装 LDAP 可插拔认证中所述，并且客户端主机上有适当的客户端插件可用。

本节不描述 LDAP 配置或管理。假定您已熟悉这些主题。

两个服务器端 LDAP 插件分别与特定的客户端端插件配合使用：

+   服务器端`authentication_ldap_simple`插件执行简单的 LDAP 身份验证。 对于使用此插件的帐户的连接，客户端程序使用客户端端`mysql_clear_password`插件，该插件将密码以明文形式发送到服务器。 不使用密码哈希或加密，因此建议在 MySQL 客户端和服务器之间建立安全连接，以防止密码泄露。

+   服务器端`authentication_ldap_sasl`插件执行基于 SASL 的 LDAP 身份验证。 对于使用此插件的帐户的连接，客户端程序使用客户端端`authentication_ldap_sasl_client`插件。 客户端端和服务器端 SASL LDAP 插件使用 SASL 消息来在 LDAP 协议中安全传输凭据，以避免在 MySQL 客户端和服务器之间发送明文密码。

MySQL 用户 LDAP 身份验证的总体要求：

+   每个要进行身份验证的用户必须有一个 LDAP 目录条目。

+   必须有一个 MySQL 用户帐户，指定一个服务器端 LDAP 身份验证插件，并可选择命名相关的 LDAP 用户专有名称（DN）。 （要将 LDAP 用户 DN 与 MySQL 帐户关联，包括在创建帐户的`CREATE USER`语句中包含一个`BY`子句。）如果帐户未命名任何 LDAP 字符串，则 LDAP 身份验证将使用客户端指定的用户名查找 LDAP 条目。

+   客户端程序使用适用于 MySQL 帐户使用的服务器端身份验证插件的连接方法进行连接。 对于 LDAP 身份验证，连接需要 MySQL 用户名和 LDAP 密码。 此外，对于使用服务器端`authentication_ldap_simple`插件的帐户，使用`--enable-cleartext-plugin`选项调用客户端程序以启用客户端端`mysql_clear_password`插件。

此处的说明假定以下场景：

+   MySQL 用户`betsy`和`boris`分别对`betsy_ldap`和`boris_ldap`的 LDAP 条目进行身份验证。（MySQL 和 LDAP 用户名不必不同。 在本讨论中使用不同名称有助于澄清操作上下文是 MySQL 还是 LDAP。）

+   LDAP 条目使用`uid`属性来指定用户名。 这可能会根据 LDAP 服务器而变化。 一些 LDAP 服务器使用`cn`属性而不是`uid`来表示用户名。 要更改属性，请适当修改`authentication_ldap_simple_user_search_attr`或`authentication_ldap_sasl_user_search_attr`系统变量。

+   这些 LDAP 条目可在 LDAP 服务器管理的目录中找到，以提供唯一标识每个用户的专有名称值：

    ```sql
    uid=betsy_ldap,ou=People,dc=example,dc=com
    uid=boris_ldap,ou=People,dc=example,dc=com
    ```

+   `CREATE USER` 语句创建 MySQL 账户时，在 `BY` 子句中指定 LDAP 用户，以指示 MySQL 账户进行身份验证的 LDAP 条目。

使用 LDAP 认证设置帐户的说明取决于使用的服务器端 LDAP 插件。 以下各节描述了几种使用场景。

##### 简单 LDAP 认证

要为简单 LDAP 认证配置 MySQL 账户，`CREATE USER` 语句指定 `authentication_ldap_simple` 插件，并可选择命名 LDAP 用户的区分名称（DN）：

```sql
CREATE USER *user*
  IDENTIFIED WITH authentication_ldap_simple
  [BY '*LDAP user DN*'];
```

假设 MySQL 用户 `betsy` 在 LDAP 目录中有以下条目：

```sql
uid=betsy_ldap,ou=People,dc=example,dc=com
```

然后创建 `betsy` 的 MySQL 账户的语句如下：

```sql
CREATE USER 'betsy'@'localhost'
  IDENTIFIED WITH authentication_ldap_simple
  AS 'uid=betsy_ldap,ou=People,dc=example,dc=com';
```

在 `BY` 子句中指定的认证字符串不包括 LDAP 密码。 客户端用户必须在连接时提供密码。

客户端通过提供 MySQL 用户名和 LDAP 密码连接到 MySQL 服务器，并启用客户端端 `mysql_clear_password` 插件：

```sql
$> mysql --user=betsy --password --enable-cleartext-plugin
Enter password: *betsy_password* *(betsy_ldap LDAP password)*
```

注意

客户端端 `mysql_clear_password` 认证插件不会更改密码，因此客户端程序将其作为明文发送到 MySQL 服务器。 这使得密码可以原样传递到 LDAP 服务器。 在没有 SASL 的情况下使用服务器端 LDAP 库需要明文密码，但在某些配置中可能存在安全问题。 这些措施最小化了风险：

+   为了减少意外使用 `mysql_clear_password` 插件的可能性，MySQL 客户端必须显式启用它（例如，使用 `--enable-cleartext-plugin` 选项）。 请参见 Section 8.4.1.4, “Client-Side Cleartext Pluggable Authentication”。

+   为了避免启用 `mysql_clear_password` 插件时密码暴露，MySQL 客户端应该使用加密连接连接到 MySQL 服务器。 请参见 Section 8.3.1, “Configuring MySQL to Use Encrypted Connections”。

认证过程如下进行：

1.  客户端插件将 `betsy` 和 *`betsy_password`* 作为客户端用户名和 LDAP 密码发送到 MySQL 服务器。

1.  连接尝试匹配 `'betsy'@'localhost'` 账户。 服务器端 LDAP 插件发现该账户具有一个认证字符串 `'uid=betsy_ldap,ou=People,dc=example,dc=com'` 用于命名 LDAP 用户 DN。 该插件将此字符串和 LDAP 密码发送到 LDAP 服务器。

1.  LDAP 服务器找到了 `betsy_ldap` 的 LDAP 条目，并且密码匹配，因此 LDAP 认证成功。

1.  LDAP 条目没有组属性，因此服务器端插件将客户端用户名（`betsy`）作为经过身份验证的用户返回。这与客户端提供的相同用户名，因此不会发生代理，并且客户端会话使用`'betsy'@'localhost'`账户进行权限检查。

如果匹配的 LDAP 条目包含组属性，该属性值将成为经过身份验证的用户名，并且如果该值与`betsy`不同，则将发生代理。有关使用组属性的示例，请参阅具有代理功能的 LDAP 认证。

如果`CREATE USER`语句中没有包含`BY`子句来指定`betsy_ldap` LDAP 专有名称，认证尝试将使用客户端提供的用户名（在本例中为`betsy`）。在没有`betsy`的 LDAP 条目的情况下，认证将失败。

##### 基于 SASL 的 LDAP 认证

要为 SASL LDAP 认证配置 MySQL 账户，`CREATE USER` 语句指定`authentication_ldap_sasl`插件，并可选择命名 LDAP 用户的专有名称（DN）：

```sql
CREATE USER *user*
  IDENTIFIED WITH authentication_ldap_sasl
  [BY '*LDAP user DN*'];
```

假设 MySQL 用户`boris`在 LDAP 目录中有以下条目：

```sql
uid=boris_ldap,ou=People,dc=example,dc=com
```

然后创建`boris`的 MySQL 账户的语句如下：

```sql
CREATE USER 'boris'@'localhost'
  IDENTIFIED WITH authentication_ldap_sasl
  AS 'uid=boris_ldap,ou=People,dc=example,dc=com';
```

`BY`子句中指定的认证字符串不包括 LDAP 密码。这必须由客户端用户在连接时提供。

客户端通过提供 MySQL 用户名和 LDAP 密码连接到 MySQL 服务器：

```sql
$> mysql --user=boris --password
Enter password: *boris_password* *(boris_ldap LDAP password)*
```

对于服务器端的`authentication_ldap_sasl`插件，客户端使用客户端的`authentication_ldap_sasl_client`插件。如果客户端程序找不到客户端插件，请指定一个`--plugin-dir`选项，指定安装插件库文件的目录。

`boris`的认证过程与之前描述的`betsy`的简单 LDAP 认证类似，只是客户端和服务器端的 SASL LDAP 插件使用 SASL 消息来在 LDAP 协议中安全传输凭据，以避免在 MySQL 客户端和服务器之间发送明文密码。

##### 具有代理功能的 LDAP 认证

LDAP 认证插件支持代理，使用户可以以一个用户连接到 MySQL 服务器，但假定另一个用户的权限。本节描述了基本的 LDAP 插件代理支持。LDAP 插件还支持指定组偏好和代理用户映射；请参阅 LDAP 认证组偏好和映射规范。

此处描述的代理实现基于使用 LDAP 组属性值将通过 LDAP 进行身份验证的连接 MySQL 用户映射到定义不同权限集的其他 MySQL 账户。用户不会直接通过定义权限的账户连接。相反，他们通过使用 LDAP 进行身份验证的默认代理账户连接，以便将所有外部登录映射到持有权限的代理 MySQL 账户。通过代理账户连接的任何用户都将映射到其中一个代理 MySQL 账户，其权限确定了允许外部用户执行的数据库操作。

此处的说明假定以下场景：

+   LDAP 条目使用`uid`和`cn`属性分别指定用户名和组值。要使用不同的用户和组属性名称，请设置适当的插件特定系统变量：

    +   对于`authentication_ldap_simple`插件：设置`authentication_ldap_simple_user_search_attr`和`authentication_ldap_simple_group_search_attr`。

    +   对于`authentication_ldap_sasl`插件：设置`authentication_ldap_sasl_user_search_attr`和`authentication_ldap_sasl_group_search_attr`。

+   这些 LDAP 条目可在由 LDAP 服务器管理的目录中找到，以提供唯一标识每个用户的可区分名称值：

    ```sql
    uid=basha,ou=People,dc=example,dc=com,cn=accounting
    uid=basil,ou=People,dc=example,dc=com,cn=front_office
    ```

    在连接时，组属性值成为经过身份验证的用户名称，因此它们命名了`accounting`和`front_office`代理账户。

+   示例假定使用 SASL LDAP 身份验证。对于简单 LDAP 身份验证，请进行适当调整。

创建默认的代理 MySQL 账户：

```sql
CREATE USER ''@'%'
  IDENTIFIED WITH authentication_ldap_sasl;
```

代理账户定义中没有`AS '*`auth_string`*'`子句来命名 LDAP 用户 DN。因此：

+   当客户端连接时，客户端用户名将成为要搜索的 LDAP 用户名。

+   预期匹配的 LDAP 条目将包括命名定义客户端应具有的权限的代理 MySQL 账户的组属性。

注意

如果您的 MySQL 安装中存在匿名用户，则可能会与默认代理用户发生冲突。有关此问题的更多信息以及处理方法，请参阅默认代理用户和匿名用户冲突。

创建代理账户并授予每个账户应具有的权限：

```sql
CREATE USER 'accounting'@'localhost'
  IDENTIFIED WITH mysql_no_login;
CREATE USER 'front_office'@'localhost'
  IDENTIFIED WITH mysql_no_login;

GRANT ALL PRIVILEGES
  ON accountingdb.*
  TO 'accounting'@'localhost';
GRANT ALL PRIVILEGES
  ON frontdb.*
  TO 'front_office'@'localhost';
```

代理帐户使用`mysql_no_login`身份验证插件防止客户端直接使用帐户登录到 MySQL 服务器。相反，使用 LDAP 进行身份验证的用户应该使用默认的`''@'%'`代理帐户。（这假定安装了`mysql_no_login`插件。有关说明，请参见 Section 8.4.1.9, “No-Login Pluggable Authentication”。）有关保护代理帐户免受直接使用的替代方法，请参见 Preventing Direct Login to Proxied Accounts。

为每个代理帐户授予`PROXY`权限：

```sql
GRANT PROXY
  ON 'accounting'@'localhost'
  TO ''@'%';
GRANT PROXY
  ON 'front_office'@'localhost'
  TO ''@'%';
```

使用**mysql**命令行客户端连接到 MySQL 服务器作为`basha`。

```sql
$> mysql --user=basha --password
Enter password: *basha_password* *(basha LDAP password)*
```

认证过程如下：

1.  服务器使用默认的`''@'%'`代理帐户对客户端用户`basha`进行连接认证。

1.  匹配的 LDAP 条目是：

    ```sql
    uid=basha,ou=People,dc=example,dc=com,cn=accounting
    ```

1.  匹配的 LDAP 条目具有组属性`cn=accounting`，因此`accounting`成为认证的代理用户。

1.  认证用户与客户端用户名`basha`不同，导致`basha`被视为`accounting`的代理，并且`basha`承担了代理`accounting`帐户的权限。以下查询返回如下输出：

    ```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
    +-----------------+----------------------+--------------+
    | USER()          | CURRENT_USER()       | @@proxy_user |
    +-----------------+----------------------+--------------+
    | basha@localhost | accounting@localhost | ''@'%'       |
    +-----------------+----------------------+--------------+
    ```

这表明`basha`使用授予代理`accounting` MySQL 帐户的权限，并且代理通过默认代理用户帐户进行。

现在改为连接为`basil`：

```sql
$> mysql --user=basil --password
Enter password: *basil_password* *(basil LDAP password)*
```

对于`basil`的身份验证过程与先前描述的`basha`类似：

1.  服务器使用默认的`''@'%'`代理帐户对客户端用户`basil`进行连接认证。

1.  匹配的 LDAP 条目是：

    ```sql
    uid=basil,ou=People,dc=example,dc=com,cn=front_office
    ```

1.  匹配的 LDAP 条目具有组属性`cn=front_office`，因此`front_office`成为认证的代理用户。

1.  认证用户与客户端用户名`basil`不同，导致`basil`被视为`front_office`的代理，并且`basil`承担了代理`front_office`帐户的权限。以下查询返回如下输出：

    ```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
    +-----------------+------------------------+--------------+
    | USER()          | CURRENT_USER()         | @@proxy_user |
    +-----------------+------------------------+--------------+
    | basil@localhost | front_office@localhost | ''@'%'       |
    +-----------------+------------------------+--------------+
    ```

这表明`basil`使用授予代理`front_office` MySQL 帐户的权限，并且代理通过默认代理用户帐户进行。

##### LDAP 认证组首选项和映射规范

如 LDAP 代理认证中所述，基本的 LDAP 认证代理工作原理是插件使用 LDAP 服务器返回的第一个组名作为 MySQL 代理用户账户名。这种简单的功能不允许指定任何偏好，关于如果 LDAP 服务器返回多个组名该使用哪个，或者指定任何名称而不是组名作为代理用户名称。

截至 MySQL 8.0.14，对于使用 LDAP 认证的 MySQL 账户，认证字符串可以指定以下信息以实现更大的代理灵活性：

+   一组按照偏好顺序排列的组，使得插件使用列表中与 LDAP 服务器返回的组匹配的第一个组名。

+   从组名到代理用户名称的映射，使得匹配的组名可以提供指定的名称作为代理用户使用。这提供了一个替代方案，不使用组名作为代理用户。

考虑以下 MySQL 代理账户定义：

```sql
CREATE USER ''@'%'
  IDENTIFIED WITH authentication_ldap_sasl
  AS '+ou=People,dc=example,dc=com#grp1=usera,grp2,grp3=userc';
```

认证字符串有一个以`+`字符为前缀的用户 DN 后缀`ou=People,dc=example,dc=com`。因此，如 LDAP 认证用户 DN 后缀中所述，完整的用户 DN 是由指定的用户 DN 后缀加上客户端用户名作为`uid`属性构建而成。

认证字符串的剩余部分以`#`开头，表示组偏好和映射信息的开始。认证字符串的这部分按照`grp1`、`grp2`、`grp3`的顺序列出组名。LDAP 插件将该列表与 LDAP 服务器返回的组名集合进行比较，按照列表顺序查找与返回的名称匹配的组。插件使用第一个匹配项，如果没有匹配项，则认证失败。

假设 LDAP 服务器返回组`grp3`、`grp2`和`grp7`。LDAP 插件使用`grp2`，因为它是认证字符串中第一个匹配的组，即使它不是 LDAP 服务器返回的第一个组。如果 LDAP 服务器返回`grp4`、`grp2`和`grp1`，插件使用`grp1`，即使`grp2`也匹配。`grp1`的优先级高于`grp2`，因为它在认证字符串中较早列出。

假设插件找到组名匹配，它将从该组名映射到 MySQL 代理用户名称，如果有的话。对于示例代理账户，映射如下进行：

+   如果匹配的组名是`grp1`或`grp3`，它们在认证字符串中与用户名称`usera`和`userc`相关联。插件使用相应的关联用户名称作为代理用户名称。

+   如果匹配的组名是 `grp2`，认证字符串中没有关联的用户名。插件使用 `grp2` 作为代理用户名称。

如果 LDAP 服务器以 DN 格式返回组，LDAP 插件会解析组 DN 以从中提取组名。

要指定 LDAP 组偏好和映射信息，应遵循以下原则：

+   用 `#` 前缀字符开始认证字符串的组偏好和映射部分。

+   组偏好和映射规范是由逗号分隔的一个或多个项目的列表。每个项目的形式为 `*`group_name`*=*`user_name`*` 或 *`group_name`*。项目应按组名偏好顺序列出。对于插件从 LDAP 服务器返回的组名集合中选择的组名，两种语法在效果上有所不同：

    +   对于指定为 `*`group_name`*=*`user_name`*`（带有用户名）的项目，组名映射到用户名，该用户名用作 MySQL 代理用户名称。

    +   对于指定为 *`group_name`*（没有用户名）的项目，组名用作 MySQL 代理用户名称。

+   要引用包含特殊字符（如空格）的组或用户名，用双引号 (`"`) 括起来。例如，如果一个项目的组名和用户名分别为 `my group name` 和 `my user name`，则必须在组映射中使用引号：

    ```sql
    "my group name"="my user name"
    ```

    如果一个项目的组名和用户名分别为 `my_group_name` 和 `my_user_name`（不包含特殊字符），可以但不必使用引号。以下任何一种都是有效的：

    ```sql
    my_group_name=my_user_name
    my_group_name="my_user_name"
    "my_group_name"=my_user_name
    "my_group_name"="my_user_name"
    ```

+   要转义字符，请在其前面加上反斜杠 (`\`)。这对于包含文字双引号或反斜杠（否则不会被字面包含）特别有用。

+   用户 DN 不一定要出现在认证字符串中，但如果出现，必须在组偏好和映射部分之前。用户 DN 可以作为完整用户 DN 给出，也可以作为带有 `+` 前缀字符的用户 DN 后缀给出。（参见 LDAP 认证用户 DN 后缀.）

##### LDAP 认证用户 DN 后缀

LDAP 认证插件允许提供用户 DN 信息的认证字符串以 `+` 前缀字符开头：

+   在没有 `+` 字符的情况下，认证字符串值将按原样处理，不做修改。

+   如果认证字符串以`+`开头，插件将从客户端发送的用户名和认证字符串中指定的 DN（去除`+`）构造完整的用户 DN 值。在构造的 DN 中，客户端用户名成为指定 LDAP 用户名的属性值。默认情况下为`uid`；要更改属性，请修改相应的系统变量（`authentication_ldap_simple_user_search_attr`或`authentication_ldap_sasl_user_search_attr`）。认证字符串存储在`mysql.user`系统表中，认证之前动态构造完整的用户 DN。

此帐户认证字符串不以`+`开头，因此被视为完整的用户 DN：

```sql
CREATE USER 'baldwin'
  IDENTIFIED WITH authentication_ldap_simple
  AS 'uid=admin,ou=People,dc=example,dc=com';
```

客户端使用帐户中指定的用户名（`baldwin`）连接。在这种情况下，该名称未被使用，因为认证字符串没有前缀，因此完全指定了用户 DN。

此帐户认证字符串以`+`开头，因此被视为用户 DN 的一部分：

```sql
CREATE USER 'accounting'
  IDENTIFIED WITH authentication_ldap_simple
  AS '+ou=People,dc=example,dc=com';
```

客户端使用帐户中指定的用户名（`accounting`）连接，该用户名与认证字符串一起用作构造用户 DN 的`uid`属性：`uid=accounting,ou=People,dc=example,dc=com`

前面示例中的帐户具有非空用户名，因此客户端始终使用帐户定义中指定的相同名称连接到 MySQL 服务器。如果帐户具有空用户名，例如使用代理进行 LDAP 认证中描述的默认匿名`''@'%'`代理帐户，客户端可能使用不同的用户名连接到 MySQL 服务器。但原则是相同的：如果认证字符串以`+`开头，插件将使用客户端发送的用户名和认证字符串一起构造用户 DN。

##### LDAP 认证方法

LDAP 认证插件使用可配置的认证方法。适当的系统变量和可用的方法选择是特定于插件的：

+   对于`authentication_ldap_simple`插件：设置`authentication_ldap_simple_auth_method_name`系统变量以配置方法。允许的选择是`SIMPLE`和`AD-FOREST`。

+   对于 `authentication_ldap_sasl` 插件：设置 `authentication_ldap_sasl_auth_method_name` 系统变量以配置方法。允许的选择是 `SCRAM-SHA-1`、`SCRAM-SHA-256` 和 `GSSAPI`。（要确定主机系统上实际可用的 SASL LDAP 方法，请检查 `Authentication_ldap_sasl_supported_methods` 状态变量的值。）

有关每种允许方法的信息，请参阅系统变量描述。此外，根据方法的不同，可能需要额外的配置，如下面的部分所述。

##### GSSAPI/Kerberos 认证方法

通用安全服务应用程序接口（GSSAPI）是一个安全抽象接口。Kerberos 是可以通过该抽象接口使用的特定安全协议的一个实例。使用 GSSAPI，应用程序通过 Kerberos 进行身份验证以获取服务凭证，然后再使用这些凭证来实现对其他服务的安全访问。

其中一种服务是 LDAP，它被客户端和服务器端的 SASL LDAP 认证插件使用。当 `authentication_ldap_sasl_auth_method_name` 系统变量设置为 `GSSAPI` 时，这些插件使用 GSSAPI/Kerberos 认证方法。在这种情况下，插件通过 Kerberos 安全通信，而不直接使用 LDAP 消息。服务器端插件然后与 LDAP 服务器通信以解释 LDAP 认证消息并检索 LDAP 组。

GSSAPI/Kerberos 在 Linux 上作为 MySQL 服务器和客户端的 LDAP 认证方法得到支持。在 Linux 环境中，当应用程序通过默认启用 Kerberos 的 Microsoft Active Directory 访问 LDAP 时，这是非常有用的。

以下讨论提供了使用 GSSAPI 方法的配置要求信息。假定熟悉 Kerberos 概念和操作。以下列表简要定义了几个常见的 Kerberos 术语。您还可以在 [RFC 4120](https://tools.ietf.org/html/rfc4120) 的术语表部分找到有用的信息。

+   主体：一个命名实体，比如用户或服务器。

+   KDC：密钥分发中心，包括 AS 和 TGS：

    +   AS：认证服务器；提供获取额外票证所需的初始票证授予票证。

    +   TGS：票证授予服务器；为拥有有效 TGT 的 Kerberos 客户端提供额外票证。

+   TGT：票据授予票据；提交给 TGS 以获取用于服务访问的服务票据。

使用 Kerberos 进行 LDAP 身份验证需要 KDC 服务器和 LDAP 服务器。可以通过不同方式满足此要求：

+   Active Directory 包括两个服务器，Active Directory LDAP 服务器默认启用 Kerberos 身份验证。

+   OpenLDAP 提供了一个 LDAP 服务器，但可能需要一个单独的 KDC 服务器，并需要额外的 Kerberos 设置。

客户端主机上也必须可用 Kerberos。客户端使用密码联系 AS 以获取 TGT。然后，客户端使用 TGT 从 TGS 获取其他服务的访问权限，例如 LDAP。

以下各节讨论了在 MySQL 中使用 GSSAPI/Kerberos 进行 SASL LDAP 身份验证的配置步骤：

+   验证 Kerberos 和 LDAP 的可用性

+   为 GSSAPI/Kerberos 配置服务器端 SASL LDAP 身份验证插件

+   创建一个使用 GSSAPI/Kerberos 进行 LDAP 身份验证的 MySQL 帐户

+   使用 MySQL 帐户连接到 MySQL 服务器

+   LDAP 身份验证的客户端配置参数

###### 验证 Kerberos 和 LDAP 的可用性

以下示例显示如何在 Active Directory 中测试 Kerberos 的可用性。示例做出以下假设：

+   Active Directory 运行在名为`ldap_auth.example.com`的主机上，IP 地址为`198.51.100.10`。

+   与 MySQL 相关的 Kerberos 身份验证和 LDAP 查找使用`MYSQL.LOCAL`域。

+   名为`bredon@MYSQL.LOCAL`的主体已在 KDC 中注册。（在后续讨论中，此主体名称还与使用 GSSAPI/Kerberos 对 MySQL 服务器进行身份验证的 MySQL 帐户相关联。）

在满足这些假设的情况下，按照以下步骤操作：

1.  验证操作系统中 Kerberos 库是否已安装和配置正确。例如，要为 MySQL 身份验证期间使用`MYSQL.LOCAL`域，`/etc/krb5.conf` Kerberos 配置文件应包含类似于以下内容：

    ```sql
    [realms]
     MYSQL.LOCAL = {
     kdc = ldap_auth.example.com
     admin_server = ldap_auth.example.com
     default_domain = MYSQL.LOCAL
      }
    ```

1.  您可能需要为服务器主机在`/etc/hosts`中添加一个条目：

    ```sql
    198.51.100.10 ldap_auth ldap_auth.example.com
    ```

1.  检查 Kerberos 身份验证是否正常工作：

    1.  使用**kinit**进行 Kerberos 身份验证：

        ```sql
        $> kinit bredon@MYSQL.LOCAL
        Password for bredon@MYSQL.LOCAL: *(enter password here)*
        ```

        该命令用于验证名为`bredon@MYSQL.LOCAL`的 Kerberos 主体。当命令提示时，请输入主体的密码。KDC 返回一个 TGT，该 TGT 在客户端缓存中供其他支持 Kerberos 的应用程序使用。

    1.  使用**klist**检查 TGT 是否正确获取。输出应类似于以下内容：

        ```sql
        $> klist
        Ticket cache: FILE:/tmp/krb5cc_244306
        Default principal: bredon@MYSQL.LOCAL

        Valid starting       Expires              Service principal
        03/23/2021 08:18:33  03/23/2021 18:18:33  krbtgt/MYSQL.LOCAL@MYSQL.LOCAL
        ```

1.  使用此命令检查**ldapsearch**是否与 Kerberos TGT 一起工作，该命令在`MYSQL.LOCAL`域中搜索用户：

    ```sql
    ldapsearch -h 198.51.100.10 -Y GSSAPI -b "dc=MYSQL,dc=LOCAL"
    ```

###### 配置服务器端 SASL LDAP 认证插件以使用 GSSAPI/Kerberos

假设 LDAP 服务器可以通过上述方式访问 Kerberos，配置服务器端 SASL LDAP 认证插件以使用 GSSAPI/Kerberos 认证方法。（有关一般 LDAP 插件安装信息，请参阅安装 LDAP 可插入认证。）以下是服务器`my.cnf`文件可能包含的插件相关设���示例：

```sql
[mysqld]
plugin-load-add=authentication_ldap_sasl.so
authentication_ldap_sasl_auth_method_name="GSSAPI"
authentication_ldap_sasl_server_host=198.51.100.10
authentication_ldap_sasl_server_port=389
authentication_ldap_sasl_bind_root_dn="cn=admin,cn=users,dc=MYSQL,dc=LOCAL"
authentication_ldap_sasl_bind_root_pwd="*password*"
authentication_ldap_sasl_bind_base_dn="cn=users,dc=MYSQL,dc=LOCAL"
authentication_ldap_sasl_user_search_attr="sAMAccountName"
```

这些选项文件设置配置了 SASL LDAP 插件如下：

+   `--plugin-load-add`选项加载插件（根据需要调整`.so`后缀以适应您的平台）。如果之前使用`INSTALL PLUGIN`语句加载了插件，则此选项是不必要的。

+   `authentication_ldap_sasl_auth_method_name`必须设置为`GSSAPI`以使用 GSSAPI/Kerberos 作为 SASL LDAP 认证方法。

+   `authentication_ldap_sasl_server_host`和`authentication_ldap_sasl_server_port`指示用于身份验证的 Active Directory 服务器主机的 IP 地址和端口号。

+   `authentication_ldap_sasl_bind_root_dn`和`authentication_ldap_sasl_bind_root_pwd`配置了用于组搜索功能的根 DN 和密码。此功能是必需的，但用户可能没有搜索权限。在这种情况下，有必要提供根 DN 信息：

    +   在 DN 选项值中，`admin`应该是具有执行用户搜索权限的管理 LDAP 帐户的名称。

    +   在密码选项值中，*`password`*应该是`admin`帐户的密码。

+   `authentication_ldap_sasl_bind_base_dn`指示用户 DN 基本路径，以便搜索在`MYSQL.LOCAL`域中的用户。

+   `authentication_ldap_sasl_user_search_attr` 指定了一个标准的 Active Directory 搜索属性，`sAMAccountName`。此属性用于搜索以匹配登录名；属性值与用户 DN 值不同。

###### 创建一个使用 GSSAPI/Kerberos 进行 LDAP 认证的 MySQL 账户

使用 GSSAPI/Kerberos 方法的 SASL LDAP 认证插件进行 MySQL 认证是基于一个作为 Kerberos 主体的用户。以下讨论使用一个名为 `bredon@MYSQL.LOCAL` 的主体作为此用户，必须在多个地方注册：

+   Kerberos 管理员应该将用户名称注册为 Kerberos 主体。此名称应包括域名。客户端使用主体名称和密码进行 Kerberos 认证并获取 TGT。

+   LDAP 管理员应在 LDAP 条目中注册用户名称。例如：

    ```sql
    uid=bredon,dc=MYSQL,dc=LOCAL
    ```

    注意

    在 Active Directory（默认使用 Kerberos 作为认证方法），创建用户会同时创建 Kerberos 主体和 LDAP 条目。

+   MySQL DBA 应创建一个账户，其用户名称为 Kerberos 主体名称，并使用 SASL LDAP 插件进行认证。

假设适当的服务管理员已经注册了 Kerberos 主体和 LDAP 条目，并且，如前面在 Installing LDAP Pluggable Authentication 和 Configure the Server-Side SASL LDAP Authentication Plugin for GSSAPI/Kerberos 中描述的那样，MySQL 服务器已经以适当的配置设置启动了服务器端 SASL LDAP 插件。然后 MySQL DBA 创建一个对应于 Kerberos 主体名称的 MySQL 账户，包括域名。

注意

SASL LDAP 插件对于 Kerberos 认证使用一个固定的用户 DN，并忽略从 MySQL 配置的任何用户 DN。这有一定的影响：

+   对于使用 GSSAPI/Kerberos 认证的任何 MySQL 账户，在 `CREATE USER` 或 `ALTER USER` 语句中的认证字符串不应包含用户 DN，因为它没有任何效果。

+   因为认证字符串不包含用户 DN，所以应包含组映射信息，以使用户能够作为映射到所需代理用户的代理用户进行处理。有关使用 LDAP 认证插件进行代理的信息，请参阅 LDAP Authentication with Proxying。

以下语句创建一个名为`bredon@MYSQL.LOCAL`的代理用户，该用户假定了被代理用户`proxied_krb_usr`的权限。其他应具有相同权限的 GSSAPI/Kerberos 用户可以类似地为同一被代理用户创建代理用户。

```sql
-- create proxy account
CREATE USER 'bredon@MYSQL.LOCAL'
  IDENTIFIED WITH authentication_ldap_sasl
  BY '#krb_grp=proxied_krb_user';

-- create proxied account and grant its privileges;
-- use mysql_no_login plugin to prevent direct login
CREATE USER 'proxied_krb_user'
  IDENTIFIED WITH mysql_no_login;
GRANT ALL
  ON krb_user_db.*
  TO 'proxied_krb_user';

-- grant to proxy account the
-- PROXY privilege for proxied account
GRANT PROXY
  ON 'proxied_krb_user'
  TO 'bredon@MYSQL.LOCAL';
```

仔细观察第一个`CREATE USER`语句和`GRANT PROXY`语句中代理账户名称的引用：

+   对于大多数 MySQL 账户，用户和主机是账户名称的独立部分，因此分别引用为`'*`user_name`*'@'*`host_name`*'`。

+   对于 LDAP Kerberos 认证，账户名称的用户部分包括主体域，因此`'bredon@MYSQL.LOCAL'`被引用为单个值。因为没有给出主机部分，所以完整的 MySQL 账户名称使用默认的`'%'`作为主机部分：`'bredon@MYSQL.LOCAL'@'%'`

注意

当创建一个使用 GSSAPI/Kerberos 认证方法的`authentication_ldap_sasl` SASL LDAP 认证插件进行身份验证的账户时，`CREATE USER`语句将领域作为用户名的一部分。这与使用`authentication_kerberos` Kerberos 插件创建账户不同。对于这样的账户，`CREATE USER`语句不包括领域作为用户名的一部分。而是在`BY`子句中指定领域作为认证字符串。请参阅创建使用 Kerberos 认证的 MySQL 账户。

被代理账户使用`mysql_no_login`认证插件防止客户端直接使用该账户登录到 MySQL 服务器。相反，预期使用 LDAP 进行身份验证的用户使用`bredon@MYSQL.LOCAL`代理账户。（这假设已安装`mysql_no_login`插件。有关说明，请参见第 8.4.1.9 节，“无登录可插入认证”。）有关保护被代理账户免受直接使用的替代方法，请参见防止直接登录到被代理账户。

###### 使用 MySQL 账户连接到 MySQL 服务器

设置使用 GSSAPI/Kerberos 进行身份验证的 MySQL 账户后，客户端可以使用它连接到 MySQL 服务器。Kerberos 认证可以在 MySQL 客户端程序调用之前或同时进行：

+   在调用 MySQL 客户端程序之前，客户端用户可以独立于 MySQL 从 KDC 获取 TGT。例如，客户端用户可以使用**kinit**通过提供 Kerberos 主体名称和主体密码来对 Kerberos 进行身份验证：

    ```sql
    $> kinit bredon@MYSQL.LOCAL
    Password for bredon@MYSQL.LOCAL: *(enter password here)*
    ```

    结果的 TGT 被缓存并可供其他了解 Kerberos 的应用程序使用，例如使用客户端端 SASL LDAP 认证插件的程序。在这种情况下，MySQL 客户端程序使用 TGT 对 MySQL 服务器进行认证，因此调用客户端时不需要指定用户名或密码：

    ```sql
    mysql --default-auth=authentication_ldap_sasl_client
    ```

    正如刚才所描述的，当 TGT 被缓存时，在客户端命令中不需要用户名称和密码选项。如果命令中仍然包含它们，则处理如下：

    +   如果命令包含用户名，如果该名称与 TGT 中的主体名称不匹配，则认证失败。

    +   如果命令包含密码，客户端插件会忽略它。因为认证是基于 TGT 的，*即使用户提供的密码不正确*，也可以成功。因此，如果找到有效的 TGT 导致密码被忽略，插件会产生警告。

+   如果 Kerberos 缓存中不包含 TGT，则客户端端 SASL LDAP 认证插件本身可以从 KDC 获取 TGT。使用与 MySQL 账户关联的 Kerberos 主体的名称和密码选项调用客户端（在提示时输入主体密码）：

    ```sql
    mysql --default-auth=authentication_ldap_sasl_client
      --user=bredon@MYSQL.LOCAL
      --password
    ```

+   如果 Kerberos 缓存中不包含 TGT，并且客户端命令未指定主体名称作为用户名，则认证失败。

如果您不确定是否存在 TGT，可以使用 **klist** 进行检查。

认证过程如下：

1.  客户端使用 TGT 使用 Kerberos 进行认证。

1.  服务器找到主体的 LDAP 条目并用它来认证连接到 `bredon@MYSQL.LOCAL` MySQL 代理账户。

1.  代理账户认证字符串中的组映射信息（`'#krb_grp=proxied_krb_user'`）表示认证的被代理用户应该是 `proxied_krb_user`。

1.  `bredon@MYSQL.LOCAL` 被视为 `proxied_krb_user` 的代理，并且以下查询返回如下输出：

    ```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
    +------------------------------+--------------------+--------------------------+
    | USER()                       | CURRENT_USER()     | @@proxy_user             |
    +------------------------------+--------------------+--------------------------+
    | bredon@MYSQL.LOCAL@localhost | proxied_krb_user@% | 'bredon@MYSQL.LOCAL'@'%' |
    +------------------------------+--------------------+--------------------------+
    ```

    `USER()` 值表示用于客户端命令的用户名（`bredon@MYSQL.LOCAL`）和客户端连接的主机（`localhost`）。

    `CURRENT_USER()` 值是被代理用户账户的全名，由 `proxied_krb_user` 用户部分和 `%` 主机部分组成。

    `@@proxy_user` 值表示用于连接到 MySQL 服务器的账户的全名，由 `bredon@MYSQL.LOCAL` 用户部分和 `%` 主机部分组成。

    这表明代理是通过 `bredon@MYSQL.LOCAL` 代理用户账户进行的，并且 `bredon@MYSQL.LOCAL` 承担了授予 `proxied_krb_user` 被代理用户账户的权限。

一旦获得 TGT，客户端会将其缓存在本地，并且在不再需要重新输入密码的情况下可以一直使用直到过期。无论如何获取 TGT，客户端插件都会使用它来获取服务票据并与服务器端插件通信。

注意

当客户端身份验证插件本身获取 TGT 时，客户端用户可能不希望 TGT 被重复使用。如 LDAP 身份验证的客户端配置参数所述，本地的`/etc/krb5.conf`文件可用于使客户端插件在完成后销毁 TGT。

服务器端插件无法访问 TGT 本身或用于获取 TGT 的 Kerberos 密码。

LDAP 身份验证插件无法控制缓存机制（存储在本地文件中、内存中等），但 Kerberos 实用程序如**kswitch**可能可用于此目的。

###### LDAP 身份验证的客户端配置参数

`authentication_ldap_sasl_client`客户端端 SASL LDAP 插件会读取本地的`/etc/krb5.conf`文件。如果该文件丢失或无法访问，将会出现错误。假设该文件可访问，它可以包含一个可选的`[appdefaults]`部分，提供插件使用的信息。将信息放在部分的`mysql`部分内。例如：

```sql
[appdefaults]
 mysql = {
 ldap_server_host = "ldap_host.example.com"
 ldap_destroy_tgt = true
  }
```

客户端插件在`mysql`部分识别这些参数：

+   `ldap_server_host`值指定 LDAP 服务器主机，当该主机与`[realms]`部分指定的 KDC 服务器主机不同时，这可能很有用。默认情况下，插件使用 KDC 服务器主机作为 LDAP 服务器主机。

+   `ldap_destroy_tgt`值指示客户端插件在获取和使用 TGT 后是否销毁 TGT。默认情况下，`ldap_destroy_tgt`为`false`，但可以设置为`true`以避免 TGT 重复使用。（此设置仅适用于由客户端插件创建的 TGT，不适用于由 MySQL 的其他插件或外部创建的 TGT。）

##### LDAP 搜索引荐

LDAP 服务器可以配置为将 LDAP 搜索委派给另一个 LDAP 服务器，这种功能称为 LDAP 引荐。假设服务器`a.example.com`拥有一个`"dc=example,dc=com"`根 DN，并希望将搜索委派给另一个服务器`b.example.com`。为了启用这个功能，`a.example.com`将被配置为具有以下属性的命名引荐对象：

```sql
dn: dc=subtree,dc=example,dc=com
objectClass: referral
objectClass: extensibleObject
dc: subtree
ref: ldap://b.example.com/dc=subtree,dc=example,dc=com
```

启用 LDAP 引荐的一个问题是，当搜索基本 DN 为根 DN 且未设置引荐对象时，搜索可能会因 LDAP 操作错误而失败。即使在`ldap.conf`配置文件中全局设置了 LDAP 引荐，MySQL DBA 可能希望避免 LDAP 认证插件出现此类引荐错误。要根据插件特定的基础配置 LDAP 服务器在与每个插件通信时是否应使用 LDAP 引荐，需设置`authentication_ldap_simple_referral`和`authentication_ldap_sasl_referral`系统变量。将任一变量设置为`ON`或`OFF`会导致相应的 LDAP 认证插件告知 LDAP 服务器在 MySQL 认证期间是否使用引荐。每个变量具有插件特定的效果，不会影响与 LDAP 服务器通信的其他应用程序。这两个变量默认为`OFF`。
