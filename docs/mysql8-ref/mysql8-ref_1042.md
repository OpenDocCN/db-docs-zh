> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-user.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-user.html)

#### 15.7.1.1 ALTER USER Statement

```sql
ALTER USER [IF EXISTS]
    *user* [*auth_option*] [, *user* [*auth_option*]] ...
    [REQUIRE {NONE | *tls_option* [[AND] *tls_option*] ...}]
    [WITH *resource_option* [*resource_option*] ...]
    [*password_option* | *lock_option*] ...
    [COMMENT '*comment_string*' | ATTRIBUTE '*json_object*']

ALTER USER [IF EXISTS]
    USER() *user_func_auth_option*

ALTER USER [IF EXISTS]
    *user* [*registration_option*]

ALTER USER [IF EXISTS]
    USER() [*registration_option*]

ALTER USER [IF EXISTS]
    *user* DEFAULT ROLE
    {NONE | ALL | *role* [, *role* ] ...}

*user*:
    (see Section 8.2.4, “Specifying Account Names”)

*auth_option*: {
    IDENTIFIED BY '*auth_string*'
        [REPLACE '*current_auth_string*']
        [RETAIN CURRENT PASSWORD]
  | IDENTIFIED BY RANDOM PASSWORD
        [REPLACE '*current_auth_string*']
        [RETAIN CURRENT PASSWORD]
  | IDENTIFIED WITH *auth_plugin*
  | IDENTIFIED WITH *auth_plugin* BY '*auth_string*'
        [REPLACE '*current_auth_string*']
        [RETAIN CURRENT PASSWORD]
  | IDENTIFIED WITH *auth_plugin* BY RANDOM PASSWORD
        [REPLACE '*current_auth_string*']
        [RETAIN CURRENT PASSWORD]
  | IDENTIFIED WITH *auth_plugin* AS '*auth_string*'
  | DISCARD OLD PASSWORD
  | ADD *factor* *factor_auth_option* [ADD *factor* *factor_auth_option*]
  | MODIFY *factor* *factor_auth_option* [MODIFY *factor* *factor_auth_option*]
  | DROP *factor* [DROP *factor*]
}

*user_func_auth_option*: {
    IDENTIFIED BY '*auth_string*'
        [REPLACE '*current_auth_string*']
        [RETAIN CURRENT PASSWORD]
  | DISCARD OLD PASSWORD
}

*factor_auth_option*: {
    IDENTIFIED BY '*auth_string*'
  | IDENTIFIED BY RANDOM PASSWORD
  | IDENTIFIED WITH *auth_plugin* BY '*auth_string*'
  | IDENTIFIED WITH *auth_plugin* BY RANDOM PASSWORD
  | IDENTIFIED WITH *auth_plugin* AS '*auth_string*'
}

*registration_option*: {
    *factor* INITIATE REGISTRATION
  | *factor* FINISH REGISTRATION SET CHALLENGE_RESPONSE AS '*auth_string*'
  | *factor* UNREGISTER
}

*factor*: {2 | 3} FACTOR

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

`ALTER USER`语句修改 MySQL 账户。它允许对现有账户进行身份验证、角色、SSL/TLS、资源限制、密码管理、注释和属性属性的修改。它还可以用于锁定和解锁账户。

在大多数情况下，`ALTER USER`需要全局`CREATE USER`权限，或者`mysql`系统模式的`UPDATE`权限。例外情况包括：

+   任何使用非匿名账户连接到服务器的客户端都可以更改该账户的密码。（特别是，您可以更改自己的密码。）要查看服务器对您进行身份验证的账户，请调用`CURRENT_USER()`函数：

    ```sql
    SELECT CURRENT_USER();
    ```

+   对于`DEFAULT ROLE`语法，`ALTER USER`需要这些权限：

    +   为另一个用户设置默认角色需要全局`CREATE USER`权限，或者`mysql.default_roles`系统表的`UPDATE`权限。

    +   为自己设置默认角色不需要特殊权限，只要您想要的默认角色已经授予您。

+   修改次要密码的语句需要这些权限：

    +   使用`RETAIN CURRENT PASSWORD`或`DISCARD OLD PASSWORD`子句需要`APPLICATION_PASSWORD_ADMIN`权限，用于适用于您自己账户的`ALTER USER`语句。该权限用于操作您自己的次要密码，因为大多数用户只需要一个密码。

    +   如果要允许一个账户操作所有账户的次要密码，则需要`CREATE USER`权限，而不是`APPLICATION_PASSWORD_ADMIN`。

当启用`read_only`系统变量时，`ALTER USER`还需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。

从 MySQL 8.0.27 开始，这些额外的权限考虑因素适用：

+   `authentication_policy`系统变量对`ALTER USER`语句中与身份验证相关的子句的使用施加了一定的约束；有关详细信息，请参阅该变量的描述。如果具有`AUTHENTICATION_POLICY_ADMIN`权限，则不适用这些约束。

+   要修改使用无密码身份验证的帐户，必须具有`PASSWORDLESS_USER_ADMIN`权限。

默认情况下，如果尝试修改不存在的用户，则会发生错误。如果提供了`IF EXISTS`子句，则该语句会对每个不存在的命名用户产生警告，而不是错误。

重要提示

在某些情况下，`ALTER USER`可能会记录在服务器日志中或客户端的历史文件中，例如`~/.mysql_history`，这意味着明文密码可能会被任何具有读取权限的人读取。有关服务器日志中发生这种情况的条件以及如何控制它的信息，请参阅第 8.1.2.3 节，“密码和日志记录”。有关客户端日志记录的类似信息，请参阅第 6.5.1.3 节，“mysql 客户端日志记录”。

`ALTER USER`语句有几个方面，分别在以下主题下描述：

+   修改用户概述

+   修改用户身份验证选项

+   修改用户多因素身份验证选项

+   修改用户注册选项

+   修改用户角色选项

+   修改用户 SSL/TLS 选项

+   修改用户资源限制选项

+   修改用户密码管理选项

+   修改用户注释和属性选项

+   修改用户账户锁定选项

+   修改用户二进制日志记录

##### 修改用户概述

对于每个受影响的帐户，`ALTER USER`修改`mysql.user`系统表中对应行，以反映语句中指定的属性。未指定的属性保留其当前值。

每个帐户名称使用第 8.2.4 节“指定帐户名称”中描述的格式。如果省略帐户名称的主机名部分，默认为`'%'`。还可以指定`CURRENT_USER`或`CURRENT_USER()`来引用与当前会话关联的帐户。

仅在一个情况下，可以使用`USER()`函数指定帐户：

```sql
ALTER USER USER() IDENTIFIED BY '*auth_string*';
```

这种语法使您可以在不明确命名您的帐户的情况下更改自己的密码。（该语法还支持 ALTER USER 身份验证选项中描述的`REPLACE`、`RETAIN CURRENT PASSWORD`和`DISCARD OLD PASSWORD`子句。）

对于允许*`auth_option`*值跟随*`user`*值的`ALTER USER`语法，*`auth_option`*通过指定帐户身份验证插件、凭据（例如密码）或两者来指示帐户如何进行身份验证。每个*`auth_option`*值仅适用于紧随其前的帐户。

根据*`user`*规范，该语句可能包括 SSL/TLS、资源限制、密码管理和锁定属性的选项。所有这些选项都是语句的*全局*选项，并适用于语句中命名的*所有*帐户。

示例：更改帐户的密码并将其过期。结果是，用户必须使用指定的密码连接，并在下次连接时选择一个新密码：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED BY '*new_password*' PASSWORD EXPIRE;
```

