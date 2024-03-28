# 8.2.15 密码管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/password-management.html`](https://dev.mysql.com/doc/refman/8.0/en/password-management.html)

MySQL 支持这些密码管理功能：

+   密码过期，要求定期更改密码。

+   密码重用限制，防止选择旧密码。

+   密码验证，要求密码更改时也需指定要替换的当前密码。

+   双密码，使客户端能够使用主密码或辅助密码连接。

+   密码强度评估，要求使用强密码。

+   随机密码生成，作为要求明确管理员指定文字密码的替代方案。

+   密码失败跟踪，以在连续太多次密码错误登录失败后启用临时帐户锁定。

以下各节描述了这些功能，除了密码强度评估，该功能使用`validate_password`组件实现，并在第 8.4.3 节，“密码验证组件”中进行了描述。

+   内部与外部凭据存储

+   密码过期策略

+   密码重用策略

+   密码验证要求策略

+   双密码支持

+   随机密码生成

+   登录失败跟踪和临时帐户锁定

重要

MySQL 使用`mysql`系统数据库中的表来实现密码管理功能。如果您从早期版本升级 MySQL，则您的系统表可能不是最新的。在这种情况下，服务器在启动过程中写入类似以下的错误日志消息（确切的数字可能有所不同）：

```sql
[ERROR] Column count of mysql.user is wrong. Expected
49, found 47\. The table is probably corrupted
[Warning] ACL table mysql.password_history missing.
Some operations may fail.
```

要纠正此问题，请执行 MySQL 升级过程。请参阅第三章，*升级 MySQL*。在此之前，*无法更改密码*。

#### 内部与外部凭据存储

一些认证插件将帐户凭据存储在 MySQL 内部，存储在`mysql.user`系统表中：

+   `mysql_native_password`

+   `caching_sha2_password`

+   `sha256_password`

本节中的大部分讨论适用于这些认证插件，因为这里描述的大多数密码管理功能都是基于 MySQL 本身处理的内部凭据存储。其他认证插件将账户凭据存储在 MySQL 之外。对于使用针对外部凭据系统执行认证的插件的账户，密码管理也必须在该系统外部处理。

例外情况是，对于所有账户，而不仅仅是使用内部凭据存储的账户，失败登录跟踪和临时账户锁定选项都适用，因为 MySQL 能够评估任何账户的登录尝试状态，无论它使用内部还是外部凭据存储。

有关各个认证插件的信息，请参见第 8.4.1 节，“认证插件”。

#### 密码过期策略

MySQL 使数据库管理员能够手动使账户密码过期，并建立自动密码过期策略。过期策略可以在全局范围内建立，并且可以设置个别账户要么遵循全局策略，要么使用特定的每个账户行为覆盖全局策略。

要手动使账户密码过期，请使用`ALTER USER`语句：

```sql
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE;
```

此操作会在`mysql.user`系统表中的相应行中标记密码过期。

根据策略，密码过期是自动的，并且基于密码的年龄，对于给定账户，从其最近更改密码的日期和时间进行评估。`mysql.user`系统表指示每个账户的密码上次更改的时间，并且如果其年龄大于其允许的寿命，则服务器会在客户端连接时自动将密码视为过期。这可以在没有明确手动密码过期的情况下工作。

要在全局范围内建立自动密码过期策略，请使用`default_password_lifetime`系统变量。其默认值为 0，这将禁用自动密码过期。如果`default_password_lifetime`的值是正整数*`N`*，则表示允许的密码寿命，即密码必须每*`N`*天更改一次。

示例：

+   要建立一个全局策略，使密码的寿命约为六个月，请在服务器的`my.cnf`文件中添加以下行：

    ```sql
    [mysqld]
    default_password_lifetime=180
    ```

+   要建立一个全局策略，使密码永不过期，请将`default_password_lifetime`设置为 0：

    ```sql
    [mysqld]
    default_password_lifetime=0
    ```

+   `default_password_lifetime`也可以在运行时设置和持久化：

    ```sql
    SET PERSIST default_password_lifetime = 180;
    SET PERSIST default_password_lifetime = 0;
    ```

    `SET PERSIST` 为正在运行的 MySQL 实例设置一个值。它还保存该值以便在后续服务器重启时继续使用；参见 Section 15.7.6.1, “变量赋值的 SET 语法”。要更改正在运行的 MySQL 实例的值，而不希望其在后续重启时继续使用，使用 `GLOBAL` 关键字而不是 `PERSIST`。

