# 3.5 MySQL 8.0 的变化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)

在升级到 MySQL 8.0 之前，请查看本节中描述的更改，以确定哪些适用于您当前的 MySQL 安装和应用程序。执行任何建议的操作。

标记为**不兼容更改**的更改与早期版本的 MySQL 不兼容，并且可能需要您在升级之前*注意*。我们的目标是避免这些更改，但偶尔它们是必要的，以纠正比版本之间的不兼容性更糟糕的问题。如果适用于您的安装的升级问题涉及不兼容性，请按照描述中给出的说明操作。

+   数据字典变化

+   将 caching_sha2_password 作为首选身份验证插件

+   配置更改

+   服务器更改

+   InnoDB 变化

+   SQL 更改

+   更改的服务器默认值

+   有效性能回归

### 数据字典变化

MySQL Server 8.0 包含一个全局数据字典，其中包含事务表中数据库对象的信息。在以前的 MySQL 系列中，字典数据存储在元数据文件和非事务系统表中。因此，升级过程要求您通过检查特定先决条件来验证安装的升级准备情况。有关更多信息，请参见第 3.6 节，“准备升级安装”。启用数据字典的服务器存在一些一般操作上的差异；请参见第 16.7 节，“数据字典使用差异”。

### 将 caching_sha2_password 作为首选身份验证插件

`caching_sha2_password` 和 `sha256_password` 认证插件提供比 `mysql_native_password` 插件更安全的密码加密，而 `caching_sha2_password` 提供比 `sha256_password` 更好的性能。由于 `caching_sha2_password` 具有卓越的安全性和性能特性，因此从 MySQL 8.0 开始，它是首选的认证插件，并且也是默认的认证插件，而不是 `mysql_native_password`。这一变化影响了服务器和 `libmysqlclient` 客户端库：

+   对于服务器，`default_authentication_plugin` 系统变量的默认值从 `mysql_native_password` 更改为 `caching_sha2_password`。

    这一变化仅适用于安装或升级到 MySQL 8.0 或更高版本后创建的新帐户。对于已存在于升级安装中的帐户，它们的认证插件保持不变。希望切换到 `caching_sha2_password` 的现有用户可以使用 `ALTER USER` 语句进行切换：

    ```sql
    ALTER USER *user*
      IDENTIFIED WITH caching_sha2_password
      BY '*password*';
    ```

+   `libmysqlclient` 库将 `caching_sha2_password` 视为默认的认证插件，而不是 `mysql_native_password`。

以下部分讨论了 `caching_sha2_password` 更显著角色的影响：

+   caching_sha2_password 兼容性问题和解决方案

+   caching_sha2_password 兼容的客户端和连接器

+   caching_sha2_password 和 root 管理员帐户

+   caching_sha2_password 和复制

#### `caching_sha2_password` 兼容性问题和解决方案

重要

如果您的 MySQL 安装必须为 8.0 之前的客户端提供服务，并且在升级到 MySQL 8.0 或更高版本后遇到兼容性问题，解决这些问题并恢复到 8.0 之前的兼容性的最简单方法是重新配置服务器以恢复到先前的默认认证插件 (`mysql_native_password`)。例如，在服务器选项文件中使用以下行：

```sql
[mysqld]
default_authentication_plugin=mysql_native_password
```

该设置使得 8.0 之前的客户端可以连接到 8.0 服务器，直到您的安装中使用的客户端和连接器升级以了解`caching_sha2_password`为止。然而，该设置应被视为临时解决方案，而不是长期或永久解决方案，因为在该设置生效时创建的新帐户将放弃`caching_sha2_password`提供的改进的身份验证安全性。

使用`caching_sha2_password`比`mysql_native_password`提供更安全的密码哈希（以及随之改进的客户端连接身份验证）。然而，它也具有兼容性影响，可能会影响现有的 MySQL 安装：

+   客户端和连接器如果没有更新以了解`caching_sha2_password`，可能会在连接到配置了`caching_sha2_password`作为默认身份验证插件的 MySQL 8.0 服务器时遇到问题，即使是使用不使用`caching_sha2_password`进行身份验证的帐户也是如此。这个问题的原因是服务器向客户端指定了其默认身份验证插件的名称。如果客户端或连接器基于一个不能优雅处理未识别的默认身份验证插件的客户端/服务器协议实现，可能会出现以下错误之一：

    ```sql
    Authentication plugin 'caching_sha2_password' is not supported
    ```

    ```sql
    Authentication plugin 'caching_sha2_password' cannot be loaded:
    dlopen(/usr/local/mysql/lib/plugin/caching_sha2_password.so, 2):
    image not found
    ```

    ```sql
    Warning: mysqli_connect(): The server requested authentication
    method unknown to the client [caching_sha2_password]
    ```

    有关编写连接器以优雅处理服务器对未知默认身份验证插件的请求的信息，请参见身份验证插件连接器编写注意事项。

+   使用一个使用`caching_sha2_password`进行身份验证的帐户的客户端必须使用安全连接（使用 TLS/SSL 凭据通过 TCP 进行连接，一个 Unix 套接字文件，或共享内存），或者支持使用 RSA 密钥对进行密码交换的未加密连接。这个安全要求不适用于`mysql_native_passsword`，因此切换到`caching_sha2_password`可能需要额外的配置（参见 8.4.1.2 节，“Caching SHA-2 Pluggable Authentication”）。然而，在 MySQL 8.0 中，默认情况下客户端连接更倾向于使用 TLS/SSL，因此已经符合该偏好的客户端可能不需要额外的配置。

+   未更新以了解`caching_sha2_password`的客户端和连接器*无法*连接到使用`caching_sha2_password`进行身份验证的账户，因为它们不认可此插件为有效。（这是客户端/服务器认证插件兼容性要求的一个特殊实例，如认证插件客户端/服务器兼容性中所讨论的。）要解决此问题，请重新链接客户端到 MySQL 8.0 或更高版本的`libmysqlclient`，或获取一个识别`caching_sha2_password`的更新连接器。

+   因为`caching_sha2_password`现在也是`libmysqlclient`客户端库中的默认认证插件，所以从 MySQL 8.0 客户端连接到使用`mysql_native_password`（之前的默认认证插件）的账户时，认证需要在客户端/服务器协议中进行额外的往返，除非客户端程序使用`--default-auth=mysql_native_password`选项调用。

用于 8.0 之前 MySQL 版本的`libmysqlclient`客户端库能够连接到 MySQL 8.0 服务器（除了使用`caching_sha2_password`进行身份验证的账户）。这意味着基于`libmysqlclient`的 8.0 之前的客户端也应该能够连接。例如：

+   标准的 MySQL 客户端，如**mysql**和**mysqladmin**都是基于`libmysqlclient`。

+   Perl DBI 的 DBD::mysql 驱动程序是基于`libmysqlclient`的。

+   MySQL Connector/Python 具有基于`libmysqlclient`的 C 扩展模块。要使用它，请在连接时包含`use_pure=False`选项。

当现有的 MySQL 8.0 安装升级到 MySQL 8.0.4 或更高版本时，一些较旧的基于`libmysqlclient`的客户端可能会“自动”升级，因为它们是动态链接的，因此它们使用升级安装的新客户端库。例如，如果 Perl DBI 的 DBD::mysql 驱动程序使用动态链接，它可以在升级到 MySQL 8.0.4 或更高版本后直接使用`libmysqlclient`，结果如下：

