# 3.4 MySQL 升级过程升级的内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrading-what-is-upgraded.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrading-what-is-upgraded.html)

安装新版本的 MySQL 可能需要升级现有安装的这些部分：

+   包含 MySQL 服务器运行所需信息的表的 `mysql` 系统模式（请参阅 第 7.3 节，“mysql 系统模式”）。 `mysql` 模式表分为两大类：

    +   存储数据库对象元数据的数据字典表。

    +   系统表（即剩余的非数据字典表），用于其他操作目的。

+   其他模式，其中一些是内置的，可能被服务器“拥有”，而另一些则不是：

    +   `performance_schema`, `INFORMATION_SCHEMA`, `ndbinfo`, 和 `sys` 模式。

    +   用户模式。

与可能需要升级的安装部分相关联的两个不同版本号：

+   数据字典版本。 这适用于数据字典表。

+   服务器版本，也称为 MySQL 版本。 这适用于系统表和其他模式中的对象。

在这两种情况下，现有 MySQL 安装适用的实际版本存储在数据字典中，当前期望的版本编译到新版本的 MySQL 中。 当实际版本低于当前期望版本时，必须将与该版本相关联的安装部分升级到当前版本。 如果两个版本都指示需要升级，则必须首先进行数据字典升级。

作为刚提到的两个不同版本的反映，升级分为两个步骤：

+   步骤 1：数据字典升级。

    此步骤升级：

    +   `mysql` 模式中的数据字典表。 如果实际数据字典版本低于当前期望版本，则服务器会创建具有更新定义的数据字典表，将持久化的元数据复制到新表中，原子性地用新表替换旧表，并重新初始化数据字典。

    +   性能模式，`INFORMATION_SCHEMA` 和 `ndbinfo`。

+   步骤 2：服务器升级。

    此步骤包括所有其他升级任务。 如果现有 MySQL 安装的服务器版本低于新安装的 MySQL 版本，则必须升级其他所有内容：

    +   `mysql` 模式中的系统表（剩余的非数据字典表）。

    +   `sys` 模式。

    +   用户模式。

数据字典升级（步骤 1）是服务器的责任，服务器会在启动时根据需要执行此任务，除非使用阻止其执行的选项。MySQL 8.0.16 中的选项是 `--upgrade=NONE`，MySQL 8.0.16 之前是 `--no-dd-upgrade`。

如果数据字典过时但服务器被阻止升级它，服务器将不运行，并���错退出。例如：

```sql
[ERROR] [MY-013381] [Server] Server shutting down because upgrade is
required, yet prohibited by the command line option '--upgrade=NONE'.
[ERROR] [MY-010334] [Server] Failed to initialize DD Storage Engine
[ERROR] [MY-010020] [Server] Data Dictionary initialization failed.
```

MySQL 8.0.16 对步骤 2 的责任发生了一些变化：

+   在 MySQL 8.0.16 之前，**mysql_upgrade** 升级性能模式、`INFORMATION_SCHEMA` 和步骤 2 中描述的对象。预期 DBA 在启动服务器后手动调用 **mysql_upgrade**。

+   从 MySQL 8.0.16 开始，服务器执行以前由 **mysql_upgrade** 处理的所有任务。尽管升级仍然是一个两步操作，但服务器会执行这两步，从而简化了流程。

根据您要升级到的 MySQL 版本，在 原地升级 和 逻辑升级 中的说明指示服务器是否执行所有升级任务，或者您必须在服务器启动后手动调用 **mysql_upgrade**。

注意

因为服务器在 MySQL 8.0.16 中升级了性能模式、`INFORMATION_SCHEMA` 和步骤 2 中描述的对象，所以 **mysql_upgrade** 不再需要，并且在该版本中已被弃用；预计在未来的 MySQL 版本中将其移除。

在 MySQL 8.0.16 之前和之后，步骤 2 中发生的大部分情况是相同的，尽管可能需要不同的命令选项来实现特定效果。

从 MySQL 8.0.16 开始，`--upgrade` 服务器选项控制服务器在启动时是否以及如何执行自动升级：

+   没有选项或使用 `--upgrade=AUTO`，服务器会升级任何被确定为过时的内容（步骤 1 和 2）。

+   使用 `--upgrade=NONE`，服务器不执行任何升级（跳过步骤 1 和 2），但如果数据字典必须升级，服务器也会报错退出。服务器不允许使用过时的数据字典运行；服务器要么升级它，要么退出。

+   使用 `--upgrade=MINIMAL`，如果需要（步骤 1），服务器将升级数据字典、性能模式和 `INFORMATION_SCHEMA`。请注意，使用此选项进行升级后，无法启动组复制，因为复制内部依赖的系统表未更新，并且在其他领域也可能出现功能减少的情况。

+   使用 `--upgrade=FORCE`，如果需要（步骤 1），服务器将升级数据字典、性能模式和 `INFORMATION_SCHEMA`，并强制升级其他所有内容（步骤 2）。请注意，使用此选项可能会导致服务器启动时间较长，因为服务器会检查所有模式中的所有对象。

`FORCE` 用于强制执行第 2 步操作，如果服务器认为这些操作是必要的。 `FORCE` 与 `AUTO` 的一个区别是，使用 `FORCE` 时，如果系统表（如帮助表或时区表）丢失，服务器会重新创建这些表。

