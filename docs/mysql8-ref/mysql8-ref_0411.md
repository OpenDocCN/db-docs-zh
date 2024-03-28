# 8.4.3 密码验证组件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/validate-password.html`](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html)

8.4.3.1 密码验证组件的安装和卸载

8.4.3.2 密码验证选项和变量

8.4.3.3 过渡到密码验证组件

`validate_password` 组件通过要求帐户密码并启用潜在密码的强度测试来提高安全性。该组件公开了系统变量，使您能够配置密码策略，并公开了用于组件监视的状态变量。

注意

在 MySQL 8.0 中，`validate_password` 插件被重新实现为 `validate_password` 组件。（有关组件的一般信息，请参见 第 7.5 节，“MySQL 组件”。）以下说明描述了如何使用组件，而不是插件。有关使用 `validate_password` 插件形式的说明，请参见 密码验证插件，在 MySQL 5.7 参考手册 中。

`validate_password` 插件形式仍然可用，但已被弃用；预计将在将来的 MySQL 版本中删除。使用插件的 MySQL 安装应该过渡到使用组件。请参见 第 8.4.3.3 节，“过渡到密码验证组件”。

`validate_password` 组件实现了以下功能：

+   对于将明文值作为密码分配的 SQL 语句，`validate_password` 会根据当前密码策略检查密码，并在密码弱时拒绝密码（该语句返回一个 `ER_NOT_VALID_PASSWORD` 错误）。这适用于 `ALTER USER`、`CREATE USER` 和 `SET PASSWORD` 语句。

+   对于 `CREATE USER` 语句，`validate_password` 要求提供密码，并且该密码必须符合密码策略。即使帐户最初被锁定，解锁帐户后也需要满足策略要求的密码，否则稍后访问该帐户将不需要密码。

+   `validate_password` 实现了一个 `VALIDATE_PASSWORD_STRENGTH()` SQL 函数，用于评估潜在密码的强度。该函数接受一个密码参数，并返回一个从 0（弱）到 100（强）的整数。

注意

对于分配或修改帐户密码的语句（`ALTER USER`，`CREATE USER`和`SET PASSWORD`），这里描述的`validate_password`功能仅适用于使用将凭据内部存储到 MySQL 的身份验证插件的帐户。对于使用针对 MySQL 外部凭据系统执行身份验证的插件的帐户，密码管理也必须在该系统外部处理。有关内部凭据存储的更多信息，请参阅 Section 8.2.15, “Password Management”。

前面的限制不适用于使用`VALIDATE_PASSWORD_STRENGTH()`函数，因为它不直接影响帐户。

例子：

+   `validate_password`在以下语句中检查明文密码。根据要求密码至少为 8 个字符长的默认密码策略，密码较弱，该语句将产生错误：

    ```sql
    mysql> ALTER USER USER() IDENTIFIED BY 'abc';
    ERROR 1819 (HY000): Your password does not satisfy the current
    policy requirements
    ```

+   由于原始密码值不可用进行检查，指定为散列值的密码不会被检查：

    ```sql
    mysql> ALTER USER 'jeffrey'@'localhost'
           IDENTIFIED WITH mysql_native_password
           AS '*0D3CED9BEC10A777AEC23CCC353A8C08A633045E';
    Query OK, 0 rows affected (0.01 sec)
    ```

+   即使帐户最初被锁定，由于没有包含符合当前密码策略的密码，此帐户创建语句将失败：

    ```sql
    mysql> CREATE USER 'juanita'@'localhost' ACCOUNT LOCK;
    ERROR 1819 (HY000): Your password does not satisfy the current
    policy requirements
    ```

+   要检查密码，请使用`VALIDATE_PASSWORD_STRENGTH()`函数：

    ```sql
    mysql> SELECT VALIDATE_PASSWORD_STRENGTH('weak');
    +------------------------------------+
    | VALIDATE_PASSWORD_STRENGTH('weak') |
    +------------------------------------+
    |                                 25 |
    +------------------------------------+
    mysql> SELECT VALIDATE_PASSWORD_STRENGTH('lessweak$_@123');
    +----------------------------------------------+
    | VALIDATE_PASSWORD_STRENGTH('lessweak$_@123') |
    +----------------------------------------------+
    |                                           50 |
    +----------------------------------------------+
    mysql> SELECT VALIDATE_PASSWORD_STRENGTH('N0Tweak$_@123!');
    +----------------------------------------------+
    | VALIDATE_PASSWORD_STRENGTH('N0Tweak$_@123!') |
    +----------------------------------------------+
    |                                          100 |
    +----------------------------------------------+
    ```

要配置密码检查，请修改名称形式为`validate_password.*`xxx`*`的系统变量；这些是控制密码策略的参数。请参阅 Section 8.4.3.2, “Password Validation Options and Variables”。

如果未安装`validate_password`，则`validate_password.*`xxx`*`系统变量不可用，语句中的密码不会被检查，`VALIDATE_PASSWORD_STRENGTH()`函数始终返回 0。例如，如果没有安装插件，帐户可以被分配少于 8 个字符的密码，甚至可以没有密码。

假设已安装`validate_password`，它实现了三个级别的密码检查：`LOW`，`MEDIUM`和`STRONG`。默认为`MEDIUM`；要更改此设置，请修改`validate_password.policy`的值。这些策略实施越来越严格的密码测试。以下描述是针对默认参数值的，可以通过更改相应的系统变量来修改这些值。

+   `LOW`策略仅测试密码长度。密码必须至少为 8 个字符长。要更改此长度，请修改`validate_password.length`。

+   `MEDIUM`策略添加了密码必须包含至少 1 个数字字符、1 个小写字符、1 个大写字符和 1 个特殊（非字母数字）字符的条件。要更改这些值，请修改`validate_password.number_count`、`validate_password.mixed_case_count`和`validate_password.special_char_count`。

+   `STRONG`策略添加了一个条件，即长度为 4 或更长的密码子串不得与字典文件中的单词匹配（如果已指定）。要指定字典文件，请修改`validate_password.dictionary_file`。

此外，`validate_password`支持拒绝与当前会话的有效用户帐户的用户名部分匹配的密码，无论是正向还是反向。为了控制这种能力，`validate_password`暴露了一个`validate_password.check_user_name`系统变量，默认情况下已启用。