+   在升级之前，使用 DBD::mysql 的 DBI 脚本可以连接到 MySQL 8.0 服务器，除了使用`caching_sha2_password`进行身份验证的账户。

+   升级后，相同的脚本也能够使用`caching_sha2_password`账户。

然而，前述结果发生是因为 MySQL 8.0 安装中 8.0.4 之前的`libmysqlclient`实例是二进制兼容的：它们都使用共享库主版本号 21。对于链接到来自 MySQL 5.7 或更早版本的`libmysqlclient`的客户端，它们链接到具有不兼容的不同版本号的共享库。在这种情况下，客户端必须重新编译以便与 MySQL 8.0 服务器和`caching_sha2_password`帐户完全兼容。

MySQL Connector/J 5.1 至 8.0.8 能够连接到 MySQL 8.0 服务器，除了使用`caching_sha2_password`进行身份验证的帐户。（连接器/J 8.0.9 或更高版本需要连接到`caching_sha2_password`帐户。）

使用除`libmysqlclient`之外的客户端/服务器协议实现的客户端可能需要升级到了解新身份验证插件的较新版本。例如，在 PHP 中，MySQL 连接通常基于`mysqlnd`，目前不了解`caching_sha2_password`。在更新的`mysqlnd`版本可用之前，使 PHP 客户端连接到 MySQL 8.0 的方法是重新配置服务器，将`mysql_native_password`恢复为默认身份验证插件，如前所述。

如果客户端或连接器支持显式指定默认身份验证插件的选项，请使用它来命名除`caching_sha2_password`之外的插件。示例：

+   一些 MySQL 客户端支持`--default-auth`选项。（标准 MySQL 客户端，如**mysql**和**mysqladmin**支持此选项，但可以成功连接到 8.0 服务器而无需它。但是，其他客户端可能支持类似的选项。如果是这样，值得尝试。）

+   使用`libmysqlclient` C API 的程序可以使用`MYSQL_DEFAULT_AUTH`选项调用`mysql_options()`函数。

+   使用客户端/服务器协议的本机 Python 实现的 MySQL Connector/Python 脚本可以指定`auth_plugin`连接选项。（或者，使用 Connector/Python C 扩展，可以连接到 MySQL 8.0 服务器而无需`auth_plugin`。）

#### 兼容`caching_sha2_password`的客户端和连接器

如果有已更新以了解`caching_sha2_password`的客户端或连接器可用，则使用它是连接到配置为默认身份验证插件为`caching_sha2_password`的 MySQL 8.0 服务器时确保兼容性的最佳方法。

这些客户端和连接器已升级以支持`caching_sha2_password`：

+   MySQL 8.0（8.0.4 或更高版本）中的 `libmysqlclient` 客户端库。标准 MySQL 客户端，如 **mysql** 和 **mysqladmin** 基于 `libmysqlclient`，因此它们也是���容的。

+   MySQL 5.7（5.7.23 或更高版本）中的 `libmysqlclient` 客户端库。标准 MySQL 客户端，如 **mysql** 和 **mysqladmin** 基于 `libmysqlclient`，因此它们也是兼容的。

+   MySQL Connector/C++ 1.1.11 或更高版本或 8.0.7 或更高版本。

+   MySQL Connector/J 8.0.9 或更高版本。

+   MySQL Connector/NET 8.0.10 或更高版本（通过经典 MySQL 协议）。

+   MySQL Connector/Node.js 8.0.9 或更高版本。

+   PHP：X DevAPI PHP 扩展（mysql_xdevapi）支持 `caching_sha2_password`。

    PHP：PDO_MySQL 和 ext/mysqli 扩展不支持 `caching_sha2_password`。此外，当与 PHP 版本 7.1.16 之前的版本和 PHP 7.2 之前的版本一起使用时，即使未使用 `caching_sha2_password`，它们也无法连接到 `default_authentication_plugin=caching_sha2_password`。

#### caching_sha2_password 和 root 管理员帐户

对于升级到 MySQL 8.0，现有帐户的身份验证插件保持不变，包括 `'root'@'localhost'` 管理员帐户的插件。

对于新的 MySQL 8.0 安装，在初始化数据目录时（使用 第 2.9.1 节，“初始化数据目录” 中的说明），会创建 `'root'@'localhost'` 帐户，并且该帐户默认使用 `caching_sha2_password`。因此，在数据目录初始化后连接到服务器时，您必须使用支持 `caching_sha2_password` 的客户端或连接器。如果您可以这样做，但希望安装后 `root` 帐户使用 `mysql_native_password`，请按照通常的方式安装 MySQL 并初始化数据目录。然后连接到服务器作为 `root`，并使用 `ALTER USER` 如下更改帐户身份验证插件和密码：

```sql
ALTER USER 'root'@'localhost'
  IDENTIFIED WITH mysql_native_password
  BY '*password*';
```

如果您使用的客户端或连接器尚不支持 `caching_sha2_password`，则可以使用修改后的数据目录初始化过程，一旦创建帐户，就将 `root` 帐户与 `mysql_native_password` 关联起来。为此，可以使用以下任一技术：

+   在 `--initialize` 或 `--initialize-insecure` 选项中提供一个 `--default-authentication-plugin=mysql_native_password` 选项。

+   在选项文件中将`default_authentication_plugin`设置为`mysql_native_password`，并使用`--defaults-file`选项命名该选项文件，以及`--initialize`或`--initialize-insecure`。 （在这种情况下，如果您继续使用该选项文件进行后续服务器启动，则新帐户将使用`mysql_native_password`而不是`caching_sha2_password`创建，除非从选项文件中删除`default_authentication_plugin`设置。）

#### `caching_sha2_password`和复制

对于所有服务器都已升级到 MySQL 8.0.4 或更高版本的复制场景，副本连接到源服务器可以使用使用`caching_sha2_password`进行身份验证的账户。对于这样的连接，与使用`caching_sha2_password`进行身份验证的其他客户端相同的要求适用：使用安全连接或基于 RSA 的密码交换。

要连接到用于源/副本复制的`caching_sha2_password`账户：

+   使用以下任何一个`CHANGE MASTER TO`选项：

    ```sql
    MASTER_SSL = 1
    GET_MASTER_PUBLIC_KEY = 1
    MASTER_PUBLIC_KEY_PATH='*path to RSA public key file*'
    ```

+   或者，如果在服务器启动时提供所需的密钥，则可以使用 RSA 公钥相关选项。

要连接到使用`caching_sha2_password`账户的组复制：

+   对于使用 OpenSSL 构建的 MySQL，设置以下任何一个系统变量：

    ```sql
    SET GLOBAL group_replication_recovery_use_ssl = ON;
    SET GLOBAL group_replication_recovery_get_public_key = 1;
    SET GLOBAL group_replication_recovery_public_key_path = '*path to RSA public key file*';
    ```

+   或者，如果在服务器启动时提供所需的密钥，则可以使用 RSA 公钥相关选项。

### 配置更改

