> 原文：[`dev.mysql.com/doc/refman/8.0/en/persisted-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/persisted-system-variables.html)

#### 7.1.9.3 持久化系统变量

MySQL 服务器维护配置其操作的系统变量。系统变量可以具有影响整个服务器操作的全局值，影响当前会话的会话值，或两者都有。许多系统变量是动态的，可以使用`SET`语句在运行时更改，以影响当前服务器实例的操作。`SET`也可以用于将某些全局系统变量持久化到数据目录中的`mysqld-auto.cnf`文件中，以影响后续启动时的服务器操作。`RESET PERSIST`会从`mysqld-auto.cnf`中删除持久化的设置。

以下讨论描述了持久化系统变量的方面：

+   持久化系统变量概述

+   持久化系统变量的语法

+   获取有关持久化系统变量的信息

+   mysqld-auto.cnf 文件的格式和服务器处理方式

+   持久化敏感系统变量

##### 持久化系统变量概述

在运行时持久化全局系统变量的能力使得服务器配置可以持续跨越服务器启动。尽管许多系统变量可以在启动时从`my.cnf`选项文件设置，或者使用`SET`语句在运行时设置，但这些配置服务器的方法要么需要登录到服务器主机，要么不能提供在运行时或远程持久配置服务器的能力：

+   修改选项文件需要直接访问该文件，这需要登录到 MySQL 服务器主机。这并不总是方便的。

+   使用`SET GLOBAL`修改系统变量是一种运行时功能，可以从本地运行的客户端或远程主机执行，但更改仅影响当前运行的服务器实例。这些设置不是持久的，也不会传递到后续的服务器启动。

为了增强服务器配置的管理能力，超出了通过编辑选项文件或使用`SET GLOBAL`所能实现的范围，MySQL 提供了`SET`语法的变体，将系统变量设置持久化到名为`mysqld-auto.cnf`的文件中。示例：

```sql
SET PERSIST max_connections = 1000;
SET @@PERSIST.max_connections = 1000;

SET PERSIST_ONLY back_log = 100;
SET @@PERSIST_ONLY.back_log = 100;
```

MySQL 还提供了一个`RESET PERSIST`语句，用于从`mysqld-auto.cnf`中删除持久化的系统变量。

通过持久化系统变量执行的服务器配置具有以下特点：

+   持久化设置是在运行时进行的。

+   持久化设置是永久的。它们在服务器重新启动时生效。

+   持久化设置可以来自本地客户端或从远程主机连接的客户端。这提供了从中央客户端主机远程配置多个 MySQL 服务器的便利。

+   要持久化系统变量，您无需登录 MySQL 服务器主机或具有选项文件的文件系统访问权限。持久化设置的能力是通过 MySQL 权限系统控制的。请参见 Section 7.1.9.1, “系统变量权限”。

+   拥有足够权限的管理员可以通过持久化系统变量重新配置服务器，然后通过执行`RESTART`语句立即使服务器使用更改后的设置。

+   持久化设置提供有关错误的即时反馈。手动输入设置中的错误可能要到很久之后才能发现。持久化系统变量的`SET`语句避免了设置格式错误的可能性，因为具有语法错误的设置不会成功，也不会更改服务器配置。

##### 持久化系统变量的语法

这些`SET`语法选项可用于持久化系统变量：

+   要将全局系统变量持久化到数据目录中的`mysqld-auto.cnf`选项文件中，需在变量名之前加上`PERSIST`关键字或`@@PERSIST.`限定符：

    ```sql
    SET PERSIST max_connections = 1000;
    SET @@PERSIST.max_connections = 1000;
    ```

    与`SET GLOBAL`类似，`SET PERSIST`设置全局变量的运行时值，但也将变量设置写入`mysqld-auto.cnf`文件（如果存在任何现有变量设置，则会替换它）。

+   要将全局系统变量持久化到`mysqld-auto.cnf`文件中，而不设置全局变量运行时值，需在变量名之前加上`PERSIST_ONLY`关键字或`@@PERSIST_ONLY.`限定符：

    ```sql
    SET PERSIST_ONLY back_log = 1000;
    SET @@PERSIST_ONLY.back_log = 1000;
    ```

    与 `PERSIST` 类似，`PERSIST_ONLY` 将变量设置写入 `mysqld-auto.cnf`。但是，与 `PERSIST` 不同，`PERSIST_ONLY` 不会修改全局变量的运行时值。这使得 `PERSIST_ONLY` 适用于配置只能在服务器启动时设置的只读系统变量。

有关 `SET` 的更多信息，请参阅 Section 15.7.6.1, “SET Syntax for Variable Assignment”。

这些 `RESET PERSIST` 语法选项可用于移除持久化系统变量：

+   要从 `mysqld-auto.cnf` 中删除所有持久化变量，请使用 `RESET PERSIST` 而不命名任何系统变量：

    ```sql
    RESET PERSIST;
    ```

+   要从 `mysqld-auto.cnf` 中删除特定的持久化变量，请在语句中命名它：

    ```sql
    RESET PERSIST *system_var_name*;
    ```

    这包括插件系统变量，即使插件当前未安装。如果变量不存在于文件中，则会发生错误。

+   要从 `mysqld-auto.cnf` 中删除特定的持久化变量，但如果变量不存在于文件中则产生警告而不是错误，请在先前的语法中添加 `IF EXISTS` 子句：

    ```sql
    RESET PERSIST IF EXISTS *system_var_name*;
    ```

有关 `RESET PERSIST` 的更多信息，请参阅 Section 15.7.8.7, “RESET PERSIST Statement”。

使用 `SET` 将全局系统变量持久化为 `DEFAULT` 值或其字面默认值会将变量分配为其默认值，并向 `mysqld-auto.cnf` 添加变量设置。要从文件中删除变量，请使用 `RESET PERSIST`。

一些系统变量无法持久化。请参阅 Section 7.1.9.4, “Nonpersistible and Persist-Restricted System Variables”。

如果插件在执行 `SET` 语句时已安装，则插件实现的系统变量可以持久化。如果插件仍然安装，则持久化插件变量的分配将在后续服务器重新启动时生效。如果插件不再安装，则当服务器读取 `mysqld-auto.cnf` 文件时插件变量不存在。在这种情况下，服务器会向错误日志写入警告并继续：

```sql
currently unknown variable '*var_name*'
was read from the persisted config file
```

##### 获取有关持久化系统变量的信息

Performance Schema `persisted_variables` 表提供了对 `mysqld-auto.cnf` 文件的 SQL 接口，使得可以使用 `SELECT` 语句在运行时检查其内容。参见 Section 29.12.14.1, “Performance Schema persisted_variables Table”。

Performance Schema `variables_info` 表包含了显示每个系统变量最近由哪个用户何时设置的信息。参见 Section 29.12.14.2, “Performance Schema variables_info Table”。

`RESET PERSIST` 会影响 `persisted_variables` 表的内容，因为表内容对应于 `mysqld-auto.cnf` 文件的内容。另一方面，因为 `RESET PERSIST` 不会更改变量值，所以在服务器重新启动之前，它不会对 `variables_info` 表的内容产生影响。

##### `mysqld-auto.cnf` 文件的格式和服务器处理

`mysqld-auto.cnf` 文件使用类似以下的 `JSON` 格式（稍作整理以提高可读性）：

```sql
{
  "Version": 1,
  "mysql_server": {
    "max_connections": {
      "Value": "152",
      "Metadata": {
        "Timestamp": 1519921341372531,
        "User": "root",
        "Host": "localhost"
      }
    },
    "transaction_isolation": {
      "Value": "READ-COMMITTED",
      "Metadata": {
        "Timestamp": 1519921553880520,
        "User": "root",
        "Host": "localhost"
      }
    },
    "mysql_server_static_options": {
      "innodb_api_enable_mdl": {
        "Value": "0",
        "Metadata": {
          "Timestamp": 1519922873467872,
          "User": "root",
          "Host": "localhost"
        }
      },
      "log_slave_updates": {
        "Value": "1",
        "Metadata": {
          "Timestamp": 1519925628441588,
          "User": "root",
          "Host": "localhost"
        }
      }
    }
  }
}
```

在启动时，服务器在处理所有其他选项文件之后处理 `mysqld-auto.cnf` 文件（参见 Section 6.2.2.2, “使用选项文件”）。服务器处理文件内容如下：

+   如果 `persisted_globals_load` 系统变量已禁用，则服务器会忽略 `mysqld-auto.cnf` 文件。

+   `"mysql_server_static_options"` 部分包含使用 `SET PERSIST_ONLY` 持久化的只读变量。该部分也可能（尽管其名称如此）包含某些不是只读的动态变量。此部分中的所有变量都会附加��命令行并与其他命令行选项一起处理。

+   所有剩余的持久化变量都是通过在服务器开始监听客户端连接之前执行等效的 `SET GLOBAL` 语句来设置的。因此，这些设置直到启动过程的后期才会生效，这对于某些系统变量可能不太合适。在 `my.cnf` 而不是 `mysqld-auto.cnf` 中设置这些变量可能更可取。

`mysqld-auto.cnf`文件的管理应留给服务器处理。文件的操作应仅使用`SET`和`RESET PERSIST`语句执行，而不是手动操作：

+   删除文件会导致在下次服务器启动时丢失所有持久化设置。（如果您的意图是重新配置服务器而不需要这些设置，则可以这样做。）要删除文件中的所有设置而不删除文件本身，请使用以下语句：

    ```sql
    RESET PERSIST;
    ```

+   对文件的手动更改可能会导致服务器启动时出现解析错误。在这种情况下，服务器会报告错误并退出。如果出现此问题，请使用`persisted_globals_load`系统变量禁用或使用`--no-defaults`选项启动服务器。或者，删除`mysqld-auto.cnf`文件。但是，如前所述，删除此文件会导致所有持久化设置丢失。

##### 持久化敏感系统变量

从 MySQL 8.0.29 开始，MySQL 服务器具有安全存储包含私钥或密码等敏感数据的持久化系统变量值的能力，并限制查看这些值。目前没有 MySQL 服务器系统变量被标记为敏感，但新功能允许将包含敏感数据的系统变量在未来安全地持久化。升级到 MySQL 8.0.29 后，`mysqld-auto.cnf`选项文件的格式保持不变，直到首次发出`SET PERSIST`或`SET PERSIST ONLY`语句，此时即使涉及的系统变量不敏感，也会更改为新格式。在新格式中，即使涉及的系统变量不敏感，旧版本的 MySQL 服务器也无法读取选项文件。

注意

MySQL 服务器实例必须启用一个密钥环组件，以支持对持久化系统变量值进行安全存储，而不是使用不支持此功能的密钥环插件。请参阅第 8.4.4 节，“MySQL 密钥环”。

在`mysqld-auto.cnf`选项文件中，敏感系统变量的名称和值以加密格式存储，同时还有一个生成的文件密钥用于解密它们。生成的文件密钥又使用一个存储在密钥环中的主密钥(`persisted_variables_key`)进行加密。服务器启动时，持久化的敏感系统变量会被解密并使用。默认情况下，如果选项文件中存在加密值但在启动时无法成功解密，将使用它们的默认设置。可选的最安全设置会使服务器在无法解密加密值时停止启动。

系统变量`persist_sensitive_variables_in_plaintext`控制服务器是否允许以未加密格式存储敏感系统变量的值，如果在使用`SET PERSIST`设置值时，此时不支持密钥环组件。它还控制服务器是否可以在无法解密加密值时启动。

+   默认设置为`ON`，如果支持密钥环组件，则对值进行加密，并在不支持时以未加密方式持久化（附带警告）。下次设置任何持久化的系统变量时，如果此时支持密钥环，则服务器会加密任何未加密的敏感系统变量的值。`ON`设置还允许服务器在无法解密加密的系统变量值时启动，此时会发出警告并使用系统变量的默认值。在这种情况下，直到可以解密为止，它们的值不能被更改。

+   最安全的设置为`OFF`，意味着如果密钥环组件支持不可用，则敏感系统变量值无法持久化。`OFF`设置还意味着如果无法解密加密的系统变量值，则服务器不会启动。

特权`SENSITIVE_VARIABLES_OBSERVER`允许持有者查看性能模式表`global_variables`、`session_variables`、`variables_by_thread`和`persisted_variables`中敏感系统变量的值，发出`SELECT`语句返回它们的值，并在会话跟踪器中跟踪对它们的更改。没有此特权的用户无法查看或跟踪这些系统变量的值。

如果对敏感系统变量发出`SET`语句，则在记录到一般日志和审计日志之前，查询会被重写以将值替换为“`<redacted>`”。即使服务器实例上没有通过密钥环组件进行安全存储，也会发生这种情况。