以下列表显示了 MySQL 8.0.16 之前的升级命令以及 MySQL 8.0.16 及更高版本的等效命令：

+   执行正常升级（根据需要执行步骤 1 和 2）：

    +   MySQL 8.0.16 之前：**mysqld** 后跟 **mysql_upgrade**

    +   截至 MySQL 8.0.16：**mysqld**

+   根据需要仅执行步骤 1：

    +   MySQL 8.0.16 之前：不可能执行步骤 1 中描述的所有升级任务，同时排除步骤 2 中描述的任务。但是，您可以使用 **mysqld** 后跟 **mysql_upgrade** 以 `--upgrade-system-tables` 和 `--skip-sys-schema` 选项来避免升级用户模式和 `sys` 模式。

    +   截至 MySQL 8.0.16：**mysqld --upgrade=MINIMAL**

+   根据需要执行第 1 步，并强制执行第 2 步：

    +   MySQL 8.0.16 之前：**mysqld** 后跟 **mysql_upgrade --force**

    +   截至 MySQL 8.0.16：**mysqld --upgrade=FORCE**

MySQL 8.0.16 之前，某些 **mysql_upgrade** 选项会影响其执行的操作。以下表格显示了截至 MySQL 8.0.16，要实现类似效果应使用哪些服务器 `--upgrade` 选项值。（这些不一定是确切的等效值，因为给定的 `--upgrade` 选项值可能具有额外效果。）

| mysql_upgrade 选项 | 服务器选项 |
| --- | --- |
| `--skip-sys-schema` | `--upgrade=NONE` 或 `--upgrade=MINIMAL` |
| `--upgrade-system-tables` | `--upgrade=NONE` 或 `--upgrade=MINIMAL` |
| `--force` | `--upgrade=FORCE` |

升级步骤 2 的附加说明：

+   步骤 2 会安装`sys`模式（如果尚未安装），否则将其升级到当前版本。如果存在`sys`模式但没有`version`视图，则会出现错误，因为缺少`version`视图表明这是用户创建的模式：

    ```sql
    A sys schema exists with no sys.version view. If
    you have a user created sys schema, this must be renamed for the
    upgrade to succeed.
    ```

    在这种情况下进行升级，首先删除或重命名现有的`sys`模式。然后再次执行升级过程。（可能需要强制执行步骤 2。）

    为了防止`sys`模式检查：

    +   截至 MySQL 8.0.16：使用`--upgrade=NONE`或`--upgrade=MINIMAL`选项启动服务器。

    +   在 MySQL 8.0.16 之前：使用`--skip-sys-schema`选项调用**mysql_upgrade**。

+   步骤 2 会升级系统表以确保其具有当前结构。无论是服务器还是**mysql_upgrade**执行该步骤都是如此。关于帮助表和时区表的内容，**mysql_upgrade**不会加载任何类型的表，而服务器会加载帮助表，但不会加载时区表。（即，在 MySQL 8.0.16 之前，服务器仅在数据目录初始化时加载帮助表。从 MySQL 8.0.16 开始，它在初始化和升级时加载帮助表。）加载时区表的过程取决于平台，并且需要由 DBA 做出决策，因此无法自动完成。

+   从 MySQL 8.0.30 开始，当第 2 步升级`mysql`模式中的系统表时，`mysql.db`、`mysql.tables_priv`、`mysql.columns_priv`和`mysql.procs_priv`表的主键中的列顺序被更改为将主机名和用户名列放在一起。将主机名和用户名放在一起意味着可以使用索引查找，这提高了`CREATE USER`、`DROP USER`和`RENAME USER`语句的性能，以及对具有多个权限的多个用户进行 ACL 检查。如果系统具有大量用户和权限，则需要删除并重新创建索引，这可能需要一些时间。

+   第 2 步根据需要处理所有用户模式中的所有表。表检查可能需要很长时间才能完成。在处理表时，每个表都会被锁定，因此在处理过程中对其他会话不可用。检查和修复操作可能需要很长时间，特别是对于大表。表检查使用`CHECK TABLE`语句的`FOR UPGRADE`选项。有关此选项的详细信息，请参见第 15.7.3.2 节，“CHECK TABLE Statement”。

    要防止表检查：

    +   截至 MySQL 8.0.16 版本：使用`--upgrade=NONE`或`--upgrade=MINIMAL`选项启动服务器。

    +   在 MySQL 8.0.16 版本之前：使用`--upgrade-system-tables`选项调用**mysql_upgrade**。

    要强制进行表检查：

    +   截至 MySQL 8.0.16 版本：使用`--upgrade=FORCE`选项启动服务器。

    +   在 MySQL 8.0.16 版本之前：使用`--force`选项调用**mysql_upgrade**。

+   第 2 步将 MySQL 版本号保存在名为`mysql_upgrade_info`的文件中，该文件位于数据目录中。

    要忽略`mysql_upgrade_info`文件并执行检查，请执行以下操作：

    +   截至 MySQL 8.0.16 版本：使用`--upgrade=FORCE`选项启动服务器。

    +   在 MySQL 8.0.16 版本之前：使用`--force`选项调用**mysql_upgrade**。

    注意

    `mysql_upgrade_info`文件已被弃用；预计将在 MySQL 的未来版本中删除。

+   第 2 步使用当前 MySQL 版本号标记所有已检查和修复的表。这确保了下次使用相同版本的服务器进行升级检查时，可以确定是否需要再次检查或修复给定的表。