示例：修改帐户以使用`caching_sha2_password`身份验证插件和给定密码。要求每 180 天选择一个新密码，并启用失败登录跟踪，以便连续三次输入错误密码导致临时锁定帐户两天：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED WITH caching_sha2_password BY '*new_password*'
  PASSWORD EXPIRE INTERVAL 180 DAY
  FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2;
```

示例：锁定或解锁帐户：

```sql
ALTER USER 'jeffrey'@'localhost' ACCOUNT LOCK;
ALTER USER 'jeffrey'@'localhost' ACCOUNT UNLOCK;
```

示例：要求帐户使用 SSL 连接，并在每小时建立 20 个连接的限制：

```sql
ALTER USER 'jeffrey'@'localhost'
  REQUIRE SSL WITH MAX_CONNECTIONS_PER_HOUR 20;
```

示例：更改多个帐户，指定一些每个帐户的属性和一些全局属性：

```sql
ALTER USER
  'jeffrey'@'localhost'
    IDENTIFIED BY '*jeffrey_new_password*',
  'jeanne'@'localhost',
  'josh'@'localhost'
    IDENTIFIED BY '*josh_new_password*'
    REPLACE '*josh_current_password*'
    RETAIN CURRENT PASSWORD
  REQUIRE SSL WITH MAX_USER_CONNECTIONS 2
  PASSWORD HISTORY 5;
