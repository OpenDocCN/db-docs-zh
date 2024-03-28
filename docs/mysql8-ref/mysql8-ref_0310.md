> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-remote.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-remote.html)

#### 7.6.7.3 克隆远程数据

克隆插件支持以下语法用于克隆远程数据；即从远程 MySQL 服务器实例（捐赠者）克隆数据并将其传输到发起克隆操作的 MySQL 实例（接收方）。

```sql
CLONE INSTANCE FROM '*user*'@'*host*':*port*
IDENTIFIED BY '*password*'
[DATA DIRECTORY [=] '*clone_dir*']
[REQUIRE [NO] SSL];
```

其中：

+   `*user*`是在捐赠者 MySQL 服务器实例上的克隆用户。

+   `*password*`是`*user*`的密码。

+   `*host*`是捐赠者 MySQL 服务器实例的`主机名`地址。不支持 Internet 协议版本 6（IPv6）地址格式。可以使用 IPv6 地址的别名。IPv4 地址可以直接使用。

+   `*port*`是捐赠者 MySQL 服务器实例的`端口`号。（不支持由`mysqlx_port`指定的 X 协议端口。也不支持通过 MySQL 路由器连接到捐赠者 MySQL 服务器实例。）

+   `DATA DIRECTORY [=] '*clone_dir*'`是一个可选子句，用于指定您正在克隆的数据在接收方的目录。如果不想从接收方数据目录中删除现有的用户创建数据（模式、表、表空间）和二进制日志，则使用此选项。需要绝对路径，并且目录不能存在。MySQL 服务器必须具有必要的写访问权限以创建目录。

    当不使用可选的`DATA DIRECTORY [=] '*clone_dir*'`子句时，克隆操作会从接收方数据目录中删除用户创建的数据（模式、表、表空间）和二进制日志，将新数据克隆到接收方数据目录，并在之后自动重新启动服务器。

+   `[REQUIRE [NO] SSL]`明确指定在通过网络传输克隆数据时是否使用加密连接。如果无法满足明确规定，将返回错误。如果未指定 SSL 子句，默认情况下，克隆尝试建立加密连接，如果安全连接尝试失败，则退回到不安全连接。无论是否指定了此子句，克隆加密数据时都需要安全连接。有关更多信息，请参见为克隆配置加密连接。

注意

默认情况下，位于捐赠者 MySQL 服务器实例数据目录中的用户创建的`InnoDB`表和表空间将被克隆到接收者 MySQL 服务器实例的数据目录中。如果指定了`DATA DIRECTORY [=] '*clone_dir*'`子句，则它们将被克隆到指定目录。

用户创建的`InnoDB`表和表空间，如果位于捐赠 MySQL 服务器实例上的数据目录之外，则会被克隆到接收 MySQL 服务器实例上的相同路径。如果表或表空间已经存在，则会报告错误。

默认情况下，`InnoDB`系统表空间、重做日志和撤销表空间会被克隆到在捐赠端配置的相同位置（由`innodb_data_home_dir`和`innodb_data_file_path`、`innodb_log_group_home_dir`和`innodb_undo_directory`定义）。如果指定了`DATA DIRECTORY [=] '*clone_dir*'`子句，则这些表空间和日志会被克隆到指定目录。

##### 远程克隆先决条件

要执行克隆操作，克隆插件必须在捐赠端和接收端的 MySQL 服务器实例上处于活动状态。有关安装说明，请参阅 Section 7.6.7.1, “Installing the Clone Plugin”。

在捐赠端和接收端需要一个 MySQL 用户来执行克隆操作（“克隆用户”）。

+   在捐赠端，克隆用户需要`BACKUP_ADMIN`权限，以便在克隆操作期间访问和传输来自捐赠端的数据，并阻止并发的 DDL。在 MySQL 8.0.27 之前，克隆操作期间会在捐赠端阻止并发的 DDL。从 MySQL 8.0.27 开始，默认情况下在捐赠端允许并发的 DDL。请参阅 Section 7.6.7.4, “Cloning and Concurrent DDL”。

+   在接收端，克隆用户需要`CLONE_ADMIN`权限来替换接收端数据，在克隆操作期间阻止 DDL，并自动重新启动服务器。`CLONE_ADMIN`权限隐含包括`BACKUP_ADMIN`和`SHUTDOWN`权限。

包括创建克隆用户和授予所需权限的说明在接下来的远程克隆示例中。