+   **不兼容更改**：MySQL 存储引擎现在负责提供自己的分区处理程序，MySQL 服务器不再提供通用分区支持。`InnoDB`和`NDB`是唯一提供本机分区处理程序并在 MySQL 8.0 中受支持的存储引擎。使用任何其他存储引擎的分区表必须在升级服务器之前进行更改，要么将其转换为`InnoDB`或`NDB`，要么删除其分区，*否则*之后无法使用。

    有关将`MyISAM`表转换为`InnoDB`的信息，请参见第 17.6.1.5 节，“从 MyISAM 转换表到 InnoDB”。

    在 MySQL 8.0 中，使用不支持的存储引擎创建分区表的表创建语句将失败并显示错误（ER_CHECK_NOT_IMPLEMENTED）。如果您从在 MySQL 5.7（或更早版本）中创建的转储文件中导入数据库到 MySQL 8.0 服务器，请确保任何创建分区表的语句不会同时指定不支持的存储引擎，可以通过删除任何与分区相关的引用，或者将存储引擎指定为`InnoDB`或允许其默认设置为`InnoDB`来实现。

    注意

    在第 3.6 节，“准备升级安装”中描述了如何在升级到 MySQL 8.0 之前识别必须更改的分区表的过程。

    有关更多信息，请参阅第 26.6.2 节，“与存储引擎相关的分区限制”。

+   **不兼容更改**：几个服务器错误代码未被使用并已被移除（详细列表请参见 MySQL 8.0 中删除的功能）。特别测试任何这些错误代码的应用程序应该进行更新。

+   **重要更改**：默认字符集已从`latin1`更改为`utf8mb4`。这些系统变量受到影响：

    +   `character_set_server`和`character_set_database`系统变量的默认值已从`latin1`更改为`utf8mb4`。

    +   `collation_server`和`collation_database`系统变量的默认值已从`latin1_swedish_ci`更改为`utf8mb4_0900_ai_ci`。

    因此，除非显式指定字符集和校对规则，否则新对象的默认字符集和校对规则与以前不同。这包括数据库及其中的对象，如表、视图和存储过程。假设以前使用的是默认值，保留它们的一种方法是在`my.cnf`文件中使用以下行启动服务器：

    ```sql
    [mysqld]
    character_set_server=latin1
    collation_server=latin1_swedish_ci
    ```

    在复制设置中，从 MySQL 5.7 升级到 8.0 时，建议在升级之前将默认字符集更改回 MySQL 5.7 中使用的字符集。升级完成后，可以将默认字符集更改为`utf8mb4`。

    另外，您应该知道 MySQL 8.0 强制执行对给定字符集中允许字符的检查，而 MySQL 5.7 不执行；这是一个已知问题。这意味着，在尝试升级之前，您应确保没有注释包含未在使用的字符集中定义的字符。您可以通过以下两种方式修复此问题：

    +   将字符集更改为包含相关字符的字符集。

    +   删除有问题的字符或字符。

    上述适用于表、文件和索引注释。

+   **不兼容更改**：从 MySQL 8.0.11 开始，禁止使用与服务器初始化时使用的`lower_case_table_names`设置不同的设置启动服务器。这个限制是必要的，因为各种数据字典表字段使用的校对是基于服务器初始化时定义的`lower_case_table_names`设置，重新启动服务器使用不同设置会导致标识符的排序和比较出现不一致。

### 服务器更改

+   在 MySQL 8.0.11 中，已删除了与帐户管理相关的几个弃用功能，例如使用`GRANT`语句修改用户帐户的非权限特性，`NO_AUTO_CREATE_USER` SQL 模式，`PASSWORD()`函数和`old_passwords`系统变量。

    从 MySQL 5.7 复制到 8.0 的涉及这些已移除功能的语句可能导致复制失败。使用任何已移除功能的应用程序应进行修订以避免使用它们，并在可能的情况下使用替代方案，如 MySQL 8.0 中已移除的功能中所述。

    为避免在 MySQL 8.0 上启动失败，请从 MySQL 选项文件的`sql_mode`系统变量设置中删除任何`NO_AUTO_CREATE_USER`实例。

    将包含`NO_AUTO_CREATE_USER` SQL 模式的转储文件加载到 MySQL 8.0 服务器中会导致失败。从 MySQL 5.7.24 和 MySQL 8.0.13 开始，**mysqldump**会从存储程序定义中删除`NO_AUTO_CREATE_USER`。使用早期版本的`mysqldump`创建的转储文件必须手动修改以删除`NO_AUTO_CREATE_USER`的实例。

+   在 MySQL 8.0.11 中，这些已弃用的兼容性 SQL 模式被移除：`DB2`、`MAXDB`、`MSSQL`、`MYSQL323`、`MYSQL40`、`ORACLE`、`POSTGRESQL`、`NO_FIELD_OPTIONS`、`NO_KEY_OPTIONS`、`NO_TABLE_OPTIONS`。它们不能再分配给`sql_mode`系统变量，也不能作为**mysqldump** `--compatible`选项的允许值。

    移除`MAXDB`意味着`CREATE TABLE`或`ALTER TABLE`中的`TIMESTAMP`数据类型不再被视为`DATETIME`。

    从 MySQL 5.7 到 8.0 的复制中，涉及已移除 SQL 模式的语句可能导致复制失败。这包括在当前`sql_mode`值包含任何已移除模式的情况下执行的存储程序（存储过程和函数、触发器和事件）的`CREATE`语句的复制。使用任何已移除模式的应用程序应进行修改以避免使用它们。

+   许多 MySQL 8.0 错误消息的文本已经修订和改进，提供的信息比 MySQL 5.7 更多更好。如果您的应用程序依赖于特定内容或格式的错误消息，您应该测试这些内容，并准备在升级之前更新应用程序。

+   截至 MySQL 8.0.3，空间数据类型允许一个`SRID`属性，明确指示存储在列中的值的空间参考系统（SRS）。参见第 13.4.1 节，“空间数据类型”。

    具有显式`SRID`属性的空间列是 SRID 受限制的：该列仅接受具有该 ID 的值，并且该列上的`SPATIAL`索引成为优化器使用的对象。优化器会忽略没有`SRID`属性的空间列上的`SPATIAL`索引。参见第 10.3.3 节，“空间索引优化”。如果您希望优化器考虑对没有 SRID 限制的空间列上的`SPATIAL`索引，应修改每个这样的列：

    +   验证列中的所有值是否具有相同的 SRID。要确定几何列*`col_name`*中包含的 SRIDs，使用以下查询：

        ```sql
        SELECT DISTINCT ST_SRID(*col_name*) FROM *tbl_name*;
        ```

        如果查询返回多行，则该列包含混合的 SRIDs。在这种情况下，修改其内容使所有值具有相同的 SRID。

    +   重新定义列以具有显式`SRID`属性。

    +   重新创建`SPATIAL`索引。

+   由于空间函数命名空间更改实施了执行精确操作的函数的`ST_`前缀，或者基于最小边界矩形执行操作的函数的`MBR`前缀，因此在 MySQL 8.0.0 中删除了几个空间函数。在生成列定义中使用已删除的空间函数可能会导致升级失败。在升级之前，运行**mysqlcheck --check-upgrade**以查找已删除的空间函数，并用它们的`ST_`或`MBR`命名的替代函数替换任何找到的函数。有关已删除的空间函数列表，请参考 MySQL 8.0 中删除的功能。

+   当进行就地升级到 MySQL 8.0.3 或更高版本时，具有`RELOAD`权限的用户会自动被授予`BACKUP_ADMIN`权限。

