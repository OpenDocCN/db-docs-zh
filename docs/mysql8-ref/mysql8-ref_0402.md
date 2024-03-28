> 原文：[`dev.mysql.com/doc/refman/8.0/en/kerberos-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/kerberos-pluggable-authentication.html)

#### 8.4.1.8 Kerberos 可插拔认证

注意

Kerberos 可插拔认证是 MySQL 企业版中包含的扩展，这是一个商业产品。要了解更多关于商业产品的信息，请参见[`www.mysql.com/products/`](https://www.mysql.com/products/)。

MySQL 企业版支持一种认证方法，允许用户使用 Kerberos 对 MySQL 服务器进行身份验证，前提是有适当的 Kerberos 票证可用或可以获取。

该认证方法适用于 MySQL 8.0.26 及更高版本，在 Linux 上的 MySQL 服务器和客户端。在 Linux 环境中，应用程序可以访问默认启用 Kerberos 的 Microsoft Active Directory 时，这是非常有用的。从 MySQL 8.0.27 开始（MIT Kerberos 的 MySQL 8.0.32），客户端端插件也在 Windows 上受支持。服务器端插件仍然只在 Linux 上受支持。

Kerberos 可插拔认证提供了以下功能：

+   外部认证：Kerberos 认证使 MySQL 服务器能够接受来自 MySQL 授权表之外定义的用户的连接，这些用户已经获得了适当的 Kerberos 票证。

+   安全性：Kerberos 使用票证与对称密钥加密结合，实现了在网络上传输密码的身份验证。Kerberos 认证支持无用户和无密码的场景。

以下表显示了插件和库文件名称。文件名后缀可能在您的系统上有所不同。该文件必须位于由`plugin_dir`系统变量命名的目录中。有关安装信息，请参见安装 Kerberos 可插拔认证。

**表 8.24 Kerberos 认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `authentication_kerberos` |
| 客户端插件 | `authentication_kerberos_client` |
| 库文件 | `authentication_kerberos.so`，`authentication_kerberos_client.so` |

服务器端 Kerberos 认证插件仅包含在 MySQL 企业版中。它不包含在 MySQL 社区发行版中。客户端插件包含在所有发行版中，包括社区发行版。这使得来自任何发行版的客户端都可以连接到加载了服务器端插件的服务器。

以下各节提供了特定于 Kerberos 可插拔认证的安装和使用信息：

+   Kerberos 可插拔认证的先决条件

+   MySQL 用户的 Kerberos 认证工作原理

+   安装 Kerberos 可插拔认证

+   使用 Kerberos 可插拔认证

+   Kerberos 认证调试

有关 MySQL 中可插拔认证的一般信息，请参阅 第 8.2.17 节，“可插拔认证”。

##### Kerberos 可插拔认证的先决条件

要在 MySQL 中使用可插拔的 Kerberos 认证，必须满足以下先决条件：

+   必须提供一个 Kerberos 服务，以便 Kerberos 认证插件进行通信。

+   MySQL 要对其进行认证的每个 Kerberos 用户（主体）必须存在于 KDC 服务器管理的数据库中。

+   在使用服务器端或客户端 Kerberos 认证插件的系统上必须提供 Kerberos 客户端库。此外，GSSAPI 用作访问 Kerberos 认证的接口，因此必须提供 GSSAPI 库。

##### MySQL 用户的 Kerberos 认证工作原理

本节概述了 MySQL 和 Kerberos 如何共同工作以对 MySQL 用户进行认证。有关如何设置 MySQL 帐户以使用 Kerberos 认证插件的示例，请参阅 使用 Kerberos 可插拔认证。

假定读者熟悉 Kerberos 的概念和操作。以下列表简要定义了几个常见的 Kerberos 术语。您还可以在 [RFC 4120](https://tools.ietf.org/html/rfc4120) 的术语表部分找到有用的信息。

+   主体：命名实体，如用户或服务器。在本讨论中，某些与主体相关的术语经常出现：

    +   SPN：服务主体名称；代表服务的主体名称。

    +   UPN：用户主体名称；代表用户的主体名称。

+   KDC：密钥分发中心，包括 AS 和 TGS：

    +   AS：认证服务器；提供获取额外票证所需的初始票证授予票证。

    +   TGS：票据授予服务器；为拥有有效 TGT 的 Kerberos 客户端提供额外的票据。

+   TGT：票据授予票据；提交给 TGS 以获取用于服务访问的服务票据。

+   ST：服务票据；提供对 MySQL 服务器等服务的访问。

使用 Kerberos 进行身份验证需要一个 KDC 服务器，例如，由 Microsoft Active Directory 提供。

MySQL 中的 Kerberos 认证使用通用安全服务应用程序接口（GSSAPI），这是一个安全抽象接口。Kerberos 是可以通过该抽象接口使用的特定安全协议的一个实例。使用 GSSAPI，应用程序进行身份验证以获取服务凭据，然后依次使用这些凭据来启用对其他服务的安全访问。

在 Windows 上，`authentication_kerberos_client` 认证插件支持两种模式，客户端用户可以在运行时设置或在选项文件中指定：

+   `SSPI` 模式：安全支持提供程序接口（SSPI）实现了 GSSAPI（参见 SSPI 模式下的 Windows 客户端命令）。SSPI 在传输层面与 GSSAPI 兼容，但仅支持 Windows 单点登录场景，并且专门指代已登录用户。SSPI 是大多数 Windows 客户端的默认模式。

+   `GSSAPI` 模式：在 Windows 上通过 MIT Kerberos 库支持 GSSAPI（参见 GSSAPI 模式下的 Windows 客户端命令）。

使用 Kerberos 认证插件，应用程序和 MySQL 服务器能够使用 Kerberos 认证协议相互认证用户和 MySQL 服务。这样，用户和服务器都能够验证彼此的身份。不会通过网络发送密码，并且 Kerberos 协议消息受到窃听和重放攻击的保护。

MySQL 中的 Kerberos 认证遵循这些步骤，其中服务器端和客户端部分分别使用 `authentication_kerberos` 和 `authentication_kerberos_client` 认证插件执行：

1.  MySQL 服务器向客户端应用程序发送其服务主体名称。此 SPN 必须在 Kerberos 系统中注册，并且在服务器端使用 `authentication_kerberos_service_principal` 系统变量进行配置。

1.  使用 GSSAPI，客户端应用程序创建一个 Kerberos 客户端端身份验证会话，并与 Kerberos KDC 交换 Kerberos 消息：

    +   客户端从认证服务器获取票据授予票据。

    +   使用 TGT，客户端从票据授予服务获取 MySQL 的服务票据。

    如果 TGT、ST 或两者都已在本地缓存，则可以跳过或部分跳过此步骤。客户端可以选择使用客户端 keytab 文件来获取 TGT 和 ST，而无需提供密码。

1.  使用 GSSAPI，客户端应用程序向 MySQL 服务器呈现 MySQL ST。

1.  使用 GSSAPI，MySQL 服务器创建一个 Kerberos 服务器端认证会话。服务器验证用户身份和用户请求的有效性。它使用其服务 keytab 文件中配置的服务密钥验证 ST，以确定身份验证成功或失败，并将身份验证结果返回给客户端。

应用程序可以使用提供的用户名和密码进行身份验证，也可以使用本地缓存的 TGT 或 ST（例如，使用 **kinit** 或类似方法创建）。因此，此设计涵盖了各种用例，从完全无用户和无密码连接，其中 Kerberos 服务票据是从本地存储的 Kerberos 缓存获取的，到提供并使用用户名和密码以获取有效的 Kerberos 服务票据从 KDC 发送到 MySQL 服务器的连接。

如前述描述所示，MySQL 的 Kerberos 认证使用两种类型的 keytab 文件：

+   在客户端主机上，可以使用客户端 keytab 文件来获取 TGT 和 ST，而无需提供密码。请参阅 Client Configuration Parameters for Kerberos Authentication。

+   在 MySQL 服务器主机上，使用服务器端服务 keytab 文件来验证 MySQL 服务器从客户端接收的服务票据。 keytab 文件名是使用 `authentication_kerberos_service_key_tab` 系统变量配置的。

有关 keytab 文件的信息，请参阅 [`web.mit.edu/kerberos/krb5-latest/doc/basic/keytab_def.html`](https://web.mit.edu/kerberos/krb5-latest/doc/basic/keytab_def.html)。

##### 安装 Kerberos 可插拔认证

本节描述了如何安装服务器端 Kerberos 认证插件。有关安装插件的一般信息，请参阅 Section 7.6.1, “Installing and Uninstalling Plugins”。

注意

服务器端插件仅在 Linux 系统上受支持。在 Windows 系统上，仅支持客户端端插件（截至 MySQL 8.0.27），可以在 Windows 系统上用于连接使用 Kerberos 认证的 Linux 服务器。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。必要时，通过设置`plugin_dir`的值来配置插件目录位置以在服务器启动时生效。

服务器端插件库文件基本名称为`authentication_kerberos`。Unix 和类 Unix 系统的文件名后缀为`.so`。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，每次服务器启动时都必须提供该选项。还要为您希望配置的任何插件提供的系统变量指定值。插件公开这些系统变量，使其操作可以配置：

+   `authentication_kerberos_service_principal`: MySQL 服务主体名称（SPN）。此名称将发送给尝试使用 Kerberos 进行身份验证的客户端。SPN 必须存在于 KDC 服务器管理的数据库中。默认值为`mysql/*`host_name`*@*`realm_name`*`。

+   `authentication_kerberos_service_key_tab`: 用于验证从客户端接收到的票证的 keytab 文件。此文件必须存在并包含 SPN 的有效密钥，否则客户端的身份验证将失败。默认值为数据目录中的`mysql.keytab`。

有关所有 Kerberos 身份验证系统变量的详细信息，请参见 Section 8.4.1.13, “Pluggable Authentication System Variables”。

要加载插件并配置它，请在您的`my.cnf`文件中放入类似以下行的内容，使用适合您安装的系统变量的值：

```sql
[mysqld]
plugin-load-add=authentication_kerberos.so
authentication_kerberos_service_principal=mysql/krbauth.example.com@MYSQL.LOCAL
authentication_kerberos_service_key_tab=/var/mysql/data/mysql.keytab
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用以下语句：

```sql
INSTALL PLUGIN authentication_kerberos
  SONAME 'authentication_kerberos.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

当您在运行时安装插件而没有在`my.cnf`文件中配置其系统变量时，系统变量 `authentication_kerberos_service_key_tab` 将设置为数据目录中的默认值`mysql.keytab`。此系统变量的值不能在运行时更改，因此如果需要指定不同的文件，您需要将设置添加到您的`my.cnf`文件，然后重新启动 MySQL 服务器。例如：

```sql
[mysqld]
authentication_kerberos_service_key_tab=/var/mysql/data/mysql.keytab
```

如果 keytab 文件不在正确的位置或不包含有效的 SPN 密钥，则 MySQL 服务器不会验证此问题，但客户端会返回身份验证错误，直到您解决问题。

`authentication_kerberos_service_principal` 系统变量可以在运行时使用 `SET PERSIST` 语句设置和持久化，而无需重新启动服务器：

```sql
SET PERSIST authentication_kerberos_service_principal='mysql/krbauth.example.com@MYSQL.LOCAL';
```

`SET PERSIST` 为运行中的 MySQL 实例设置一个值。它还保存该值，导致其在后续服务器重启时保留。要更改运行中的 MySQL 实例的值，而不使其在后续重启中保留，使用`GLOBAL`关键字而不是`PERSIST`。参见 Section 15.7.6.1, “变量赋值的 SET 语法”。

要验证插件安装，请检查信息模式 `PLUGINS` 表或使用 `SHOW PLUGINS` 语句（参见 Section 7.6.2, “获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME = 'authentication_kerberos';
+-------------------------+---------------+
| PLUGIN_NAME             | PLUGIN_STATUS |
+-------------------------+---------------+
| authentication_kerberos | ACTIVE        |
+-------------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断信息。

要将 MySQL 帐户与 Kerberos 插件关联，请参见 使用 Kerberos 可插拔认证。

##### 使用 Kerberos 可插拔认证

本节描述了如何启用 MySQL 帐户使用 Kerberos 可插拔认证连接到 MySQL 服务器。假定服务器正在运行并启用了服务器端插件，如 安装 Kerberos 可插拔认证 中所述，并且客户端主机上可用客户端插件。

+   验证 Kerberos 可用性

+   创建一个使用 Kerberos 身份验证的 MySQL 帐户

+   使用 MySQL 帐户连接到 MySQL 服务器

+   Kerberos 身份验证的客户端配置参数

###### 验证 Kerberos 的可用性

以下示例显示如何测试 Active Directory 中 Kerberos 的可用性。该示例做出以下假设：

+   Active Directory 正在名为`krbauth.example.com`的主机上运行，IP 地址为`198.51.100.11`。

+   与 MySQL 相关的 Kerberos 身份验证使用`MYSQL.LOCAL`域，并且还使用`MYSQL.LOCAL`作为领域名称。

+   名为`karl@MYSQL.LOCAL`的主体已在 KDC 中注册。（在后续讨论中，此主体名称与使用 Kerberos 身份验证连接到 MySQL 服务器的 MySQL 帐户相关联。）

在满足这些假设的情况下，按照以下步骤进行：

1.  验证操作系统中是否正确安装和配置了 Kerberos 库。例如，要在 MySQL 身份验证期间使用`MYSQL.LOCAL`域和领域，`/etc/krb5.conf` Kerberos 配置文件应包含类似于以下内容：

    ```sql
    [realms]
     MYSQL.LOCAL = {
     kdc = krbauth.example.com
     admin_server = krbauth.example.com
     default_domain = MYSQL.LOCAL
      }
    ```

1.  您可能需要为服务器主机在`/etc/hosts`中添加一个条目：

    ```sql
    198.51.100.11 krbauth krbauth.example.com
    ```

1.  检查 Kerberos 身份验证是否正常工作：

    1.  使用**kinit**进行 Kerberos 身份验证：

        ```sql
        $> kinit karl@MYSQL.LOCAL
        Password for karl@MYSQL.LOCAL: *(enter password here)*
        ```

        该命令对名为`karl@MYSQL.LOCAL`的 Kerberos 主体进行身份验证。当命令提示时输入主体密码。KDC 返回一个 TGT，该 TGT 在客户端缓存以供其他了解 Kerberos 的应用程序使用。

    1.  使用**klist**检查 TGT 是否正确获取。输出应类似于以下内容：

        ```sql
        $> klist
        Ticket cache: FILE:/tmp/krb5cc_244306
        Default principal: karl@MYSQL.LOCAL

        Valid starting       Expires              Service principal
        03/23/2021 08:18:33  03/23/2021 18:18:33  krbtgt/MYSQL.LOCAL@MYSQL.LOCAL
        ```

###### 创建一个使用 Kerberos 身份验证的 MySQL 帐户

使用`authentication_kerberos`身份验证插件进行 MySQL 身份验证基于 Kerberos 用户主体名称（UPN）。这里的说明假设一个名为`karl`的 MySQL 用户使用 Kerberos 进行 MySQL 身份验证，Kerberos 领域命名为`MYSQL.LOCAL`，用户主体名称为`karl@MYSQL.LOCAL`。这个 UPN 必须在几个地方注册：

+   Kerberos 管理员应将用户名称注册为 Kerberos 主体。此名称包括领域名称。客户端使用主体名称和密码进行 Kerberos 身份验证，并获取票据授予票据（TGT）。

+   MySQL DBA 应创建一个与 Kerberos 主体名称对应并使用 Kerberos 插件进行身份验证的帐户。

假设适当的服务管理员已注册了 Kerberos 用户主体名称，并且如前所述在安装 Kerberos 可插拔认证中，MySQL 服务器已使用适当的配置设置启动了服务器端 Kerberos 插件。要创建与 Kerberos UPN `*`user`*@*`realm_name`*`对应的 MySQL 帐户，MySQL DBA 使用类似于以下语句：

```sql
CREATE USER *user*
  IDENTIFIED WITH authentication_kerberos
  BY '*realm_name*';
```

由*`user`*指定的帐户可以包含或省略主机名部分。如果省略主机名，则通常默认为`%`。*`realm_name`*存储为`mysql.user`系统表中帐户的`authentication_string`值。

要创建与 UPN `karl@MYSQL.LOCAL`对应的 MySQL 帐户，请使用此语句：

```sql
CREATE USER 'karl'
  IDENTIFIED WITH authentication_kerberos
  BY 'MYSQL.LOCAL';
```

如果 MySQL 必须为此帐户构造 UPN，例如获取或验证票证（TGT 或 ST），则通过组合帐户名（忽略任何主机名部分）和领域名来执行此操作。例如，从前述`CREATE USER`语句中得到的完整帐户名为`'karl'@'%'`。MySQL 从用户名部分`karl`（忽略主机名部分）和领域名`MYSQL.LOCAL`构造 UPN，以生成`karl@MYSQL.LOCAL`。

注意

请注意，当创建使用`authentication_kerberos`进行身份验证的帐户时，`CREATE USER`语句不包括 UPN 领域作为用户名的一部分。而是在`BY`子句中指定领域（在本例中为`MYSQL.LOCAL`）作为认证字符串。这与使用 GSSAPI/Kerberos 认证方法的`authentication_ldap_sasl` SASL LDAP 认证插件创建帐户不同。对于这样的帐户，`CREATE USER`语句包括 UPN 领域作为用户名的一部分。请参阅创建使用 GSSAPI/Kerberos 进行 LDAP 认证的 MySQL 帐户。

设置好帐户后，客户端可以使用它连接到 MySQL 服务器。该过程取决于客户端主机是运行 Linux 还是 Windows，如下面的讨论所示。

使用`authentication_kerberos`受到限制，即不支持具有相同用户部分但不同领域部分的 UPN。例如，您不能创建对应于以下这两个 UPN 的 MySQL 帐户：

```sql
kate@MYSQL.LOCAL
kate@EXAMPLE.COM
```

两个 UPN 都具有`kate`的用户部分，但领域部分不同（`MYSQL.LOCAL`与`EXAMPLE.COM`）。这是不允许的。

###### 使用 MySQL 帐户连接到 MySQL 服务器

设置了使用 Kerberos 进行身份验证的 MySQL 帐户后，客户端可以按以下方式连接到 MySQL 服务器：

1.  使用用户主体名称（UPN）和其密码对 Kerberos 进行身份验证，以获取票据授予票据（TGT）。

1.  使用 TGT 获取 MySQL 的服务票据（ST）。

1.  通过呈现 MySQL ST 向 MySQL 服务器进行身份验证。

第一步（向 Kerberos 进行身份验证）可以通过各种方式执行：

+   在连接到 MySQL 之前：

    +   在 Linux 或在`GSSAPI`模式下的 Windows 上，调用**kinit**以获取 TGT 并将其保存在 Kerberos 凭据缓存中。

    +   在`SSPI`模式下的 Windows 中，认证可能已在登录时完成，这会将登录用户的 TGT 保存在 Windows 内存缓存中。**kinit**不会被使用，也没有 Kerberos 缓存。

+   连接到 MySQL 时，客户端程序本身可以获取 TGT，如果它可以确定所需的 Kerberos UPN 和密码：

    +   信息可以来自诸如命令选项或操作系统之类的来源。

    +   在 Linux 上，客户端还可以使用 keytab 文件或`/etc/krb5.conf`配置文件。在`GSSAPI`模式下的 Windows 客户端使用配置文件。在`SSPI`模式下��Windows 客户端两者都不使用。

连接到 MySQL 服务器的客户端命令的详细信息因 Linux 和 Windows 而异，因此每个主机类型都会单独讨论，但这些命令属性适用于所有主机类型：

+   每个显示的命令都包括以下选项，但在某些情况下可以省略每个选项：

    +   `--default-auth`选项指定客户端端身份验证插件的名称（`authentication_kerberos_client`）。当指定`--user`选项时，可以省略此选项，因为在这种情况下，MySQL 可以从 MySQL 服务器发送的用户帐户信息中确定插件。

    +   `--plugin-dir`选项指示客户端程序`authentication_kerberos_client`插件的位置。如果插件安装在默认（编译内）位置，则可以省略此选项。

+   命令还应包括任何其他选项，如`--host`或`--port`，以指定连接到哪个 MySQL 服务器。

+   每个命令都要单独输入一行。如果命令包括`--password`选项以获取密码，请在提示时输入与 MySQL 用户关联的 Kerberos UPN 的密码。

**Linux 客户端连接命令**

在 Linux 上，连接到 MySQL 服务器的适当客户端命令因命令是使用来自 Kerberos 缓存的 TGT 进行身份验证，还是基于 MySQL 用户名和 UPN 密码的命令选项而异：

+   在调用 MySQL 客户端程序之前，客户端用户可以独立于 MySQL 从 KDC 获取 TGT。例如，客户端用户可以使用**kinit**通过提供 Kerberos 用户主体名称和主体密码来对 Kerberos 进行身份验证：

    ```sql
    $> kinit karl@MYSQL.LOCAL
    Password for karl@MYSQL.LOCAL: *(enter password here)*
    ```

    生成的 UPN 的 TGT 被缓存，并可供其他了解 Kerberos 的应用程序使用，例如使用客户端端 Kerberos 认证插件的程序。在这种情况下，调用客户端时不指定用户名称或密码选项：

    ```sql
    mysql
      --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
    ```

    客户端插件在缓存中找到 TGT，使用它获取 MySQL ST，并使用 ST 对 MySQL 服务器进行身份验证。

    正如刚才描述的，当 UPN 的 TGT 被缓存时，在客户端命令中不需要用户名称和密码选项。如果命令仍然包含它们，则处理如下：

    +   此命令包括一个用户名选项：

        ```sql
        mysql
          --default-auth=authentication_kerberos_client
          --plugin-dir=*path/to/plugin/directory*
          --user=karl
        ```

        在这种情况下，如果选项指定的用户名与 TGT 中的 UPN 的用户名部分不匹配，则身份验证将失败。

    +   此命令包括一个密码选项，在提示时输入：

        ```sql
        mysql
          --default-auth=authentication_kerberos_client
          --plugin-dir=*path/to/plugin/directory*
          --password
        ```

        在这种情况下，客户端端插件会忽略密码。因为身份验证是基于 TGT 的，即使用户提供的密码不正确，也可以成功*。因此，如果找到导致密码被忽略的有效 TGT，插件会产生警告。

+   如果 Kerberos 缓存中不包含 TGT，则客户端端 Kerberos 认证插件本身可以从 KDC 获取 TGT。使用 MySQL 用户名和密码的选项调用客户端，然后在提示时输入 UPN 密码：

    ```sql
    mysql --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
      --user=karl
      --password
    ```

    客户端端的 Kerberos 认证插件将用户名称（`karl`）与用户帐户中指定的领域（`MYSQL.LOCAL`）结合起来构建 UPN（`karl@MYSQL.LOCAL`）。客户端插件使用 UPN 和密码获取 TGT，使用 TGT 获取 MySQL ST，并使用 ST 对 MySQL 服务器进行身份验证。

    或者，假设 Kerberos 缓存中不包含 TGT，并且命令指定了密码选项但没有用户名称选项：

    ```sql
    mysql --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
      --password
    ```

    客户端端 Kerberos 认证插件使用操作系统登录名作为 MySQL 用户名。它将该用户名与用户的 MySQL 帐户中的领域结合起来构建 UPN。客户端插件使用 UPN 和密码获取 TGT，使用 TGT 获取 MySQL ST，并使用 ST 对 MySQL 服务器进行身份验证。

如果您不确定是否存在 TGT，可以使用**klist**进行检查。

注意

当客户端端 Kerberos 认证插件本身获取 TGT 时，客户端用户可能不希望 TGT 被重用。如用于 Kerberos 认证的客户端配置参数中所述，本地`/etc/krb5.conf`文件可用于使客户端插件在完成后销毁 TGT。

**SSPI 模式下 Windows 客户端的连接命令**

在 Windows 上，使用默认的客户端插件选项（SSPI），连接到 MySQL 服务器的适当客户端命令取决于命令是基于 MySQL 用户名和 UPN 密码的命令选项进行身份验证，还是使用 Windows 内存缓存中的 TGT。有关 Windows 上 GSSAPI 模式的详细信息，请参阅 GSSAPI 模式下 Windows 客户端的命令。

一个命令可以明确指定 MySQL 用户名和 UPN 密码的选项，也可以省略这些选项：

+   此命令包括 MySQL 用户名和 UPN 密码的选项：

    ```sql
    mysql --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
      --user=karl
      --password
    ```

    客户端端的 Kerberos 认证插件将用户名称（`karl`）和用户帐户中指定的领域（`MYSQL.LOCAL`）结合起来构建 UPN（`karl@MYSQL.LOCAL`）。客户端插件使用 UPN 和密码获取 TGT，使用 TGT 获取 MySQL ST，并使用 ST 对 MySQL 服务器进行身份验证。

    Windows 内存缓存中的任何信息都将被忽略；用户名称和密码选项值优先。

+   此命令包括 UPN 密码选项，但不包括 MySQL 用户名选项：

    ```sql
    mysql
      --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
      --password
    ```

    客户端端的 Kerberos 认证插件使用已登录用户名称作为 MySQL 用户名，并将该用户名与用户 MySQL 帐户中的领域结合起来构建 UPN。客户端插件使用 UPN 和密码获取 TGT，使用 TGT 获取 MySQL ST，并使用 ST 对 MySQL 服务器进行身份验证。

+   此命令不包括 MySQL 用户名或 UPN 密码的选项：

    ```sql
    mysql
      --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
    ```

    客户端端插件从 Windows 内存缓存中获取 TGT，使用 TGT 获取 MySQL ST，并使用 ST 对 MySQL 服务器进行身份验证。

    这种方法要求客户端主机是 Windows Server Active Directory (AD) 域的一部分。如果不是这种情况，请通过手动输入 AD 服务器和领域作为 DNS 服务器和前缀来帮助 MySQL 客户端发现 AD 域的 IP 地址：

    1.  启动 `console.exe` 并选择网络和共享中心。

    1.  从网络和共享中心窗口的侧边栏中，选择更改适配器设置。

    1.  在网络连接窗口中，右键单击要配置的网络或 VPN 连接，然后选择属性。

    1.  从网络选项卡中，找到并单击 Internet 协议版本 4 (TCP/IPv4)，然后单击属性。

    1.  在 Internet 协议版本 4 (TCP/IPv4) 属性对话框中，单击高级。打开高级 TCP/IP 设置对话框。

    1.  从 DNS 选项卡中，将 Active Directory 服务器和领域添加为 DNS 服务器和前缀。

+   此命令包括 MySQL 用户名选项，但不包括 UPN 密码选项：

    ```sql
    mysql
      --default-auth=authentication_kerberos_client
      --plugin-dir=*path/to/plugin/directory*
      --user=karl
    ```

    客户端端的 Kerberos 身份验证插件将由用户名称选项指定的名称与登录用户名进行比较。如果名称相同，则插件使用登录用户的 TGT 进行身份验证。如果名称不同，则身份验证失败。

**在 GSSAPI 模式下的 Windows 客户端连接命令**

在 Windows 上，客户端用户必须明确使用 `plugin_authentication_kerberos_client_mode` 插件选项指定 `GSSAPI` 模式，以通过 MIT Kerberos 库启用支持。默认模式为 `SSPI`（请参阅 SSPI 模式下 Windows 客户端的命令）。

可以指定 `GSSAPI` 模式：

+   在运行 Windows 主机上始终从 MIT Kerberos 缓存中检索或放置票证。

    ```sql
    [mysql]
    plugin_authentication_kerberos_client_mode=GSSAPI
    ```

    或：

    ```sql
    [mysql]
    plugin-authentication-kerberos-client-mode=GSSAPI
    ```

+   在运行时从命令行使用 **mysql** 或 **mysqldump** 客户端程序。例如，以下命令（带有下划线或破折号）使 **mysql** 通过 MIT Kerberos 库连接到 Windows 上的服务器。

    ```sql
    mysql *[connection-options]* --plugin_authentication_kerberos_client_mode=GSSAPI
    ```

    或：

    ```sql
    mysql *[connection-options]* --plugin-authentication-kerberos-client-mode=GSSAPI
    ```

+   客户端用户可以从 MySQL Workbench 和一些 MySQL 连接器中选择 `GSSAPI` 模式。在运行 Windows 的客户端主机上，您可以覆盖默认位置：

    +   通过设置 `KRB5_CONFIG` 环境变量来配置 Kerberos 配置文件。

    +   使用 `KRB5CCNAME` 环境变量设置默认凭据缓存名称（例如，`KRB5CCNAME=DIR:/mydir/`）。

    有关特定客户端插件信息，请参阅 `dev.mysql.com/doc/` 文档。

连接到 MySQL 服务器的适当客户端命令因命令是使用 MIT Kerberos 缓存中的 TGT 进行身份验证，还是基于 MySQL 用户名称和 UPN 密码的命令选项而异。在 Windows 上通过 MIT 库支持的 GSSAPI 与 Linux 上的 GSSAPI 类似（请参阅 Linux 客户端命令），但有以下例外：

+   在运行 MySQL 客户端程序之前，在选项文件中。插件变量名称可以使用下划线或破折号：

+   **kinit** 在 Windows 上以具有狭窄权限和特定角色的功能帐户运行。客户端用户不知道 **kinit** 密码。有关概述，请参阅 [`docs.oracle.com/en/java/javase/11/tools/kinit.html`](https://docs.oracle.com/en/java/javase/11/tools/kinit.html)。

+   如果客户端用户提供密码，MIT Kerberos 库在 Windows 上决定是使用密码还是依赖现有票证。

+   描述在用于 Kerberos 认证的客户端配置参数中的`destroy_tickets`参数不受支持，因为 Windows 上的 MIT Kerberos 库不支持所需的 API 成员（`get_profile_boolean`）来从配置文件中读取其值。

###### 用于 Kerberos 认证的客户端配置参数

本节仅适用于运行 Linux 的客户端主机，不适用于运行 Windows 的客户端主机。

注意

运行 Windows 的客户端主机，将`authentication_kerberos_client`客户端 Kerberos 插件设置为`GSSAPI`模式支持客户端配置参数，但 Windows 上的 MIT Kerberos 库不支持本节中描述的`destroy_tickets`参数。

如果在 MySQL 客户端应用程序调用时不存在有效的票证授予票证（TGT），应用程序本身可以获取并缓存 TGT。如果在 Kerberos 认证过程中，客户端应用程序导致 TGT 被缓存，任何添加的 TGT 在不再需要时可以通过设置适当的配置参数销毁。

`authentication_kerberos_client`客户端 Kerberos 插件读取本地的`/etc/krb5.conf`文件。如果该文件丢失或无法访问，将会出现错误。假设文件可访问，它可以包含一个可选的`[appdefaults]`部分，提供插件使用的信息。将信息放在该部分的`mysql`部分内。例如：

```sql
[appdefaults]
 mysql = {
 destroy_tickets = true
  }
```

客户端插件在`mysql`部分识别这些参数：

+   `destroy_tickets`值指示客户端插件在获取和使用 TGT 后是否销毁 TGT。默认情况下，`destroy_tickets`为`false`，但可以设置为`true`以避免 TGT 重用。（此设置仅适用于由客户端插件创建的 TGT，不适用于由其他插件或外部 MySQL 创建的 TGT。）

在客户端主机上，可以使用客户端 keytab 文件来获取 TGT 和 TS 而无需提供密码。有关 keytab 文件的信息，请参阅[`web.mit.edu/kerberos/krb5-latest/doc/basic/keytab_def.html`](https://web.mit.edu/kerberos/krb5-latest/doc/basic/keytab_def.html)。

##### Kerberos 认证调试

`AUTHENTICATION_KERBEROS_CLIENT_LOG`环境变量用于启用或禁用 Kerberos 认证的调试输出。

注意

尽管`AUTHENTICATION_KERBEROS_CLIENT_LOG`中包含`CLIENT`，但同一环境变量也适用于服务器端插件以及客户端插件。

在服务器端，允许的值为 0（关闭）和 1（开启）。日志消息写入服务器错误日志，受服务器错误日志详细级别的影响。例如，如果您正在使用基于优先级的日志过滤，`log_error_verbosity` 系统变量控制详细程度，如第 7.4.2.5 节“基于优先级的错误日志过滤（log_filter_internal）”中所述。

在客户端，允许的值为 1 到 5，并写入标准错误输出。以下表格显示了每个日志级别值的含义。

| 日志级别 | 含义 |
| --- | --- |
| 1 或未设置 | 无日志记录 |
| 2 | 错误消息 |
| 3 | 错误和警告消息 |
| 4 | 错误、警告和信息消息 |
| 5 | 错误、警告、信息和调试消息 |