当执行`CLONE INSTANCE`语句时，会检查以下先决条件：

+   克隆插件在 MySQL 8.0.17 及更高版本中受支持。捐赠端和接收端必须是相同的 MySQL 服务器系列，例如 8.0.37 和 8.0.41。在 8.0.37 之前的版本中，它们必须是相同的点发布版本。

    ```sql
    mysql> SHOW VARIABLES LIKE 'version';
     +---------------+--------+
    | Variable_name | Value  |
    +---------------+--------+
    | version       | 8.0.36 |
    +---------------+--------+
    ```

    从捐赠 MySQL 服务器实例克隆到相同版本和发布版本的热修复 MySQL 服务器实例在 MySQL 8.0.26 中得到支持。

    从系列内不同的点发布版本克隆是从 MySQL 8.0.37 开始支持的。对于早于 8.0.37 的版本，仍然适用先前的限制。例如，不允许将 8.0.36 克隆到 8.0.42 或反之亦然。

+   捐赠者和接收方 MySQL 服务器实例必须在相同的操作系统和平台上运行。例如，如果捐赠者实例在 Linux 64 位平台上运行，则接收方实例也必须在该平台上运行。请参考您的操作系统文档以了解如何确定您的操作系统平台。

+   接收方必须有足够的磁盘空间来存储克隆的数据。默认情况下，在克隆捐赠者数据之前，接收方会删除用户创建的数据（模式、表、表空间）和二进制日志，因此您只需要足够的空间来存储捐赠者数据。如果您使用`DATA DIRECTORY`子句克隆到命名目录，则必须有足够的磁盘空间来存储现有接收方数据和克隆的数据。您可以通过检查文件系统上的数据目录大小和驻留在数据目录之外的任何表空间的大小来估算数据大小。在估算捐赠者的数据大小时，请记住只有`InnoDB`数据会被克隆。如果您在其他存储引擎中存储数据，请相应调整数据大小的估算。

+   `InnoDB`允许在数据目录之外创建一些表空间类型。如果捐赠者 MySQL 服务器实例具有驻留在数据目录之外的表空间，则克隆操作必须能够访问这些表空间。您可以查询信息模式`FILES`表来识别驻留在数据目录之外的表空间。驻留在数据目录之外的文件具有指向数据目录以外目录的完全限定路径。

    ```sql
    mysql> SELECT FILE_NAME FROM INFORMATION_SCHEMA.FILES;
    ```

+   在捐赠者上激活的插件，包括任何密钥环插件，也必须在接收方上激活。您可以通过发出`SHOW PLUGINS`语句或查询信息模式`PLUGINS`表来识别活动插件。

+   捐赠者和接收方必须具有相同的 MySQL 服务器字符集和校对规则。有关 MySQL 服务器字符集和校对规则配置的信息，请参见第 12.15 节，“字符集配置”。

+   捐赠者和接收者必须具有相同的`innodb_page_size`和`innodb_data_file_path`设置。捐赠者和接收者的`innodb_data_file_path`设置必须指定相同数量的等效大小的数据文件。您可以使用`SHOW VARIABLES`语法检查变量设置。

    ```sql
    mysql> SHOW VARIABLES LIKE 'innodb_page_size';
    mysql> SHOW VARIABLES LIKE 'innodb_data_file_path';
    ```

+   如果克隆加密或页面压缩数据，则捐赠者和接收者必须具有相同的文件系统块大小。对于页面压缩数据，接收方文件系统必须支持稀疏文件和孔打孔，以便在接收方上进行孔打孔。有关这些功能以及如何识别使用这些功能的表和表空间的信息，请参见 Section 7.6.7.5，“克隆加密数据”和 Section 7.6.7.6，“克隆压缩数据”。要确定您的文件系统块大小，请参考您的操作系统文档。

+   如果要克隆加密数据，则需要安全连接。请参见为克隆配置加密连接。

+   接收方的`clone_valid_donor_list`设置必须包括捐赠者 MySQL 服务器实例的主机地址。您只能从有效捐赠者列表中的主机克隆数据。需要具有`SYSTEM_VARIABLES_ADMIN`权限的 MySQL 用户来配置此变量。在本节之后的远程克隆示例中提供了设置`clone_valid_donor_list`变量的说明。您可以使用`SHOW VARIABLES`语法检查`clone_valid_donor_list`设置。

    ```sql
    mysql> SHOW VARIABLES LIKE 'clone_valid_donor_list';
    ```