+   从 MySQL 8.0.13 开始，由于基于行或混合复制模式与基于语句的复制模式在处理临时表的方式上的差异，切换二进制日志格式在运行时有了新的限制。

    +   如果会话有任何打开的临时表，则不能使用`SET @@SESSION.binlog_format`。

    +   如果任何复制通道有任何打开的临时表，则不能使用`SET @@global.binlog_format`和`SET @@persist.binlog_format`。如果复制通道有打开的临时表，则允许使用`SET @@persist_only.binlog_format`，因为与`PERSIST`不同，`PERSIST_ONLY`不会修改运行时全局系统变量值。

    +   如果任何复制通道应用程序正在运行，则不能使用`SET @@global.binlog_format`和`SET @@persist.binlog_format`。这是因为更改仅在复制通道的应用程序重新启动时生效，此时复制通道可能有打开的临时表。这种行为比以前更为严格。如果任何复制通道应用程序正在运行，则允许使用`SET @@persist_only.binlog_format`。

    +   从 MySQL 8.0.27 开始，为`internal_tmp_mem_storage_engine`配置会话设置需要`SESSION_VARIABLES_ADMIN`或`SYSTEM_VARIABLES_ADMIN`权限。

    +   截至 MySQL 8.0.27 版本，克隆插件允许在捐赠者 MySQL 服务器实例上进行并发 DDL 操作，同时克隆操作正在进行中。以前，在克隆操作期间会持有备份锁，阻止捐赠者上的并发 DDL。要恢复到在克隆操作期间阻止捐赠者上的并发 DDL 的先前行为，请启用 `clone_block_ddl` 变量。参见 第 7.6.7.4 节，“克隆和并发 DDL”。

+   从 MySQL 8.0.30 版本开始，在启动时列出的 `log_error_services` 值中的错误日志组件将在 MySQL 服务器启动序列的早期隐式加载。如果您以前使用 `INSTALL COMPONENT` 安装了可加载的错误日志组件，并且您在启动时列出了这些组件（例如，从选项文件中读取的 `log_error_services` 设置），则应更新配置以避免启动警告。有关更多信息，请参见 错误日志配置方法。

### InnoDB 更改

+   基于 `InnoDB` 系统表的 `INFORMATION_SCHEMA` 视图已被内部数据字典表上的内部系统视图所取代。受影响的 `InnoDB` `INFORMATION_SCHEMA` 视图已更名为：

    **表 3.1 重命名的 InnoDB 信息模式视图**

    | 旧名称 | 新名称 |
    | --- | --- |
    | `INNODB_SYS_COLUMNS` | `INNODB_COLUMNS` |
    | `INNODB_SYS_DATAFILES` | `INNODB_DATAFILES` |
    | `INNODB_SYS_FIELDS` | `INNODB_FIELDS` |
    | `INNODB_SYS_FOREIGN` | `INNODB_FOREIGN` |
    | `INNODB_SYS_FOREIGN_COLS` | `INNODB_FOREIGN_COLS` |
    | `INNODB_SYS_INDEXES` | `INNODB_INDEXES` |
    | `INNODB_SYS_TABLES` | `INNODB_TABLES` |
    | `INNODB_SYS_TABLESPACES` | `INNODB_TABLESPACES` |
    | `INNODB_SYS_TABLESTATS` | `INNODB_TABLESTATS` |
    | `INNODB_SYS_VIRTUAL` | `INNODB_VIRTUAL` |
    | 旧名称 | 新名称 |

    升级到 MySQL 8.0.3 或更高版本后，请更新任何引用先前 `InnoDB` `INFORMATION_SCHEMA` 视图名称的脚本。

+   MySQL 捆绑的 zlib 库版本从 1.2.3 版升级到 1.2.11 版。

    zlib 1.2.11 中的 zlib `compressBound()`函数返回的压缩给定长度字节所需的缓冲区大小的估计比 zlib 版本 1.2.3 中的估计略高。`compressBound()`函数由确定在创建压缩的`InnoDB`表或在压缩的`InnoDB`表中插入和更新行时允许的最大行大小的`InnoDB`函数调用。因此，与早期版本中成功的最大行大小非常接近的`CREATE TABLE ... ROW_FORMAT=COMPRESSED`，`INSERT`和`UPDATE`操作现在可能会失败。为避免此问题，在升级之前，在 MySQL 8.0 测试实例上测试具有大行的压缩`InnoDB`表的`CREATE TABLE`语句。

+   随着`--innodb-directories`功能的引入，使用绝对路径创建的文件表和通用表空间文件或位于数据目录之外的位置应添加到`innodb_directories`参数值中。否则，在恢复过程中，`InnoDB`将无法定位这些文件。要查看表空间文件位置，请查询 Information Schema `FILES`表：

    ```sql
    SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES \G
    ```

+   撤销日志不再可以驻留在系统表空间中。在 MySQL 8.0 中，默认情况下，撤销日志驻留在两个 undo 表空间中。有关更多信息，请参见第 17.6.3.4 节，“Undo Tablespaces”。

    当从 MySQL 5.7 升级到 MySQL 8.0 时，MySQL 5.7 实例中存在的任何 undo 表空间都将被删除，并由两个新的默认 undo 表空间替换。默认的 undo 表空间是在`innodb_undo_directory`变量定义的位置创建的。如果`innodb_undo_directory`变量未定义，则 undo 表空间将在数据目录中创建。从 MySQL 5.7 升级到 MySQL 8.0 需要进行缓慢关闭，以确保 MySQL 5.7 实例中的 undo 表空间为空，从而可以安全地删除它们。

    从较早的 MySQL 8.0 版本升级到 MySQL 8.0.14 或更高版本时，作为 `innodb_undo_tablespaces` 设置大于 2 的结果存在于升级前实例中的撤销表空间被视为用户定义的撤销表空间，可以在升级后使用 `ALTER UNDO TABLESPACE` 和 `DROP UNDO TABLESPACE` 语法分别停用和删除。在 MySQL 8.0 版本系列内进行升级可能不总是需要慢速关闭，这意味着现有的撤销表空间可能包含撤销日志。因此，升级过程不会删除现有的撤销表空间。

+   **不兼容更改**：从 MySQL 8.0.17 开始，`CREATE TABLESPACE ... ADD DATAFILE` 子句不允许循环目录引用。例如，以下语句中的循环目录引用 (`/../`) 是不允许的：

    ```sql
    CREATE TABLESPACE ts1 ADD DATAFILE ts1.ibd '*any_directory*/../ts1.ibd';
    ```

    在 Linux 上存在对限制的例外情况，如果前面的目录是符号链接，则允许循环目录引用。例如，如果上面示例中的数据文件路径是 *`any_directory`* 是符号链接，则允许。 (数据文件路径仍然可以以 '`../`' 开头。)

    为避免升级问题，在升级到 MySQL 8.0.17 或更高版本之前，请从表空间数据文件路径中删除任何循环目录引用。要检查表空间路径，请查询信息模式 `INNODB_DATAFILES` 表。