```

`jeffrey`后面的`IDENTIFIED BY`值仅适用于其紧接的帐户，因此它仅为`jeffrey`更改密码为`'*`jeffrey_new_password`*'`。对于`jeanne`，没有每个帐户的值（因此密码保持不变）。对于`josh`，`IDENTIFIED BY`建立了一个新密码（`'*`josh_new_password`*'`），指定`REPLACE`以验证发出`ALTER USER`语句的用户知道当前密码（`'*`josh_current_password`*'`），并且当前密码也保留为帐户的次要密码。（因此，`josh`可以使用主密码或次要密码连接。）

剩余属性全局适用于语句中命名的所有帐户，因此对于两个帐户：

+   连接需要使用 SSL。

+   该帐户最多可用于两个同时连接。

+   密码更改不能重复使用最近的五个密码。

示例：丢弃`josh`的次要密码，只留下主密码：

```sql
ALTER USER 'josh'@'localhost' DISCARD OLD PASSWORD;
```

在缺少特定类型选项的情况下，帐户在这方面保持不变。例如，没有锁定选项，帐户的锁定状态不会改变。

##### ALTER USER 认证选项

一个帐户名后面可以跟着一个*`auth_option`*认证选项，指定帐户认证插件、凭据，或两者兼有。它还可以包括一个密码验证子句，指定要替换的帐户当前密码，并管理帐户是否有次要密码。

注意

仅适用于使用将凭据存储在 MySQL 内部的认证插件的帐户的随机密码生成、密码验证和次要密码子句。对于使用针对 MySQL 外部凭据系统执行身份验证的插件的帐户，密码管理也必须在该系统外部处理。有关内部凭据存储的更多信息，请参见第 8.2.15 节，“密码管理”。

+   *`auth_plugin`*指定认证插件。插件名称可以是带引号的字符串文字，也可以是未带引号的名称。插件名称存储在`mysql.user`系统表的`plugin`列中。

    对于不指定认证插件的*`auth_option`*语法，服务器分配默认插件，如默认认证插件中所述确定。有关每个插件的描述，请参见第 8.4.1 节，“认证插件”。

+   存储在内部的凭据存储在 `mysql.user` 系统表中。`'*`auth_string`*'` 值或 `RANDOM PASSWORD` 指定账户凭据，分别作为明文（未加密）字符串或以与账户关联的认证插件期望的格式进行哈希处理的形式：

    +   对于使用 `BY '*`auth_string`*'` 语法的情况，该字符串是明文的，并且传递给认证插件进行可能的哈希处理。插件返回的结果存储在 `mysql.user` 表中。插件可以按照指定的值使用该值，这种情况下不会进行哈希处理。

    +   对于使用 `BY RANDOM PASSWORD` 语法的情况，MySQL 生成一个随机密码并作为明文传递给认证插件进行可能的哈希处理。插件返回的结果存储在 `mysql.user` 表中。插件可以按照指定的值使用该值，这种情况下不会进行哈希处理。

        随机生成的密码从 MySQL 8.0.18 开始可用，并具有 随机密码生成 中描述的特性。

    +   对于使用 `AS '*`auth_string`*'` 语法的情况，该字符串被假定已经是认证插件所需的格式，并且原样存储在 `mysql.user` 表中。如果插件需要哈希值，则该值必须已经以适合插件的格式进行哈希处理；否则，插件无法使用该值，也无法正确验证客户端连接。

        从 MySQL 8.0.17 开始，哈希字符串可以是字符串文字或十六进制值。后者对应于当启用 `print_identified_with_as_hex` 系统变量时，包含不可打印字符的密码哈希的 `SHOW CREATE USER` 显示的值类型。

    +   如果认证插件不对认证字符串进行哈希处理，则 `BY '*`auth_string`*'` 和 `AS '*`auth_string`*'` 子句具有相同的效果：认证字符串原样存储在 `mysql.user` 系统表中。

+   `REPLACE '*`current_auth_string`*'` 子句执行密码验证，并从 MySQL 8.0.13 开始可用。如果给出：

    +   `REPLACE` 指定要替换的账户当前密码，作为明文（未加密）字符串。

    +   如果账户的密码更改需要指定当前密码以验证尝试进行更改的用户实际上知道当前密码，则必须给出该子句。

    +   如果账户的密码更改可能但不一定需要指定当前密码，则该子句是可选的。

    +   如果给出了该子句但与当前密码不匹配，则语句将失败，即使该子句是可选的。

    +   只有在更改当前用户的账户密码时才能指定 `REPLACE`。

    有关通过指定当前密码进行密码验证的更多信息，请参见第 8.2.15 节，“密码管理”。

+   `RETAIN CURRENT PASSWORD`和`DISCARD OLD PASSWORD`子句实现了双密码功能，并自 MySQL 8.0.14 起可用。两者都是可选的，但如果给出，则具有以下效果：

    +   `RETAIN CURRENT PASSWORD`保留账户当前密码作为其次要密码，替换任何现有的次要密码。新密码成为主密码，但客户端可以使用账户使用主密码或次要密码连接到服务器。（例外情况：如果`ALTER USER`语句指定的新密码为空，则次要密码也变为空，即使给出了`RETAIN CURRENT PASSWORD`。）

    +   如果为具有空主密码的账户指定了`RETAIN CURRENT PASSWORD`，则该语句将失败。

    +   如果一个账户有次要密码，并且您更改其主密码而没有指定`RETAIN CURRENT PASSWORD`，则次要密码保持不变。

    +   如果更改分配给账户的身份验证插件，则次要密码将被丢弃。如果更改身份验证插件并且还指定了`RETAIN CURRENT PASSWORD`，则该语句将失败。

    +   `DISCARD OLD PASSWORD`丢弃次要密码（如果存在）。账户仅保留其主密码，客户端只能使用主密码连接到服务器。

    有关双密码使用的更多信息，请参见第 8.2.15 节，“密码管理”。

`ALTER USER`允许这些*`auth_option`*语法：

+   `IDENTIFIED BY '*`auth_string`*' [REPLACE '*`current_auth_string`*'] [RETAIN CURRENT PASSWORD]`

    将账户身份验证插件设置为默认插件，将明文`'*`auth_string`*'`值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的账户行中。

    如果指定了`REPLACE`子句，则指定了当前账户密码，如本节前文所述。

    如果给出了`RETAIN CURRENT PASSWORD`子句，则导致保留账户当前密码作为其次要密码，如本节前文所述。

+   `IDENTIFIED BY RANDOM PASSWORD [REPLACE '*`current_auth_string`*'] [RETAIN CURRENT PASSWORD]`

    将账户认证插件设置为默认插件，生成一个随机密码，将明文密码值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的账户行中。该语句还会将明文密码作为结果集返回，以便用户或应用程序执行该语句时使用。有关结果集和随机生成密码特性的详细信息，请参见随机密码生成。

    `REPLACE`子句，如果给定，指定账户当前密码，如本节前文所述。

    `RETAIN CURRENT PASSWORD`子句，如果给定，会导致账户当前密码保留为其次要密码，如本节前文所述。

+   `IDENTIFIED WITH *`auth_plugin`*`

    将账户认证插件设置为*`auth_plugin`*，将凭据清空为空字符串（凭据与旧认证插件相关联，而不是新的插件），并将结果存储在`mysql.user`系统表中的账户行中。

    此外，密码被标记为过期。用户在下次连接时必须选择新密码。

+   `IDENTIFIED WITH *`auth_plugin`* BY '*`auth_string`*' [REPLACE '*`current_auth_string`*'] [RETAIN CURRENT PASSWORD]`

    将账户认证插件设置为*`auth_plugin`*，将明文`'*`auth_string`*'`值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的账户行中。

    `REPLACE`子句，如果给定，指定账户当前密码，如本节前文所述。

    `RETAIN CURRENT PASSWORD`子句，如果给定，会导致账户当前密码保留为其次要密码，如本节前文所述。

+   `IDENTIFIED WITH *`auth_plugin`* BY RANDOM PASSWORD [REPLACE '*`current_auth_string`*'] [RETAIN CURRENT PASSWORD]`

    将账户认证插件设置为*`auth_plugin`*，生成一个随机密码，将明文密码值传递给插件进行可能的哈希处理，并将结果存储在`mysql.user`系统表中的账户行中。该语句还会将明文密码作为结果集返回，以便用户或应用程序执行该语句时使用。有关结果集和随机生成密码特性的详细信息，请参见随机密码生成。

    `REPLACE`子句，如果给定，指定账户当前密码，如本节前文所述。

    `RETAIN CURRENT PASSWORD`子句，如果给定，会导致账户当前密码保留为其次要密码，如本节前文所述。

+   `IDENTIFIED WITH *`auth_plugin`* AS '*`auth_string`*'`

    将账户认证插件设置为*`auth_plugin`*，并将`'*`auth_string`*'`值原样存储在`mysql.user`账户行中。如果插件需要哈希字符串，则假定字符串已经以插件所需的格式进行了哈希处理。

+   `DISCARD OLD PASSWORD`

    丢弃账户的次要密码，如果存在的话，如本节前面描述的。

示例：将密码指定为明文；使用默认插件：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED BY '*password*';
```

