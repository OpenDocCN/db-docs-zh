> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-key-migration.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-key-migration.html)

#### 8.4.4.14 在密钥环密钥库之间迁移密钥

密钥环迁移将密钥从一个密钥库复制到另一个，使得 DBA 可以将 MySQL 安装切换到不同的密钥库。成功的迁移操作会产生以下结果：

+   目标密钥库包含迁移前具有的密钥，以及源密钥库中的密钥。

+   在迁移前后，源密钥库保持不变（因为密钥是复制而不是移动）。

如果要复制的密钥已经存在于目标密钥库中，则会发生错误，并且目标密钥库将恢复到迁移前的状态。

密钥环使用密钥环组件和密钥环插件管理密钥库。这与迁移策略有关，因为源密钥库和目标密钥库的管理方式决定了是否可能进行特定类型的密钥迁移以及执行该操作的程序：

+   从一个密钥环插件迁移到另一个：MySQL 服务器具有提供此功能的操作模式。

+   从密钥环插件迁移到密钥环组件：MySQL 服务器在 MySQL 8.0.24 版本中提供了此功能的操作模式。

+   从一个密钥环组件迁移到另一个：**mysql_migrate_keyring** 实用程序提供了这种能力。**mysql_migrate_keyring** 自 MySQL 8.0.24 版本起可用。

+   从密钥环组件迁移到密钥环插件：没有提供此功能。

以下部分讨论了离线和在线迁移的特点，并描述了如何执行迁移。

+   离线和在线密钥迁移

+   使用迁移服务器进行密钥迁移

+   使用 mysql_migrate_keyring 实用程序进行密钥迁移

+   涉及多个运行服务器的密钥迁移

##### 离线和在线密钥迁移

密钥迁移可以是离线或在线的：

+   离线迁移：用于确保本地主机上没有运行中的服务器使用源或目标密钥库时使用。在这种情况下，迁移操作可以将密钥从源密钥库复制到目标密钥库，而在操作期间不会发生运行中服务器修改密钥库内容的可能性。

+   在线迁移：用于当本地主机上运行的服务器正在使用源密钥库时。在这种情况下，必须小心防止服务器在迁移过程中更新密钥库。这涉及连接到运行的服务器并指示其暂停密钥环操作，以便可以安全地从源密钥库复制密钥到目标位置。当密钥复制完成后，允许运行的服务器恢复密钥环操作。

当计划进行密钥迁移时，请使用以下要点来决定是离线还是在线：

+   不要执行涉及正在运行的服务器使用的密钥库的离线迁移。

+   在线迁移过程中暂停密钥环操作是通过连接到运行的服务器并将其全局 `keyring_operations` 系统变量设置为 `OFF` 来实现的，然后在复制密钥之前设置为 `ON`。这有几个影响：

    +   `keyring_operations` 是在 MySQL 5.7.21 中引入的，因此只有运行的服务器是 MySQL 5.7.21 或更高版本时才能进行在线迁移。如果运行的服务器较旧，则必须停止它，执行离线迁移，然后重新启动。所有其他地方提到的与 `keyring_operations` 相关的迁移说明都受到此条件的约束。

    +   用于连接到运行的服务器的帐户必须具有修改 `keyring_operations` 所需的特权。这些特权是 `ENCRYPTION_KEY_ADMIN` 以及 `SYSTEM_VARIABLES_ADMIN` 或已弃用的 `SUPER` 特权。

    +   如果在线迁移操作异常退出（例如，被强制终止），则可能导致 `keyring_operations` 在运行的服务器上保持禁用，导致无法执行密钥环操作。在这种情况下，可能需要连接到运行的服务器并使用以下语句手动启用 `keyring_operations`：

        ```sql
        SET GLOBAL keyring_operations = ON;
        ```

+   在线密钥迁移允许在单个运行的服务器上暂停密钥环操作。如果多个运行的服务器正在使用涉及的密钥库，则使用在 涉及多个运行服务器的密钥迁移 中描述的过程。

##### 使用迁移服务器进行密钥迁移

注意

仅当运行的服务器允许套接字连接或使用 TLS 的 TCP/IP 连接时，才支持使用迁移服务器进行在线密钥迁移；例如，当服务器在 Windows 平台上运行并且仅允许共享内存连接时，不支持此功能。

如果以支持密钥迁移的特殊操作模式调用，则 MySQL 服务器将成为迁移服务器。迁移服务器不接受客户端连接。相反，它只运行足够长的时间来迁移密钥，然后退出。迁移服务器将错误报告给控制台（标准错误输出）。

迁移服务器支持以下迁移类型：

+   从一个密钥环插件迁移到另一个。

+   从密钥环插件迁移到密钥环组件。此功能自 MySQL 8.0.24 版本开始提供。较旧的服务器仅支持从一个密钥环插件迁移到另一个，此时这些指令中涉及密钥环组件的部分不适用。