全局密码过期策略适用于未设置为覆盖它的所有帐户。要为单个帐户建立策略，请使用 `CREATE USER` 和 `ALTER USER` 语句的 `PASSWORD EXPIRE` 选项。参见 Section 15.7.1.3, “CREATE USER 语句”，以及 Section 15.7.1.1, “ALTER USER 语句”。

示例特定帐户语句：

+   要求每 90 天更改一次密码：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;
    ```

    此过期选项会覆盖语句指定的所有帐户的全局策略。

+   禁用密码过期：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;
    ```

    此过期选项会覆盖语句指定的所有帐户的全局策略。

+   延迟到语句指定的所有帐户的全局过期策略：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE DEFAULT;
    ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE DEFAULT;
    ```

当客户端成功连接时，服务器会确定帐户密码是否已过期：

+   服务器会检查密码是否已手动过期。

+   否则，服务器会检查密码的年龄是否超过根据自动密码过期策略允许的寿命。如果是，则服务器会将密码视为过期。

如果密码已过期（无论是手动还是自动），服务器要么断开客户端连接，要么限制其允许的操作（参见 Section 8.2.16, “过期密码的服务器处理”）。受限客户端执行的操作会导致错误，直到用户建立新的帐户密码：

```sql
mysql> SELECT 1;
ERROR 1820 (HY000): You must reset your password using ALTER USER
statement before executing this statement. 
mysql> ALTER USER USER() IDENTIFIED BY '*password*';
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT 1;
+---+
| 1 |
+---+
| 1 |
+---+
1 row in set (0.00 sec)
```

客户端重置密码后，服务器将为该会话以及使用该帐户的后续连接恢复正常访问。管理员用户也可以重置帐户密码，但该帐户的任何现有受限会话仍然受限。在执行语句成功之前，使用该帐户的客户端必须断开连接并重新连接。

注意

尽管可以通过将过期密码设置为当前值来“重置”过期密码，但作为良好策略，最好选择不同的密码。DBA 可以通过建立适当的密码重用策略来强制不重用。参见 密码重用策略。

#### 密码重用策略

MySQL 允许对先前密码的重用施加限制。重用限制可以基于密码更改次数、经过的时间或两者来建立。重用策略可以在全局范围内建立，并且可以设置个别帐户要么遵循全局策略，要么用特定的每个帐户行为覆盖全局策略。

帐户的密码历史包括其过去分配的密码。MySQL 可以限制新密码从此历史中选择：

+   如果一个帐户受到密码更改次数的限制，新密码不能选择最近一定数量的最近密码之一。例如，如果最小密码更改次数设置为 3，新密码不能与最近的任何 3 个密码相同。

+   如果一个帐户受到经过时间限制，新密码不能从历史中选择的密码中选择。例如，如果密码重用间隔设置为 60，新密码必须不是在过去 60 天内先前选择的密码之一。

注意

空密码不计入密码历史，并可随时重新使用。

要在全局范围内建立密码重用策略，请使用`password_history`和`password_reuse_interval`系统变量。

例子：

+   要禁止重用最后 6 个密码或新于 365 天的密码，请将以下行放入服务器`my.cnf`文件中：

    ```sql
    [mysqld]
    password_history=6
    password_reuse_interval=365
    ```

+   要在运行时设置和持久化变量，请使用以下语句：

    ```sql
    SET PERSIST password_history = 6;
    SET PERSIST password_reuse_interval = 365;
    ```

    `SET PERSIST`为运行中的 MySQL 实例设置一个值。它还保存该值以在后续服务器重新启动时传递；参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。要更改运行中的 MySQL 实例的值，而不使其在后续重新启动时传递，请使用`GLOBAL`关键字而不是`PERSIST`。

全局密码重用策略适用于未被设置为覆盖它的所有帐户。要为个别帐户建立策略，请使用`CREATE USER`和`ALTER USER`语句的`PASSWORD HISTORY`和`PASSWORD REUSE INTERVAL`选项。参见 Section 15.7.1.3, “CREATE USER Statement”和 Section 15.7.1.1, “ALTER USER Statement”。

个别帐户语句示例：

+   要求至少更改 5 次密码才允许重用：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD HISTORY 5;
    ALTER USER 'jeffrey'@'localhost' PASSWORD HISTORY 5;
    ```

    此历史长度选项将覆盖语句命名的所有帐户的全局策略。