示例：指定认证插件，以及一个明文密码值：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED WITH mysql_native_password
             BY '*password*';
```

示例：与前面的示例类似，但另外指定当前密码作为明文值，以满足用户更改时需要知道该密码的任何账户要求：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED WITH mysql_native_password
             BY '*password*'
             REPLACE '*current_password*';
```

除非当前用户是`jeffrey`，否则上述语句将失败，因为`REPLACE`仅允许更改当前用户的密码。

示例：建立一个新的主密码，并保留现有密码作为次要密码：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED BY '*new_password*'
  RETAIN CURRENT PASSWORD;
```

示例：丢弃次要密码，仅保留账户的主密码：

```sql
ALTER USER 'jeffery'@'localhost' DISCARD OLD PASSWORD;
```

示例：指定认证插件，以及一个哈希密码值：

```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED WITH mysql_native_password
             AS '*6C8989366EAF75BB670AD8EA7A7FC1176A95CEF4';
```

有关设置密码和认证插件的更多信息，请参见第 8.2.14 节，“分配账户密码”和第 8.2.17 节，“可插拔认证”。

##### 更改用户多因素认证选项

截至 MySQL 8.0.27，`ALTER USER`具有`ADD`、`MODIFY`和`DROP`子句，允许添加、修改或删除认证因素。在每种情况下，子句指定要对一个认证因素执行的操作，以及可选地对另一个认证因素执行的操作。对于每个操作，*`factor`*项指定了`FACTOR`关键字，前面跟着数字 2 或 3，以指示操作是应用于第二个还是第三个认证因素。（在此上下文中不允许使用 1。要对第一个认证因素执行操作，请使用 ALTER USER Authentication Options 中描述的语法。）

`ALTER USER`多因素认证子句的约束由`authentication_policy`系统变量定义。例如，`authentication_policy`设置控制账户可以拥有的认证因素数量，以及对于每个因素，允许使用哪些认证方法。请参阅配置多因素认证策略。

当`ALTER USER`在单个语句中添加、修改或删除第二个和第三个因素时，操作是按顺序执行的，但如果序列中的任何操作失败，则整个`ALTER USER`语句将失败。

对于`ADD`，每个命名因素都不得已存在，否则无法添加。对于`MODIFY`和`DROP`，每个命名因素必须存在才能被修改或删除。如果定义了第二个和第三个因素，删除第二个因素会导致第三个因素取代它成为第二个因素。

此语句删除认证因素 2 和 3，从而将帐户从 3FA 转换为 1FA：

```sql
ALTER USER '*user*' DROP 2 FACTOR 3 FACTOR;
```

有关其他`ADD`，`MODIFY`和`DROP`示例，请参阅开始使用多因素认证。

有关确定未命名插件的认证子句的默认认证插件的特定规则的信息，请参阅默认认证插件。

##### ALTER USER 注册选项

截至 MySQL 8.0.27，`ALTER USER`具有允许注册和注销 FIDO 设备的子句。有关更多信息，请参阅使用 FIDO 认证，FIDO 设备注销以及**mysql**客户端`--fido-register-factor`选项描述。

**mysql**客户端`--fido-register-factor`选项，用于 FIDO 设备注册，会导致**mysql**客户端生成并执行`INITIATE REGISTRATION`和`FINISH REGISTRATION`语句。这些语句不适用于手动执行。

##### ALTER USER 角色选项

`ALTER USER ... DEFAULT ROLE`定义了用户连接到服务器并进行身份验证时激活的角色，或者用户在会话期间执行`SET ROLE DEFAULT`语句时激活的角色。

`ALTER USER ... DEFAULT ROLE` 是 `SET DEFAULT ROLE` 的替代语法（参见 第 15.7.1.9 节，“SET DEFAULT ROLE Statement”）。然而，`ALTER USER` 只能为单个用户设置默认值，而 `SET DEFAULT ROLE` 可以为多个用户设置默认值。另一方面，您可以将 `CURRENT_USER` 指定为 `ALTER USER` 语句的用户名，而对于 `SET DEFAULT ROLE` 则不行。

每个用户账户名称使用先前描述的格式。

每个角色名称使用 第 8.2.5 节，“指定角色名称” 中描述的格式。例如：

```sql
ALTER USER 'joe'@'10.0.0.1' DEFAULT ROLE administrator, developer;
```

如果省略角色���称的主机名部分，则默认为 `'%'`。

在 `DEFAULT ROLE` 关键字后的子句允许这些值：

+   `NONE`：将默认设置为 `NONE`（无角色）。

+   `ALL`：将默认设置为授予该账户的所有角色。

+   `*`role`* [, *`role`* ] ...`：将默认设置为指定的角色，这些角色必须在执行 `ALTER USER ... DEFAULT ROLE` 时存在并授予该账户。

##### 修改用户 SSL/TLS 选项

MySQL 可以检查 X.509 证书属性，除了基于用户名和凭据的常规身份验证外。有关在 MySQL 中使用 SSL/TLS 的背景信息，请参见 第 8.3 节，“使用加密连接”。

要为 MySQL 账户指定 SSL/TLS 相关选项，请使用包含一个或多个 *`tls_option`* 值的 `REQUIRE` 子句。

`REQUIRE` 选项的顺序无关紧要，但不能指定两次选项。在 `REQUIRE` 选项之间的 `AND` 关键字是可选的。

`ALTER USER` 允许这些 *`tls_option`* 值：

+   `NONE`

    表明由该语句指定的所有账户没有 SSL 或 X.509 要求。如果用户名和密码有效，则允许非加密连接。如果客户端具有正确的证书和密钥文件，则客户端可以选择使用加密连接。

    ```sql
    ALTER USER 'jeffrey'@'localhost' REQUIRE NONE;
    ```

    客户端默认尝试建立安全连接。对于具有 `REQUIRE NONE` 的客户端，如果无法建立安全连接，则连接尝试会回退到非加密连接。要求加密连接，客户端只需指定 `--ssl-mode=REQUIRED` 选项；如果无法建立安全连接，则连接尝试失败。

+   `SSL`

    告诉服务器只允许通过加密连接访问由该语句指定的所有账户。

    ```sql
    ALTER USER 'jeffrey'@'localhost' REQUIRE SSL;
    ```

    客户端默认尝试建立安全连接。对于具有`REQUIRE SSL`的帐户，如果无法建立安全连接，则连接尝试失败。

+   `X509`

    对于所有由该语句命名的帐户，要求客户端提供有效证书，但确切的证书、颁发者和主题并不重要。唯一的要求是应该能够使用其中一个 CA 证书验证其签名。使用 X.509 证书始终意味着加密，因此在这种情况下`SSL`选项是不必要的。

    ```sql
    ALTER USER 'jeffrey'@'localhost' REQUIRE X509;
    ```

    对于具有`REQUIRE X509`的帐户，客户端必须指定`--ssl-key`和`--ssl-cert`选项进行连接。（建议但不是必须还指定`--ssl-ca`，以便验证服务器提供的公共证书。）对于`ISSUER`和`SUBJECT`也是如此，因为这些`REQUIRE`选项暗示了`X509`的要求。

+   `颁发者 '*`issuer`*'`

    对于所有由该语句命名的帐户，要求客户端提供由 CA`'*`issuer`*'`颁发的有效 X.509 证书。如果客户端提供的证书有效但颁发者不同，服务器将拒绝连接。使用 X.509 证书始终意味着加密，因此在这种情况下`SSL`选项是不必要的。

    ```sql
    ALTER USER 'jeffrey'@'localhost'
      REQUIRE ISSUER '/C=SE/ST=Stockholm/L=Stockholm/
        O=MySQL/CN=CA/emailAddress=ca@example.com';
    ```

    因为`ISSUER`暗示了`X509`的要求，客户端必须指定`--ssl-key`和`--ssl-cert`选项进行连接。（建议但不是必须还指定`--ssl-ca`，以便验证服务器提供的公共证书。）

+   `主题 '*`subject`*'`

    对于所有由该语句命名的帐户，要求客户端提供包含主题*`subject`*的有效 X.509 证书。如果客户端提供的证书有效但主题不同，服务器将拒绝连接。使用 X.509 证书始终意味着加密，因此在这种情况下`SSL`选项是不必要的。

    ```sql
    ALTER USER 'jeffrey'@'localhost'
      REQUIRE SUBJECT '/C=SE/ST=Stockholm/L=Stockholm/
        O=MySQL demo client certificate/
        CN=client/emailAddress=client@example.com';
    ```

    MySQL 对`'*`subject`*'`值与证书中的值进行简单的字符串比较，因此大小写和组件顺序必须与证书中的完全相同。

    因为`SUBJECT`暗示了`X509`的要求，客户端必须指定`--ssl-key`和`--ssl-cert`选项进行连接。（建议但不是必须还指定`--ssl-ca`，以便验证服务器提供的公共证书。）

+   `密码 '*`cipher`*'`

    对于语句命名的所有帐户，需要特定的密码方法来加密连接。需要此选项以确保使用足够强度的密码和密钥长度的密码和密钥。如果使用旧算法和短加密密钥的算法，则加密可能较弱。

    ```sql
    ALTER USER 'jeffrey'@'localhost'
      REQUIRE CIPHER 'EDH-RSA-DES-CBC3-SHA';
    ```

`SUBJECT`、`ISSUER`和`CIPHER`选项可以在`REQUIRE`子句中组合使用：

```sql
ALTER USER 'jeffrey'@'localhost'
  REQUIRE SUBJECT '/C=SE/ST=Stockholm/L=Stockholm/
    O=MySQL demo client certificate/
    CN=client/emailAddress=client@example.com'
  AND ISSUER '/C=SE/ST=Stockholm/L=Stockholm/
    O=MySQL/CN=CA/emailAddress=ca@example.com'
  AND CIPHER 'EDH-RSA-DES-CBC3-SHA';
