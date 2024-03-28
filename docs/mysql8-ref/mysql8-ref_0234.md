> 原文：[`dev.mysql.com/doc/refman/8.0/en/system-variable-privileges.html`](https://dev.mysql.com/doc/refman/8.0/en/system-variable-privileges.html)

#### 7.1.9.1 系统变量权限

系统变量可以具有影响整个服务器操作的全局值，影响仅当前会话的会话值，或两者：

+   对于动态系统变量，`SET`语句可用于更改它们的全局或会话运行时值（或两者），以影响当前服务器实例的操作。（有关动态变量的信息，请参见第 7.1.9.2 节，“动态系统变量”。）

+   对于某些全局系统变量，可以使用`SET`将它们的值持久化到数据目录中的`mysqld-auto.cnf`文件，以影响后续启动的服务器操作。（有关持久化系统变量和`mysqld-auto.cnf`文件的信息，请参见第 7.1.9.3 节，“持久化系统变量”。）

+   对于持久化的全局系统变量，可以使用`RESET PERSIST`来从`mysqld-auto.cnf`中删除它们的值，以影响后续启动的服务器操作。

本节描述了在运行时为系统变量分配值所需的权限。这包括影响运行时值的操作和持久化值的操作。

要设置全局系统变量，请使用带有适当关键字的`SET`语句。这些权限适用：

+   要设置全局系统变量的运行时值，请使用`SET GLOBAL`语句，需要`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。

+   要将全局系统变量持久化到`mysqld-auto.cnf`文件（并设置运行时值），请使用`SET PERSIST`语句，需要`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限。

+   要将全局系统变量持久化到`mysqld-auto.cnf`文件（而不设置运行时值），使用`SET PERSIST_ONLY`语句，需要`SYSTEM_VARIABLES_ADMIN`和`PERSIST_RO_VARIABLES_ADMIN`特权。`SET PERSIST_ONLY`可用于动态和只读系统变量，但特别适用于持久化只读变量，对于这些变量，不能使用`SET PERSIST`。

+   一些全局系统变量是持久限制的（参见第 7.1.9.4 节，“不可持久化和持久限制的系统变量”）。要持久化这些变量，使用`SET PERSIST_ONLY`语句，需要之前描述的特权。此外，您必须使用加密连接连接到服务器，并提供一个 SSL 证书，其 Subject 值由`persist_only_admin_x509_subject`系统变量指定。

要从`mysqld-auto.cnf`文件中删除持久化的全局系统变量，请使用`RESET PERSIST`语句。这些特权适用：

+   对于动态系统变量，`RESET PERSIST`需要`SYSTEM_VARIABLES_ADMIN`或`SUPER`特权。

+   对于只读系统变量，`RESET PERSIST`需要`SYSTEM_VARIABLES_ADMIN`和`PERSIST_RO_VARIABLES_ADMIN`特权。

+   对于持久限制的变量，`RESET PERSIST`不需要使用特定 SSL 证书建立的加密连接到服务器。

如果全局系统变量有任何对前述特权要求的例外情况，变量描述中会指出这些例外情况。例如，`default_table_encryption`和`mandatory_roles`需要额外的特权。这些额外的特权适用于设置全局运行时值的操作，但不适用于持久化值的操作。

要设置会话系统变量的运行时值，请使用`SET SESSION`语句。与设置全局运行时值相比，设置会话运行时值通常不需要特殊权限，并且可以由任何用户执行以影响当前会话。对于某些系统变量，设置会话值可能会影响当前会话之外的效果，因此这是一项受限制的操作，只能由具有特殊权限的用户执行：

+   从 MySQL 8.0.14 开始，所需的权限是`SESSION_VARIABLES_ADMIN`。

    注意

    任何具有`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限的用户实际上通过暗示具有`SESSION_VARIABLES_ADMIN`权限，因此不需要显式授予`SESSION_VARIABLES_ADMIN`。

+   在 MySQL 8.0.14 之前，所需的权限是`SYSTEM_VARIABLES_ADMIN`或`SUPER`。

如果会话系统变量受限制，变量描述会指出该限制。例如包括`binlog_format`和`sql_log_bin`。设置这些变量的会话值会影响当前会话的二进制日志记录，但也可能对服务器复制和备份的完整性产生更广泛的影响。

`SESSION_VARIABLES_ADMIN`使管理员能够最小化先前可能已被授予`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限的用户的权限足迹，以便使他们能够修改受限制的会话系统变量。假设管理员已创建以下角色以授予设置受限制会话系统变量的能力：

```sql
CREATE ROLE set_session_sysvars;
GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO set_session_sysvars;
```

任何被授予`set_session_sysvars`角色（并且该角色处于活动状态）的用户都能够设置受限制的会话系统变量。但是，该用户也能够设置全局系统变量，这可能是不希望的。

通过修改角色为`SESSION_VARIABLES_ADMIN`而不是`SYSTEM_VARIABLES_ADMIN`，可以将角色权限减少为仅能够设置受限制的会话系统变量而已。要修改角色，请使用以下语句：

```sql
GRANT SESSION_VARIABLES_ADMIN ON *.* TO set_session_sysvars;
REVOKE SYSTEM_VARIABLES_ADMIN ON *.* FROM set_session_sysvars;
```

修改角色会立即生效：任何被授予`set_session_sysvars`角色的账户将不再具有`SYSTEM_VARIABLES_ADMIN`，也无法在未明确授予该权限的情况下设置全局系统变量。类似的`GRANT`/`REVOKE`序列可以应用于任何直接被授予`SYSTEM_VARIABLES_ADMIN`而非通过角色授予的账户。
