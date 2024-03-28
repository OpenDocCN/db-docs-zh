# 3.7 在 Unix/Linux 上升级 MySQL 二进制或基于包的安装

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrade-binary-package.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrade-binary-package.html)

本节描述了如何在 Unix/Linux 上升级 MySQL 二进制和基于包的安装。介绍了原地和逻辑升级方法。

+   原地升级

+   逻辑升级

+   MySQL Cluster 升级

### 原地升级

原地升级涉及关闭旧的 MySQL 服务器，用新的 MySQL 二进制文件或包替换旧的文件，重新启动现有数据目录上的 MySQL，并升级需要升级的现有安装的任何剩余部分。有关可能需要升级的详细信息，请参阅 Section 3.4, “MySQL 升级过程升级的内容”。

注意

如果您正在升级最初通过安装多个 RPM 包生成的安装程序，请升级所有包，而不仅仅是一些。例如，如果您之前安装了服务器和客户端 RPM 包，请不要仅升级服务器 RPM 包。

对于某些 Linux 平台，从 RPM 或 Debian 包安装 MySQL 包括 systemd 支持以管理 MySQL 服务器的启动和关闭。在这些平台上，**mysqld_safe**未安装。在这种情况下，使用 systemd 代替以下说明中使用的方法进行服务器的启动和关闭。请参阅 Section 2.5.9, “使用 systemd 管理 MySQL 服务器”。

对于 MySQL Cluster 安装的升级，请参阅 MySQL Cluster 升级。

���行原地升级：

1.  查看 Section 3.1, “开始之前”中的信息。

1.  通过完成 Section 3.6, “准备升级安装”中的初步检查，确保您的安装准备好升级。

1.  如果您在 `InnoDB` 中使用 XA 事务，请在升级之前运行`XA RECOVER`以检查未提交的 XA 事务。如果返回结果，请通过发出`XA COMMIT`或`XA ROLLBACK`语句提交或回滚 XA 事务。

1.  如果您从 MySQL 5.7.11 或更早版本升级到 MySQL 8.0，并且存在加密的 `InnoDB` 表空间，请通过执行此语句旋转密钥环主密钥：

    ```sql
    ALTER INSTANCE ROTATE INNODB MASTER KEY;
    ```

1.  如果您通常将 MySQL 服务器配置为将`innodb_fast_shutdown`设置为`2`（冷关闭），请通过执行以下任一语句之一配置它执行快速或慢速关闭：

    ```sql
    SET GLOBAL innodb_fast_shutdown = 1; -- fast shutdown
    SET GLOBAL innodb_fast_shutdown = 0; -- slow shutdown
    ```

    使用快速或慢速关闭，`InnoDB`会将其撤销日志和数据文件保留在一种状态，以便在不同版本之间的文件格式差异情况下进行处理。

1.  关闭旧的 MySQL 服务器。例如：

    ```sql
    mysqladmin -u root -p shutdown
    ```

1.  升级 MySQL 二进制文件或软件包。如果升级二进制安装，解压新的 MySQL 二进制分发包。参见获取和解压分发包。对于基于软件包的安装，请安装新软件包。

1.  启动 MySQL 8.0 服务器，使用现有的数据目录。例如：

    ```sql
    mysqld_safe --user=mysql --datadir=*/path/to/existing-datadir* &
    ```

    如果存在加密的`InnoDB`表空间，请使用`--early-plugin-load`选项加载密钥环插件。

    当您启动 MySQL 8.0 服务器时，它会自动检测数据字典表是否存在。如果不存在，服务器将在数据目录中创建这些表，填充元数据，然后继续正常的启动序列。在此过程中，服务器会升级所有数据库对象的元数据，包括数据库、表空间、系统和用户表、视图以及存储程序（存储过程和函数、触发器和事件调度器事件）。服务器还会删除以前用于存储元数据的文件。例如，在从 MySQL 5.7 升级到 MySQL 8.0 后，您可能会注意到表不再具有`.frm`文件。

    如果此步骤失败，服务器将还原对数据目录的所有更改。在这种情况下，您应删除所有重做日志文件，在相同数据目录上启动您的 MySQL 5.7 服务器，并修复任何错误的原因。然后执行另一个慢速关闭 5.7 服务器，并启动 MySQL 8.0 服务器再次尝试。