+   不得运行其他克隆操作。一次只允许进行单个克隆操作。要确定是否正在运行克隆操作，请查询`clone_status`表。请参见使用性能模式克隆表监视克隆操作。

+   克隆插件以 1MB 数据包加元数据的方式传输数据。因此，捐赠者和接收方 MySQL 服务器实例上最低要求的`max_allowed_packet`值为 2MB。小于 2MB 的`max_allowed_packet`值会导致错误。使用以下查询检查您的`max_allowed_packet`设置：

    ```sql
    mysql> SHOW VARIABLES LIKE 'max_allowed_packet';
    ```

还有以下先决条件：

+   捐赠者上的撤销表空间文件名必须是唯一的。当数据克隆到接收方时，无论在捐赠者上的位置如何，撤销表空间都会克隆到接收方的`innodb_undo_directory`位置或使用`DATA DIRECTORY [=] '*clone_dir*'`子句指定的目录。由于这个原因，捐赠者上重复的撤销表空间文件名是不允许的。从 MySQL 8.0.18 开始，在克隆操作期间遇到重复的撤销表空间文件名时会报告错误。在 MySQL 8.0.18 之前，克隆具有相同文件名的撤销表空间可能会导致接���方上的撤销表空间文件被覆盖。

    要查看捐赠者上的撤销表空间文件名以确保它们是唯一的，请查询`INFORMATION_SCHEMA.FILES`：

    ```sql
    mysql> SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES
           WHERE FILE_TYPE LIKE 'UNDO LOG';
    ```

    有关删除和添加撤销表空间文件的信息，请参见 Section 17.6.3.4，“撤销表空间”。

+   默认情况下，在数据克隆后，接收方 MySQL 服务器实例会自动重启（停止和启动）。要实现自动重启，接收方必须有一个监控进程来检测服务器的关闭。否则，在数据克隆后，克隆操作会因为以下错误而停止，并且接收方 MySQL 服务器实例会关闭：

    ```sql
    ERROR 3707 (HY000): Restart server failed (mysqld is not managed by supervisor process).
    ```

    此错误并不表示克隆失败。这意味着在数据克隆后，接收方 MySQL 服务器实例必须手动重新启动。在手动启动服务器后，您可以连接到接收方 MySQL 服务器实例，并检查性能模式克隆表，以验证克隆操作是否成功完成（请参见使用性能模式克隆表监控克隆操作）。`RESTART`语句也具有相同的监控进程要求。有关更多信息，请参见 Section 15.7.8.8，“RESTART Statement”。如果使用`DATA DIRECTORY`子句克隆到命名目录，则不适用此要求，因为在这种情况下不会执行自动重启。

+   几个变量控制远程克隆操作的各个方面。在执行远程克隆操作之前，请查看这些变量并根据需要调整设置以适应您的计算环境。克隆变量设置在执行克隆操作的接收方 MySQL 服务器实例上。请参阅 Section 7.6.7.13, “克隆系统变量”。

##### 克隆远程数据

以下示例演示了克隆远程数据的过程。默认情况下，远程克隆操作会在接收方上删除用户创建的数据（模式、表、表空间）和二进制日志，将新数据克隆到接收方数据目录，并在之后重新启动 MySQL 服务器。

本示例假定远程克隆的先决条件已满足。请参阅远程克隆先决条件。

1.  使用管理员用户帐户登录到捐赠方 MySQL 服务器实例。

    1.  创建一个具有`BACKUP_ADMIN`权限的克隆用户。

        ```sql
        mysql> CREATE USER 'donor_clone_user'@'example.donor.host.com' IDENTIFIED BY '*password*';
        mysql> GRANT BACKUP_ADMIN on *.* to 'donor_clone_user'@'example.donor.host.com';
        ```

    1.  安装克隆插件：

        ```sql
        mysql> INSTALL PLUGIN clone SONAME 'mysql_clone.so';
        ```

1.  使用管理员用户帐户登录到接收方 MySQL 服务器实例。

    1.  创建一个具有`CLONE_ADMIN`权限的克隆用户。

        ```sql
        mysql> CREATE USER 'recipient_clone_user'@'example.recipient.host.com' IDENTIFIED BY '*password*';
        mysql> GRANT CLONE_ADMIN on *.* to 'recipient_clone_user'@'example.recipient.host.com';
        ```

    1.  安装克隆插件：

        ```sql
        mysql> INSTALL PLUGIN clone SONAME 'mysql_clone.so';
        ```

    1.  将捐赠方 MySQL 服务器实例的主机地址添加到`clone_valid_donor_list`变量设置中。

        ```sql
        mysql> SET GLOBAL clone_valid_donor_list = '*example.donor.host.com*:*3306*';
        ```

