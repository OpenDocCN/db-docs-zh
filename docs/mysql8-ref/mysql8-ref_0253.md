# 7.1.16 资源组

> 原文：[`dev.mysql.com/doc/refman/8.0/en/resource-groups.html`](https://dev.mysql.com/doc/refman/8.0/en/resource-groups.html)

MySQL 支持创建和管理资源组，并允许将在服务器内运行的线程分配给特定组，以便线程根据组可用的资源执行。组属性使其资源可控，以启用或限制组中线程的资源消耗。DBA 可以根据不同的工作负载适当修改这些属性。

目前，CPU 时间是一种可管理的资源，由“虚拟 CPU”概念表示，该术语包括 CPU 核心、超线程、硬件线程等。服务器在启动时确定有多少虚拟 CPU 可用，并且具有适当权限的数据库管理员可以将这些 CPU 与资源组关联，并将线程分配给组。

例如，为了管理不需要以高优先级执行的批处理作业的执行，DBA 可以创建一个`Batch`资源组，并根据服务器的繁忙程度调整其优先级。 （也许应该在白天以较低优先级运行分配给该组的批处理作业，在晚上以较高优先级运行。）DBA 还可以调整组可用的 CPU 集。可以启用或禁用组以控制是否可以将线程分配给它们。

以下部分描述了 MySQL 中资源组使用的方面：

+   资源组元素

+   资源组属性

+   资源组管理

+   资源组复制

+   资源组限制

重要

在某些平台或 MySQL 服务器配置中，资源组可能不可用或存在限制。特别是，Linux 系统可能需要一些安装方法的手动步骤。详情请参阅资源组限制。

#### 资源组元素

这些功能为 MySQL 中资源组管理提供了 SQL 接口：

+   SQL 语句使得可以创建、更改和删除资源组，并允许将线程分配给资源组。优化器提示使得可以将单个语句分配给资源组。

+   资源组权限提供对哪些用户可以执行资源组操作的控制。

+   信息模式`RESOURCE_GROUPS`表公开了有关资源组定义的信息，性能模式`threads`表显示了每个线程的资源组分配。

+   状态变量为每个管理 SQL 语句提供执行计数。

#### 资源组属性

资源组具有定义组的属性。所有属性都可以在组创建时设置。一些属性在创建时是固定的，其他属性可以在之后的任何时候修改。

这些属性在资源组创建时被定义，不能被修改：

+   每个组都有一个名称。资源组名称是像表和列名称一样的标识符，在 SQL 语句中不需要用引号括起，除非它们包含特殊字符或是保留字。组名称不区分大小写，最长可达 64 个字符。

+   每个组都有一个类型，可以是`SYSTEM`或`USER`。资源组类型影响可分配给组的优先级值范围，如后文所述。这个属性与允许的优先级差异一起，使系统线程能够被识别，以免与用户线程争夺 CPU 资源。

    系统和用户线程对应于性能模式`threads`表中列出的后台和前台线程。

这些属性在资源组创建时被定义，之后可以随时修改：

+   CPU 亲和性是资源组可以使用的虚拟 CPU 集合。亲和性可以是可用 CPU 的任何非空子集。如果一个组没有亲和性，它可以使用所有可用的 CPU。

+   线程优先级是分配给资源组的线程的执行优先级。优先级值范围从-20（最高优先级）到 19（最低优先级）。默认优先级为 0，对于系统组和用户组都是如此。

    系统组允许比用户组更高的优先级，确保用户线程永远不会比系统线程具有更高的优先级：

    +   对于系统资源组，允许的优先级范围是-20 到 0。

    +   对于用户资源组，允许的优先级范围是 0 到 19。

+   每个组可以启用或禁用，管理员可以控制线程分配。只有启用的组才能分配线程。

#### 资源组管理

默认情况下，有一个系统组和一个用户组，分别命名为`SYS_default`和`USR_default`。这些默认组不能被删除，它们的属性也不能被修改。每个默认组没有 CPU 亲和性和优先级 0。

新创建的系统和用户线程分别分配给`SYS_default`和`USR_default`组。

对于用户定义的资源组，所有属性在组创建时被分配。创建组后，除了名称和类型属性外，其属性可以被修改。

要创建和管理用户定义的资源组，请使用以下 SQL 语句：

+   `CREATE RESOURCE GROUP` 创建一个新的组。参见 第 15.7.2.2 节，“创建资源组语句”。

+   `ALTER RESOURCE GROUP` 修改现有组。参见 第 15.7.2.1 节，“修改资源组语句”。

+   `DROP RESOURCE GROUP` 删除现有组。参见 第 15.7.2.3 节，“删除资源组语句”。

这些语句需要 `RESOURCE_GROUP_ADMIN` 权限。

要管理资源组分配，请使用以下功能：

+   `SET RESOURCE GROUP` 将线程分配给一个组。参见 第 15.7.2.4 节，“设置资源组语句”。

+   `RESOURCE_GROUP` 优化器提示将单个语句分配给一个组。参见 第 10.9.3 节，“优化器提示”。

这些操作需要 `RESOURCE_GROUP_ADMIN` 或 `RESOURCE_GROUP_USER` 权限。

资源组定义存储在 `resource_groups` 数据字典表中，以便在服务器重新启动时保持组的持久性。由于 `resource_groups` 是数据字典的一部分，用户无法直接访问它。资源组信息可通过信息模式 `RESOURCE_GROUPS` 表获取，该表实际上是数据字典表上的视图。参见 第 28.3.26 节，“INFORMATION_SCHEMA RESOURCE_GROUPS 表”。

最初，`RESOURCE_GROUPS` 表中有这些行描述默认组：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.RESOURCE_GROUPS\G
*************************** 1\. row ***************************
   RESOURCE_GROUP_NAME: USR_default
   RESOURCE_GROUP_TYPE: USER
RESOURCE_GROUP_ENABLED: 1
              VCPU_IDS: 0-3
       THREAD_PRIORITY: 0
*************************** 2\. row ***************************
   RESOURCE_GROUP_NAME: SYS_default
   RESOURCE_GROUP_TYPE: SYSTEM
RESOURCE_GROUP_ENABLED: 1
              VCPU_IDS: 0-3
       THREAD_PRIORITY: 0
```

`THREAD_PRIORITY` 值为 0，表示默认优先级。`VCPU_IDS` 值显示一个包含所有可用 CPU 的范围。对于默认组，显示的值取决于 MySQL 服务器运行的系统。

早些讨论提到了一个名为`Batch`的资源组来管理不需要以高优先级执行的批处理作业的情景。要创建这样一个组，使用类似于以下语句：

```sql
CREATE RESOURCE GROUP Batch
  TYPE = USER
  VCPU = 2-3            -- assumes a system with at least 4 CPUs
  THREAD_PRIORITY = 10;
```

要验证资源组是否按预期创建，请检查`RESOURCE_GROUPS`表：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.RESOURCE_GROUPS
       WHERE RESOURCE_GROUP_NAME = 'Batch'\G
*************************** 1\. row ***************************
   RESOURCE_GROUP_NAME: Batch
   RESOURCE_GROUP_TYPE: USER
RESOURCE_GROUP_ENABLED: 1
              VCPU_IDS: 2-3
       THREAD_PRIORITY: 10
```

如果`THREAD_PRIORITY`值为 0 而不是 10，请检查您的平台或系统配置是否限制了资源组功能；请参阅资源组限制。

要将线程分配给`Batch`组，请执行以下操作：

```sql
SET RESOURCE GROUP Batch FOR *thread_id*;
```

之后，命名线程中的语句将使用`Batch`组资源执行。

如果会话的当前线程应该在`Batch`组中，那么在会话中执行此语句：

```sql
SET RESOURCE GROUP Batch;
```

之后，会话中的语句将使用`Batch`组资源执行。

要使用`Batch`组执行单个语句，请使用`RESOURCE_GROUP`优化器提示：

```sql
INSERT /*+ RESOURCE_GROUP(Batch) */ INTO t2 VALUES(2);
```

分配给`Batch`组的线程将使用其资源执行，可以根据需要进行修改：

+   在系统负载高的时候，减少分配给组的 CPU 数量，降低其优先级，或者（如所示）两者都做：

    ```sql
    ALTER RESOURCE GROUP Batch
      VCPU = 3
      THREAD_PRIORITY = 19;
    ```

+   在系统负载轻的时候，增加分配给组的 CPU 数量，提高其优先级，或者（如所示）两者都做：

    ```sql
    ALTER RESOURCE GROUP Batch
      VCPU = 0-3
      THREAD_PRIORITY = 0;
    ```

#### 资源组复制

资源组管理是在发生的服务器上本地的。资源组 SQL 语句和对`resource_groups`数据字典表的修改不会被写入二进制日志，也不会被复制。

#### 资源组限制

在某些平台或 MySQL 服务器配置中，资源组不可用或存在限制：

+   如果安装了线程池插件，则资源组不可用。

+   在 macOS 上，资源组不可用，因为它没有提供将 CPU 绑定到线程的 API。

+   在 FreeBSD 和 Solaris 上，资源组线程优先级会被忽略。（实际上，所有线程都以优先级 0 运行。）尝试更改优先级会导致警告：

    ```sql
    mysql> ALTER RESOURCE GROUP abc THREAD_PRIORITY = 10;
    Query OK, 0 rows affected, 1 warning (0.18 sec)

    mysql> SHOW WARNINGS;
    +---------+------+-------------------------------------------------------------+
    | Level   | Code | Message                                                     |
    +---------+------+-------------------------------------------------------------+
    | Warning | 4560 | Attribute thread_priority is ignored (using default value). |
    +---------+------+-------------------------------------------------------------+
    ```

+   在 Linux 上，除非设置了`CAP_SYS_NICE`能力，否则将忽略资源组线程优先级。授予进程`CAP_SYS_NICE`能力将启用一系列特权；请参考[`man7.org/linux/man-pages/man7/capabilities.7.html`](http://man7.org/linux/man-pages/man7/capabilities.7.html)获取完整列表。在启用此能力时请小心。

    在使用 systemd 和内核支持环境能力（Linux 4.3 或更新版本）的 Linux 平台上，启用`CAP_SYS_NICE`能力的推荐方法是修改 MySQL 服务文件，保持**mysqld**二进制文件不变。要调整 MySQL 的服务文件，请使用以下步骤：

    1.  根据您的平台运行适当的命令：

        +   Oracle Linux、Red Hat 和 Fedora 系统：

            ```sql
            $> sudo systemctl edit mysqld
            ```

        +   SUSE、Ubuntu 和 Debian 系统：

            ```sql
            $> sudo systemctl edit mysql
            ```

    1.  使用编辑器，将以下文本添加到服务文件中：

        ```sql
        [Service]
        AmbientCapabilities=CAP_SYS_NICE
        ```

    1.  重新启动 MySQL 服务。

    如果无法像刚才描述的那样启用`CAP_SYS_NICE`功能，则可以使用**setcap**命令手动设置，指定路径名到**mysqld**可执行文件（这需要**sudo**访问权限）。您可以使用**getcap**检查权限。例如：

    ```sql
    $> sudo setcap cap_sys_nice+ep */path/to/mysqld*
    $> getcap */path/to/mysqld*
    */path/to/mysqld* = cap_sys_nice+ep
    ```

    作为安全措施，将**mysqld**二进制文件的执行权限限制为`root`用户和具有`mysql`组成员资格的用户：

    ```sql
    $> sudo chown root:mysql */path/to/mysqld*
    $> sudo chmod 0750 */path/to/mysqld*
    ```

    重要提示

    如果需要手动使用**setcap**，则必须在每次重新安装后执行。

+   在 Windows 上，线程以五个线程优先级级别之一运行。资源组线程优先级范围从-20 到 19 映射到以下表中指示的级别。

    **表 7.6 Windows 上的资源组线程优先级**

    | 优先级范围 | Windows 优先级级别 |
    | --- | --- |
    | -20 到 -10 | `THREAD_PRIORITY_HIGHEST` |
    | -9 到 -1 | `THREAD_PRIORITY_ABOVE_NORMAL` |
    | 0 | `THREAD_PRIORITY_NORMAL` |
    | 1 到 10 | `THREAD_PRIORITY_BELOW_NORMAL` |
    | 11 到 19 | `THREAD_PRIORITY_LOWEST` |