```

##### ALTER USER 资源限制选项

可以对帐户使用服务器资源的限制进行限制，如第 8.2.21 节“设置帐户资源限制”中所讨论的。为此，请使用指定一个或多个*`resource_option`*值的`WITH`子句。

`WITH`选项的顺序无关紧要，除非给定资源限制多次指定，否则最后一次实例优先。

`ALTER USER`允许使用以下*`resource_option`*值：

+   `MAX_QUERIES_PER_HOUR *`count`*`、`MAX_UPDATES_PER_HOUR *`count`*`、`MAX_CONNECTIONS_PER_HOUR *`count`*`

    对于语句命名的所有帐户，这些选项限制了在任何给定的一小时内每个帐户对服务器执行多少查询、更新和连接。如果*`count`*为`0`（默认值），这意味着该帐户没有限制。

+   `MAX_USER_CONNECTIONS *`count`*`

    对于语句命名的所有帐户，限制每个帐户对服务器的最大同时连接数。非零*`count`*明确指定了帐户的限制。如果*`count`*为`0`（默认值），服务器将根据`max_user_connections`系统变量的全局值确定帐户的同时连接数。如果`max_user_connections`也为零，则该帐户没有限制。

示例：

```sql
ALTER USER 'jeffrey'@'localhost'
  WITH MAX_QUERIES_PER_HOUR 500 MAX_UPDATES_PER_HOUR 100;