迁移服务器不支持从一个密钥环组件迁移到另一个密钥环组件。对于这种类型的迁移，请参阅使用 mysql_migrate_keyring 实用程序进行密钥迁移。

要使用迁移服务器执行密钥迁移操作，请确定需要的密钥迁移选项，以指定涉及哪些密钥环插件或组件，以及迁移是离线还是在线的：

+   要指示源密钥环插件和目标密钥环插件或组件，请指定这些选项：

    +   `--keyring-migration-source`: 管理要迁移的密钥的源密钥环插件。

    +   `--keyring-migration-destination`: 要将迁移的密钥复制到的目标密钥环插件或组件。

    +   `--keyring-migration-to-component`: 如果目标是密钥环组件而不是密钥环插件，则需要此选项。

    `--keyring-migration-source` 和 `--keyring-migration-destination` 选项表示服务器应该运行在密钥迁移模式下。对于密钥迁移操作，这两个选项都是必需的。每个插件或组件都使用其库文件的名称指定，包括任何特定于平台的扩展，如`.so`或`.dll`。源和目标必须不同，并且迁移服务器必须支持它们两者。

+   对于离线迁移，不需要额外的密钥迁移选项。

+   对于在线迁移，某个运行服务器当前正在使用源或目标密钥库。要调用迁移服务器，请指定额外的密钥迁移选项，指示如何连接到运行服务器。这是必要的，以便迁移服务器可以连接到运行服务器并告诉其在迁移操作期间暂停密钥环使用。

    使用以下任何选项表示在线迁移：

    +   `--keyring-migration-host`：运行服务器所在的主机。这始终是本地主机，因为迁移服务器只能在本地插件和组件管理的密钥库之间迁移密钥。

    +   `--keyring-migration-user`，`--keyring-migration-password`：用于连接到运行服务器的帐户凭据。

    +   `--keyring-migration-port`：对于 TCP/IP 连接，连接到运行服务器的端口号。

    +   `--keyring-migration-socket`：对于 Unix 套接字文件或 Windows 命名管道连接，连接到运行服务器的套接字文件或命名管道。

有关密钥迁移选项的更多详细信息，请参见第 8.4.4.18 节，“密钥环命令选项”。

使用指示源和目标密钥库以及迁移是离线还是在线的密钥迁移选项启动迁移服务器，可能还有其他选项。请牢记以下考虑事项：

+   可能需要其他服务器选项，例如两个密钥环插件的配置参数。例如，如果`keyring_file`是源或目标，则必须设置`keyring_file_data`系统变量，如果密钥环数据文件位置不是默认位置。还可能需要其他非密钥环选项。指定这些选项的一种方法是使用`--defaults-file`命名包含所需选项的选项文件。

+   迁移服务器期望路径名选项值为完整路径。相对路径名可能不会按您的预期解析。

+   调用服务器以密钥迁移模式启动的用户不得是`root`操作系统用户，除非使用`--user`选项指定一个非`root`用户名来以该用户身份运行服务器。

+   以密钥迁移模式运行的服务器的用户必须具有读取和写入任何本地密钥环文件的权限，例如基于文件的插件的数据文件。

    如果您以不同于通常用于运行 MySQL 的系统帐户调用迁移服务器，则可能会创建对服务器在正常操作期间无法访问的密钥环目录或文件。假设**mysqld**通常以`mysql`操作系统用户运行，但您以`isabel`登录时调用迁移服务器。迁移服务器创建的任何新目录或文件都归`isabel`所有。当以`mysql`操作系统用户运行的服务器尝试访问由`isabel`拥有的文件系统对象时，后续启动将失败。

    为避免此问题，请将迁移服务器作为`root`操作系统用户启动，并提供`--user=*`user_name`*`选项，其中*`user_name`*是通常用于运行 MySQL 的系统帐户。或者，在迁移后，使用**chown**、**chmod**或类似命令检查与密钥环相关的文件系统对象，并根据需要更改其所有权和权限，以便运行服务器可以访问这些对象。

离线迁移两个密钥环插件之间的示例命令行（在一行上输入命令）：

```sql
mysqld --defaults-file=/usr/local/mysql/etc/my.cnf
  --keyring-migration-source=keyring_file.so
  --keyring-migration-destination=keyring_encrypted_file.so
  --keyring_encrypted_file_password=*password*
```

两个密钥环插件之间的在线迁移示例命令行：

```sql
mysqld --defaults-file=/usr/local/mysql/etc/my.cnf
  --keyring-migration-source=keyring_file.so
  --keyring-migration-destination=keyring_encrypted_file.so
  --keyring_encrypted_file_password=*password*
  --keyring-migration-host=127.0.0.1
  --keyring-migration-user=root
  --keyring-migration-password=*root_password*
```

