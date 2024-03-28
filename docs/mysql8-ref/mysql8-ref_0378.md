# 8.2.18 多因素认证

> 原文：[`dev.mysql.com/doc/refman/8.0/en/multifactor-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/multifactor-authentication.html)

认证涉及一方向第二方证明其身份。多因素认证（MFA）是在认证过程中使用多个认证值（或“因素”）的方法。MFA 比单因素认证（1FA）提供更高的安全性，单因素认证仅使用一种认证方法，比如密码。MFA 可以启用额外的认证方法，比如使用多个密码进行认证，或者使用智能卡、安全密钥和生物识别读卡器进行认证。

MySQL 8.0.27 及更高版本支持多因素认证。该功能包括需要最多三个认证值的 MFA 形式。也就是说，MySQL 账户管理支持使用 2FA 或 3FA 的账户，除了现有的 1FA 支持。

当客户端尝试使用单因素账户连接到 MySQL 服务器时，服务器调用账户定义中指定的认证插件，并根据插件报告的成功或失败来接受或拒绝连接。

对于具有多个认证因素的账户，流程类似。服务器按照账户定义中列出的顺序调用认证插件。如果插件报告成功，服务器会接受连接（如果插件是最后一个），或者如果还有插件剩余，则继续调用下一个插件。如果任何插件报告失败，服务器将拒绝连接。

以下章节详细介绍了 MySQL 中的多因素认证。

+   多因素认证支持的元素

+   配置多因素认证策略

+   开始使用多因素认证

#### 多因素认证支持的元素

认证因素通常包括以下类型的信息：

+   你知道的东西，比如秘密密码或口令。

+   你拥有的东西，比如安全密钥或智能卡。

+   你的生物特征；比如指纹或面部扫描。

“你所知道的”因素类型依赖于在身份验证过程的双方都保密的信息。不幸的是，秘密可能会受到威胁：有人可能看到你输入密码，或者用网络钓鱼攻击欺骗你，服务器端存储的密码可能会在安全漏洞中暴露等。使用多个密码可以提高安全性，但每个密码仍然可能会受到威胁。使用其他因素类型可以在减少威胁的情况下提高安全性。

MySQL 中多因素身份验证的实现包括以下元素：

+   `authentication_policy` 系统变量控制可以使用多少身份验证因素以及每个因素允许的身份验证类型。也就是说，它对于 `CREATE USER` 和 `ALTER USER` 语句在多因素身份验证方面施加了约束。

+   `CREATE USER` 和 `ALTER USER` 具有语法，允许为新帐户指定多个身份验证方法，并为现有帐户添加、修改或删除身份验证方法。如果帐户使用 2FA 或 3FA，`mysql.user` 系统表会在 `User_attributes` 列中存储有关额外身份验证因素的信息。

+   为了启用使用需要多个密码的帐户进行 MySQL 服务器身份验证，客户端程序具有 `--password1`、`--password2` 和 `--password3` 选项，允许指定最多三个密码。对于使用 C API 的应用程序，`mysql_options4()` C API 函数的 `MYSQL_OPT_USER_PASSWORD` 选项可以实现相同的功能。

+   服务器端的 `authentication_fido` 插件（已弃用）允许使用设备进行身份验证。这个服务器端的 FIDO 身份验证插件仅包含在 MySQL Enterprise Edition 发行版中。它不包含在 MySQL 社区发行版中。然而，客户端的 `authentication_fido_client` 插件（已弃用）包含在所有发行版中，包括社区发行版。这使得来自任何发行版的客户端可以连接到使用 `authentication_fido` 在加载了该插件的服务器上进行身份验证的帐户。参见 Section 8.4.1.11, “FIDO Pluggable Authentication”。

+   `authentication_fido` 还可以实现免密码认证，如果它是账户唯一使用的认证插件。查看 FIDO 免密码认证。

+   多因素认证可以使用非 FIDO MySQL 认证方法、FIDO 认证方法或两者的组合。

+   这些权限使用户能够执行某些受限的多因素认证相关操作：

    +   拥有`AUTHENTICATION_POLICY_ADMIN` 权限的用户不受`authentication_policy` 系统变量强加的限制。 （对于否则不允许的语句会发出警告。）

    +   `PASSWORDLESS_USER_ADMIN` 权限允许创建免密码认证账户并对其进行操作复制。

#### 配置多因素认证策略

`authentication_policy` 系统变量定义了多因素认证策略。 具体来说，它定义了账户可以拥有（或需要拥有）多少认证因素以及可以用于每个因素的认证方法。

`authentication_policy` 的值是一个由 1、2 或 3 个逗号分隔的元素组成的列表。 列表中的每个元素对应一个认证因素，可以是认证插件名称、星号（`*`）、空或缺失。 （例外：元素 1 不能是空或缺失。） 整个列表用单引号括起来。 例如，以下`authentication_policy` 值包括一个星号、一个认证插件名称和一个空元素：

```sql
authentication_policy = '*,authentication_fido,'
```

星号（`*`）表示需要认证方法，但允许任何方法。 空元素表示认证方法是可选的，且允许任何方法。 缺失的元素（没有星号、空元素或认证插件名称）表示不允许认证方法。 当指定插件名称时，创建或修改账户时需要该认证方法作为相应因素的必需方法。

默认`authentication_policy` 值为`'*,,'`（一个星号和两个空元素），需要第一因素，并可选择允许第二和第三因素。 因此，默认`authentication_policy` 值与现有的 1FA 账户向后兼容，但也允许创建或修改账户以使用 2FA 或 3FA。

拥有`AUTHENTICATION_POLICY_ADMIN`权限的用户不受`authentication_policy`设置所施加的约束。 （对于否则不允许的语句会发出警告。）

`authentication_policy`值可以在选项文件中定义或使用`SET GLOBAL`语句指定：

```sql
SET GLOBAL authentication_policy='*,*,';
```

有几条规则规定了如何定义`authentication_policy`值。请参考`authentication_policy`系统变量描述，了解这些规则的完整说明。以下表格提供了几个`authentication_policy`示例值及其所建立的策略。

**表 8.11 示例 authentication_policy 值**

| authentication_policy 值 | 有效策略 |
| --- | --- |
| `'*'` | 仅允许创建或更改具有一个因素的帐户。 |
| `'*,*'` | 仅允许创建或更改具有两个因素的帐户。 |
| `'*,*,*'` | 仅允许创建或更改具有三个因素的帐户。 |
| `'*,'` | 允许创建或更改具有一或两个因素的帐户。 |
| `'*,,'` | 允许创建或更改具有一、两或三个因素的帐户。 |
| `'*,*,'` | 允许创建或更改具有两个或三个因素的帐户。 |
| `'*,*`auth_plugin`*'` | 允许创建或更改具有两个因素的帐户，其中第一个因素可以是任何认证方法，第二个因素必须是指定的插件。 |
| `'*`auth_plugin`*,*,'` | 允许创建或更改具有两个或三个因素的帐户，其中第一个因素必须是指定的插件。 |
| `'*`auth_plugin`*,'` | 允许创建或更改具有一或两个因素的帐户，其中第一个因素必须是指定的插件。 |
| `'*`auth_plugin`*,*`auth_plugin`*,*`auth_plugin`*'` | 允许创建或更改具有三个因素的帐户，其中因素必须使用指定的插件。 |
| authentication_policy 值 | 有效策略 |

#### 开始使用多因素认证

默认情况下，MySQL 使用一个多因素认证策略，允许任何认证插件作为第一个因素，并可选择允许第二和第三个认证因素。此策略是可配置的；有关详细信息，请参阅配置多因素认证策略。

注意

不允许使用任何内部凭据存储插件（`caching_sha2_password`或`mysql_native_password`）作为第 2 或第 3 因素。

假设您希望一个账户首先使用 `caching_sha2_password` 插件进行认证，然后再使用 `authentication_ldap_sasl` SASL LDAP 插件进行认证。（这假设 LDAP 认证已经按照 Section 8.4.1.7, “LDAP Pluggable Authentication” 中描述的设置完成，并且用户在 LDAP 目录中有一个与示例中显示的认证字符串对应的条目。）使用以下语句创建账户：

```sql
CREATE USER 'alice'@'localhost'
  IDENTIFIED WITH caching_sha2_password
    BY '*sha2_password*'
  AND IDENTIFIED WITH authentication_ldap_sasl
    AS 'uid=u1_ldap,ou=People,dc=example,dc=com';
```

要连接，用户必须提供两个密码。为了启用使用需要多个密码的账户连接到 MySQL 服务器的认证，客户端程序具有 `--password1`、`--password2` 和 `--password3` 选项，允许指定最多三个密码。这些选项类似于 `--password` 选项，可以在命令行上跟随选项后面输入密码值（这是不安全的），或者如果没有密码值，则会提示用户输入密码。对于刚创建的账户，因素 1 和 2 需要密码，因此使用 `--password1` 和 `--password2` 选项调用 **mysql** 客户端。**mysql** 依次提示输入每个密码：

```sql
$> mysql --user=alice --password1 --password2
Enter password: *(enter factor 1 password)* Enter password: *(enter factor 2 password)*
```

假设您想要添加第三个认证因素。可以通过删除并重新创建具有第三个因素的用户，或者使用 `ALTER USER *`user`* ADD *`factor`*` 语法来实现。以下两种方法都显示如下：

```sql
DROP USER 'alice'@'localhost';

CREATE USER 'alice'@'localhost'
  IDENTIFIED WITH caching_sha2_password
    BY '*sha2_password*'
  AND IDENTIFIED WITH authentication_ldap_sasl
    AS 'uid=u1_ldap,ou=People,dc=example,dc=com'
  AND IDENTIFIED WITH authentication_fido;
```

`ADD *`factor`*` 语法包括因素编号和 `FACTOR` 关键字：

```sql
ALTER USER 'alice'@'localhost' ADD 3 FACTOR IDENTIFIED WITH authentication_fido;
```

`ALTER USER *`user`* DROP *`factor`*` 语法允许删除一个因素。以下示例删除了前一个示例中添加的第三个因素（`authentication_fido`）：

```sql
ALTER USER 'alice'@'localhost' DROP 3 FACTOR;
```

`ALTER USER *`user`* MODIFY *`factor`*` 语法允许更改特定因素的插件或认证字符串，前提是该因素存在。以下示例修改了第二个因素，将认证方法从 `authentication_ldap_sasl` 更改为 `authetication_fido`：

```sql
ALTER USER 'alice'@'localhost' MODIFY 2 FACTOR IDENTIFIED WITH authentication_fido;
```

使用 `SHOW CREATE USER` 查看为账户定义的认证方法：

```sql
SHOW CREATE USER 'u1'@'localhost'\G
*************************** 1\. row ***************************
CREATE USER for u1@localhost: CREATE USER `u1`@`localhost` 
IDENTIFIED WITH 'caching_sha2_password' AS '*sha2_password*' 
AND IDENTIFIED WITH 'authentication_fido' REQUIRE NONE 
PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK PASSWORD HISTORY 
DEFAULT PASSWORD REUSE INTERVAL DEFAULT PASSWORD REQUIRE 
CURRENT DEFAULT
```