+   由于 MySQL 8.0.14 引入的一个回归，从 MySQL 5.7 或 MySQL 8.0.14 之前的 MySQL 8.0 版本在区分大小写文件系统上进行原地升级到 MySQL 8.0.16 时，对于具有分区表和 `lower_case_table_names=1` 的实例会失败。失败是由于与分区表文件名相关的大小写不匹配问题引起的。导致回归的修复已被撤销，这允许从 MySQL 5.7 或 MySQL 8.0.14 之前的 MySQL 8.0 版本升级到 MySQL 8.0.17 以正常运行。然而，回归仍然存在于 MySQL 8.0.14、8.0.15 和 8.0.16 版本中。

    在从 MySQL 8.0.14、8.0.15 或 8.0.16 升级到 MySQL 8.0.17 的区分大小写文件系统上进行原地升级时，如果存在分区表且 `lower_case_table_names=1`，在升级二进制文件或软件包到 MySQL 8.0.17 后启动服务器时会出现以下错误：

    ```sql
    Upgrading from server version *version_number* with
    partitioned tables and lower_case_table_names == 1 on a case sensitive file
    system may cause issues, and is therefore prohibited. To upgrade anyway, restart
    the new server version with the command line option 'upgrade=FORCE'. When
    upgrade is completed, please execute 'RENAME TABLE *part_table_name*
    TO *new_table_name*; RENAME TABLE *new_table_name*
    TO *part_table_name*;' for each of the partitioned tables.
    Please see the documentation for further information.
    ```

    如果在升级到 MySQL 8.0.17 时遇到此错误，请执行以下解决方法：

    1.  使用 `--upgrade=force` 重新启动服务器以强制进行升级操作。

    1.  识别具有小写分区名称分隔符 `(#p#` 或 `#sp#`）的分区表文件名：

        ```sql
        mysql> SELECT FILE_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_NAME LIKE '%#p#%' OR FILE_NAME LIKE '%#sp#%';
        ```

    1.  对于每个识别的文件，将相关表重命名为临时名称，然后将表重新命名为原始名称。

        ```sql
        mysql> RENAME TABLE *table_name* TO *temporary_table_name*;
        mysql> RENAME TABLE *temporary_table_name* TO *table_name*;
        ```

    1.  确保没有具有小写分区名称分隔符的分区表文件名（应返回空结果集）。

        ```sql
        mysql> SELECT FILE_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_NAME LIKE '%#p#%' OR FILE_NAME LIKE '%#sp#%';
        Empty set (0.00 sec)
        ```

    1.  在每个重命名的表上运行 `ANALYZE TABLE` 以更新 `mysql.innodb_index_stats` 和 `mysql.innodb_table_stats` 表中的优化器统计信息。

    由于 MySQL 8.0.14、8.0.15 和 8.0.16 版本中仍存在的回归，从 MySQL 8.0.14、8.0.15 或 8.0.16 导入分区表到 MySQL 8.0.17 在 `lower_case_table_names=1` 的区分大小写文件系统上不受支持。尝试这样做会导致“表空间缺失于表”错误。

+   MySQL 在构建表分区的表空间名称和文件名时使用分隔符字符串。一个“ `#p#` ”分隔符字符串位于分区名称之前，一个“ `#sp#` ”分隔符字符串位于子分区名称之前，如下所示：

    ```sql
     *schema_name*.*table_name*#p#*partition_name*#sp#*subpartition_name*
          *table_name*#p#*partition_name*#sp#*subpartition_name*.ibd
    ```

    历史上，在诸如 Linux 等区分大小写的文件系统上，分隔符字符串为大写（`#P#` 和 `#SP#`），而在诸如 Windows 等不区分大小写的文件系统上为小写（`#p#` 和 `#sp#`）。从 MySQL 8.0.19 开始，分隔符字符串在所有文件系统上都为小写。此更改可防止在区分大小写和不区分大小写的文件系统之间迁移数据目录时出现问题。不再使用大写分隔符字符串。

    另外，基于用户指定的分区或子分区名称生成的分区表空间名称和文件名，无论 `lower_case_table_names` 设置如何，现在都会生成（并在内部存储）为小写，以确保不区分大小写。例如，如果创建了一个名为 `PART_1` 的表分区，则表空间名称和文件名将以小写形式生成：

    ```sql
     *schema_name*.*table_name*#p#*part_1*
          *table_name*#p#*part_1*.ibd
    ```

    在升级期间，MySQL 检查并根据需要修改：

    +   在磁盘上和数据字典中识别分区表文件名，以确保小写分隔符和分区名称。

    +   数据字典中的分区元数据，以解决之前 bug 修复引入的相关问题。

    +   `InnoDB` 统计数据用于之前 bug 修复引入的相关问题。

    在表空间导入操作期间，会检查并根据需要修改磁盘上的分区表空间文件名，以确保小写分隔符和分区名称。

+   从 MySQL 8.0.21 开始，在启动时或从 MySQL 5.7 升级时，如果发现表空间数据文件位于未知目录中，将向错误日志写入警告。已知目录是由`datadir`、`innodb_data_home_dir`和`innodb_directories`变量定义的目录。要使目录成为已知目录，请将其添加到`innodb_directories`设置中。使目录成为已知目录可确保在恢复期间可以找到数据文件。有关更多信息，请参见崩溃恢复期间的表空间发现。

+   **重要变更**：从 MySQL 8.0.30 开始，`innodb_redo_log_capacity`变量控制重做日志文件占用的磁盘空间量。随着这一变更，默认的重做日志文件数量和位置也发生了变化。从 MySQL 8.0.30 开始，`InnoDB`在数据目录中的`#innodb_redo`目录中维护 32 个重做日志文件。以前，`InnoDB`默认在数据目录中创建两个重做日志文件，并且重做日志文件的数量和大小由`innodb_log_files_in_group`和`innodb_log_file_size`变量控制。这两个变量现已弃用。

    当定义了`innodb_redo_log_capacity`设置时，将忽略`innodb_log_files_in_group`和`innodb_log_file_size`设置；否则，将使用这些设置来计算`innodb_redo_log_capacity`设置（`innodb_log_files_in_group` * `innodb_log_file_size` = `innodb_redo_log_capacity`）。如果这些变量都没有设置，则重做日志容量将设置为`innodb_redo_log_capacity`的默认值，即 104857600 字节（100MB）。

    与任何升级一样，此更改在升级之前需要干净的关闭。

    有关此功能的更多信息，请参见第 17.6.5 节，“重做日志”。

+   在 MySQL 5.7.35 之前，具有冗余或紧凑行格式的表中的索引没有大小限制。从 MySQL 5.7.35 开始，限制为 767 字节。从 MySQL 5.7.35 之前的 MySQL 版本升级到 MySQL 8.0 可能会导致无法访问的表。如果具有冗余或紧凑行格式的表的索引大于 767 字节，请在升级到 MySQL 8.0 之前删除索引并重新创建。错误消息为：

    ```sql
    mysql> ERROR 1709 (HY000): Index column size too large. The maximum column size is 767 bytes.
    ```

### SQL 变更

+   **不兼容更改**：从 MySQL 8.0.13 开始，已删除了对 `GROUP BY` 子句的已弃用 `ASC` 或 `DESC` 修饰符。先前依赖于 `GROUP BY` 排序的查询可能会产生与先前 MySQL 版本不同的结果。为了产生给定的排序顺序，请提供一个 `ORDER BY` 子句。

    从 MySQL 8.0.12 或更低版本中使用 `ASC` 或 `DESC` 修饰符进行 `GROUP BY` 子句的查询和存储程序定义应进行修改。否则，升级到 MySQL 8.0.13 或更高版本可能会失败，复制到 MySQL 8.0.13 或更高版本的复制服务器也可能失败。

+   在 MySQL 8.0 中可能会保留一些在 MySQL 5.7 中未保留的关键字。请参阅第 11.3 节，“关键字和保留字”。这可能导致先前用作标识符的单词变得非法。要修复受影响的语句，请使用标识符引用。请参阅第 11.2 节，“模式对象名称”。

+   升级后，建议测试应用程序代码中指定的优化器提示，以确保这些提示仍然需要实现所需的优化策略。优化器增强有时可能使某些优化器提示变得不必要。在某些情况下，不必要的优化器提示甚至可能适得其反。