若要执行将目标设置为密钥环组件而不是密钥环插件的迁移，请指定`--keyring-migration-to-component`选项，并将组件命名为`--keyring-migration-destination`选项的值。

从密钥环插件迁移到密钥环组件的离线迁移示例命令行：

```sql
mysqld --defaults-file=/usr/local/mysql/etc/my.cnf
  --keyring-migration-to-component
  --keyring-migration-source=keyring_file.so
  --keyring-migration-destination=component_keyring_encrypted_file.so
```

请注意，在这种情况下，未指定`keyring_encrypted_file_password`值。组件数据文件的密码列在组件配置文件中。

从密钥环插件迁移到密钥环组件的在线迁移示例命令行：

```sql
mysqld --defaults-file=/usr/local/mysql/etc/my.cnf
  --keyring-migration-to-component
  --keyring-migration-source=keyring_file.so
  --keyring-migration-destination=component_keyring_encrypted_file.so
  --keyring-migration-host=127.0.0.1
  --keyring-migration-user=root
  --keyring-migration-password=*root_password*
```

密钥迁移服务器执行以下迁移操作：

1.  （仅在线迁移）使用连接选项连接到运行中的服务器。

1.  （仅在线迁移）在运行中的服务器上禁用`keyring_operations`。

1.  加载源和目标密钥环插件/组件库。

1.  将密钥从源密钥库复制到目标密钥库。

1.  卸载源和目标密钥环插件/组件库。

1.  （仅在线迁移）在运行中的服务器上启用`keyring_operations`。

1.  （仅在线迁移）断开与运行中的服务器的连接。

如果在密钥迁移过程中发生错误，则目标密钥库将恢复到迁移前的状态。

成功进行在线密钥迁移操作后，可能需要重新启动运行中的服务器：

+   如果运行的服务器在迁移前使用源密钥库，并且在迁移后应继续使用它，则在迁移后无需重新启动。

+   如果运行的服务器在迁移前使用目标密钥库，并且在迁移后应继续使用它，则应在迁移后重新启动以加载所有迁移到目标密钥库中的密钥。

+   如果运行的服务器在迁移前使用源密钥库，但在迁移后应使用目标密钥库，则必须重新配置为使用目标密钥库并重新启动。在这种情况下，请注意，尽管在迁移过程中运行的服务器暂停修改源密钥库，但在迁移和随后的重新启动之间的间隔期间不会暂停。应注意在此间隔期间服务器不要修改源密钥库，因为这样的任何更改将不会反映在目标密钥库中。

##### 使用 mysql_migrate_keyring 实用程序进行密钥迁移

**mysql_migrate_keyring** 实用程序将密钥从一个密钥环组件迁移到另一个。它不支持涉及密钥环插件的迁移。对于这种类型的迁移，请使用在密钥迁移模式下运行的 MySQL 服务器；参见使用迁移服务器进行密钥迁移。

使用**mysql_migrate_keyring**执行密钥迁移操作时，确定所需的密钥迁移选项，以指定涉及哪些密钥环组件，以及迁移是离线还是在线的：

+   要指示源和目标密钥环组件及其位置，请指定以下选项：

    +   `--source-keyring`：管理要迁移的密钥的源密钥环组件。

    +   `--destination-keyring`：要将迁移的密钥复制到的目标密钥环组件。

    +   `--component-dir`：包含密钥环组件库文件的目录。这通常是本地 MySQL 服务器的`plugin_dir`系统变量的值。

    所有三个选项都是必需的。每个密钥库组件名称都是一个组件库文件名，不带任何平台特定的扩展名，如`.so`或`.dll`。例如，要使用库文件为`component_keyring_file.so`的组件，请将选项指定为`--source-keyring=component_keyring_file`。源和目标必须不同，并且**mysql_migrate_keyring**必须同时支持它们。

+   对于离线迁移，不需要额外的选项。

+   对于在线迁移，某个正在运行的服务器当前正在使用源或目标密钥库。在这种情况下，请指定`--online-migration`选项表示在线迁移。此外，请指定连接选项，指示如何连接到运行的服务器，以便**mysql_migrate_keyring**可以连接到它并告诉它在迁移操作期间暂停密钥环使用。

    `--online-migration`选项通常与这些连接选项一起使用：

    +   `--host`：运行服务器所在的主机。这始终是本地主机，因为**mysql_migrate_keyring**只能在本地组件管理的密钥库之间迁移密钥。

    +   `--user`、`--password`：用于连接到运行服务器的帐户凭据。

    +   `--port`：对于 TCP/IP 连接，在运行服务器上连接的端口号。

    +   `--socket`：对于 Unix 套接字文件或 Windows 命名管道连接，在运行服务器上连接的套接字文件或命名管道。