```

##### ALTER USER 密码管理选项

`ALTER USER`支持几个*`password_option`*值用于密码管理：

+   密码过期选项：您可以手动使帐户密码过期并建立其密码过期策略。策略选项不会使密码过期。相反，它们确定服务器如何根据密码年龄自动使帐户密码过期，该年龄是从最近更改帐户密码的日期和时间评估的。

+   密码重用选项：您可以基于密码更改次数、经过的时间或两者限制密码重用。

+   密码验证必需选项：您可以指示更改帐户密码的尝试是否必须指定当前密码，以验证试图进行更改的用户实际上知道当前密码。

+   不正确密码的失败登录跟踪选项：您可以导致服务器跟踪失败的登录尝试，并临时锁定给出太多连续不正确密码的帐户。可配置失败次数和锁定时间。

本节描述了密码管理选项的语法。有关建立密码管理策略的信息，请参见第 8.2.15 节“密码管理”。

如果指定了给定类型的多个密码管理选项，则最后一个优先。例如，`PASSWORD EXPIRE DEFAULT PASSWORD EXPIRE NEVER`与`PASSWORD EXPIRE NEVER`相同。

注意

除了与失败登录跟踪相关的选项外，密码管理选项仅适用于使用将凭据存储在 MySQL 内部的身份验证插件的帐户。对于使用针对 MySQL 外部凭据系统执行身份验证的插件的帐户，密码管理也必须在该系统外部处理。有关内部凭据存储的更多信息，请参见第 8.2.15 节“密码管理”。

如果客户端的帐户密码已手动过期或密码年龄被认为大于其允许的生命周期，根据自动过期策略，客户端的密码已过期。在这种情况下，服务器要么断开客户端的连接，要么限制其允许的操作（请参见第 8.2.16 节“服务器处理过期密码”）。受限客户端执行的操作会导致错误，直到用户建立新的帐户密码。

注意

尽管可以通过将过期密码重置为当前值来“重置”过期密码，但作为良好政策的一部分，最好选择不同的密码。 DBA 可以通过建立适当的密码重用策略来强制执行不重用。请参见密码重用策略。

`ALTER USER`允许这些*`password_option`*值来控制密码过期：

+   `PASSWORD EXPIRE`

    立即标记由语句命名的所有帐户的密码过期。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE;
    ```

+   `PASSWORD EXPIRE DEFAULT`

    将由语句命名的所有帐户设置为适用全局过期策略，由`default_password_lifetime`系统变量指定。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE DEFAULT;
    ```

+   `PASSWORD EXPIRE NEVER`

    此过期选项覆盖了语句命名的所有帐户的全局策略。对于每个帐户，它禁用密码过期，使密码永不过期。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;
    ```

