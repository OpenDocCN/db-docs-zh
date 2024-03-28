> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-password.html`](https://dev.mysql.com/doc/refman/8.0/en/set-password.html)

#### 15.7.1.10 `SET PASSWORD`语句

```sql
SET PASSWORD [FOR *user*] *auth_option*
    [REPLACE '*current_auth_string*']
    [RETAIN CURRENT PASSWORD]

*auth_option*: {
    = '*auth_string*'
  | TO RANDOM
}
```

`SET PASSWORD`语句为 MySQL 用户帐户分配密码。密码可以在语句中明确指定，也可以由 MySQL 随机生成。该语句还可以包括一个密码验证条款，该条款指定要替换的帐户当前密码，以及一个管理帐户是否具有次要密码的条款。`'*`auth_string`*'`和`'*`current_auth_string`*'`分别表示明文（未加密）密码。

注意

与使用`SET PASSWORD`分配密码不同，`ALTER USER`是首选语句，用于帐户更改，包括分配密码。例如：

```sql
ALTER USER *user* IDENTIFIED BY '*auth_string*';
```

注意

仅适用于使用将凭据存储在 MySQL 内部的身份验证插件的帐户的随机密码生成、密码验证和次要密码的条款。对于使用针对 MySQL 外部凭据系统执行身份验证的插件的帐户，密码管理也必须在该系统外部处理。有关内部凭据存储的更多信息，请参见第 8.2.15 节，“密码管理”。

`REPLACE '*`current_auth_string`*'`条款执行密码验证，并自 MySQL 8.0.13 起可用。如果给出：

+   `REPLACE`指定要替换的帐户当前密码，作为明文（未加密）字符串。

+   如果需要更改帐户密码，则必须提供该条款，以指定当前密码，以验证试图进行更改的用户实际知道当前密码。

+   如果需要更改帐户密码，但不需要指定当前密码，则该条款是可选的。

+   如果给出该条款但与当前密码不匹配，则该语句将失败，即使该条款是可选的。

+   只有在更改当前用户的帐户密码时才可以指定`REPLACE`。

有关通过指定当前密码进行密码验证的更多信息，请参见第 8.2.15 节，“密码管理”。

`RETAIN CURRENT PASSWORD`条款实现双密码功能，并自 MySQL 8.0.14 起可用。如果给出：

+   `RETAIN CURRENT PASSWORD` 保留账户当前密码作为其次要密码，替换任何现有的次要密码。新密码成为主密码，但客户端可以使用该账户使用主密码或次要密码连接到服务器。 （例外情况：如果`SET PASSWORD`语句指定的新密码为空，则次要密码也变为空，即使给出了 `RETAIN CURRENT PASSWORD`。）

+   如果为一个主密码为空的账户指定 `RETAIN CURRENT PASSWORD`，该语句将失败。

+   如果一个账户有一个次要密码，并且您更改其主密码而不指定 `RETAIN CURRENT PASSWORD`，则次要密码保持不变。

有关双重密码使用的更多信息，请参阅第 8.2.15 节，“密码管理”。

`SET PASSWORD` 允许使用���些 *`auth_option`* 语法：

+   `= '*`auth_string`*'`

    为账户分配指定的明文密码。

+   `TO RANDOM`

    为账户分配由 MySQL 随机生成的密码。该语句还会在结果集中返回明文密码，以便用户或执行该语句的应用程序使用。

    有关结果集和随机生成密码的特性的详细信息，请参阅随机密码生成。

    随机密码生成功能自 MySQL 8.0.18 版本开始提供。

重要提示

在某些情况下，`SET PASSWORD` 可能会记录在服务器日志中或客户端的历史文件中，例如 `~/.mysql_history`，这意味着明文密码可能被任何具有读取权限的人读取。有关在服务器日志中发生这种情况的条件以及如何控制它的信息，请参阅第 8.1.2.3 节，“密码和日志记录”。有关客户端日志记录的类似信息，请参阅第 6.5.1.3 节，“mysql 客户端日志记录”。

`SET PASSWORD` 可以使用或不使用显式命名用户账户的 `FOR` 子句：

+   使用 `FOR *`user`*` 子句，该语句为指定的账户设置密码，该账户必须存在：

    ```sql
    SET PASSWORD FOR 'jeffrey'@'localhost' = '*auth_string*';
    ```

+   没有 `FOR *`user`*` 子句，该语句为当前用户设置密码：

    ```sql
    SET PASSWORD = '*auth_string*';
    ```

    任何使用非匿名账户连接到服务器的客户端都可以更改该账户的密码（特别是可以更改自己的密码）。要查看服务器对您进行身份验证的账户，请调用`CURRENT_USER()`函数：

    ```sql
    SELECT CURRENT_USER();
    ```

如果给出了`FOR *`user`*`子句，则账户名使用第 8.2.4 节“指定账户名”中描述的格式。 例如：

```sql
SET PASSWORD FOR 'bob'@'%.example.org' = '*auth_string*';
```

如果省略了账户名的主机名部分，则默认为`'%'`。

`SET PASSWORD`将字符串解释为明文字符串，将其传递给与账户关联的认证插件，并将插件返回的结果存储在`mysql.user`系统表中的账户行中。（插件有机会将值哈希为其期望的加密格式。插件可以按照指定的值使用该值，这种情况下不会发生哈希。）

为具名账户（使用`FOR`子句）设置密码需要对`mysql`系统模式具有`UPDATE`权限。 为自己设置密码（对于没有`FOR`子句的非匿名账户）不需要特殊权限。

修改次要密码的语句需要以下权限：

+   需要`APPLICATION_PASSWORD_ADMIN`权限才能使用`RETAIN CURRENT PASSWORD`子句来对自己的账户执行`SET PASSWORD`语句。 大多数用户只需要一个密码，因此需要该权限来操作自己的次要密码。

+   如果要允许一个账户操作所有账户的次要密码，则应授予`CREATE USER`权限，而不是`APPLICATION_PASSWORD_ADMIN`。

当启用`read_only`系统变量时，`SET PASSWORD`需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限），以及任何其他所需权限。

有关设置密码和认证插件的更多信息，请参见第 8.2.14 节“分配账户密码”和第 8.2.17 节“可插拔认证”。