要查看所有可用选项的描述，请参阅 Section 6.6.8, “mysql_migrate_keyring — Keyring Key Migration Utility”。

启动**mysql_migrate_keyring**时，需要指定源和目标密钥库以及迁移是离线还是在线，可能还有其他选项。请记住以下几点：

+   调用**mysql_migrate_keyring**的用户不能是`root`操作系统用户。

+   调用**mysql_migrate_keyring**的用户必须具有读取和写入任何本地密钥环文件的权限，例如基于文件的插件的数据文件。

    如果您以不同于通常用于运行 MySQL 的系统帐户调用**mysql_migrate_keyring**，可能会创建对运行服务器在正常操作期间无法访问的密钥环目录或文件。假设**mysqld**通常以`mysql`操作系统用户身份运行，但您在`isabel`登录时调用**mysql_migrate_keyring**。**mysql_migrate_keyring**创建的任何新目录或文件都归`isabel`所有。当以`mysql`操作系统用户身份运行的服务器尝试访问由`isabel`拥有的文件系统对象时，随后的启动将失败。

    为避免此问题，请以`mysql`操作系统用户的身份调用**mysql_migrate_keyring**。或者，在迁移后，检查与密钥环相关的文件系统对象，并根据需要使用**chown**、**chmod**或类似命令更改它们的所有权和权限，以便运行服务器可以访问这些对象。

假设您想要从`component_keyring_file`迁移密钥到`component_keyring_encrypted_file`，并且本地服务器将其密钥环组件库文件存储在`/usr/local/mysql/lib/plugin`中。

如果没有运行中的服务器正在使用密钥环，允许进行离线迁移。像这样调用**mysql_migrate_keyring**（在一行中输入命令）：

```sql
mysql_migrate_keyring
  --component-dir=/usr/local/mysql/lib/plugin
  --source-keyring=component_keyring_file
  --destination-keyring=component_keyring_encrypted_file
```

如果运行中的服务器正在使用密钥环，则必须执行在线迁移。在这种情况下，必须提供`--online-migration`选项，以及指定要连接的服务器和要使用的 MySQL 帐户所需的任何连接选项。

以下命令执行在线迁移。它使用 TCP/IP 连接和`admin`帐户连接到本地服务器。命令提示输入密码，您应在提示时输入密码：

```sql
mysql_migrate_keyring
  --component-dir=/usr/local/mysql/lib/plugin
  --source-keyring=component_keyring_file
  --destination-keyring=component_keyring_encrypted_file
  --online-migration --host=127.0.0.1 --user=admin --password
```

**mysql_migrate_keyring**执行迁移操作如下：

1.  （仅限在线迁移）使用连接选项连接到运行中的服务器。

1.  （仅限在线迁移）在运行中的服务器上禁用`keyring_operations`。

1.  加载源和目标密钥库的密钥环组件库。

1.  将密钥从源密钥库复制到目标密钥库。

1.  卸载源和目标密钥库的密钥环组件库。

1.  （仅限在线迁移）在运行中的服务器上启用`keyring_operations`。

1.  （仅限在线迁移）断开与运行中的服务器的连接。

如果在密钥迁移过程中发生错误，则目标密钥库将恢复到迁移前的状态。

在成功的在线密钥迁移操作之后，运行中的服务器可能需要重新启动：

+   如果运行中的服务器在迁移前使用源密钥库，并且在迁移后应继续使用它，则在迁移后无需重新启动。

+   如果在迁移前运行中的服务器使用目标密钥库，并且在迁移后应继续使用它，则应在迁移后重新启动以加载所有迁移到目标密钥库中的密钥。

+   如果在迁移前运行中的服务器使用源密钥库，但在迁移后应使用目标密钥库，则必须重新配置以使用目标密钥库并重新启动。在这种情况下，请注意，尽管在迁移过程中暂停了运行中的服务器对源密钥库的修改，但在迁移和随后的重新启动之间的间隔期间，服务器并未暂停。应注意确保服务器在此间隔期间不修改源密钥库，因为这些更改不会反映在目标密钥库中。

##### 涉及多个运行中的服务器的密钥迁移

在线密钥迁移允许暂停单个运行中的服务器上的密钥环操作。如果多个运行中的服务器正在使用涉及的密钥库，则使用以下过程执行迁移：

1.  手动连接到每个运行中的服务器，并设置`keyring_operations=OFF`。这确保没有运行中的服务器正在使用源密钥库或目标密钥库，并满足离线迁移所需的条件。

1.  使用迁移服务器或**mysql_migrate_keyring**为每个暂停的服务器执行离线密钥迁移。

1.  手动连接到每个运行中的服务器，并设置`keyring_operations=ON`。

所有运行中的服务器必须支持`keyring_operations`系统变量。任何不支持的服务器在迁移前必须停止，并在迁移后重新启动。