1.  以前创建的克隆用户（`recipient_clone_user'@'example.recipient.host.com`）登录到接收方 MySQL 服务器实例，并执行`CLONE INSTANCE`语句。

    ```sql
    mysql> CLONE INSTANCE FROM 'donor_clone_user'@'example.donor.host.com':3306
           IDENTIFIED BY '*password*';
    ```

    数据克隆完成后，接收方 MySQL 服务器实例将自动重新启动。

    有关监视克隆操作状态和进度的信息，请参见 Section 7.6.7.10, “监视克隆操作”。

##### 克隆到指定目录

默认情况下，远程克隆操作会在克隆数据之前从接收方数据目录中删除用户创建的数据（模式、表、表空间）和二进制日志。通过克隆到指定目录，您可以避免从当前接收方数据目录中删除数据。

克隆到指定目录的过程与克隆远程数据中描述的过程相同，唯一的例外是：`CLONE INSTANCE`语句必须包含`DATA DIRECTORY`子句。例如：

```sql
mysql> CLONE INSTANCE FROM '*user*'@'*example.donor.host.com*':*3306*
       IDENTIFIED BY '*password*'
       DATA DIRECTORY = '*/path/to/clone_dir*';
```

需要绝对路径，并且目录不能存在。MySQL 服务器必须具有必要的写访问权限以创建目录。

在克隆到命名目录时，接收方 MySQL 服务器实例在克隆数据后不会自动重新启动。如果要在命名目录上重新启动 MySQL 服务器，则必须手动执行：

```sql
$> mysqld_safe --datadir=*/path/to/clone_dir*
```

其中*`/path/to/clone_dir`*是接收方命名目录的路径。

##### 为克隆配置加密连接

您可以配置远程克隆操作的加密连接，以保护在网络上传输时克隆的数据。默认情况下，当克隆加密数据时需要加密连接。（参见第 7.6.7.5 节，“克隆加密数据”。）

接下来的说明描述了如何配置接收方 MySQL 服务器实例以使用加密连接。假定捐赠方 MySQL 服务器实例已配置为使用加密连接。如果没有，请参考第 8.3.1 节，“配置 MySQL 使用加密连接”进行服务器端配置说明。

要配置接收方 MySQL 服务器实例以使用加密连接：

1.  使捐赠方 MySQL 服务器实例的客户端证书和密钥文件可供接收方主机使用。可以通过安全通道将文件分发到接收方主机，或将它们放在对接收方主机可访问的挂载分区上。要提供的客户端证书和密钥文件包括：

    +   `ca.pem`

        自签名证书颁发机构（CA）文件。

    +   `client-cert.pem`

        客户端公钥证书文件。

    +   `client-key.pem`

        客户端私钥文件。

1.  在接收方 MySQL 服务器实例上配置以下 SSL 选项。

    +   `clone_ssl_ca`

        指定自签名证书颁发机构（CA）文件的路径。

    +   `clone_ssl_cert`

        指定客户端公钥证书文件的路径。

    +   `clone_ssl_key`

        指定客户端私钥文件的路径。

    例如：

    ```sql
    clone_ssl_ca=/*path*/*to*/ca.pem
    clone_ssl_cert=/*path*/*to*/client-cert.pem
    clone_ssl_key=/*path*/*to*/client-key.pem
    ```

1.  要求使用加密连接，请在接收方发出`CLONE`语句时包含`REQUIRE SSL`子句。

    ```sql
    mysql> CLONE INSTANCE FROM '*user*'@'*example.donor.host.com*':*3306*
           IDENTIFIED BY '*password*'
           DATA DIRECTORY = '*/path/to/clone_dir*'
           REQUIRE SSL;
    ```

    如果未指定 SSL 子句，则克隆插件默认尝试建立加密连接，如果加密连接尝试失败，则回退到非加密连接。

    注意

    如果您正在克隆加密数据，则默认情况下需要加密连接，无论是否指定了`REQUIRE SSL`子句。使用`REQUIRE NO SSL`会在尝试克隆加密数据时导致错误。