+   要求至少 365 天经过才允许重用：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REUSE INTERVAL 365 DAY;
    ALTER USER 'jeffrey'@'localhost' PASSWORD REUSE INTERVAL 365 DAY;
    ```

    此经过时间的选项覆盖了语句命名的所有账户的全局策略。

+   要结合两种类型的重用限制，一起使用`PASSWORD HISTORY`和`PASSWORD REUSE INTERVAL`：

    ```sql
    CREATE USER 'jeffrey'@'localhost'
      PASSWORD HISTORY 5
      PASSWORD REUSE INTERVAL 365 DAY;
    ALTER USER 'jeffrey'@'localhost'
      PASSWORD HISTORY 5
      PASSWORD REUSE INTERVAL 365 DAY;
    ```

    这些选项覆盖了语句命名的所有账户的全局策略重用限制。

+   对于两种类型的重用限制，应遵循全局策略：

    ```sql
    CREATE USER 'jeffrey'@'localhost'
      PASSWORD HISTORY DEFAULT
      PASSWORD REUSE INTERVAL DEFAULT;
    ALTER USER 'jeffrey'@'localhost'
      PASSWORD HISTORY DEFAULT
      PASSWORD REUSE INTERVAL DEFAULT;
    ```

#### 需要密码验证的策略

从 MySQL 8.0.13 开始，可以要求验证更改账户密码的尝试，方法是指定要替换的当前密码。这使得 DBA 可以防止用户在未证明知道当前密码的情况下更改密码。否则，例如，如果一个用户暂时离开终端会话而没有注销，那么恶意用户可以使用该会话更改原始用户的 MySQL 密码。这可能会产生不幸的后果：

+   原始用户在管理员重置账户密码之前无法访问 MySQL。

+   直到密码重置发生，恶意用户可以使用良性用户更改的凭据访问 MySQL。

可以在全局范围内建立密码验证策略，并且可以设置单独的账户以延迟全局策略或使用特定的每个账户行为覆盖全局策略。

对于每个账户，其`mysql.user`行指示是否有一个账户特定设置，要求在尝试更改密码时验证当前密码。该设置由`CREATE USER`和`ALTER USER`语句的`PASSWORD REQUIRE`选项建立：

+   如果账户设置为`PASSWORD REQUIRE CURRENT`，密码更改必须指定当前密码。

+   如果账户设置为`PASSWORD REQUIRE CURRENT OPTIONAL`，密码更改可能需要但不一定需要指定当前密码。

+   如果账户设置为`PASSWORD REQUIRE CURRENT DEFAULT`，`password_require_current`系统变量确定账户的验证要求策略：

    +   如果`password_require_current`被启用，密码更改必须指定当前密码。

    +   如果`password_require_current`被禁用，密码更改可能需要但不一定需要指定当前密码。

换句话说，如果账户设置不是`PASSWORD REQUIRE CURRENT DEFAULT`，则账户设置优先于由`password_require_current`系统变量建立的全局策略。否则，账户将遵循`password_require_current`设置。

默认情况下，密码验证是可选的：`password_require_current`已禁用，并且使用无`PASSWORD REQUIRE`选项创建的帐户默认为`PASSWORD REQUIRE CURRENT DEFAULT`。

以下表格显示了每个帐户设置如何与`password_require_current`系统变量值交互，以确定帐户密码验证所需策略。

**表 8.10 密码验证策略**

| 每个帐户设置 | password_require_current 系统变量 | 密码更改是否需要当前密码？ |
| --- | --- | --- |
| `PASSWORD REQUIRE CURRENT` | `OFF` | 是 |
| `PASSWORD REQUIRE CURRENT` | `ON` | 是 |
| `PASSWORD REQUIRE CURRENT OPTIONAL` | `OFF` | 否 |
| `PASSWORD REQUIRE CURRENT OPTIONAL` | `ON` | 否 |
| `PASSWORD REQUIRE CURRENT DEFAULT` | `OFF` | 否 |
| `PASSWORD REQUIRE CURRENT DEFAULT` | `ON` | 是 |

注意

特权用户可以在不指定当前密码的情况下更改任何帐户密码，而不受需要验证的策略限制。特权用户是具有全局`CREATE USER`权限或`mysql`系统数据库的`UPDATE`权限的用户。

要在全局范围内建立密码验证策略，请使用`password_require_current`系统变量。其默认值为`OFF`，因此不需要指定当前密码来更改帐户密码。

示例：

+   要建立一个全局策略，要求密码更改必须指定当前密码，请在服务器`my.cnf`文件中使用以下行启动服务器：

    ```sql
    [mysqld]
    password_require_current=ON
    ```

+   要在运行时设置并持久化`password_require_current`，可以使用以下语句之一：

    ```sql
    SET PERSIST password_require_current = ON;
    SET PERSIST password_require_current = OFF;
    ```

    `SET PERSIST`为正在运行的 MySQL 实例设置一个值。它还保存该值以在后续服务器重新启动时继续使用；请参阅第 15.7.6.1 节，“变量赋值的 SET 语法”。要更改正在运行的 MySQL 实例的值，而不使其在后续重新启动时继续使用，使用`GLOBAL`关键字而不是`PERSIST`。

全局密码验证所需策略适用于未被设置为覆盖它的所有帐户。要为单个帐户建立策略，请使用`CREATE USER`和`ALTER USER`语句的`PASSWORD REQUIRE`选项。请参阅第 15.7.1.3 节，“CREATE USER 语句”和第 15.7.1.1 节，“ALTER USER 语句”。

示例特定帐户语句：

+   要求更改密码时指定当前密码：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT;
    ALTER USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT;
    ```

    此验证选项覆盖了语句命名的所有帐户的全局策略。