+   **不兼容更改**：在 MySQL 5.7 中，为 `InnoDB` 表指定 `FOREIGN KEY` 定义而不带 `CONSTRAINT *`symbol`*` 子句，或者指定 `CONSTRAINT` 关键字而不带 `symbol`，会导致 `InnoDB` 使用生成的约束名。在 MySQL 8.0 中，`InnoDB` 的行为发生了变化，使用 `FOREIGN KEY *`index_name`*` 值而不是生成的名称。由于约束名必须在模式（数据库）中唯一，这种更改导致由于外键索引名称在模式中不唯一而导致错误。为避免此类错误，新的约束命名行为已在 MySQL 8.0.16 中恢复，`InnoDB` 再次使用生成的约束名。

    为了与 `InnoDB` 保持一致，基于 MySQL 8.0.16 或更高版本的 `NDB` 发行版如果未指定 `CONSTRAINT *`symbol`*` 子句，或者指定 `CONSTRAINT` 关键字而不带 `symbol`，则使用生成的约束名。基于 MySQL 5.7 和早期 MySQL 8.0 发行版的 `NDB` 发行版使用 `FOREIGN KEY *`index_name`*` 值。

    上述描述的更改可能会对依赖于先前外键约束命名行为的应用程序造成不兼容性。

+   MySQL 8.0 中的系统变量值处理方式已更改，例如 `IFNULL()` 和 `CASE()` 等 MySQL 流控制函数；现在系统变量值被视为相同字符和排序规则的列值，而不是常量。一些使用这些函数与系统变量的查询可能会被拒绝，出现 Illegal mix of collations。在这种情况下，将系统变量转换为正确的字符集和排序规则。

+   **不兼容的更改**：MySQL 8.0.28 修复了先前 MySQL 8.0 版本中的问题，即 `CONVERT()` 函数有时允许将 `BINARY` 值无效地转换为非二进制字符集。可能依赖于此行为的应用程序应在升级之前进行检查，并在必要时进行修改。

    特别是，在索引生成列的表达式中使用 `CONVERT()`，函数行为的更改可能导致在升级到 MySQL 8.0.28 之后索引损坏。您可以通过以下步骤防止这种情况发生：

    1.  在执行升级之前，纠正任何无效的输入数据。

    1.  删除然后重新创建索引。

        您还可以使用 `ALTER TABLE *`table`* FORCE` 强制重建表。

    1.  升级 MySQL 软件。

    如果您无法事先验证输入数据，则在升级到 MySQL 8.0.28 之后，不应重新创建索引或重建表。

### 更改的服务器默认值

MySQL 8.0 提供了改进的默认值，旨在为大多数用户提供最佳的开箱即用体验。这些变化是由于技术的进步（机器拥有更多 CPU，使用 SSD 等），存储的数据更多，MySQL 正在发展（InnoDB，Group Replication，AdminAPI）等。以下表格总结了已更改的默认值，以为大多数用户提供最佳的 MySQL 体验。

| 选项/参数 | 旧默认值 | 新默认值 |
| --- | --- | --- |
| *服务器更改* |  |  |
| `character_set_server` | latin1 | utf8mb4 |
| `collation_server` | latin1_swedish_ci | utf8mb4_0900_ai_ci |
| `explicit_defaults_for_timestamp` | OFF | ON |
| `optimizer_trace_max_mem_size` | 16KB | 1MB |
| `validate_password_check_user_name` | OFF | ON |
| `back_log` | -1 (autosize) changed from : back_log = 50 + (max_connections / 5) | -1 (autosize) changed to : back_log = max_connections |
| `max_allowed_packet` | 4194304 (4MB) | 67108864 (64MB) |
| `max_error_count` | 64 | 1024 |
| `event_scheduler` | OFF | ON |
| `table_open_cache` | 2000 | 4000 |
| `log_error_verbosity` | 3 (Notes) | 2 (Warning) |
| `local_infile` | ON (5.7) | OFF |
| *InnoDB 更改* |  |  |
| `innodb_undo_tablespaces` | 0 | 2 |
| `innodb_undo_log_truncate` | OFF | ON |
| `innodb_flush_method` | NULL | fsync (Unix), unbuffered (Windows) |
| `innodb_autoinc_lock_mode` | 1 (consecutive) | 2 (interleaved) |
| `innodb_flush_neighbors` | 1 (enable) | 0 (disable) |
| `innodb_max_dirty_pages_pct_lwm` | 0 (%) | 10 (%) |
| `innodb_max_dirty_pages_pct` | 75 (%) | 90 (%) |
| *性能模式更改* |  |  |
| `performance-schema-instrument='wait/lock/metadata/sql/%=ON'` | OFF | ON |
| `performance-schema-instrument='memory/%=COUNTED'` | OFF | COUNTED |
| `performance-schema-consumer-events-transactions-current=ON` | OFF | ON |
| `performance-schema-consumer-events-transactions-history=ON` | OFF | ON |
| `performance-schema-instrument='transaction%=ON'` | OFF | ON |
| *复制更改* |  |  |
| `log_bin` | OFF | ON |
| `server_id` | 0 | 1 |
| `log-slave-updates` | OFF | ON |
| `expire_logs_days` | 0 | 30 |
| `master-info-repository` | FILE | TABLE |
| `relay-log-info-repository` | FILE | TABLE |
| `transaction-write-set-extraction` | OFF | XXHASH64 |
| `slave_rows_search_algorithms` | INDEX_SCAN, TABLE_SCAN | INDEX_SCAN, HASH_SCAN |
| `slave_pending_jobs_size_max` | 16M | 128M |
| `gtid_executed_compression_period` | 1000 | 0 |
| *组复制更改* |  |  |
| `group_replication_autorejoin_tries` | 0 | 3 |
| `group_replication_exit_state_action` | ABORT_SERVER | READ_ONLY |
| `group_replication_member_expel_timeout` | 0 | 5 |
| 选项/参数 | 旧默认值 | 新默认值 |

有关已添加的选项或变量的更多信息，请参阅 MySQL 8.0 的选项和变量更改，在*MySQL 服务器版本参考*中。

以下部分解释了默认值的更改以及它们可能对您的部署产生的影响。

**服务器默认值**

+   `character_set_server`系统变量和命令行选项`--character-set-server`的默认值从`latin1`更改为`utf8mb4`。这是服务器的默认字符集。目前，UTF8MB4 是网络的主要字符编码，这一变化使得绝大多数 MySQL 用户的生活更加便利。从 5.7 升级到 8.0 不会更改任何现有数据库对象的字符集，但是，除非您明确设置`character_set_server`（要么回到以前的值，要么设置为新值），否则新模式默认使用`utf8mb4`。我们建议尽可能迁移到`utf8mb4`。

+   `collation_server`系统变量和命令行参数`--collation-server`的默认值从`latin1_swedish_ci`更改为`utf8mb4_0900_ai_ci`。这是服务器的默认排序规则，即字符集中字符的排序。排序规则和字符集之间存在链接，因为每个字符集都有可能的排序规则列表。从 5.7 升级到 8.0 不会更改任何现有数据库对象的排序规则，但会影响新对象。

+   `explicit_defaults_for_timestamp`系统变量的默认值从`OFF`（MySQL 传统行为）更改为`ON`（SQL 标准行为）。此选项最初在 5.6 中引入，在 5.6 和 5.7 中为`OFF`。

