> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-user.html`](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)

#### 15.7.1.3 CREATE USER Statement

```sql
CREATE USER [IF NOT EXISTS]
    *user* [*auth_option*] [, *user* [*auth_option*]] ...
    DEFAULT ROLE *role* [, *role* ] ...
    [REQUIRE {NONE | *tls_option* [[AND] *tls_option*] ...}]
    [WITH *resource_option* [*resource_option*] ...]
    [*password_option* | *lock_option*] ...
    [COMMENT '*comment_string*' | ATTRIBUTE '*json_object*']

*user*:
    (see Section 8.2.4, “Specifying Account Names”)

*auth_option*: {
    IDENTIFIED BY '*auth_string*' [AND *2fa_auth_option*]
  | IDENTIFIED BY RANDOM PASSWORD [AND *2fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* [AND *2fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* BY '*auth_string*' [AND *2fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* BY RANDOM PASSWORD [AND *2fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* AS '*auth_string*' [AND *2fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* [*initial_auth_option*]
}

*2fa_auth_option*: {
    IDENTIFIED BY '*auth_string*' [AND *3fa_auth_option*]
  | IDENTIFIED BY RANDOM PASSWORD [AND *3fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* [AND *3fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* BY '*auth_string*' [AND *3fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* BY RANDOM PASSWORD [AND *3fa_auth_option*]
  | IDENTIFIED WITH *auth_plugin* AS '*auth_string*' [AND *3fa_auth_option*]
}

*3fa_auth_option*: {
    IDENTIFIED BY '*auth_string*'
  | IDENTIFIED BY RANDOM PASSWORD
  | IDENTIFIED WITH *auth_plugin*
  | IDENTIFIED WITH *auth_plugin* BY '*auth_string*'
  | IDENTIFIED WITH *auth_plugin* BY RANDOM PASSWORD
  | IDENTIFIED WITH *auth_plugin* AS '*auth_string*'
}

*initial_auth_option*: {
    INITIAL AUTHENTICATION IDENTIFIED BY {RANDOM PASSWORD | '*auth_string*'}
  | INITIAL AUTHENTICATION IDENTIFIED WITH *auth_plugin* AS '*auth_string*'
}

*tls_option*: {
   SSL
 | X509
 | CIPHER '*cipher*'
 | ISSUER '*issuer*'
 | SUBJECT '*subject*'
}

*resource_option*: {
    MAX_QUERIES_PER_HOUR *count*
  | MAX_UPDATES_PER_HOUR *count*
  | MAX_CONNECTIONS_PER_HOUR *count*
  | MAX_USER_CONNECTIONS *count*
}

*password_option*: {
    PASSWORD EXPIRE [DEFAULT | NEVER | INTERVAL *N* DAY]
  | PASSWORD HISTORY {DEFAULT | *N*}
  | PASSWORD REUSE INTERVAL {DEFAULT | *N* DAY}
  | PASSWORD REQUIRE CURRENT [DEFAULT | OPTIONAL]
  | FAILED_LOGIN_ATTEMPTS *N*
  | PASSWORD_LOCK_TIME {*N* | UNBOUNDED}
}

*lock_option*: {
    ACCOUNT LOCK
  | ACCOUNT UNLOCK
}
```

`CREATE USER`语句创建新的 MySQL 账户。它使得可以为新账户建立认证、角色、SSL/TLS、资源限制、密码管理、注释和属性属性。它还控制了账户最初是锁定还是解锁的状态。

要使用`CREATE USER`，您必须具有全局`CREATE USER`权限，或者对`mysql`系统模式具有`INSERT`权限。当启用`read_only`系统变量时，`CREATE USER`还需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。

截至 MySQL 8.0.27，以下附加特权考虑因素适用：

+   `authentication_policy` 系统变量对`CREATE USER`语句中与认证相关的子句的使用施加了一定的约束；详情请参阅该变量的描述。如果具有`AUTHENTICATION_POLICY_ADMIN`权限，则这些约束不适用。

+   要创建一个使用无密码认证的账户，您必须具有`PASSWORDLESS_USER_ADMIN`权限。

截至 MySQL 8.0.22，如果要创建的任何账户被命名为任何存储对象的`DEFINER`属性，则`CREATE USER`将失败并显示错误。（也就是说，如果创建账户会导致该账户接管当前孤立的存储对象，则该语句将失败。）要执行该操作，您必须具有`SET_USER_ID`权限；在这种情况下，该语句将以警告成功而不是错误失败。如果没有`SET_USER_ID`，要执行用户创建操作，请删除孤立对象，创建账户并授予其权限，然后重新创建已删除的对象。有关更多信息，包括如何识别哪些对象将给定账户命名为`DEFINER`属性，请参阅孤立存储对象。

`CREATE USER` 对所有命名用户都成功，或者如果发生任何错误则回滚并不起作用。默认情况下，如果尝试创建已存在的用户，则会发生错误。如果给出了 `IF NOT EXISTS` 子句，则该语句会对每个已存在的命名用户产生警告，而不是错误。

重要提示

在某些情况下，`CREATE USER` 可能会记录在服务器日志中或客户端的历史文件中，例如 `~/.mysql_history`，这意味着明文密码可能被任何具有读取权限的人读取。有关在服务器日志中发生这种情况的条件以及如何控制它的信息，请参见 第 8.1.2.3 节，“密码和日志记录”。有关客户端日志记录的类似信息，请参见 第 6.5.1.3 节，“mysql 客户端日志记录”。

`CREATE USER` 语句有几个方面，描述如下主题：

+   创建用户概述

+   创建用户认证选项

+   创建用户多因素认证选项

+   创建用户角色选项

+   创建用户 SSL/TLS 选项

+   创建用户资源限制选项

+   创建用户密码管理选项

+   创建用户注释和属性选项

+   创建用户账户锁定选项

+   创建用户二进制日志记录

##### 创建用户概述

对于每个账户，`CREATE USER` 在 `mysql.user` 系统表中创建一行新记录。账户行反映了语句中指定的属性。未指定的属性将设置为它们的默认值：

+   认证：默认认证插件（如 默认认证插件 中所述确定的），以及空凭据

+   默认角色：`NONE`

+   SSL/TLS：`NONE`

+   资源限制：无限制

+   密码管理：`PASSWORD EXPIRE DEFAULT PASSWORD HISTORY DEFAULT PASSWORD REUSE INTERVAL DEFAULT PASSWORD REQUIRE CURRENT DEFAULT`；禁用了失败登录跟踪和临时帐户锁定

+   帐户锁定：`ACCOUNT UNLOCK`

创建时，帐户没有任何权限和默认角色`NONE`。要为此帐户分配权限或角色，请使用一个或多个`GRANT`语句。

每个帐户名称都采用第 8.2.4 节，“指定帐户名称”中描述的格式。例如：

```sql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY '*password*';
```

如果帐户名称的主机名部分被省略，则默认为`'%'`。您应该知道，虽然 MySQL 8.0 将授予此类用户的授权视为已授予`'*`user`*'@'localhost'`，但此行为已在 MySQL 8.0.35 中被弃用，并因此可能在将来的 MySQL 版本中被移除。

每个命名帐户的*`user`*值后面可以跟一个可选的*`auth_option`*值，表示帐户的认证方式。这些值可以指定帐户认证插件和凭据（例如密码）。每个*`auth_option`*值仅适用于紧随其后的帐户。

根据*`user`*规范，语句可以包括 SSL/TLS、资源限制、密码管理和锁定属性的选项。所有这些选项都是语句的*全局*属性，并适用于语句中命名的*所有*帐户。

示例：创建一个使用默认认证插件和给定密码的帐户。标记密码已过期，以便用户必须在首次连接到服务器时选择新密码：

```sql
CREATE USER 'jeffrey'@'localhost'
  IDENTIFIED BY '*new_password*' PASSWORD EXPIRE;
```

示例：创建一个使用`caching_sha2_password`认证插件和给定密码的帐户。要求每 180 天选择一个新密码，并启用失败登录跟踪，以便三次连续输入错误密码导致帐户被临时锁定两天：

```sql
CREATE USER 'jeffrey'@'localhost'
  IDENTIFIED WITH caching_sha2_password BY '*new_password*'
  PASSWORD EXPIRE INTERVAL 180 DAY
  FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2;
```

示例：创建多个帐户，指定一些每个帐户的属性和一些全局属性：

```sql
CREATE USER
  'jeffrey'@'localhost' IDENTIFIED WITH mysql_native_password
                                   BY '*new_password1*',
  'jeanne'@'localhost' IDENTIFIED WITH caching_sha2_password
                                  BY '*new_password2*'
  REQUIRE X509 WITH MAX_QUERIES_PER_HOUR 60
  PASSWORD HISTORY 5
  ACCOUNT LOCK;
```

每个*`auth_option`*值（在本例中为`IDENTIFIED WITH ... BY`）仅适用于紧随其后的帐户，因此每个帐户使用紧随其后的认证插件和密码。

剩余的属性全局适用于语句中命名的所有帐户，因此对于两个帐户：

+   必须使用有效的 X.509 证书进行连接。

+   每小时最多允许 60 个查询。

+   密码更改不能重复使用最近的五个密码。

+   帐户最初被锁定，因此实际上它是一个占位符，直到管理员解锁它才能使用。

##### 创建用户认证选项

帐户名称后面可以跟一个*`auth_option`*认证选项，指定帐户认证插件、凭据或两者。

注意

在 MySQL 8.0.27 之前，*`auth_option`* 定义了账户进行身份验证的唯一方法。也就是说，所有账户都使用单因素/单因素身份验证（1FA/SFA）。MySQL 8.0.27 及更高版本支持多因素身份验证（MFA），因此账户可以使用最多三种身份验证方法。也就是说，账户可以使用双因素身份验证（2FA）或三因素身份验证（3FA）。*`auth_option`* 的语法和语义保持不变，但*`auth_option`* 可以后跟额外身份验证方法的规范。本节描述了*`auth_option`*。有关可选 MFA 相关后续子句的详细信息，请参阅 CREATE USER 多因素身份验证选项。

注意

仅适用于使用将凭据存储在 MySQL 内部的身份验证插件的账户的随机密码生成子句。对于使用针对 MySQL 外部凭据系统执行身份验证的插件的账户，密码管理也必须在该系统外部处理。有关内部凭据存储的更多信息，请参阅 第 8.2.15 节，“密码管理”。

+   *`auth_plugin`* 指定了一个身份验证插件。插件名称可以是带引号的字符串文字或未带引号的名称。插件名称存储在 `mysql.user` 系统表的 `plugin` 列中。

    对于未指定身份验证插件的*`auth_option`* 语法，服务器会分配默认插件，具体确定方法请参见 默认身份验证插件。有关每个插件的描述，请参见 第 8.4.1 节，“身份验证插件”。

+   存储在内部的凭据存储在 `mysql.user` 系统表中。`'*`auth_string`*'` 值或 `RANDOM PASSWORD` 指定账户凭据，可以是明文（未加密）字符串或以与账户关联的身份验证插件期望的格式进行哈希处理：

    +   对于使用 `BY '*`auth_string`*'` 语法的情况，字符串是明文的，并传递给身份验证插件进行可能的哈希处理。插件返回的结果存储在 `mysql.user` 表中。插件可以按照指定的值使用该值，这种情况下不会发生哈希处理。

    +   对于使用 `BY RANDOM PASSWORD` 语法的情况，MySQL 生成一个随机密码并以明文形式传递给身份验证插件进行可能的哈希处理。插件返回的结果存储在 `mysql.user` 表中。插件可以按照指定的值使用该值，这种情况下不会发生哈希处理。

        随机生成的密码可在 MySQL 8.0.18 中使用，并具有随机密码生成中描述的特性。

    +   对于使用`AS '*`auth_string`*'`语法的语法，假定字符串已经是认证插件所需的格式，并按原样存储在`mysql.user`表中。如果插件需要哈希值，则该值必须已经以适合插件的格式进行哈希处理；否则，插件无法使用该值，客户端连接的正确认证也不会发生。

        截至 MySQL 8.0.17，哈希字符串可以是字符串文字或十六进制值。当启用`print_identified_with_as_hex`系统变量时，后者对应于包含不可打印字符的密码哈希的值类型，该变量由`SHOW CREATE USER`显示。

    +   如果认证插件不对认证字符串进行哈希处理，则`BY '*`auth_string`*'`和`AS '*`auth_string`*'`子句具有相同的效果：认证字符串按原样存储在`mysql.user`系统表中。

`CREATE USER`允许这些*`auth_option`*语法：

+   `IDENTIFIED BY '*`auth_string`*'`

    将帐户认证插件设置为默认插件，将明文`'*`auth_string`*'`值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的帐户行中。

+   `IDENTIFIED BY RANDOM PASSWORD`

    将帐户认证插件设置为默认插件，生成随机密码，将明文密码值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的帐户行中。该语句还将明文密码作为结果集返回，以便用户或执行该语句的应用程序可以使用。有关结果集和随机生成密码的特性的详细信息，请参见随机密码生成。

+   `IDENTIFIED WITH *`auth_plugin`*`

    将帐户认证插件设置为*`auth_plugin`*，将凭据清除为空字符串，并将结果存储在`mysql.user`系统表中的帐户行中。

+   `IDENTIFIED WITH *`auth_plugin`* BY '*`auth_string`*'`

    将帐户认证插件设置为*`auth_plugin`*，将明文`'*`auth_string`*'`值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的帐户行中。

+   `IDENTIFIED WITH *`auth_plugin`* BY RANDOM PASSWORD`

    将账户认证插件设置为 *`auth_plugin`*，生成一个随机密码，将明文密码值传递给插件进行可能的哈希处理，并将结果存储在 `mysql.user` 系统表中的账户行中。该语句还将明文密码作为结果集返回，以便用户或执行该语句的应用程序可以访问。有关结果集和随机生成密码的特性的详细信息，请参阅 随机密码生成。

+   `IDENTIFIED WITH *`auth_plugin`* AS '*`auth_string`*'`

    将账户认证插件设置为 *`auth_plugin`*，并将 `'*`auth_string`*'` 值原样存储在 `mysql.user` 账户行中。如果插件需要哈希字符串，则假定字符串已经以插件所需的格式进行了哈希处理。

示例：将密码指定为明文；使用默认插件：

```sql
CREATE USER 'jeffrey'@'localhost'
  IDENTIFIED BY '*password*';
```

示例：指定认证插件，以及明文密码值：

```sql
CREATE USER 'jeffrey'@'localhost'
  IDENTIFIED WITH mysql_native_password BY '*password*';
```

在每种情况下，存储在账户行中的密码值是与账户关联的认证插件对其进行哈希后的明文值 `'*`password`*'`。

有关设置密码和认证插件的其他信息，请参阅 第 8.2.14 节，“分配账户密码” 和 第 8.2.17 节，“可插拔认证”。

##### 创建用户多因素认证选项

`CREATE USER` 中的 *`auth_option`* 部分定义了一种用于单因素认证（1FA/SFA）的认证方法。从 MySQL 8.0.27 开始，`CREATE USER` 具有支持多因素认证（MFA）的子句，因此账户可以拥有最多三种认证方法。也就是说，账户可以使用双因素认证（2FA）或三因素认证（3FA）。

`authentication_policy` 系统变量定义了带有多因素认证（MFA）子句的 `CREATE USER` 语句的约束条件。例如，`authentication_policy` 设置控制了账户可以拥有的认证因素数量，以及对于每个因素，允许使用的认证方法。请参阅 配置多因素认证策略。

关于确定未指定插件名称的认证子句的默认认证插件的特定规则的信息，请参见默认认证插件。

在*`auth_option`*之后，可能会出现不同的可选 MFA 子句：

+   *`2fa_auth_option`*：指定二要素认证方法。以下示例将`caching_sha2_password`定义为第一要素认证方法，将`authentication_ldap_sasl`定义为第二要素认证方法。

    ```sql
    CREATE USER 'u1'@'localhost'
      IDENTIFIED WITH caching_sha2_password
        BY '*sha2_password*'
      AND IDENTIFIED WITH authentication_ldap_sasl
        AS 'uid=u1_ldap,ou=People,dc=example,dc=com';
    ```

+   *`3fa_auth_option`*：在*`2fa_auth_option`*之后，可能会出现一个*`3fa_auth_option`*子句，用于指定第三要素认证方法。以下示例将`caching_sha2_password`定义为第一要素认证方法，将`authentication_ldap_sasl`定义为第二要素认证方法，将`authentication_fido`定义为第三要素认证方法。

    ```sql
    CREATE USER 'u1'@'localhost'
      IDENTIFIED WITH caching_sha2_password
        BY '*sha2_password*'
      AND IDENTIFIED WITH authentication_ldap_sasl
        AS 'uid=u1_ldap,ou=People,dc=example,dc=com'
      AND IDENTIFIED WITH authentication_fido;
    ```

+   *`initial_auth_option`*：指定用于配置 FIDO 无密码认证的初始认证方法。如下所示，需要使用生成的随机密码或用户指定的*`auth-string`*进行临时认证，以启用 FIDO 无密码认证。

    ```sql
    CREATE USER *user*
      IDENTIFIED WITH authentication_fido
      INITIAL AUTHENTICATION IDENTIFIED BY {RANDOM PASSWORD | '*auth_string*'};
    ```

    有关使用 FIDO 可插拔认证配置无密码认证的信息，请参见 FIDO 无密码认证。

##### 创建用户角色选项

`DEFAULT ROLE`子句定义了用户连接到服务器并进行身份验证时或用户在会话期间执行`SET ROLE DEFAULT`语句时激活的角色。

每个角色名称使用第 8.2.5 节，“指定角色名称”中描述的格式。例如：

```sql
CREATE USER 'joe'@'10.0.0.1' DEFAULT ROLE administrator, developer;
```

如果省略角色名称的主机名部分，默认为`'%'`。

`DEFAULT ROLE`子句允许一个或多个逗号分隔的角色名称列表。这些角色必须在执行`CREATE USER`时存在；否则语句会引发错误(`ER_USER_DOES_NOT_EXIST`)，并且用户不会被创建。

##### 创建用户 SSL/TLS 选项

MySQL 可以检查 X.509 证书属性，除了基于用户名和凭据的通常认证外。有关在 MySQL 中使用 SSL/TLS 的背景信息，请参见第 8.3 节，“使用加密连接”。

要为 MySQL 账户指定与 SSL/TLS 相关的选项，请使用指定一个或多个*`tls_option`*值的`REQUIRE`子句。

`REQUIRE`选项的顺序不重要，但不能指定两次选项。`REQUIRE`选项之间的`AND`关键字是可选的。

`CREATE USER` 允许这些 *`tls_option`* 值：

+   `NONE`

    表示语句指定的所有帐户没有 SSL 或 X.509 要求。如果用户名和密码有效，则允许未加密连接。如果客户端具有正确的证书和密钥文件，则客户端可以选择使用加密连接。

    ```sql
    CREATE USER 'jeffrey'@'localhost' REQUIRE NONE;
    ```

    客户端默认尝试建立安全连接。对于具有 `REQUIRE NONE` 的客户端，如果无法建立安全连接，则连接尝试回退到未加密连接。要求加密连接，客户端只需指定 `--ssl-mode=REQUIRED` 选项；如果无法建立安全连接，则连接尝试失败。

    如果未指定任何与 SSL 相关的 `REQUIRE` 选项，则 `NONE` 是默认值。

+   `SSL`

    通知服务器只允许通过语句指定的所有帐户进行加密连接。

    ```sql
    CREATE USER 'jeffrey'@'localhost' REQUIRE SSL;
    ```

    客户端默认尝试建立安全连接。对于具有 `REQUIRE SSL` 的帐户，如果无法建立安全连接，则连接尝试失败。

+   `X509`

    对于语句指定的所有帐户，要求客户端提供有效证书，但确切的证书、颁发者和主题并不重要。唯一的要求是应该能够使用其中一个 CA 证书验证其签名。使用 X.509 证书始终意味着加密，因此在这种情况下 `SSL` 选项是不必要的。

    ```sql
    CREATE USER 'jeffrey'@'localhost' REQUIRE X509;
    ```

    对于具有 `REQUIRE X509` 的帐户，客户端必须指定 `--ssl-key` 和 `--ssl-cert` 选项进行连接。（建议但不是必须还指定 `--ssl-ca`，以便验证服务器提供的公共证书。）对于 `ISSUER` 和 `SUBJECT` 也是如此，因为这些 `REQUIRE` 选项暗示了 `X509` 的要求。

+   `ISSUER '*`issuer`*'`

    对于语句指定的所有帐户，要求客户端提供由 CA `'*`issuer`*'` 颁发的有效 X.509 证书。如果客户端提供的证书有效但颁发者不同，则服务器拒绝连接。使用 X.509 证书始终意味着加密，因此在这种情况下 `SSL` 选项是不必要的。

    ```sql
    CREATE USER 'jeffrey'@'localhost'
      REQUIRE ISSUER '/C=SE/ST=Stockholm/L=Stockholm/
        O=MySQL/CN=CA/emailAddress=ca@example.com';
    ```

    因为 `ISSUER` 暗示了 `X509` 的要求，客户端必须指定 `--ssl-key` 和 `--ssl-cert` 选项进行连接。（建议但不是必须还指定 `--ssl-ca`，以便验证服务器提供的公共证书。）

+   `SUBJECT '*`subject`*'`

    对于陈述中命名的所有帐户，要求客户端提供包含主题*`subject`*的有效 X.509 证书。如果客户端提供了一个有效但主题不同的证书，服务器将拒绝连接。使用 X.509 证书总是意味着加密，因此在这种情况下`SSL`选项是不必要的。

    ```sql
    CREATE USER 'jeffrey'@'localhost'
      REQUIRE SUBJECT '/C=SE/ST=Stockholm/L=Stockholm/
        O=MySQL demo client certificate/
        CN=client/emailAddress=client@example.com';
    ```

    MySQL 对`'*`subject`*'`值与证书中的值进行简单的字符串比较，因此字母大小写和组件顺序必须与证书中的完全一致。

    因为`SUBJECT`暗示了`X509`的要求，客户端必须指定`--ssl-key`和`--ssl-cert`选项进行连接。（建议但不是必须还指定`--ssl-ca`，以便验证服务器提供的公共证书。）

+   `CIPHER '*`cipher`*'`

    对于陈述中命名的所有帐户，需要为加密连接指定特定的密码方法。这个选项是必需的，以确保使用足够强度的密码和密钥长度。如果使用旧算法和短加密密钥，加密可能会很弱。

    ```sql
    CREATE USER 'jeffrey'@'localhost'
      REQUIRE CIPHER 'EDH-RSA-DES-CBC3-SHA';
    ```

`SUBJECT`、`ISSUER`和`CIPHER`选项可以在`REQUIRE`子句中组合使用：

```sql
CREATE USER 'jeffrey'@'localhost'
  REQUIRE SUBJECT '/C=SE/ST=Stockholm/L=Stockholm/
    O=MySQL demo client certificate/
    CN=client/emailAddress=client@example.com'
  AND ISSUER '/C=SE/ST=Stockholm/L=Stockholm/
    O=MySQL/CN=CA/emailAddress=ca@example.com'
  AND CIPHER 'EDH-RSA-DES-CBC3-SHA';
```

##### 创建用户资源限制选项

可以通过讨论中提到的第 8.2.21 节，“设置帐户资源限制”来限制帐户对服务器资源的使用。为此，请使用指定一个或多个*`resource_option`*值的`WITH`子句。

`WITH`选项的顺序无关紧要，除非给定资源限制被多次指定，最后一次实例优先。

`CREATE USER`允许这些*`resource_option`*值：

+   `MAX_QUERIES_PER_HOUR *`count`*`，`MAX_UPDATES_PER_HOUR *`count`*`，`MAX_CONNECTIONS_PER_HOUR *`count`*`

    对于陈述中命名的所有帐户，这些选项限制了每个帐户在任何给定的一个小时内对服务器的查询、更新和连接的次数。如果*`count`*是`0`（默认值），这意味着该帐户没有限制。

+   `MAX_USER_CONNECTIONS *`count`*`

    对于陈述中命名的所有帐户，限制了每个帐户对服务器的最大同时连接数。非零的*`count`*明确指定了该帐户的限制。如果*`count`*是`0`（默认值），服务器从`max_user_connections`系统变量的全局值确定该帐户的同时连接数。如果`max_user_connections`也是零，则该帐户没有限制。

示例：

```sql
CREATE USER 'jeffrey'@'localhost'
  WITH MAX_QUERIES_PER_HOUR 500 MAX_UPDATES_PER_HOUR 100;
```

##### 创建用户密码管理选项

`创建用户` 支持几个*`password_option`*值用于密码管理：

+   密码过期选项：您可以手动使账户密码过期并建立其密码过期策略。策略选项不会使密码过期。相反，它们确定服务器如何根据密码年龄自动使账户过期，密码年龄是从最近一次账户密码更改的日期和时间开始评估的。

+   密码重用选项：您可以基于密码更改次数、经过的时间或两者限制密码重用。

+   需要密码验证的选项：您可以指示尝试更改账户密码是否必须指定当前密码，以验证试图进行更改的用户实际上知道当前密码。

+   不正确密码失败登录跟踪选项：您可以导致服务器跟踪失败的登录尝试，并临时锁定给出太多连续不正确密码的账户。失败次数和锁定时间可配置。

本节描述了密码管理选项的语法。有关建立密码管理策略的信息，请参阅第 8.2.15 节，“密码管理”。

如果指定了给定类型的多个密码管理选项，则最后一个优先。例如，`密码过期默认 密码过期从不`等同于`密码过期从不`。

注意

除了与失败登录跟踪相关的选项外，密码管理选项仅适用于使用将凭据存储在 MySQL 内部的身份验证插件的账户。对于使用针对 MySQL 外部凭据系统执行身份验证的插件的账户，密码管理也必须在该系统外部处理。有关内部凭据存储的更多信息，请参阅第 8.2.15 节，“密码管理”。

如果账户密码已被手动过期或密码年龄被认为大于其允许的生命周期根据自动过期策略。在这种情况下，服务器要么断开客户端连接，要么限制其允许的操作（请参阅第 8.2.16 节，“过期密码的服务器处理”）。受限客户端执行的操作会导致错误，直到用户建立新的账户密码。

`创建用户` 允许这些*`password_option`*值来控制密码过期：

+   `密码过期`

    立即标记语句命名的所有账户的密码过期。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE;
    ```

+   `密码过期默认`

    设置所有语句中命名的帐户，使全局过期策略适用，如`default_password_lifetime`系统变量指定的那样。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE DEFAULT;
    ```

+   `PASSWORD EXPIRE NEVER`

    此过期选项覆盖了语句中命名的所有帐户的全局策略。对于每个帐户，它禁用密码过期，使密码永不过期。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;
    ```

+   `PASSWORD EXPIRE INTERVAL *`N`* DAY`

    此过期选项覆盖了语句中命名的所有帐户的全局策略。对于每个帐户，它将密码寿命设置为*`N`*天。以下语句要求每 180 天更改一次密码：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE INTERVAL 180 DAY;
    ```

`CREATE USER`允许这些*`password_option`*值来控制基于所需最小密码更改次数的先前密码重用：

+   `PASSWORD HISTORY DEFAULT`

    设置所有语句中命名的帐户，使全局关于密码历史长度的策略适用，以禁止在`password_history`系统变量指定的更改次数之前重用密码。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD HISTORY DEFAULT;
    ```

+   `PASSWORD HISTORY *`N`*`

    此历史长度选项覆盖了语句中命名的所有帐户的全局策略。对于每个帐户，它将密码历史长度设置为*`N`*个密码，以禁止重用最近选择的*`N`*个密码中的任何一个。以下语句禁止重用之前的 6 个密码中的任何一个：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD HISTORY 6;
    ```

`CREATE USER`允许这些*`password_option`*值来控制基于经过时间的先前密码重用：

+   `PASSWORD REUSE INTERVAL DEFAULT`

    设置所有帐户中命名的语句，使全局关于经过时间的策略适用，以禁止重用新于`password_reuse_interval`系统变量指定的天数的密码。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REUSE INTERVAL DEFAULT;
    ```

+   `PASSWORD REUSE INTERVAL *`N`* DAY`

    此经过时间选项覆盖了语句中命名的所有帐户的全局策略。对于每个帐户，它将密码重用间隔设置为*`N`*天，以禁止重用新于该天数的密码。以下语句禁止密码在 360 天内重用：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REUSE INTERVAL 360 DAY;
    ```

`CREATE USER`允许这些*`password_option`*值来控制是否尝试更改帐户密码必须指定当前密码，以验证尝试进行更改的用户实际上知道当前密码：

+   `PASSWORD REQUIRE CURRENT`

    此验证选项覆盖了语句中命名的所有帐户的全局策略。对于每个帐户，它要求密码更改时指定当前密码。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT;
    ```

+   `PASSWORD REQUIRE CURRENT OPTIONAL`

    此验证选项会覆盖语句指定的所有帐户的全局策略。对于每个帐户，不需要密码更改指定当前密码。（当前密码可以给出，但不是必需的。）

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT OPTIONAL;
    ```

+   `PASSWORD REQUIRE CURRENT DEFAULT`

    设置所有由帐户命名的语句，以便全局关于密码验证的策略适用，如`password_require_current` 系统变量所指定的那样。

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT DEFAULT;
    ```

从 MySQL 8.0.19 开始，`CREATE USER` 允许这些 *`password_option`* 值来控制登录失败跟踪：

+   `FAILED_LOGIN_ATTEMPTS *`N`*`

    是否跟踪指定错误密码的帐户登录尝试。*`N`* 必须是从 0 到 32767 的数字。值为 0 会禁用登录失败跟踪。大于 0 的值表示多少连续密码失败会导致临时帐户锁定（如果 `PASSWORD_LOCK_TIME` 也是非零）。

+   `PASSWORD_LOCK_TIME {*`N`* | UNBOUNDED}`

    连续登录尝试提供错误密码后锁定帐户的时间。*`N`* 必须是从 0 到 32767 的数字，或者 `UNBOUNDED`。值为 0 会禁用临时帐户锁定。大于 0 的值表示锁定帐户的天数。值为 `UNBOUNDED` 会导致帐户锁定持续时间无限制；一旦锁定，帐户将保持锁定状态直到解锁。有关解锁发生的条件的信息，请参阅登录失败跟踪和临时帐户锁定。

要进行登录失败跟踪和临时锁定，帐户的 `FAILED_LOGIN_ATTEMPTS` 和 `PASSWORD_LOCK_TIME` 选项都必须非零。以下语句创建一个帐户，在连续四次密码失败后保持锁定两天：

```sql
CREATE USER 'jeffrey'@'localhost'
  FAILED_LOGIN_ATTEMPTS 4 PASSWORD_LOCK_TIME 2;
```

##### 创建用户注释和属性选项

从 MySQL 8.0.21 开始，您可以创建一个带有可选注释或属性的帐户，如下所述：

+   **用户注释**

    要设置用户注释，请在 `CREATE USER` 语句中添加 `COMMENT '*`user_comment`*'`，其中 *`user_comment`* 是用户注释的文本。

    示例（省略其他选项）：

    ```sql
    CREATE USER 'jon'@'localhost' COMMENT 'Some information about Jon';
    ```

+   **用户属性**

    用户属性是由一个或多个键值对组成的 JSON 对象，并通过在 `CREATE USER` 中包含 `ATTRIBUTE '*`json_object`*'` 来设置。*`json_object`* 必须是一个有效的 JSON 对象。

    示例（省略其他选项）：

    ```sql
    CREATE USER 'jim'@'localhost'
        ATTRIBUTE '{"fname": "James", "lname": "Scott", "phone": "123-456-7890"}';
    ```

用户注释和用户属性一起存储在信息模式 `USER_ATTRIBUTES` 表的 `ATTRIBUTE` 列中。此查询显示了刚刚用于创建用户 `jim@localhost` 的语句插入的此表中的行：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->    WHERE USER = 'jim' AND HOST = 'localhost'\G
*************************** 1\. row ***************************
     USER: jim
     HOST: localhost
ATTRIBUTE: {"fname": "James", "lname": "Scott", "phone": "123-456-7890"} 1 row in set (0.00 sec)
```

实际上，`COMMENT`选项提供了一个快捷方式，用于设置一个只有`comment`作为其键且其值为选项提供的参数的用户属性。通过执行语句`CREATE USER 'jon'@'localhost' COMMENT 'Some information about Jon'`，并观察它插入到`USER_ATTRIBUTES`表中的行，您可以看到这一点：

```sql
mysql> CREATE USER 'jon'@'localhost' COMMENT 'Some information about Jon';
Query OK, 0 rows affected (0.06 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->    WHERE USER = 'jon' AND HOST = 'localhost';
+------+-----------+-------------------------------------------+
| USER | HOST      | ATTRIBUTE                                 |
+------+-----------+-------------------------------------------+
| jon  | localhost | {"comment": "Some information about Jon"} |
+------+-----------+-------------------------------------------+
1 row in set (0.00 sec)
```

不能在同一`CREATE USER`语句中同时使用`COMMENT`和`ATTRIBUTE`；尝试这样做会导致语法错误。要同时设置用户评论和用户属性，使用`ATTRIBUTE`并在其参数中包含具有`comment`键的值，如下所示：

```sql
mysql> CREATE USER 'bill'@'localhost'
 ->        ATTRIBUTE '{"fname":"William", "lname":"Schmidt",
    ->        "comment":"Website developer"}';
Query OK, 0 rows affected (0.16 sec)
```

由于`ATTRIBUTE`行的内容是一个 JSON 对象，您可以使用任何适当的 MySQL JSON 函数或运算符来操作它，如下所示：

```sql
mysql> SELECT
 ->   USER AS User,
 ->   HOST AS Host,
 ->   CONCAT(ATTRIBUTE->>"$.fname"," ",ATTRIBUTE->>"$.lname") AS 'Full Name',
 ->   ATTRIBUTE->>"$.comment" AS Comment
 -> FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 -> WHERE USER='bill' AND HOST='localhost';
+------+-----------+-----------------+-------------------+
| User | Host      | Full Name       | Comment           |
+------+-----------+-----------------+-------------------+
| bill | localhost | William Schmidt | Website developer |
+------+-----------+-----------------+-------------------+
1 row in set (0.00 sec)
```

要为现有用户设置或更改用户评论或用户属性，可以使用带有`ALTER USER`语句的`COMMENT`或`ATTRIBUTE`选项。

因为用户评论和用户属性在单个`JSON`列中内部存储在一起，这为它们的最大组合大小设置了一个上限；有关更多信息，请参阅 JSON 存储要求。

有关更多信息和示例，请参阅信息模式`USER_ATTRIBUTES`表的描述。

##### 创建用户帐户锁定选项

MySQL 支持使用`ACCOUNT LOCK`和`ACCOUNT UNLOCK`选项对帐户进行锁定和解锁，这些选项指定帐户的锁定状态。有关更多讨论，请参阅 Section 8.2.20，“帐户锁定”。

如果指定了多个帐户锁定选项，则最后一个优先。

##### 创建用户二进制日志

如果`CREATE USER`成功，则写入二进制日志，但如果失败则不写入；在这种情况下，将发生回滚并且不会进行任何更改。写入二进制日志的语句包括所有命名用户。如果给出了`IF NOT EXISTS`子句，这甚至包括已经存在且未创建的用户。

写入二进制日志的语句为每个用户指定一个身份验证插件，确定如下：

+   原始语句中指定的插件。

+   否则，默认的身份验证插件。特别是，如果用户`u1`已经存在并使用非默认身份验证插件，则为`CREATE USER IF NOT EXISTS u1`写入二进制日志的语句将列出默认身份验证插件。（如果必须为用户指定非默认身份验证插件，请在原始语句中包含它。）

如果服务器为二进制日志中写入的任何不存在的用户添加默认身份验证插件，则会在错误日志中写入警告，列出这些用户的名称。

如果原始语句指定了`FAILED_LOGIN_ATTEMPTS`或`PASSWORD_LOCK_TIME`选项，则写入二进制日志的语句将包括该选项。

支持多因素认证（MFA）的子句的`CREATE USER`语句将写入二进制日志。

+   `CREATE USER ... IDENTIFIED WITH .. INITIAL AUTHENTICATION IDENTIFIED WITH ...`语句将作为`CREATE USER .. IDENTIFIED WITH .. INITIAL AUTHENTICATION IDENTIFIED WITH .. AS '*`password-hash`*'`写入二进制日志，其中*`password-hash`*是用户指定的*`auth-string`*或服务器在指定`RANDOM PASSWORD`子句时生成的随机密码。
