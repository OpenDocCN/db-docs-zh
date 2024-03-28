> 原文：[`dev.mysql.com/doc/refman/8.0/en/fido-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/fido-pluggable-authentication.html)

#### 8.4.1.11 FIDO 可插拔认证

注意

FIDO 可插拔认证是包含在 MySQL Enterprise Edition 中的扩展，这是一款商业产品。要了解更多关于商业产品的信息，请参见 [`www.mysql.com/products/`](https://www.mysql.com/products/)。

MySQL Enterprise Edition 支持一种认证方法，允许用户使用 FIDO 认证对 MySQL 服务器进行认证。从 MySQL 8.0.35 开始，此认证方法已被弃用，并可能在未来的 MySQL 版本中被移除。为了获得类似的功能，请考虑升级到 MySQL 8.2（或更高版本），用户可以使用 WebAuthn 认证 对 MySQL 服务器进行认证。在进行升级之前，您需要了解 MySQL 创新和长期支持（LTS）版本的发布模型。有关更多信息，请参见 Section 3.2, “升级路径”。

FIDO 代表快速身份在线，提供了不需要使用密码的认证标准。

FIDO 可插拔认证提供以下功能：

+   FIDO 可以使用智能卡、安全密钥和生物识别读卡器等设备对 MySQL 服务器进行认证。

+   因为认证可以通过除提供密码外的其他方式进行，所以 FIDO 实现了无密码认证。

+   另一方面，设备认证通常与密码认证一起使用，因此 FIDO 认证可用于使用多因素认证的 MySQL 帐户；参见 Section 8.2.18, “多因素认证”。

以下表格显示了插件和库文件的名称。文件名后缀可能因系统而异。Unix 和类 Unix 系统通常使用 `.so`，Windows 系统使用 `.dll`。文件必须位于由 `plugin_dir` 系统变量指定的目录中。有关安装信息，请参见 安装 FIDO 可插拔认证。

**表 8.27 FIDO 认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `authentication_fido` |
| 客户端插件 | `authentication_fido_client` |
| 库文件 | `authentication_fido.so`，`authentication_fido_client.so` |

注意

在使用服务器端或客户端 FIDO 认证插件的系统上必须可用 `libfido2` 库。如果主机上有多个 FIDO 设备，则 `libfido2` 库决定用于注册和认证的设备。`libfido2` 库不提供设备选择功能。

服务器端的 FIDO 身份验证插件仅包含在 MySQL 企业版中。它不包含在 MySQL 社区发行版中。客户端插件包含在所有发行版中，包括社区发行版，这使得来自任何发行版的客户端都可以连接到加载了服务器端插件的服务器。

以下各节提供了特定于 FIDO 可插拔认证的安装和使用信息：

+   安装 FIDO 可插拔认证

+   使用 FIDO 认证

+   FIDO 免密码认证

+   FIDO 设备注销

+   MySQL 用户 FIDO 认证的工作原理

有关 MySQL 中可插拔认证的一般信息，请参见第 8.2.17 节，“可插拔认证”。

##### 安装 FIDO 可插拔认证

本节描述了如何安装服务器端的 FIDO 身份验证插件。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如果需要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

服务器端插件库文件基本名称为`authentication_fido`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为`.so`，对于 Windows 为`.dll`）。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，选项必须在每次服务器启动时给出。

要加载插件，请在您的`my.cnf`文件中添加类似以下行，根据需要调整`.so`后缀以适应您的平台：

```sql
[mysqld]
plugin-load-add=authentication_fido.so
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用此语句，根据需要调整`.so`后缀以适应您的平台：

```sql
INSTALL PLUGIN authentication_fido
  SONAME 'authentication_fido.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见 Section 7.6.2，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME = 'authentication_fido';
+---------------------+---------------+
| PLUGIN_NAME         | PLUGIN_STATUS |
+---------------------+---------------+
| authentication_fido | ACTIVE        |
+---------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

要将 MySQL 帐户与 FIDO 认证插件关联，请参见使用 FIDO 认证。

##### 使用 FIDO 认证

FIDO 认证通常用于多因素认证的背景下（参见 Section 8.2.18，“多因素认证”）。本节展示了如何将基于 FIDO 设备的认证纳入多因素帐户中，使用`authentication_fido`插件。

在以下讨论中假定服务器正在运行，并启用了服务器端 FIDO 认证插件，如安装 FIDO 可插拔认证中所述，并且客户端 FIDO 插件在客户端主机的插件目录中可用。

注意

在 Windows 上，只有当客户端进程以具有管理员权限的用户身份运行时，FIDO 认证才能正常工作。

还假定 FIDO 认证与非 FIDO 认证一起使用（这意味着 2FA 或 3FA 帐户）。FIDO 也可以单独使用，创建以无密码方式进行身份验证的 1FA 帐户。在这种情况下，设置过程略有不同。有关说明，请参见 FIDO 无密码认证。

配置为使用`authentication_fido`插件的帐户与 FIDO 设备关联。因此，在进行 FIDO 认证之前需要进行一次设备注册步骤。设备注册过程具有以下特点：

+   与帐户关联的任何 FIDO 设备必须在使用帐户之前注册。

+   注册要求客户端主机上有一个 FIDO 设备，否则注册将失败。

+   用户在注册过程中收到提示时，应执行适当的 FIDO 设备操作（例如，触摸设备或进行生物识别扫描）。

+   要执行设备注册，客户端用户必须调用**mysql**客户端程序或 MySQL Shell，并指定`--fido-register-factor`选项以指定正在注册设备的因素或因素。例如，如果帐户设置为将 FIDO 用作第二身份验证因素，则用户使用`--fido-register-factor=2`选项调用**mysql**。

+   如果用户帐户配置为将`authentication_fido`插件设置为第二或第三因素，则在注册步骤可以继续之前，所有前面因素的身份验证必须成功。

+   服务器根据用户帐户中的信息知道 FIDO 设备是否需要注册或已经注册。当客户端程序连接时，如果设备必须注册，服务器会将客户端会话置于沙盒模式，以便在执行其他操作之前必须进行注册。用于 FIDO 设备注册的沙盒模式类似于处理过期密码的模式。请参阅第 8.2.16 节，“服务器处理过期密码”。

+   在沙盒模式下，除了`ALTER USER`语句之外，不允许使用其他语句。注册是通过此语句的形式执行的。当使用`--fido-register-factor`选项调用时，**mysql**客户端会生成执行注册所需的`ALTER USER`语句。注册完成后，服务器会将会话从沙盒模式切换出来，客户端可以正常进行操作。有关生成的`ALTER USER`语句的信息，请参考`--fido-register-factor`的描述。

+   当为帐户执行设备注册后，服务器会更新该帐户的`mysql.user`系统表行，以更新设备注册状态并存储公钥和凭证 ID。

+   只能由帐户指定的用户执行注册步骤。如果一个用户尝试为另一个用户执行注册，则会出现错误。

+   用户在注册和身份验证过程中应使用相同的 FIDO 设备。如果在客户端主机上注册了 FIDO 设备后，重置设备或插入不同设备，则身份验证将失败。在这种情况下，必须取消与帐户关联的设备注册，并重新进行注册。

假设您希望一个账户首先使用 `caching_sha2_password` 插件进行认证，然后再使用 `authentication_fido` 插件进行认证。可以使用类似以下语句创建多因素账户：

```sql
CREATE USER 'u2'@'localhost'
  IDENTIFIED WITH caching_sha2_password
    BY '*sha2_password*'
  AND IDENTIFIED WITH authentication_fido;
```

要连接，提供因素 1 密码以满足该因素的认证，并将 `--fido-register-factor` 设置为因素 2 以启动 FIDO 设备的注册。

```sql
$> mysql --user=u2 --password1 --fido-register-factor=2
Enter password: *(enter factor 1 password)*
```

一旦因素 1 密码被接受，客户端会进入沙盒模式，以便为因素 2 进行设备注册。在注册过程中，您将被提示执行适当的 FIDO 设备操作，例如触摸设备或进行生物识别扫描。

当注册过程完成时，连接到服务器是允许的。

注意

在注册后，无论账户的认证链中是否存在额外的认证因素，连接到服务器都是允许的。例如，如果前面的示例中的账户定义了第三个认证因素（使用非 FIDO 认证），则在成功注册后连接将被允许，而无需对第三个因素进行认证。然而，后续的连接将需要对所有三个因素进行认证。

##### FIDO 无密码认证

本节描述了如何单独使用 FIDO 创建支持无密码认证的 1FA 账户。在这种情况下，“无密码”意味着认证会发生，但使用的是除密码之外的方法，例如安全密钥或生物识别扫描。这并不是指使用密码为空的基于密码的认证插件的账户。那种“无密码”是完全不安全的，不建议使用。

使用 `authentication_fido` 插件实现无密码认证时，需要满足以下先决条件：

+   创建无密码认证账户的用户需要`PASSWORDLESS_USER_ADMIN`权限，以及`CREATE USER`权限。

+   `authentication_policy` 值的第一个元素必须是星号（`*`），而不是插件名称。例如，默认的 `authentication_policy` 值支持启用无密码认证，因为第一个元素是星号：

    ```sql
    authentication_policy='*,,'
    ```

    有关配置 `authentication_policy` 值的信息，请参阅配置多因素认证策略。

要将`authentication_fido`用作免密码认证方法，必须将帐户创建为将`authentication_fido`作为第一因素认证方法。还必须为第一因素指定`INITIAL AUTHENTICATION IDENTIFIED BY`子句（不支持第 2 或第 3 因素）。此子句指定 FIDO 设备注册时将使用随机生成的还是用户指定的密码。设备注册后，服务器会删除密码并修改帐户，使`authentication_fido`成为唯一的认证方法（1FA 方法）。

所需的`CREATE USER`语法如下：

```sql
CREATE USER *user*
  IDENTIFIED WITH authentication_fido
  INITIAL AUTHENTICATION IDENTIFIED BY {RANDOM PASSWORD | '*auth_string*'};
```

以下示例使用`RANDOM PASSWORD`语法：

```sql
mysql> CREATE USER 'u1'@'localhost'
         IDENTIFIED WITH authentication_fido
         INITIAL AUTHENTICATION IDENTIFIED BY RANDOM PASSWORD;
+------+-----------+----------------------+-------------+
| user | host      | generated password   | auth_factor |
+------+-----------+----------------------+-------------+
| u1   | localhost | 9XHK]M{l2rnD;VXyHzeF |           1 |
+------+-----------+----------------------+-------------+
```

要执行注册，用户必须使用与`INITIAL AUTHENTICATION IDENTIFIED BY`子句关联的密码对服务器进行身份验证，可以是随机生成的密码，也可以是`'*`auth_string`*'`值。如果帐户是刚刚创建的，用户执行此命令并在提示处粘贴先前生成的随机密码（`9XHK]M{l2rnD;VXyHzeF`）：

```sql
$> mysql --user=u1 --password --fido-register-factor=2
Enter password:
```

选项`--fido-register-factor=2`用于表示`INITIAL AUTHENTICATION IDENTIFIED BY`子句当前充当第一因素认证方法。因此，用户必须使用第二因素提供临时密码。成功注册后，服务器将删除临时密码并修改`mysql.user`系统表中的帐户条目，将`authentication_fido`列为唯一（1FA）认证方法。

创建无密码认证帐户时，重要的是在`CREATE USER`语句中包含`INITIAL AUTHENTICATION IDENTIFIED BY`子句。服务器将接受没有子句的语句，但由于没有办法连接到服务器注册设备，因此生成的帐户无法使用。假设您执行了这样的语句：

```sql
CREATE USER 'u2'@'localhost'
  IDENTIFIED WITH authentication_fido;
```

后续尝试使用帐户连接将失败，如下所示：

```sql
$> mysql --user=u2 --skip-password
Failed to open FIDO device.
ERROR 1 (HY000): Unknown MySQL error
```

注意

使用通用第二因素（U2F）协议实现无密码认证，该协议不支持设置要注册的设备的 PIN 等其他安全措施。因此，设备持有者有责任确保设备以安全方式处理。

##### FIDO 设备注销

可以注销与 MySQL 帐户关联的 FIDO 设备。在多种情况下，这可能是可取或必要的：

+   要用不同设备替换 FIDO 设备。必须注销先前的设备并注册新设备。

    在这种情况下，帐户所有者或任何具有`CREATE USER`权限的用户都可以注销设备。帐户所有者可以注册新设备。

+   FIDO 设备被重置或丢失。直到当前设备被注销并执行新的注册为止，认证尝试将失败。

    在这种情况下，由于账户所有者无法进行身份验证，因此无法注销当前设备，必须联系 DBA（或任何具有`CREATE USER` 权限的用户）来执行此操作。然后账户所有者可以重新注册重置的设备或注册新设备。

注销 FIDO 设备可以由账户所有者或任何具有`CREATE USER` 权限的用户执行。使用以下语法：

```sql
ALTER USER *user* {2 | 3} FACTOR UNREGISTER;
```

要重新注册设备或执行新的注册，请参考使用 FIDO 认证 中的说明。

##### FIDO 认证 MySQL 用户的工作原理

本节概述了 MySQL 和 FIDO 如何共同工作以对 MySQL 用户进行认证。有关如何设置 MySQL 账户以使用 FIDO 认证插件的示例，请参见使用 FIDO 认证。

使用 FIDO 认证的账户必须在连接到服务器之前执行初始设备注册步骤。设备注册后，认证可以继续进行。FIDO 设备注册过程如下：

1.  服务器向客户端发送随机挑战、用户 ID 和依赖方 ID（唯一标识服务器），依赖方 ID 由`authentication_fido_rp_id` 系统变量定义。默认值为 `MySQL`。

1.  客户端接收该信息并将其发送给客户端 FIDO 认证插件，后者再将其提供给 FIDO 设备。

1.  用户执行适当的设备操作（例如，触摸设备或进行生物识别扫描）后，FIDO 设备生成公钥/私钥对、密钥句柄、X.509 证书和签名，然后返回给服务器。

1.  服务器端 FIDO 认证插件验证签名。验证成功后，服务器将凭证 ID 和公钥存储在 `mysql.user` 系统表中。

注册成功后，FIDO 认证遵循以下流程：

1.  服务器向客户端发送随机挑战、用户 ID、依赖方 ID 和凭证。

1.  客户端将相同的信息发送给 FIDO 设备。

1.  FIDO 设备提示用户执行适当的设备操作，根据注册时所做的选择。

1.  此操作解锁私钥并签署挑战。

1.  签署的挑战返回给服务器。

1.  服务器端 FIDO 认证插件使用公钥验证签名，并响应以指示认证成功或失败。