1.  在前一步骤中，服务器根据需要升级数据字典。现在需要执行任何剩余的升级操作：

    +   从 MySQL 8.0.16 开始，服务器会在前一步骤中执行这些操作，对`mysql`系统数据库在 MySQL 5.7 和 MySQL 8.0 之间所需的任何更改进行处理，以便您可以利用新的权限或功能。它还会为 MySQL 8.0 更新性能模式、`INFORMATION_SCHEMA`和`sys`数据库，并检查所有用户数据库是否与当前版本的 MySQL 不兼容。

    +   在 MySQL 8.0.16 之前，服务器仅在前一步骤中升级数据字典。成功启动 MySQL 8.0 服务器后，执行**mysql_upgrade**来执行剩余的升级任务：

        ```sql
        mysql_upgrade -u root -p
        ```

        然后关闭并重新启动 MySQL 服务器，以确保对系统表所做的任何更改生效。例如：

        ```sql
        mysqladmin -u root -p shutdown
        mysqld_safe --user=mysql --datadir=*/path/to/existing-datadir* &
        ```

        第一次启动 MySQL 8.0 服务器（在之前的步骤中），您可能会在错误日志中看到有关未升级表的消息。如果**mysql_upgrade**已成功运行，则第二次启动服务器时不应出现此类消息。

注意

升级过程不会升级时区表的内容。有关升级说明，请参见第 7.1.15 节，“MySQL 服务器时区支持”。

如果升级过程使用**mysql_upgrade**（即在 MySQL 8.0.16 之前），该过程也不会升级帮助表的内容。在这种情况下的升级说明，请参见第 7.1.17 节，“服务器端帮助支持”。

### 逻辑升级

逻辑升级涉及使用备份或导出工具（如**mysqldump**或**mysqlpump**）从旧的 MySQL 实例中导出 SQL，安装新的 MySQL 服务器，并将 SQL 应用于新的 MySQL 实例。有关可能需要升级的详细信息，请参见第 3.4 节，“MySQL 升级过程升级了什么”。

注意

对于某些 Linux 平台，从 RPM 或 Debian 软件包安装 MySQL 包括 systemd 支持以管理 MySQL 服务器的启动和关闭。在这些平台上，不安装**mysqld_safe**。在这种情况下，请使用 systemd 来启动和关闭服务器，而不是以下说明中使用的方法。请参见第 2.5.9 节，“使用 systemd 管理 MySQL 服务器”。

警告

将从先前的 MySQL 版本中提取的 SQL 应用于新的 MySQL 版本可能会由于新功能和能力的不兼容性而导致错误。因此，从先前的 MySQL 版本中提取的 SQL 可能需要修改以实现逻辑升级。

在升级到最新的 MySQL 8.0 版本之前，执行第 3.6 节，“准备升级安装”中描述的步骤以识别不兼容性。

执行逻辑升级：

1.  查看第 3.1 节，“开始之前”中的信息。

1.  从先前的 MySQL 安装中导出现有数据：

    ```sql
    mysqldump -u root -p
      --add-drop-table --routines --events
      --all-databases --force > data-for-upgrade.sql
    ```

    注意

    如果您的数据库包含存储程序，请使用`--routines`和`--events`选项与**mysqldump**（如上所示）。`--all-databases`选项包括转储中的所有数据库，包括保存系统表的`mysql`数据库。

    重要

    如果您的表包含生成列，请使用 MySQL 5.7.9 或更高版本提供的**mysqldump**实用程序创建转储文件。较早版本提供的**mysqldump**实用程序对生成列定义使用了不正确的语法（Bug #20769542）。您可以使用信息模式`COLUMNS`表来识别具有生成列的表。

1.  关闭旧的 MySQL 服务器。例如：

    ```sql
    mysqladmin -u root -p shutdown
    ```

1.  安装 MySQL 8.0。有关安装说明，请参见 Chapter 2, *Installing MySQL*。

1.  初始化新的数据目录，如 Section 2.9.1, “Initializing the Data Directory”中所述。例如：

    ```sql
    mysqld --initialize --datadir=*/path/to/8.0-datadir*
    ```

    复制临时显示在屏幕上或写入错误日志以供以后使用的 `'root'@'localhost'` 密码。

1.  启动 MySQL 8.0 服务器，使用新的数据目录。例如：

    ```sql
    mysqld_safe --user=mysql --datadir=*/path/to/8.0-datadir* &
    ```