+   不要求更改密码时指定当前密码（当前密码可以但不必给出）：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT OPTIONAL;
    ALTER USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT OPTIONAL;
    ```

    此验证选项覆盖了语句命名的所有帐户的全局策略。

+   延迟到语句命名的所有帐户的全局密码验证要求策略：

    ```sql
    CREATE USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT DEFAULT;
    ALTER USER 'jeffrey'@'localhost' PASSWORD REQUIRE CURRENT DEFAULT;
    ```

当用户使用`ALTER USER`或`SET PASSWORD`语句更改密码时，当前密码的验证起作用。示例使用`ALTER USER`，这比使用`SET PASSWORD`更可取，但这里描述的原则对两个语句都是相同的。

在更改密码的语句中，`REPLACE`子句指定要替换的当前密码。示例：

+   更改当前用户的密码：

    ```sql
    ALTER USER USER() IDENTIFIED BY '*auth_string*' REPLACE '*current_auth_string*';
    ```

+   更改命名用户的密码：

    ```sql
    ALTER USER 'jeffrey'@'localhost'
      IDENTIFIED BY '*auth_string*'
      REPLACE '*current_auth_string*';
    ```

+   更改命名用户的身份验证插件和密码：

    ```sql
    ALTER USER 'jeffrey'@'localhost'
      IDENTIFIED WITH caching_sha2_password BY '*auth_string*'
      REPLACE '*current_auth_string*';
    ```

`REPLACE`子句的工作原理如下：

+   如果需要为帐户更改密码以指定当前密码，则必须提供`REPLACE`，以验证试图进行更改的用户实际上知道当前密码。

+   如果需要为帐户更改密码但不需要指定当前密码，则`REPLACE`是可选的。

+   如果指定了`REPLACE`，则必须指定正确的当前密码，否则将出现错误。即使`REPLACE`是可选的也是如此。

+   只有在更改当前用户的帐户密码时才能指定`REPLACE`。（这意味着在刚刚显示的示例中，明确命名`jeffrey`帐户的语句将失败，除非当前用户是`jeffrey`。）即使特权用户尝试为另一个用户更改密码，也是如此；但是，这样的用户可以更改任何密码而无需指定`REPLACE`。

+   为避免将明文密码写入二进制日志，`REPLACE`被省略。

#### 双密码支持

截至 MySQL 8.0.14，用户帐户被允许具有主密码和辅助密码，称为双密码。双密码功能使得在诸如此类场景中无缝执行凭据更改成为可能：

+   一个系统有大量的 MySQL 服务器，可能涉及复制。

+   多个应用程序连接到不同的 MySQL 服务器。

+   必须定期更改用于连接到服务器的应用程序使用的帐户或帐户的凭据。

考虑在前述类型的场景中执行凭据更改时，当一个账户只允许一个密码时。在这种情况下，必须在账户密码更改和在所有服务器上传播时以及所有使用该账户的应用程序更新以使用新密码的时间上进行紧密合作。这个过程可能涉及停机，期间服务器或应用程序不可用。

通过双重密码，可以更轻松地分阶段进行凭据更改，无需紧密合作，也无需停机：

1.  对于每个受影响的账户，在服务器上建立一个新的主密码，将当前密码保留为次要密码。这使得服务器可以识别每个账户的主密码或次要密码，而应用程序可以继续使用与以前相同的密码连接到服务器（现在是次要密码）。

1.  在密码更改传播到所有服务器后，修改使用任何受影响账户的应用程序，以使用账户主密码连接。

1.  在所有应用程序已从次要密码迁移到主密码后，次要密码不再需要，可以丢弃。此更改传播到所有服务器后，每个账户只能使用主密码连接。凭据更改现在已完成。

MySQL 使用保存和丢弃次要密码的语法实现双重密码功能：

+   `RETAIN CURRENT PASSWORD`子句用于`ALTER USER`和`SET PASSWORD`语句，当您分配新的主密码时，会将账户当前密码保存为其次要密码。

+   `DISCARD OLD PASSWORD`子句用于`ALTER USER`，丢弃账户次要密码，仅保留主密码。

假设对于先前描述的凭据更改场景，一个名为`'appuser1'@'host1.example.com'`的账户被应用程序用于连接服务器，并且要将账户密码从`'*`password_a`*'`更改为`'*`password_b`*'`。

要执行凭据更改，请使用以下`ALTER USER`：

1.  在每个非副本服务器上，建立`'*`password_b`*'`作为新的`appuser1`主密码，保留当前密码作为次要密码：

    ```sql
    ALTER USER 'appuser1'@'host1.example.com'
      IDENTIFIED BY '*password_b*'
      RETAIN CURRENT PASSWORD;
    ```

1.  等待密码更改在整个系统中复制到所有副本。

1.  修改使用`appuser1`账户的每个应用程序，使其使用密码`'*`password_b`*'`而不是`'*`password_a`*'`连接到服务器。

1.  此时，次要密码不再需要。在每个非副本服务器上丢弃次要密码：

    ```sql
    ALTER USER 'appuser1'@'host1.example.com'
      DISCARD OLD PASSWORD;
    ```

1.  在丢弃密码更改已经复制到所有副本之后，凭据更改完成。

`RETAIN CURRENT PASSWORD` 和 `DISCARD OLD PASSWORD` 子句具有以下效果：

+   `RETAIN CURRENT PASSWORD` 会保留账户当前密码作为其二级密码，替换任何现有的二级密码。新密码成为主密码，但客户端可以使用主密码或二级密码连接到服务器。 （例外情况：如果由 `ALTER USER` 或 `SET PASSWORD` 语句指定的新密码为空，则即使给出 `RETAIN CURRENT PASSWORD`，二级密码也会变为空。）

+   如果您为具有空主密码的账户指定 `RETAIN CURRENT PASSWORD`，则该语句将失败。

+   如果一个账户有二级密码，并且您更改其主密码而不指定 `RETAIN CURRENT PASSWORD`，则二级密码保持不变。

+   对于 `ALTER USER`，如果更改分配给账户的身份验证插件，则二级密码将被丢弃。 如果更改身份验证插件并且还指定 `RETAIN CURRENT PASSWORD`，则该语句将失败。

+   对于 `ALTER USER`，`DISCARD OLD PASSWORD` 会丢弃二级密码（如果存在）。账户仅保留其主密码，客户端只能使用主密码连接到服务器。

修改二级密码的语句需要以下权限：

+   要使用 `RETAIN CURRENT PASSWORD` 或 `DISCARD OLD PASSWORD` 子句来操作您自己的账户的 `ALTER USER` 和 `SET PASSWORD` 语句，需要 `APPLICATION_PASSWORD_ADMIN` 权限。 大多数用户只需要一个密码，因此需要该权限来操作自己的二级密码。

+   如果要允许一个账户为所有账户操作二级密码，则应授予 `CREATE USER` 权限，而不是 `APPLICATION_PASSWORD_ADMIN`。

#### 随机密码生成

截至 MySQL 8.0.18 版本，`CREATE USER`，`ALTER USER` 和 `SET PASSWORD` 语句具有生成用户账户随机密码的功能，作为明文密码的替代选择。有关语法的详细信息，请参阅每个语句的描述。本节描述了生成随机密码的共同特征。

默认情况下，生成的随机密码长度为 20 个字符。这个长度由`generated_random_password_length`系统变量控制，范围从 5 到 255。

对于每个语句生成随机密码的账户，该语句将密码存储在`mysql.user`系统表中，适当地为账户认证插件进行哈希处理。该语句还会在结果集的一行中返回明文密码，以便用户或应用程序执行该语句。结果集列名为`user`，`host`，`generated password`和`auth_factor`，表示标识`mysql.user`系统表中受影响行的用户名和主机名值，明文生成的密码，以及显示密码值适用的认证因素。

```sql
mysql> CREATE USER
       'u1'@'localhost' IDENTIFIED BY RANDOM PASSWORD,
       'u2'@'%.example.com' IDENTIFIED BY RANDOM PASSWORD,
       'u3'@'%.org' IDENTIFIED BY RANDOM PASSWORD;