+   `PASSWORD EXPIRE INTERVAL *`N`* DAY`

    此过期选项覆盖了声明中所有命名的账户的全局策略。对于每个账户，它将密码寿命设置为*`N`*天。以下声明要求每 180 天更改一次密码：

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE INTERVAL 180 DAY;
    ```

`ALTER USER` 允许这些*`password_option`*值来控制基于所需最小密码更改次数的先前密码重用：

+   `PASSWORD HISTORY DEFAULT`

    设置所有由声明命名的账户，以便全局关于密码历史长度的策略适用，以禁止在由`password_history`系统变量指定的更改次数之前重用密码。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD HISTORY DEFAULT;
    ```

+   `PASSWORD HISTORY *`N`*`

    此历史长度选项覆盖了声明中所有命名的账户的全局策略。对于每个账户，它将密码历史长度设置为*`N`*个密码，以禁止重新使用最近选择的*`N`*个密码中的任何一个。以下声明禁止重新使用之前的 6 个密码：

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD HISTORY 6;
    ```

`ALTER USER` 允许这些*`password_option`*值来控制基于经过的时间重新使用先前密码的情况：

+   `PASSWORD REUSE INTERVAL DEFAULT`

    设置所有由账户命名的声明，以便全局关于经过时间的策略适用，以禁止重用比由`password_reuse_interval`系统变量指定的天数新的密码。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD REUSE INTERVAL DEFAULT;
    ```

+   `PASSWORD REUSE INTERVAL *`N`* DAY`

    此经过时间选项覆盖了声明中所有命名的账户的全局策略。对于每个账户，它将密码重用间隔设置为*`N`*天，以禁止重用比该天数新的密码。以下声明禁止密码在 360 天内重用：

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD REUSE INTERVAL 360 DAY;
    ```

`ALTER USER` 允许这些*`password_option`*值来控制是否尝试更改账户密码必须指定当前密码，以验证尝试进行更改的用户实际上知道当前密码：

+   `PASSWORD REQUIRE CURRENT`

    此验证选项覆盖了声明中所有命名的账户的全局策略。对于每个账户，它要求密码更改必须指定当前密码。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT;
    ```

+   `PASSWORD REQUIRE CURRENT OPTIONAL`

    此验证选项覆盖了声明中所有命名的账户的全局策略。对于每个账户，它不要求密码更改必须指定当前密码。（当前密码可以给出，但不是必须的。）

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT OPTIONAL;
    ```

+   `PASSWORD REQUIRE CURRENT DEFAULT`

    设置所有由账户命名的声明，以便全局关于密码验证的策略适用，如`password_require_current`系统变量所指定的那样。

    ```sql
    ALTER USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT DEFAULT;
    ```

从 MySQL 8.0.19 开始，`ALTER USER` 允许使用以下 *`password_option`* 值来控制失败登录跟踪：

+   `FAILED_LOGIN_ATTEMPTS *`N`*`

    是否跟踪指定错误密码的账户登录尝试。*`N`* 必须是从 0 到 32767 的数字。值为 0 禁用失败登录跟踪。大于 0 的值表示多少连续密码失败会导致临时账户锁定（如果 `PASSWORD_LOCK_TIME` 也非零）。

+   `PASSWORD_LOCK_TIME {*`N`* | UNBOUNDED}`

    在太多连续登录尝试提供错误密码后锁定账户多长时间。*`N`* 必须是从 0 到 32767 的数字，或 `UNBOUNDED`。值为 0 禁用临时账户锁定。大于 0 的值表示锁定账户的天数。值为 `UNBOUNDED` 导致账户锁定持续时间无限；一旦锁定，账户将保持锁定状态直到解锁。有关解锁发生的条件，请参阅 Failed-Login Tracking and Temporary Account Locking。

要实现失败登录跟踪和临时锁定，一个账户的 `FAILED_LOGIN_ATTEMPTS` 和 `PASSWORD_LOCK_TIME` 选项都必须非零。以下语句修改一个账户，使其在连续四次密码失败后保持锁定两天：

```sql
ALTER USER 'jeffrey'@'localhost'
  FAILED_LOGIN_ATTEMPTS 4 PASSWORD_LOCK_TIME 2;
```

##### 修改用户评论和属性选项

MySQL 8.0.21 及更高版本支持用户评论和用户属性，如 Section 15.7.1.3, “CREATE USER Statement” 中所述。这些可以通过 `ALTER USER` 使用 `COMMENT` 和 `ATTRIBUTE` 选项进行修改。不能在同一 `ALTER USER` 语句中同时指定这两个选项；尝试这样做会导致语法错误。

用户评论和用户属性存储在信息模式 `USER_ATTRIBUTES` 表中作为 JSON 对象；用户评论存储为此表的 ATTRIBUTE 列中 `comment` 键的值，如后面的讨论所示。`COMMENT` 文本可以是任意带引号的文本，并替换任何现有用户评论。`ATTRIBUTE` 值必须是 JSON 对象的有效字符串表示。这与任何现有用户属性合并，就好像在现有用户属性和新用户属性上使用了 `JSON_MERGE_PATCH()` 函数；对于重新使用的任何键，新值会覆盖旧值，如下所示：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->     WHERE USER='bill' AND HOST='localhost';
+------+-----------+----------------+
| USER | HOST      | ATTRIBUTE      |
+------+-----------+----------------+
| bill | localhost | {"foo": "bar"} |
+------+-----------+----------------+
1 row in set (0.11 sec)

mysql> ALTER USER 'bill'@'localhost' ATTRIBUTE '{"baz": "faz", "foo": "moo"}';
Query OK, 0 rows affected (0.22 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->     WHERE USER='bill' AND HOST='localhost';
+------+-----------+------------------------------+
| USER | HOST      | ATTRIBUTE                    |
+------+-----------+------------------------------+
| bill | localhost | {"baz": "faz", "foo": "moo"} |
+------+-----------+------------------------------+
1 row in set (0.00 sec)
```

