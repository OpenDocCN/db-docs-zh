> 原文：[`dev.mysql.com/doc/refman/8.0/en/pam-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/pam-pluggable-authentication.html)

#### 8.4.1.5 PAM 可插入认证

注意

PAM 可插入认证是 MySQL 企业版中包含的扩展，是一款商业产品。要了解更多关于商业产品的信息，请参阅[`www.mysql.com/products/`](https://www.mysql.com/products/)。

MySQL 企业版支持一种认证方法，使得 MySQL 服务器能够使用 PAM（可插入认证模块）来认证 MySQL 用户。PAM 使得系统能够使用标准接口访问各种认证方法，例如传统的 Unix 密码或 LDAP 目录。

PAM 可插入认证提供了以下功能：

+   外部认证：PAM 认证使得 MySQL 服务器能够接受来自 MySQL 授权表之外定义的用户的连接，并使用 PAM 支持的方法进行认证。

+   代理用户支持：PAM 认证可以根据外部用户所属的 PAM 组和提供的认证字符串，返回一个与客户端程序传递的外部用户名不同的 MySQL 用户名，这意味着插件可以返回定义外部 PAM 认证用户应具有的特权的 MySQL 用户。例如，名为`joe`的操作系统用户可以连接并具有名为`developer`的 MySQL 用户的权限。

PAM 可插入认证已在 Linux 和 macOS 上进行了测试；请注意 Windows 不支持 PAM。

以下表格显示了插件和库文件名称。文件名后缀可能在您的系统上有所不同。该文件必须位于由`plugin_dir`系统变量命名的目录中。有关安装信息，请参阅安装 PAM 可插入认证。

**表 8.20 PAM 认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `authentication_pam` |
| 客户端插件 | `mysql_clear_password` |
| 库文件 | `authentication_pam.so` |

与服务器端 PAM 插件通信的客户端`mysql_clear_password`明文插件内置于`libmysqlclient`客户端库中，并包含在所有发行版中，包括社区发行版。在所有 MySQL 发行版中包含客户端明文插件使得来自任何发行版的客户端都能连接到加载了服务器端 PAM 插件的服务器。

以下部分提供了特定于 PAM 可插入认证的安装和使用信息：

+   MySQL 用户 PAM 认证工作原理

+   安装 PAM 可插拔认证

+   卸载 PAM 可插拔认证

+   使用 PAM 可插拔认证

+   无代理用户的 PAM Unix 密码认证

+   无代理用户的 PAM LDAP 认证

+   带有代理用户和组映射的 PAM Unix 密码认证

+   访问 Unix 密码存储的 PAM 认证

+   PAM 认证调试

关于 MySQL 中可插拔认证的一般信息，请参见第 8.2.17 节，“可插拔认证”。关于`mysql_clear_password`插件的信息，请参见第 8.4.1.4 节，“客户端明文可插拔认证”。有关代理用户信息，请参见第 8.2.19 节，“代理用户”。

##### MySQL 用户的 PAM 认证工作原理

本节概述了 MySQL 和 PAM 如何共同工作来认证 MySQL 用户。有关如何设置 MySQL 帐户以使用特定 PAM 服务的示例，请参见使用 PAM 可插拔认证。

1.  客户端程序和服务器进行通信，客户端向服务器发送客户端用户名（默认为操作系统用户名）和密码：

    +   客户端用户名为外部用户名。

    +   对于使用 PAM 服务器端认证插件的帐户，相应的客户端插件是`mysql_clear_password`。这个客户端插件不执行密码哈希处理，结果客户端将密码以明文形式发送到服务器。

1.  服务器根据客户端连接的外部用户名和主机找到匹配的 MySQL 帐户。PAM 插件使用 MySQL 服务器传递给它的信息（如用户名、主机名、密码和认证字符串）。当您定义一个使用 PAM 进行身份验证的 MySQL 帐户时，认证字符串包含：

    +   PAM 服务名称，这是系统管理员可以用来引用特定应用程序的身份验证方法的名称。可以将多个应用程序关联到单个数据库服务器实例，因此服务名称的选择留给 SQL 应用程序开发人员。

    +   可选地，如果要使用代理，可以从 PAM 组到 MySQL 用户名的映射。

1.  该插件使用认证字符串中命名的 PAM 服务来检查用户凭据，并返回`'Authentication succeeded, Username is *`user_name`*'`或`'Authentication failed'`。密码必须适用于 PAM 服务使用的密码存储。示例：

    +   对于传统的 Unix 密码，服务查找存储在`/etc/shadow`文件中的密码。

    +   对于 LDAP，服务查找存储在 LDAP 目录中的密码。

    如果凭据检查失败，服务器将拒绝连接。

1.  否则，认证字符串指示是否进行代理。如果字符串不包含 PAM 组映射，则不会进行代理。在这种情况下，MySQL 用户名与外部用户名相同。

1.  否则，根据 PAM 组映射指示代理，MySQL 用户名根据映射列表中的第一个匹配组确定。“PAM 组”的含义取决于 PAM 服务。示例：

    +   对于传统的 Unix 密码，组是在`/etc/group`文件中定义的 Unix 组，可能在诸如`/etc/security/group.conf`之类的文件中补充了其他 PAM 信息。

    +   对于 LDAP，组是在 LDAP 目录中定义的 LDAP 组。

    如果代理用户（外部用户）对被代理的 MySQL 用户名具有`PROXY`权限，则进行代理，代理用户承担被代理用户的权限。

##### 安装 PAM 可插拔认证

本节描述了如何安装服务器端 PAM 认证插件。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

要使服务器能够使用插件库文件，必须将其位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如果需要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

插件库文件基本名称为`authentication_pam`，通常编译为`.so`后缀。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，每次启动服务器都必须提供该选项。例如，在服务器`my.cnf`文件中放入以下行：

```sql
[mysqld]
plugin-load-add=authentication_pam.so
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用以下语句，根据需要调整`.so`后缀：

```sql
INSTALL PLUGIN authentication_pam SONAME 'authentication_pam.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装，请检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%pam%';
+--------------------+---------------+
| PLUGIN_NAME        | PLUGIN_STATUS |
+--------------------+---------------+
| authentication_pam | ACTIVE        |
+--------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断信息。

要将 MySQL 帐户与 PAM 插件关联，请参阅使用 PAM 可插拔认证。

##### 卸载 PAM 可插拔认证

卸载 PAM 认证插件的方法取决于您安装插件的方式：

+   如果您使用`--plugin-load-add`选项在服务器启动时安装了插件，请在不带该选项的情况下重新启动服务器。

+   如果您使用`INSTALL PLUGIN`语句在运行时安装了插件，则在服务器重新启动时仍保留安装。要卸载它，请使用`UNINSTALL PLUGIN`：

    ```sql
    UNINSTALL PLUGIN authentication_pam;
    ```

##### 使用 PAM 可插拔认证

本节概述了如何使用 PAM 认证插件从 MySQL 客户端程序连接到服务器。以下各节提供了使用 PAM 认证的具体方法的说明。假定服务器正在运行并启用了服务器端 PAM 插件，如安装 PAM 可插拔认证中所述。

要在`CREATE USER`语句的`IDENTIFIED WITH`子句中引用 PAM 认证插件，请使用名称`authentication_pam`。例如：

```sql
CREATE USER *user*
  IDENTIFIED WITH authentication_pam
  AS '*auth_string*';
```

认证字符串指定以下类型的信息：

+   PAM 服务名称（参见 MySQL 用户的 PAM 认证工作原理）。以下讨论中的示例使用`mysql-unix`作为服务名称，用于使用传统 Unix 密码进行身份验证，以及`mysql-ldap`用于使用 LDAP 进行身份验证。

+   对于代理支持，PAM 提供了一种方式，让 PAM 模块在连接到服务器时，返回一个与客户端程序传递的外部用户名不同的 MySQL 用户名。使用认证字符串来控制从外部用户名到 MySQL 用户名的映射。如果要利用代理用户功能，认证字符串必须包含这种映射。

例如，如果一个账户使用`mysql-unix` PAM 服务名称，并且应该将`root`和`users` PAM 组中的操作系统用户映射到`developer`和`data_entry` MySQL 用户，可以使用类似以下语句：

```sql
CREATE USER *user*
  IDENTIFIED WITH authentication_pam
  AS 'mysql-unix, root=developer, users=data_entry';
```

PAM 认证插件的认证字符串语法遵循以下规则：

+   该字符串由 PAM 服务名称组成，可选择地后跟一个 PAM 组映射列表，其中包含一个或多个关键字/值对，每个对指定一个 PAM 组名和一个 MySQL 用户名：

    ```sql
    *pam_service_name*[,*pam_group_name*=*mysql_user_name*]...
    ```

    插件会解析每个使用该账户的连接尝试的认证字符串。为了最小化开销，尽量保持字符串尽可能短。

+   每个`*`pam_group_name`*=*`mysql_user_name`*`对前面必须有一个逗号。

+   不在双引号内的前导和尾随空格会被忽略。

+   未引用的*`pam_service_name`*、*`pam_group_name`*和*`mysql_user_name`*值可以包含除等号、逗号或空格之外的任何内容。

+   如果*`pam_service_name`*、*`pam_group_name`*或*`mysql_user_name`*值用双引号引起来，引号之间的所有内容都是值的一部分。例如，如果值包含空格字符，则这是必要的。所有字符都是合法的，除了双引号和反斜杠（`\`）。要包含这两个字符，用反斜杠进行转义。

如果插件成功验证外部用户名（客户端传递的名称），它会查找认证字符串中的 PAM 组映射列表，并且如果存在，则根据外部用户所属的 PAM 组返回不同的 MySQL 用户名给 MySQL 服务器：

+   如果认证字符串不包含 PAM 组映射列表，则插件返回外部名称。

+   如果认证字符串包含 PAM 组映射列表，则插件会从左到右检查列表中的每个`*`pam_group_name`*=*`mysql_user_name`*`对，并尝试在已认证用户分配的非 MySQL 目录中找到*`pam_group_name`*值的匹配项，并返回找到的第一个匹配项的*`mysql_user_name`*。如果插件找不到任何 PAM 组的匹配项，则返回外部名称。如果插件无法在目录中查找组，则会忽略 PAM 组映射列表并返回外部名称。

以下各节描述了如何设置使用 PAM 身份验证插件的几种身份验证方案：

+   没有代理用户。这仅使用 PAM 来检查登录名和密码。每个被允许连接到 MySQL 服务器的外部用户都应该有一个匹配的 MySQL 帐户，该帐户被定义为使用 PAM 身份验证。（为了使 MySQL 帐户`'*`user_name`*'@'*`host_name`*'`与外部用户匹配，*`user_name`*必须是外部用户名，*`host_name`*必须与客户端连接的主机匹配。）认证可以通过各种 PAM 支持的方法进行。后续讨论将展示如何使用传统的 Unix 密码和 LDAP 密码来验证客户端凭据。

    PAM 身份验证，如果不通过代理用户或 PAM 组进行，需要 MySQL 用户名与操作系统用户名相同。 MySQL 用户名限制为 32 个字符（参见第 8.2.3 节，“授权表”），这限制了 PAM 非代理身份验证仅限于最多 32 个字符的 Unix 帐户。

+   仅代理用户，带有 PAM 组映射。对于这种情况，创建一个或多个定义不同权限集的 MySQL 帐户。（理想情况下，没有人应该直接使用这些帐户连接。）然后定义一个通过 PAM 进行身份验证的默认用户，使用某种映射方案（通常基于用户所属的外部 PAM 组）将所有外部用户名映射到持有权限集的少数 MySQL 帐户。任何连接并指定外部用户名作为客户端用户名的客户端都将映射到一个 MySQL 帐户并使用其权限。本讨论将展示如何使用传统的 Unix 密码设置此功能，但也可以使用其他 PAM 方法，如 LDAP。

这些方案的变体是可能的：

+   您可以允许一些用户直接登录（无需代理），但要求其他用户通过代理帐户连接。

+   您可以通过在 PAM 身份验证帐户之间使用不同的 PAM 服务名称，为一些用户使用一种 PAM 身份验证方法，为其他用户使用另一种方法。例如，您可以为一些用户使用`mysql-unix` PAM 服务，为其他用户使用`mysql-ldap`。

示例做出以下假设。如果您的系统设置不同，您可能需要进行一些调整。

+   登录名和密码分别为`antonio`和*`antonio_password`*。请将其更改为要进行身份验证的用户。

+   PAM 配置目录为`/etc/pam.d`。

+   PAM 服务名称对应于认证方法（在本讨论中为`mysql-unix`或`mysql-ldap`）。要使用给定的 PAM 服务，您必须在 PAM 配置目录中设置一个同名的 PAM 文件（如果不存在则创建该文件）。此外，您必须在使用该 PAM 服务进行认证的任何账户的`CREATE USER`语句的认证字符串中命名 PAM 服务。

PAM 认证插件在初始化时检查服务器启动环境中是否设置了`AUTHENTICATION_PAM_LOG`环境值。如果是，则插件会启用将诊断消息记录到标准输出的功能。根据服务器的启动方式，消息可能会显示在控制台上或错误日志中。这些消息对于调试描相关问题很有帮助。有关更多信息，请参阅 PAM 认证调试。

##### 无代理用户的 PAM Unix 密码认证

这种认证场景使用 PAM 来检查以操作系统用户名和 Unix 密码定义的外部用户，而无需代理。每个被允许连接到 MySQL 服务器的外部用户都应该有一个匹配的 MySQL 账户，该账户被定义为通过传统的 Unix 密码存储使用 PAM 认证。

注意

传统的 Unix 密码是使用`/etc/shadow`文件进行检查的。有关与该文件相关的可能问题的信息，请参阅 PAM 认证访问 Unix 密码存储。

1.  验证 Unix 认证是否允许使用用户名`antonio`和密码*`antonio_password`*登录到操作系统。

1.  通过创建名为`/etc/pam.d/mysql-unix`的`mysql-unix` PAM 服务文件来设置 PAM 以使用传统的 Unix 密码对 MySQL 连接进行认证。文件内容取决于系统，因此请检查`/etc/pam.d`目录中现有的与登录相关的文件以查看其外观。在 Linux 上，`mysql-unix`文件可能如下所示：

    ```sql
    #%PAM-1.0
    auth            include         password-auth
    account         include         password-auth
    ```

    对于 macOS，请使用`login`而不是`password-auth`。

    在某些系统上，PAM 文件格式可能有所不同。例如，在 Ubuntu 和其他基于 Debian 的系统上，使用以下文件内容：

    ```sql
    @include common-auth
    @include common-account
    @include common-session-noninteractive
    ```

1.  创建一个与操作系统用户名相同的 MySQL 账户，并定义其使用 PAM 插件和`mysql-unix` PAM 服务进行认证：

    ```sql
    CREATE USER 'antonio'@'localhost'
      IDENTIFIED WITH authentication_pam
      AS 'mysql-unix';
    GRANT ALL PRIVILEGES
      ON mydb.*
      TO 'antonio'@'localhost';
    ```

    在这里，认证字符串仅包含 PAM 服务名称，`mysql-unix`，用于验证 Unix 密码。

1.  使用**mysql**命令行客户端作为`antonio`连接到 MySQL 服务器。例如：

    ```sql
    $> mysql --user=antonio --password --enable-cleartext-plugin
    Enter password: *antonio_password*
    ```

    服务器应允许连接，并且以下查询返回如下输出：

    ```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
    +-------------------+-------------------+--------------+
    | USER()            | CURRENT_USER()    | @@proxy_user |
    +-------------------+-------------------+--------------+
    | antonio@localhost | antonio@localhost | NULL         |
    +-------------------+-------------------+--------------+
    ```

    这表明`antonio`操作系统用户经过认证具有授予`antonio` MySQL 用户的权限，并且没有发生代理。

注意

客户端端的`mysql_clear_password`认证插件不会更改密码，因此客户端程序会将其以明文形式发送到 MySQL 服务器。这使得密码可以原样传递给 PAM。在某些配置中，明文密码是使用服务器端 PAM 库所必需的，但可能会存在安全问题。以下措施可最小化风险：

+   为了减少意外使用`mysql_clear_password`插件的可能性，MySQL 客户端必须显式启用它（例如，使用`--enable-cleartext-plugin`选项）。参见 Section 8.4.1.4, “Client-Side Cleartext Pluggable Authentication”。

+   为了避免启用`mysql_clear_password`插件时密码暴露，MySQL 客户端应该使用加密连接连接到 MySQL 服务器。参见 Section 8.3.1, “Configuring MySQL to Use Encrypted Connections”。

##### 无代理用户的 PAM LDAP 认证

这种认证场景使用 PAM 来检查以操作系统用户名和 LDAP 密码定义的外部用户，而无需代理。每个被允许连接到 MySQL 服务器的外部用户都应该有一个匹配的 MySQL 账户，该账户被定义为通过 LDAP 进行 PAM 认证。

要使用 PAM LDAP 可插拔认证为 MySQL，必须满足以下先决条件：

+   PAM LDAP 服务必须与 LDAP 服务器通信。

+   每个要由 MySQL 进行认证的 LDAP 用户必须存在于由 LDAP 服务器管理的目录中。

注意

另一种使用 LDAP 进行 MySQL 用户认证的方法是使用特定于 LDAP 的认证插件。参见 Section 8.4.1.7, “LDAP Pluggable Authentication”。

配置 MySQL 以进行 PAM LDAP 认证如下：

1.  验证 Unix 认证是否允许使用用户名`antonio`和密码*`antonio_password`*登录到操作系统。

1.  通过创建一个名为`/etc/pam.d/mysql-ldap`的`mysql-ldap` PAM 服务文件来设置 PAM 以使用 LDAP 认证 MySQL 连接。文件内容取决于系统，因此请检查`/etc/pam.d`目录中现有的与登录相关的文件以查看其内容。在 Linux 上，`mysql-ldap`文件可能如下所示：

    ```sql
    #%PAM-1.0
    auth        required    pam_ldap.so
    account     required    pam_ldap.so
    ```

    如果在您的系统上，PAM 对象文件的后缀与`.so`不同，请替换正确的后缀。

    PAM 文件格式可能在某些系统上有所不同。

1.  创建一个与操作系统用户名相同的 MySQL 账户，并定义其使用 PAM 插件和`mysql-ldap` PAM 服务进行验证：

    ```sql
    CREATE USER 'antonio'@'localhost'
      IDENTIFIED WITH authentication_pam
      AS 'mysql-ldap';
    GRANT ALL PRIVILEGES
      ON mydb.*
      TO 'antonio'@'localhost';
    ```

    这里，认证字符串仅包含 PAM 服务名称`mysql-ldap`，用 LDAP 进行验证。

1.  连接到服务器的方式与 PAM Unix 密码认证无代理用户中描述的相同。

##### 使用代理用户和组映射的 PAM Unix 密码认证

此处描述的认证方案使用代理和 PAM 组映射，将使用 PAM 进行验证的连接 MySQL 用户映射到定义了不同权限集的其他 MySQL 账户上。用户不直接通过定义权限的账户连接。相反，他们通过使用 PAM 进行身份验证的默认代理账户连接，以便将所有外部用户映射到具有权限的 MySQL 账户。使用代理账户连接的任何用户都将映射到这些 MySQL 账户之一，其权限确定了允许外部用户执行的数据库操作。

此处所示的过程使用 Unix 密码认证。要改用 LDAP，请参阅 PAM LDAP 认证无代理用户的早期步骤。

注意

传统的 Unix 密码使用`/etc/shadow`文件进行检查。有关与该文件相关的可能问题的信息，请参阅 PAM 认证访问 Unix 密码存储。

1.  验证 Unix 认证是否允许使用用户名`antonio`和密码*`antonio_password`*登录操作系统。

1.  验证`antonio`是否是`root`或`users` PAM 组的成员。

1.  通过创建名为`/etc/pam.d/mysql-unix`的文件，设置 PAM 通过操作系统用户验证`mysql-unix` PAM 服务。文件内容取决于系统，因此请查看`/etc/pam.d`目录中现有的与登录相关的文件以了解其内容。在 Linux 上，`mysql-unix`文件可能如下所示：

    ```sql
    #%PAM-1.0
    auth            include         password-auth
    account         include         password-auth
    ```

    对于 macOS，请使用`login`而不是`password-auth`。

    在某些系统上，PAM 文件格式可能有所不同。例如，在 Ubuntu 和其他基于 Debian 的系统上，使用以下文件内容：

    ```sql
    @include common-auth
    @include common-account
    @include common-session-noninteractive
    ```

1.  创建一个默认的代理用户(`''@''`)，将外部 PAM 用户映射到代理账户：

    ```sql
    CREATE USER ''@''
      IDENTIFIED WITH authentication_pam
      AS 'mysql-unix, root=developer, users=data_entry';
    ```

    这里，认证字符串包含了 PAM 服务名称`mysql-unix`，用于验证 Unix 密码。认证字符串还将`root`和`users` PAM 组中的外部用户映射到`developer`和`data_entry` MySQL 用户名。

    在设置代理用户时，必须在 PAM 服务名称后面提供 PAM 组映射列表。否则，插件无法确定如何将外部用户名映射到正确的被代理 MySQL 用户名。

    注意

    如果您的 MySQL 安装中存在匿名用户，则可能会与默认代理用户发生冲突。有关此问题的更多信息以及处理方法，请参见 Default Proxy User and Anonymous User Conflicts。

1.  创建被代理账户并为每个账户授予应具有的权限：

    ```sql
    CREATE USER 'developer'@'localhost'
      IDENTIFIED WITH mysql_no_login;
    CREATE USER 'data_entry'@'localhost'
      IDENTIFIED WITH mysql_no_login;

    GRANT ALL PRIVILEGES
      ON mydevdb.*
      TO 'developer'@'localhost';
    GRANT ALL PRIVILEGES
      ON mydb.*
      TO 'data_entry'@'localhost';
    ```

    代理账户使用`mysql_no_login`认证插件，以防止客户端直接使用这些账户登录到 MySQL 服务器。相反，使用 PAM 进行身份验证的用户应该通过代理使用基于他们的 PAM 组的`developer`或`data_entry`账户。（假设插件已安装。有关说明，请参见 Section 8.4.1.9, “No-Login Pluggable Authentication”。）有关保护代理账户免受直接使用的替代方法，请参见 Preventing Direct Login to Proxied Accounts。

1.  为每个被代理账户授予`PROXY`权限：

    ```sql
    GRANT PROXY
      ON 'developer'@'localhost'
      TO ''@'';
    GRANT PROXY
      ON 'data_entry'@'localhost'
      TO ''@'';
    ```

1.  使用**mysql**命令行客户端作为`安东尼奥`连接到 MySQL 服务器。

    ```sql
    $> mysql --user=antonio --password --enable-cleartext-plugin
    Enter password: *antonio_password*
    ```

    服务器使用默认的`''@''`代理账户对连接进行身份验证。`安东尼奥`的权限取决于他是哪些 PAM 组的成员。如果`安东尼奥`是`root` PAM 组的成员，PAM 插件将`root`映射到`developer` MySQL 用户名并将该名称返回给服务器。服务器验证`''@''`是否具有`developer`的`PROXY`权限，并允许连接。以下查询返回如下输出：

    ```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
    +-------------------+---------------------+--------------+
    | USER()            | CURRENT_USER()      | @@proxy_user |
    +-------------------+---------------------+--------------+
    | antonio@localhost | developer@localhost | ''@''        |
    +-------------------+---------------------+--------------+
    ```

    这表明`安东尼奥`操作系统用户被验证具有授予`developer` MySQL 用户的权限，并且通过默认代理账户进行代理。

    如果`安东尼奥`不是`root` PAM 组的成员，但是`users` PAM 组的成员，类似的过程会发生，但插件将`user` PAM 组成员映射到`data_entry` MySQL 用户名并将该名称返回给服务器：

    ```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
    +-------------------+----------------------+--------------+
    | USER()            | CURRENT_USER()       | @@proxy_user |
    +-------------------+----------------------+--------------+
    | antonio@localhost | data_entry@localhost | ''@''        |
    +-------------------+----------------------+--------------+
    ```

    这表明`安东尼奥`操作系统用户被验证具有`data_entry` MySQL 用户的权限，并且通过默认代理账户进行代理。

注意

客户端端的`mysql_clear_password`认证插件保持密码不变，因此客户端程序将其以明文形式发送到 MySQL 服务器。这使得密码可以原样传递给 PAM。在某些配置中，明文密码是使用服务器端 PAM 库所必需的，但可能在某些配置中存在安全问题。以下措施可最小化风险：

+   为了减少意外使用`mysql_clear_password`插件的可能性，MySQL 客户端必须显式启用它（例如，使用`--enable-cleartext-plugin`选项）。参见 Section 8.4.1.4, “Client-Side Cleartext Pluggable Authentication”。

+   启用`mysql_clear_password`插件以避免密码暴露，MySQL 客户端应该使用加密连接连接到 MySQL 服务器。参见 Section 8.3.1, “Configuring MySQL to Use Encrypted Connections”。

##### PAM 身份验证访问 Unix 密码存储

在某些系统上，Unix 身份验证使用密码存储，例如`/etc/shadow`，这是一个通常具有受限访问权限的文件。这可能导致 MySQL 基于 PAM 的身份验证失败。不幸的是，PAM 实现不允许区分“密码无法检查”（例如，由于无法读取`/etc/shadow`）和“密码不匹配”。如果您正在使用 Unix 密码存储进行 PAM 身份验证，您可以通过以下方法之一启用 MySQL 对其的访问：

+   假设 MySQL 服务器是从`mysql`操作系统帐户运行的，请将该帐户放入具有`/etc/shadow`访问权限的`shadow`组中：

    1.  在`/etc/group`中创建���个`shadow`组。

    1.  将`mysql`操作系统用户添加到`/etc/group`中的`shadow`组。

    1.  将`/etc/group`分配给`shadow`组并启用组读权限：

        ```sql
        chgrp shadow /etc/shadow
        chmod g+r /etc/shadow
        ```

    1.  重新启动 MySQL 服务器。

+   如果您正在使用`pam_unix`模块和**unix_chkpwd**实用程序，请按以下方式启用密码存储访问：

    ```sql
    chmod u-s /usr/sbin/unix_chkpwd
    setcap cap_dac_read_search+ep /usr/sbin/unix_chkpwd
    ```

    根据您的平台调整**unix_chkpwd**的路径。

##### PAM 身份验证调试

PAM 身份验证插件在初始化时检查`AUTHENTICATION_PAM_LOG`环境值是否已设置。在 MySQL 8.0.35 及更早版本中，该值无关紧要。如果是，则插件将启用将诊断消息记录到标准输出的功能。这些消息可能有助于调试插件执行身份验证时出现的与 PAM 相关的问题。您应该知道，在这些版本中，这些消息中包含密码。

从 MySQL 8.0.36 开始，设置`AUTHENTICATION_PAM_LOG=1`（或其他任意值）会产生相同的诊断消息，但*不*包含任何密码。如果您希望在这些消息中包含密码，请设置`AUTHENTICATION_PAM_LOG=PAM_LOG_WITH_SECRET_INFO`。

一些消息包含对 PAM 插件源文件和行号的引用，这使得插件操作与其发生的代码位置更紧密地联系在一起。

用于调试连接失败并确定连接尝试期间发生的情况的另一种技术是配置 PAM 认证以允许所有连接，然后检查系统日志文件。这种技术应仅在*临时*基础上使用，而不是在生产服务器上使用。

使用以下内容配置名为 `/etc/pam.d/mysql-any-password` 的 PAM 服务文件（在某些系统上格式可能有所不同）：

```sql
#%PAM-1.0
auth        required    pam_permit.so
account     required    pam_permit.so
```

创建一个使用 PAM 插件并命名为 `mysql-any-password` PAM 服务的帐户：

```sql
CREATE USER 'testuser'@'localhost'
  IDENTIFIED WITH authentication_pam
  AS 'mysql-any-password';
```

`mysql-any-password` 服务文件会导致任何身份验证尝试返回 true，即使密码不正确。如果身份验证尝试失败，这说明配置问题在 MySQL 方面。否则，问题在操作系统/PAM 方面。要查看可能发生的情况，请检查系统日志文件，如 `/var/log/secure`、`/var/log/audit.log`、`/var/log/syslog` 或 `/var/log/messages`。

确定问题后，删除 `mysql-any-password` PAM 服务文件以禁用任意密码访问。