1.  重置`root`密码：

    ```sql
    $> mysql -u root -p
    Enter password: ****  <- enter temporary root password
    ```

    ```sql
    mysql> ALTER USER USER() IDENTIFIED BY '*your new password*';
    ```

1.  将先前创建的转储文件加载到新的 MySQL 服务器中。例如：

    ```sql
    mysql -u root -p --force < data-for-upgrade.sql
    ```

    注意

    当服务器启用 GTIDs（`gtid_mode=ON`）时，不建议加载包含系统表的转储文件。**mysqldump** 为使用非事务性 MyISAM 存储引擎的系统表发出 DML 指令，而在启用 GTIDs 时，这种组合是不允许的。还要注意，将从启用 GTIDs 的服务器中加载的转储文件加载到另一个启用 GTIDs 的服务器中会生成不同的事务标识符。

1.  执行任何剩余的升级操作：

    +   在 MySQL 8.0.16 及更高版本中，关闭服务器，然后使用`--upgrade=FORCE`选项重新启动以执行剩余的升级任务：

        ```sql
        mysqladmin -u root -p shutdown
        mysqld_safe --user=mysql --datadir=*/path/to/8.0-datadir* --upgrade=FORCE &
        ```

        重新启动时使用`--upgrade=FORCE`，服务器会在 MySQL 5.7 和 MySQL 8.0 之间对`mysql`系统模式进行所需的任何更改，以便您可以利用新的权限或功能。它还会将性能模式、`INFORMATION_SCHEMA`和`sys`模式更新到 MySQL 8.0，并检查所有用户模式与当前版本的 MySQL 的不兼容性。

    +   在 MySQL 8.0.16 之前，执行**mysql_upgrade**执行剩余的升级任务：

        ```sql
        mysql_upgrade -u root -p
        ```

        然后关闭并重新启动 MySQL 服务器，以确保对系统表所做的任何更改生效。例如：

        ```sql
        mysqladmin -u root -p shutdown
        mysqld_safe --user=mysql --datadir=*/path/to/8.0-datadir* &
        ```

注意

升级过程不会升级时区表的内容。有关升级说明，请参见第 7.1.15 节，“MySQL 服务器时区支持”。

如果升级过程使用**mysql_upgrade**（即在 MySQL 8.0.16 之前），该过程也不会升级帮助表的内容。在这种情况下的升级说明，请参见第 7.1.17 节，“服务器端帮助支持”。

注意

加载包含 MySQL 5.7 `mysql`模式的转储文件会重新创建两个不再使用的表：`event`和`proc`。（相应的 MySQL 8.0 表是`events`和`routines`，都是数据字典表并受到保护。）在确认升级成功后，您可以通过执行以下 SQL 语句删除`event`和`proc`表：

```sql
DROP TABLE mysql.event;
DROP TABLE mysql.proc;
```

### MySQL Cluster 升级

本节信息是就地升级中描述的程序的附属内容，用于升级 MySQL Cluster。

从 MySQL 8.0.16 开始，MySQL Cluster 升级可以作为常规滚动升级执行，遵循通常的三个有序步骤：

1.  升级 MGM 节点。

1.  逐个升级数据节点。

1.  逐个升级 API 节点（包括 MySQL 服务器）。

升级每个节点的方式几乎与 MySQL 8.0.16 之前相同，因为升级数据字典和升级系统表之间有区别。升级每个单独的`mysqld`有两个步骤：

1.  导入数据字典。

    使用`--upgrade=MINIMAL`选项启动新服务器以升级数据字典但不升级系统表。这与 MySQL 8.0.16 之前启动服务器但不调用**mysql_upgrade**的操作基本相同。

    MySQL 服务器必须连接到`NDB`才能完成此阶段。如果存在任何`NDB`或`NDBINFO`表，并且服务器无法连接到集群，则会显示错误消息并退出：

    ```sql
    Failed to Populate DD tables.
    ```

1.  升级系统表。

    在 MySQL 8.0.16 之前，DBA 调用 **mysql_upgrade** 客户端来升级系统表。从 MySQL 8.0.16 开始，服务器执行此操作：为了升级系统表，重新启动每个单独的 **mysqld**，不使用 `--upgrade=MINIMAL` 选项。