要从用户属性中删除键及其值，将键设置为 JSON `null`（必须小写且不带引号），如下所示：

```sql
mysql> ALTER USER 'bill'@'localhost' ATTRIBUTE '{"foo": null}';
Query OK, 0 rows affected (0.08 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->     WHERE USER='bill' AND HOST='localhost';
+------+-----------+----------------+
| USER | HOST      | ATTRIBUTE      |
+------+-----------+----------------+
| bill | localhost | {"baz": "faz"} |
+------+-----------+----------------+
1 row in set (0.00 sec)
```

要将现有用户的注释设置为空字符串，请使用`ALTER USER ... COMMENT ''`。这将在`USER_ATTRIBUTES`表中留下一个空的`comment`值；要完全删除用户注释，请使用`ALTER USER ... ATTRIBUTE ...`，将列键的值设置为 JSON `null`（小写，不带引号）。下面是一系列 SQL 语句的示例：

```sql
mysql> ALTER USER 'bill'@'localhost' COMMENT 'Something about Bill';
Query OK, 0 rows affected (0.06 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->     WHERE USER='bill' AND HOST='localhost';
+------+-----------+---------------------------------------------------+
| USER | HOST      | ATTRIBUTE                                         |
+------+-----------+---------------------------------------------------+
| bill | localhost | {"baz": "faz", "comment": "Something about Bill"} |
+------+-----------+---------------------------------------------------+
1 row in set (0.00 sec)

mysql> ALTER USER 'bill'@'localhost' COMMENT '';
Query OK, 0 rows affected (0.09 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->     WHERE USER='bill' AND HOST='localhost';
+------+-----------+-------------------------------+
| USER | HOST      | ATTRIBUTE                     |
+------+-----------+-------------------------------+
| bill | localhost | {"baz": "faz", "comment": ""} |
+------+-----------+-------------------------------+
1 row in set (0.00 sec)

mysql> ALTER USER 'bill'@'localhost' ATTRIBUTE '{"comment": null}';
Query OK, 0 rows affected (0.07 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
 ->     WHERE USER='bill' AND HOST='localhost';
+------+-----------+----------------+
| USER | HOST      | ATTRIBUTE      |
+------+-----------+----------------+
| bill | localhost | {"baz": "faz"} |
+------+-----------+----------------+
1 row in set (0.00 sec)
```

##### ALTER USER 帐户锁定选项

MySQL 支持使用`ACCOUNT LOCK`和`ACCOUNT UNLOCK`选项进行帐户锁定和解锁，这些选项指定帐户的锁定状态。有关更多讨论，请参阅第 8.2.20 节，“帐户锁定”。

如果指定了多个帐户锁定选项，则最后一个优先。

`ALTER USER ... ACCOUNT UNLOCK` 解锁由语句指定的任何因登录失败次数过多而暂时锁定的帐户。请参阅第 8.2.15 节，“密码管理”。

##### ALTER USER 二进制日志记录

`ALTER USER` 如果成功，则写入二进制日志，但如果失败则不会；在这种情况下，将发生回滚，不会进行任何更改。写入二进制日志的语句包括所有命名用户。如果给出了`IF EXISTS`子句，则甚至包括那些不存在且未被更改的用户。

如果原始语句更改了用户的凭据，则写入二进制日志的语句指定了该用户的适用身份验证插件，��定如下：

+   如果原始语句中指定了插件的名称，则插件的名称。

+   否则，与用户帐户关联的插件（如果用户存在），或者默认身份验证插件（如果用户不存在）。（如果写入二进制日志的语句必须为用户指定特定的身份验证插件，请在原始语句中包含它。）

如果服务器为写入二进制日志的语句中的任何用户添加默认身份验证插件，则会向错误日志中写入警告，列出这些用户。

如果原始语句指定了`FAILED_LOGIN_ATTEMPTS`或`PASSWORD_LOCK_TIME`选项，则写入二进制日志的语句包括该选项。

具有支持多因素身份验证（MFA）的子句的`ALTER USER`语句被写入二进制日志，但不包括`ALTER USER *`user factor`* INITIATE REGISTRATION`语句。

+   `ALTER USER *`user factor`* FINISH REGISTRATION SET CHALLENGE_RESPONSE AS '*`auth_string`*'` 语句被写入二进制日志为`ALTER USER *`user`* MODIFY *`factor`* IDENTIFIED WITH authentication_fido AS *`fido_hash_string`*`;

+   在复制环境中，复制用户需要`PASSWORDLESS_USER_ADMIN`权限来执行对使用`authentication_fido`插件配置为无密码身份验证的帐户进行`ALTER USER ... MODIFY`操作。