+   `optimizer_trace_max_mem_size`系统变量的默认值从`16KB`更改为`1MB`。旧默认值会导致对于任何非平凡查询，优化器跟踪被截断。这一变化确保了大多数查询的优化器跟踪是有用的。

+   `validate_password_check_user_name`系统变量的默认值从`OFF`更改为`ON`。这意味着当启用`validate_password`插件时，默认情况下现在拒绝与当前会话用户名匹配的密码。

+   `back_log`系统变量的自动调整算法已更改。自动调整值（-1）现在设置为`max_connections`的值，这比由`50 + (max_connections / 5)`计算的值大。`back_log`在服务器无法跟上传入请求的情况下排队传入的 IP 连接请求。在最坏的情况下，例如在网络故障后，有`max_connections`数量的客户端尝试重新连接时，它们都可以被缓冲，并且避免了拒绝重试循环。

+   `max_allowed_packet`系统变量的默认值从`4194304`（4M）更改为`67108864`（64M）。这个更大的默认值的主要优点是减少接收关于插入或查询大于`max_allowed_packet`的错误的机会。它应该与您想要使用的最大第 13.3.4 节，“BLOB 和 TEXT 类型”一样大。要恢复到以前的行为，请设置`max_allowed_packet=4194304`。

+   `max_error_count`系统变量的默认值从`64`更改为`1024`。这确保了 MySQL 处理更多警告，例如触及数千行并且其中许多行给出转换警告的 UPDATE 语句。许多工具通常会批量更新，以帮助减少复制延迟。外部工具如 pt-online-schema-change 默认为 1000，gh-ost 默认为 100。MySQL 8.0 覆盖了这两种用例的完整错误历史。没有静态分配，因此此更改仅影响生成大量警告的语句的内存消耗。

+   `event_scheduler`系统变量的默认值从`OFF`更改为`ON`。换句话说，默认情况下启用事件调度程序。这是 SYS 中新功能的启用程序，例如“终止空闲事务”。

+   `table_open_cache`系统变量的默认值从`2000`更改为`4000`。这是一个增加表访问会话并发性的微小更改。

+   `log_error_verbosity`系统变量的默认值从`3`（Notes）更改为`2`（Warning）。目的是使 MySQL 8.0 错误日志默认情况下更少冗长。

**InnoDB 默认值**

+   **不兼容的更改** `innodb_undo_tablespaces`系统变量的默认值从`0`更改为`2`。这配置了 InnoDB 使用的撤销表空间的数量。在 MySQL 8.0 中，`innodb_undo_tablespaces`的最小值为 2，回滚段不再可以在系统表空间中创建。因此，这是一个您无法恢复到 5.7 行为的情况。这个更改的目的是能够自动截断 Undo 日志（见下一项），回收像**mysqldump**这样的（偶尔的）长事务使用的磁盘空间。

+   `innodb_undo_log_truncate`系统变量的默认值从`OFF`更改为`ON`。启用后，超过`innodb_max_undo_log_size`定义的阈值的撤销表空间将被标记为截断。只有撤销表空间可以被截断。不支持截断位于系统表空间中的撤销日志。从 5.7 升级到 8.0 会自动将系统转换为使用撤销表空间，8.0 中不再支持使用系统表空间。

+   `innodb_flush_method`系统变量的默认值在 Unix-like 系统上从`NULL`更改为`fsync`，在 Windows 系统上从`NULL`更改为`unbuffered`。这更多是术语和选项的清理，没有任何实质性影响。对于 Unix 来说，这只是一个文档更改，因为默认值在 5.7 中也是`fsync`（默认的`NULL`意味着`fsync`）。同样，在 Windows 上，`innodb_flush_method`默认的`NULL`在 5.7 中意味着`async_unbuffered`，在 8.0 中被默认的`unbuffered`替换，这与现有的默认`innodb_use_native_aio=ON`具有相同的效果。

+   **不兼容的更改** `innodb_autoinc_lock_mode`系统变量的默认值从`1`（连续）更改为`2`（交错）。将交错锁定模式作为默认设置的更改反映了从基于语句到基于行的复制作为默认复制类型的更改，该更改发生在 MySQL 5.7 中。*基于语句的复制*需要连续的自增锁定模式，以确保自增值按照可预测和可重复的顺序分配给给定序列的 SQL 语句，而*基于行的复制*不受 SQL 语句执行顺序的影响。因此，这一更改已知与基于语句的复制不兼容，并可能破坏一些依赖于顺序自增的应用程序或用户生成的测试套件。可以通过设置`innodb_autoinc_lock_mode=1;`来恢复先前的默认值。

+   `innodb_flush_neighbors`系统变量的默认值从`1`（启用）更改为`0`（禁用）。这是因为快速 IO（SSD）现在是部署的默认设置。我们预计对于大多数用户，这将带来轻微的性能提升。使用较慢硬盘的用户可能会看到性能下降，并鼓励他们通过设置`innodb_flush_neighbors=1`来恢复到先前的默认值。

+   `innodb_max_dirty_pages_pct_lwm`系统变量的默认值从`0` (%)更改为`10` (%)。当`innodb_max_dirty_pages_pct_lwm=10`时，InnoDB 在缓冲池中包含修改（'脏'）页面超过 10%时增加其刷新活动。这一变化的目的是略微降低峰值吞吐量，以换取更一致的性能。

+   `innodb_max_dirty_pages_pct`系统变量的默认值从`75` (%)更改为`90` (%)。这一变化与`innodb_max_dirty_pages_pct_lwm`的更改结合在一起，确保 InnoDB 的刷新行为平稳，避免刷新突发。要恢复到先前的行为，设置`innodb_max_dirty_pages_pct=75`和`innodb_max_dirty_pages_pct_lwm=0`。

**性能模式默认值**

+   默认情况下，性能模式元数据锁定（MDL）仪表化默认打开。`performance-schema-instrument='wait/lock/metadata/sql/%=ON'`的编译默认值从`OFF`更改为`ON`。这是为了在 SYS 中添加基于 MDL 的视图的启用器。

+   性能模式内存仪表化默认开启。`performance-schema-instrument='memory/%=COUNTED'`的编译默认值从`OFF`更改为`COUNTED`。这很重要，因为如果在服务器启动后启用了仪表化，会导致计算不正确，可能会因为错过分配而得到负余额，但捕捉到释放。

+   性能模式事务仪表化默认开启。`performance-schema-consumer-events-transactions-current=ON`，`performance-schema-consumer-events-transactions-history=ON`，和`performance-schema-instrument='transaction%=ON'`的编译默认值从`OFF`更改为`ON`。

**复制默认值**

+   `log_bin`系统变量的默认值从`OFF`更改为`ON`。换句话说，默认情况下启用了二进制日志记录。几乎所有的生产安装都启用了二进制日志，因为它用于复制和时间点恢复。因此，默认情况下启用二进制日志可以省去一个配置步骤，稍后启用需要重新启动**mysqld**。默认情况下启用它还提供更好的测试覆盖率，并且更容易发现性能退化。记得也设置`server_id`（见下面的更改）。8.0 的默认行为就好像你发出了`./mysqld --log-bin --server-id=1`。如果你在 8.0 上想要 5.7 的行为，可以发出`./mysqld --skip-log-bin --server-id=0`。

+   `server_id`系统变量的默认值从`0`更改为`1`（与`log_bin=ON`的更改相结合）。服务器可以使用这个默认 ID 启动，但实际上你必须根据部署的复制基础架构设置`server-id`，以避免出现重复的服务器 ID。