+------+---------------+----------------------+-------------+
| user | host          | generated password   | auth_factor |
+------+---------------+----------------------+-------------+
| u1   | localhost     | iOeqf>Mh9:;XD&qn(Hl} |           1 |
| u2   | %.example.com | sXTSAEvw3St-R+_-C3Vb |           1 |
| u3   | %.org         | nEVe%Ctw/U/*Md)Exc7& |           1 |
+------+---------------+----------------------+-------------+
mysql> ALTER USER
       'u1'@'localhost' IDENTIFIED BY RANDOM PASSWORD,
       'u2'@'%.example.com' IDENTIFIED BY RANDOM PASSWORD;
+------+---------------+----------------------+-------------+
| user | host          | generated password   | auth_factor |
+------+---------------+----------------------+-------------+
| u1   | localhost     | Seiei:&cw}8]@3OA64vh |           1 |
| u2   | %.example.com | j@&diTX80l8}(NiHXSae |           1 |
+------+---------------+----------------------+-------------+
mysql> SET PASSWORD FOR 'u3'@'%.org' TO RANDOM;
+------+-------+----------------------+-------------+
| user | host  | generated password   | auth_factor |
+------+-------+----------------------+-------------+
| u3   | %.org | n&cz2xF;P3!U)+]Vw52H |           1 |
+------+-------+----------------------+-------------+
```

一个`CREATE USER`，`ALTER USER`或`SET PASSWORD`语句，为一个账户生成一个随机密码，被写入二进制日志作为一个带有`IDENTIFIED WITH *`auth_plugin`* AS '*`auth_string`*`'`子句的`CREATE USER`或`ALTER USER`语句，其中*`auth_plugin`*是账户认证插件，`'*`auth_string`*`'`是账户哈希密码值。

如果安装了`validate_password`组件，则其实施的策略对生成的密码没有影响。（密码验证的目的是帮助人类创建更好的密码。）

#### 登录失败跟踪和临时账户锁定

从 MySQL 8.0.19 开始，管理员可以配置用户账户，使得连续登录失败次数过多会导致临时账户锁定。

在这种情况下，“登录失败”意味着客户端在连接尝试期间未提供正确密码。这不包括由于未知用户或网络问题等原因而无法连接的情况。对于具有双重密码的账户（参见双重密码支持），任一账户密码都算正确。

每个账户的所需登录失败次数和锁定时间可通过`CREATE USER`和`ALTER USER`语句的`FAILED_LOGIN_ATTEMPTS`和`PASSWORD_LOCK_TIME`选项进行配置。示例：

```sql
CREATE USER 'u1'@'localhost' IDENTIFIED BY '*password*'
  FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 3;

ALTER USER 'u2'@'localhost'
  FAILED_LOGIN_ATTEMPTS 4 PASSWORD_LOCK_TIME UNBOUNDED;
```

当连续登录失败次数过多时，客户端会收到如下错误消息：

```sql
ERROR 3957 (HY000): Access denied for user *user*.
Account is blocked for *D* day(s) (*R* day(s) remaining)
due to *N* consecutive failed logins.
```

使用选项如下：

+   `FAILED_LOGIN_ATTEMPTS *`N`*`

    此选项指示是否跟踪指定了不正确密码的账户登录尝试。数字*`N`*指定了多少个连续的不正确密码会导致临时账户锁定。

+   `PASSWORD_LOCK_TIME {*`N`* | UNBOUNDED}`

    此选项指示在太多连续登录尝试提供错误密码后锁定帐户的时间。该值是一个数字*`N`*，用于指定帐户保持锁定状态的天数，或`UNBOUNDED`以指定当帐户进入临时锁定状态时，该状态的持续时间是无限的，并且直到帐户被解锁为止。解锁发生的条件将在后面描述。

每个选项的允许值*`N`*的范围为 0 到 32767。值为 0 会禁用该选项。

失败登录跟踪和临时帐户锁定具有以下特征：

+   要对帐户进行失败登录跟踪和临时锁定，其`FAILED_LOGIN_ATTEMPTS`和`PASSWORD_LOCK_TIME`选项都必须为非零。

+   对于`CREATE USER`，如果未指定`FAILED_LOGIN_ATTEMPTS`或`PASSWORD_LOCK_TIME`，则其隐式默认值对于语句命名的所有帐户均为 0。这意味着禁用了失败登录跟踪和临时帐户锁定。（这些隐式默认值也适用于在引入失败登录跟踪之前创建的帐户。）

+   对于`ALTER USER`，如果未指定`FAILED_LOGIN_ATTEMPTS`或`PASSWORD_LOCK_TIME`，则其值对于语句命名的所有帐户保持不变。

+   要发生临时帐户锁定，密码失败必须是连续的。在达到失败登录的`FAILED_LOGIN_ATTEMPTS`值之前发生的任何成功登录都会导致失败计数重置。例如，如果`FAILED_LOGIN_ATTEMPTS`为 4，并且已发生三次连续密码失败，则需要再次失败一次才能开始锁定。但是，如果下次登录成功，则帐户的失败登录计数将被重置，因此再次需要四次连续失败才能锁定。

+   一旦临时锁定开始，即使使用正确密码也无法进行成功登录，直到锁定持续时间已过或帐户通过以下讨论中列出的帐户重置方法之一被解锁。

当服务器读取授权表时，它会初始化每个帐户的状态信息，包括是否启用了失败登录跟踪，帐户当前是否被临时锁定以及如果是的话锁定何时开始，以及如果帐户未被锁定，则在临时锁定发生之前的失败次数。

帐户的状态信息可以被重置，这意味着失败登录计数被重置，并且如果当前被临时锁定，则帐户被解锁。帐户重置可以是全局的，适用于所有帐户，也可以是每个帐户：

+   所有帐户的全局重置发生在以下任何条件下：

    +   服务器重新启动。

    +   执行`FLUSH PRIVILEGES`。（使用`--skip-grant-tables`选项启动服务器会导致授予权限表不被读取，从而禁用了失败登录跟踪。在这种情况下，第一次执行`FLUSH PRIVILEGES`会导致服务器读取授予权限表并启用失败登录跟踪，同时重置所有账户。）

+   对于任何以下条件，每个账户都会发生重置：

    +   账户成功登录。

    +   锁定持续时间过去。在这种情况下，失败登录计数将在下次登录尝试时重置。

    +   对于账户执行设置`FAILED_LOGIN_ATTEMPTS`或`PASSWORD_LOCK_TIME`（或两者）任何值（包括当前选项值）的`ALTER USER`语句的执行，或对于账户执行`ALTER USER ... UNLOCK`语句。

        对于账户的其他`ALTER USER`语句对其当前的失败登录计数或锁定状态没有影响。

失败登录跟踪与用于检查凭据的登录账户相关联。如果使用用户代理，跟踪将针对代理用户而不是被代理用户进行。也就是说，跟踪与`USER()`指示的账户相关联，而不是与`CURRENT_USER()`指示的账户相关联。有关代理用户和被代理用户之间区别的信息，请参阅第 8.2.19 节，“代理用户”。