+   `log-slave-updates`系统变量的默认值从`OFF`更改为`ON`。这会导致副本将复制的事件记录到自己的二进制日志中。这个选项对于组复制是必需的，并且在各种复制链设置中确保正确的行为，这在今天已经成为常态。

+   `expire_logs_days`系统变量的默认值从`0`更改为`30`。新的默认值`30`会导致**mysqld**定期清理未使用的超过 30 天的二进制日志。这个改变有助于防止过多的磁盘空间被浪费在不再需要用于复制或恢复目的的二进制日志上。旧值`0`禁用了任何自动二进制日志清理。

+   `master_info_repository`和`relay_log_info_repository`系统变量的默认值从`FILE`更改为`TABLE`。因此，在 8.0 中，默认情况下将复制元数据存储在 InnoDB 中。这增加了默认情况下尝试实现崩溃安全复制的可靠性。

+   `transaction-write-set-extraction`系统变量的默认值从`OFF`更改为`XXHASH64`。这个改变默认启用了事务写入集。通过使用事务写入集，源必须稍微增加一些工作来生成写入集，但结果对于冲突检测是有帮助的。这是 Group Replication 的要求，新的默认值使得在源上启用二进制日志写入集并行化变得更容易，以加快复制速度。

+   `slave_rows_search_algorithms`系统变量的默认值从`INDEX_SCAN,TABLE_SCAN`更改为`INDEX_SCAN,HASH_SCAN`。这个改变通过减少复制应用程序必须执行的表扫描次数来加快基于行的复制速度，以应用对没有主键的表的更改。

+   `slave_pending_jobs_size_max`系统变量的默认值从`16M`更改为`128M`。这个改变增加了可用于多线程复制的内存量。

+   `gtid_executed_compression_period`系统变量的默认值从`1000`更改为`0`。这个改变确保`mysql.gtid_executed`表的压缩只在需要时隐含发生。

**Group Replication Defaults**

+   `group_replication_autorejoin_tries`的默认值从 0 更改为 3，这意味着默认情况下启用了自动重新加入。这个系统变量指定成员在被驱逐或在达到`group_replication_unreachable_majority_timeout`设置之前无法联系到大多数组时，尝试自动重新加入组的次数。

+   `group_replication_exit_state_action`的默认值从`ABORT_SERVER`更改为`READ_ONLY`。这意味着当一个成员退出组时，例如在网络故障后，实例将变为只读，而不是被关闭。

+   `group_replication_member_expel_timeout` 的默认值从 0 更改为 5，这意味着在 5 秒检测期之后，被怀疑与组失去联系的成员将在 5 秒后被驱逐。

这些默认值大多对开发和生产环境都是合理的。但有一个例外，我们决定保持名为 `innodb_dedicated_server` 的新选项设置为 `OFF`，尽管我们建议在生产环境中将其设置为 `ON`。默认设置为 `OFF` 的原因是它会导致共享环境（如开发人员笔记本电脑）无法使用，因为它会占用 *所有* 可用内存。

对于生产环境，我们建议将 `innodb_dedicated_server` 设置为 `ON`。当设置为 `ON` 时，以下 InnoDB 变量（如果没有明确指定）将根据可用内存进行自动缩放 `innodb_buffer_pool_size`、`innodb_log_file_size` 和 `innodb_flush_method`。请参阅 第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

尽管新的默认值是大多数用例的最佳配置选择，但也存在特殊情况，以及出于遗留原因使用现有的 5.7 配置选择。例如，有些人希望尽可能少地更改他们的应用程序或运行环境来升级到 8.0。我们建议评估所有新的默认值，并尽可能多地使用它们。大多数新的默认值可以在 5.7 中进行测试，因此您可以在升级到 8.0 之前在 5.7 生产环境中验证新的默认值。对于您需要保留旧的 5.7 值的少数默认值，请在您的运行环境中设置相应的配置变量或启动选项。

MySQL 8.0 拥有性能模式 `variables_info` 表，显示了每个系统变量最近设置的来源，以及其值范围。这提供了关于配置变量及其值的所有信息的 SQL 访问。

### 有效的性能退化

在 MySQL 版本 5.7 和 8.0 之间预期会出现性能退化。MySQL 8.0 拥有更多功能，改变了默认值，更加稳健，并增加了安全功能和额外的诊断信息。以下列出了这些版本之间出现性能退化的有效原因，包括潜在的调解选项。这并非详尽列表。

MySQL 版本 5.7 和 8.0 之间默认值更改相关的变化：

+   在 5.7 中，默认情况下禁用二进制日志记录，而在 8.0 中默认启用。

    *调解*：通过在启动时指定`--skip-log-bin`或`--disable-log-bin`选项来禁用二进制日志记录。

+   默认字符集从`latin1`更改为`utf8mb4`在 8.0 中。虽然`utf8mb4`在 8.0 中表现比 5.7 好得多，但`latin1`比`utf8mb4`更快。

    *调解*：如果不需要`utf8mb4`，在 8.0 中使用`latin1`。

事务数据字典（原子 DDL）在 8.0 中引入。

+   这增加了鲁棒性/可靠性，但以 DDL 性能（CREATE / DROP 密集负载）为代价，但不应影响 DML 负载（SELECT / INSERT / UPDATE / DELETE）。

    *调解*：无

从 5.7.28 开始使用的更现代的 TLS 密码/算法在启用 TLS（SSL）时产生影响（默认情况下）：

+   在 MySQL 5.7.28 之前，MySQL 社区版使用 yaSSL 库，企业版使用 OpenSSL。

    从 MySQL 5.7.28 开始，MySQL 仅使用 OpenSSL 及其更强的 TLS 密码，这在性能方面更昂贵。

    从 MySQL 5.7.28 或更早版本升级到 MySQL 8.0 可能会导致 TLS 性能退化。

    *调解*：无（如果出于安全原因需要 TLS）

性能模式（PFS）在 8.0 中比 5.7 更广泛：

+   在 MySQL 8.0 中无法将 PFS 编译取消，但可以关闭。即使关闭了一些性能模式仍将存在，但开销会更小。

    *调解*：在 8.0 中设置 performance_schema = OFF，或者在需要部分但不是全部 PFS 功能时以更细粒度关闭性能模式仪表。

在 8.0 中默认启用截断撤消表空间，这可能会显著影响性能：

+   从历史上看，InnoDB 将撤消日志存储在系统表空间中，但无法回收撤消日志使用的空间。系统表空间只会增长而不会缩小，这激发了相关功能请求以解决此问题。

    MySQL 8.0 将撤消日志移至单独的表空间，从而允许手动和自动撤消日志截断。

    然而，自动截断会带来永久性能开销，并且可能导致停顿。

    *调解*：在 8.0 中设置 innodb_undo_log_truncate = OFF，并根据需要手动截断撤消日志。有关相关信息，请参阅截断撤消表空间。

字符类`[[:alpha:]]`或`[[:digit:]]`在 MySQL 8.0 中的正则表达式函数（如`REGEXP()`和`RLIKE()`）中的性能不如在 MySQL 5.7 中表现得好。这是因为 MySQL 8.0 中用 ICU 库替换了 Spencer 正则表达式库，ICU 库在内部使用 UTF-16。

*中介*：在`[[:alpha:]]`的位置使用`[a-zA-Z]`；在`[[:digit:]]`的位置使用`[0-9]`。
