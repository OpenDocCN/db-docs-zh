# 25.2.4 MySQL NDB Cluster 8.0 中的新功能

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-what-is-new.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-what-is-new.html)

以下各节描述了与早期版本系列相比，MySQL NDB Cluster 在 NDB Cluster 8.0 至 8.0.35 中的实现变化。NDB Cluster 8.0 作为一个通用可用性（GA）版本发布，从 NDB 8.0.19 开始。NDB Cluster 7.6 和 7.5 是之前的 GA 版本，仍在生产中得到支持；有关 NDB Cluster 7.6 的信息，请参阅 What is New in NDB Cluster 7.6。有关 NDB Cluster 7.5 的类似信息，请参阅 What is New in NDB Cluster 7.5。NDB Cluster 7.4 和 7.3 是之前的 GA 版本，已经到达生命周期终点，不再得到支持或维护。我们建议新的生产部署使用 MySQL NDB Cluster 8.0。

#### MySQL NDB Cluster 8.0 中的新功能

NDB Cluster 8.0 中的重大变化和新功能可能会引起兴趣，如下列表所示：

+   **兼容性增强。** 以下更改减少了`NDB`行为与其他 MySQL 存储引擎相比的长期非必要差异：

    +   **与 MySQL 服务器并行开发。** 从此版本开始，MySQL NDB Cluster 正在与标准 MySQL 8.0 服务器并行开发，采用新的统一发布模型，具有以下特点：

        +   NDB 8.0 是在 MySQL 8.0 源代码树中开发、构建和发布的。

        +   NDB Cluster 8.0 的编号方案遵循 MySQL 8.0 的方案。

        +   使用`NDB`支持构建源代码会在**mysql** `-V`返回的版本字符串末尾添加`-cluster`，如下所示：

            ```sql
            $> mysql -V
            mysql  Ver 8.0.35-cluster for Linux on x86_64 (Source distribution)
            ```

            `NDB`二进制文件继续显示 MySQL 服务器版本和`NDB`引擎版本，如下所示：

            ```sql
            $> ndb_mgm -V
            MySQL distrib mysql-8.0.35 ndb-8.0.35, for Linux (x86_64)
            ```

            在 MySQL Cluster NDB 8.0 中，这两个版本号始终相同。

        要使用 NDB Cluster 支持构建 MySQL 源代码，请使用 CMake 选项`-DWITH_NDB`（NDB 8.0.31 及更高版本；对于早期版本，请改用`-DWITH_NDBCLUSTER`）。

    +   **平台支持说明。** NDB 8.0 在平台支持方面做出以下更改：

        +   `NDBCLUSTER`不再支持 32 位平台。从 NDB 8.0.21 开始，NDB 构建过程会检查系统架构，并在不是 64 位平台时中止。

        +   现在可以为 64 位`ARM` CPU 从源代码构建`NDB`。目前，此支持仅限源代码，我们不为此平台提供任何预编译的二进制文件。

    +   **数据库和表名。** NDB 8.0 删除了先前对数据库和表标识符的 63 字节限制。这些标识符现在可以使用最多 64 字节，就像使用其他 MySQL 存储引擎的对象一样。请参阅 Section 25.2.7.11, “Previous NDB Cluster Issues Resolved in NDB Cluster 8.0”。

    +   **外键生成的名称。** `NDB` 现在使用模式 `*`tbl_name`*_fk_*`N`*` 用于内部生成的外键命名。这类似于 `InnoDB` 使用的模式。

+   **模式和元数据的分发和同步。** NDB 8.0 使用 MySQL 数据字典将模式信息分发给加入集群的 SQL 节点，并在现有 SQL 节点之间同步新的模式更改。以下列表描述了与此集成工作相关的各个增强功能：

    +   **模式分发增强。** `NDB` 模式分发协调器，负责处理模式操作并跟踪其进度，在 NDB 8.0 中已扩展，以确保在模式操作结束时释放使用的资源。以前，一些工作是由模式分发客户端完成的；由于客户端并非总是具有所有所需的状态信息，这可能导致在客户端决定在完成之前放弃模式操作并且未通知协调器的情况下导致资源泄漏。

        为了帮助解决这个问题，模式操作超时检测已从模式分发客户端移至协调器，使协调器有机会在模式操作期间清理任何使用的资源。协调器现在定期检查正在进行的模式操作是否超时，并在检测到超时时将尚未完成给定模式操作的参与者标记为失败。每当发生模式操作超时时，它还会提供适当的警告。（值得注意的是，在检测到超时后，模式操作本身会继续进行。）每当一个或多个这些操作正在进行时，定期打印活动模式操作列表以进行额外报告。

        作为这项工作的额外部分，一个新的 **mysqld** 选项 `--ndb-schema-dist-timeout` 可以设置等待模式操作被标记为超时的时间长度。

    +   **磁盘数据文件分发。** NDB Cluster 8.0.14 使用 MySQL 数据字典确保磁盘数据文件和相关构造（如表空间和日志文件组）在所有连接的 SQL 节点之间正确分发。

    +   **表空间对象的模式同步。** 当 MySQL 服务器作为 SQL 节点连接到 NDB 集群时，它会将其数据字典与`NDB`数据字典中的信息进行比较。

        以前，新 SQL 节点连接时同步的唯一`NDB`对象是数据库和表；MySQL NDB Cluster 8.0 还实现了磁盘数据对象的模式同步，包括表空间和日志文件组。除其他好处外，这消除了在本地备份和恢复后 MySQL 数据字典和`NDB`数据字典之间不匹配的可能性，其中表空间和日志文件组被恢复到`NDB`数据字典，但未恢复到 MySQL 服务器的数据字典。

        现在不再可能发出引用不存在表空间的`CREATE TABLE`语句。这样的语句现在会失败并显示错误。

    +   **数据库 DDL 同步增强。** 为 NDB 8.0 所做的工作确保了新加入（或重新加入）SQL 节点与现有 SQL 节点上的数据库的同步现在正确地利用数据字典，以便任何可能被该 SQL 节点忽略的数据库级操作（`CREATE DATABASE`、`ALTER DATABASE`或`DROP DATABASE`）在连接（或重新连接）到集群时现在在其上正确地复制。

        作为启动时执行的模式同步过程的一部分，SQL 节点现在会将集群数据节点上的所有数据库与其自己的数据字典进行比较，如果发现任何数据库在 SQL 节点的数据字典中缺失，则 SQL 节点通过执行`CREATE DATABASE`语句在本地安装它。因此，所创建的数据库使用的是在执行该语句时在此 SQL 节点上生效的默认 MySQL 服务器数据库属性（例如由`character_set_database`和`collation_database`确定的属性）。

    +   **NDB 元数据更改检测和同步。** NDB 8.0 实现了一种新的机制，用于检测数据对象（如表、表空间和日志文件组）的元数据更新与 MySQL 数据字典之间的不一致。这是通过一个线程，即`NDB`元数据更改监视线程，在后台定期检查`NDB`数据字典和 MySQL 数据字典之间的不一致性来完成的。

        默认情况下，监视器每隔`60 秒`执行一次元数据检查。可以通过设置`ndb_metadata_check_interval`系统变量的值来调整轮询间隔；通过将`ndb_metadata_check`系统变量设置为`OFF`可以完全禁用轮询。状态变量`Ndb_metadata_detected_count`显示自上次启动**mysqld**以来检测到不一致性的次数。

        `NDB`确保在启动后的操作期间，由元数据更改监视器线程提交的`NDB`数据库、表、日志文件组和表空间对象会被`NDB` binlog 线程自动检查不匹配并同步。

        NDB 8.0 增加了两个与自动同步相关的状态变量：`Ndb_metadata_synced_count`显示自动同步的对象数量；`Ndb_metadata_excluded_count`指示同步失败的对象数量（在 NDB 8.0.22 之前，此变量名为`Ndb_metadata_blacklist_size`）。此外，您可以通过检查集群日志查看已同步的对象。

        将`ndb_metadata_sync`系统变量设置为`true`会覆盖为`ndb_metadata_check_interval`和`ndb_metadata_check`所做的任何设置，导致更改监视器线程开始连续元数据更改检测。

        在 NDB 8.0.22 及更高版本中，将`ndb_metadata_sync`设置为`true`会清除先前同步失败的对象列表，这意味着不再需要发现单个表或通过重新连接 SQL 节点到集群来重新触发同步。此外，将此变量设置为`false`会清除等待重新尝试的对象列表。

        从 NDB 8.0.21 开始，比日志消息或状态变量提供有关自动同步当前状态的更详细信息的两个新表已添加到 MySQL 性能模式中。这些表在此列出：

        +   `ndb_sync_pending_objects`：包含在`NDB`字典和 MySQL 数据字典之间检测到不匹配的数据库对象信息（且未被排除在自动同步之外）。

        +   `ndb_sync_excluded_objects`：包含由于无法在 `NDB` 字典和 MySQL 数据字典之间同步而被排除的 `NDB` 数据库对象的信息，因此需要手动干预。

        这些表中的一行提供了数据库对象的父模式、名称和类型。对象类型包括模式、表空间、日志文件组和表。 （如果对象是日志文件组或表空间，则父模式为 `NULL`。）此外，`ndb_sync_excluded_objects` 表显示了对象被排除的原因。

        这些表仅在启用了 `NDBCLUSTER` 存储引擎支持时存在。有关这些表的更多信息，请参阅 第 29.12.12 节，“性能模式 NDB Cluster 表”。

    +   **NDB 表额外元数据的更改。** `NDB` 表的额外元数据属性用于存储来自 MySQL 数据字典的序列化元数据，而不是像以前版本中那样存储表的二进制表示。（这是一个 `.frm` 文件，MySQL Server 不再使用—参见 第十六章，*MySQL 数据字典*。）作为支持此更改的一部分，表的额外元数据的可用大小已增加。这意味着在 NDB Cluster 8.0 中创建的 `NDB` 表与以前的 NDB Cluster 版本不兼容。在以前的版本中创建的表可以在 NDB 8.0 中使用，但之后不能被较早版本打开。

        可以使用 NDB API 方法 `getExtraMetadata()` 和 `setExtraMetadata()` 访问此元数据。

        欲了解更多信息，请参阅 第 25.3.7 节，“NDB Cluster 升级和降级”。

    +   **使用 .frm 文件进行表的即时升级。** 在 NDB 7.6 及更早版本中创建的表包含以压缩的 `.frm` 文件形式存储的元数据，而这在 MySQL 8.0 中已不再支持。为了方便在线升级到 NDB 8.0，`NDB` 对此元数据进行即时转换，并将其写入 MySQL Server 的数据字典中，从而使 NDB Cluster 8.0 中的 **mysqld** 能够与该表一起工作，而不会阻止先前版本的 `NDB` 软件继续使用该表。

        重要提示

        一旦表的结构在 NDB 8.0 中被修改，其元数据将使用数据字典存储，之后将无法被 NDB 7.6 及更早版本访问。

        这一增强还使得可以将使用早期版本创建的`NDB`备份还原到运行 NDB 8.0（或更高版本）的集群中。

    +   **元数据一致性检查错误日志。** 作为之前在 NDB 8.0 中完成的工作的一部分，元数据检查作为在 NDB 字典中的`NDB`表与其在 MySQL 数据字典中的对应之间的自动同步的一部分执行，包括表的名称、存储引擎和内部 ID。 从 NDB 8.0.23 开始，所检查的属性范围扩展到包括以下数据对象的属性：

        +   列

        +   索引

        +   外键

        此外，现在任何元数据属性不匹配的详细信息都将写入 MySQL 服务器错误日志。 根据不一致性是在表级别还是在列、索引或外键级别发现的，错误日志消息的格式略有不同。 表级属性不匹配导致的日志错误的格式如下，其中*`property`*是属性名称，*`ndb_value`*是存储在 NDB 字典中的属性值，*`mysqld_value`*是存储在 MySQL 数据字典中的属性值：

        ```sql
        Diff in '*property*' detected, '*ndb_value*' != '*mysqld_value*'
        ```

        对于列、索引和外键属性不匹配的情况，格式如下，其中*`obj_type`*是`column`、`index`或`foreign key`之一，*`obj_name`*是对象的名称：

        ```sql
        Diff in *obj_type* '*obj_name*.*property*' detected, '*ndb_value*' != '*mysqld_value*'
        ```

        在将`NDB`表安装到充当 NDB 集群中 SQL 节点的任何**mysqld**的数据字典中时，将执行元数据检查。 如果**mysqld**是调试编译的，则还会在执行`CREATE TABLE`语句时进行检查，并在打开`NDB`表时进行检查。

+   **与 NDB_STORED_USER 同步用户权限。** 在 NDB 8.0 中提供了一种新的机制，使用`NDB_STORED_USER`权限在 SQL 节点之间共享和同步用户、角色和权限。 NDB 7.6 及更早版本中实现的分布式权限（请参阅使用共享授权表进行分布式权限）不再受支持。

    一旦在 SQL 节点上创建了用户帐户，用户及其权限可以存储在`NDB`中，并通过发出类似于以下语句的`GRANT`语句在集群中的所有 SQL 节点之间共享：

    ```sql
    GRANT NDB_STORED_USER ON *.* TO 'jon'@'localhost';
    ```

    `NDB_STORED_USER`始终具有全局范围，并且必须使用`ON *.*`授予。 诸如`mysql.session@localhost`或`mysql.infoschema@localhost`之类的系统保留帐户不能被分配此权限。

    通过发出适当的 `GRANT NDB_STORED_USER` 语句，角色也可以在 SQL 节点之间共享。将此类角色分配给用户不会导致用户共享；必须显式地向每个用户授予 `NDB_STORED_USER` 权限。

    拥有 `NDB_STORED_USER` 的用户或角色及其权限会在加入给定 NDB 集群的所有 SQL 节点之后立即共享。可以从任何连接的 SQL 节点进行此类更改，但建议仅从指定的 SQL 节点进行，因为无法保证不同 SQL 节点影响权限的语句执行顺序在所有 SQL 节点上相同。

    在 NDB 8.0.27 之前，用户或角色的权限更改会立即与所有连接的 SQL 节点同步。从 MySQL 8.0.27 开始，更新权限时，SQL 节点会获取全局读锁，以防止多个 SQL 节点执行的并发更改导致死锁。

    **升级的影响。** 由于 MySQL 服务器权限系统的更改（请参阅第 8.2.3 节，“授权表”），在 NDB 8.0 中，使用 `NDB` 存储引擎的权限表无法正常运行。保留在 NDB 7.6 或更早版本中创建的此类权限表是安全的，但不是必需的，因为它们不再用于访问控制。在 NDB 8.0 中，作为 SQL 节点的 **mysqld** 检测到这些表在 `NDB` 中时，会向 MySQL 服务器日志写入警告，并在本地创建 `InnoDB` 阴影表；这样的阴影表会在连接到集群的每个 MySQL 服务器上创建。在从 NDB 7.6 或更早版本升级时，可以在所有作为 SQL 节点的 MySQL 服务器升级后安全地使用 **ndb_drop_table** 删除使用 `NDB` 的权限表（请参阅第 25.3.7 节，“升级和降级 NDB 集群”）。

    **ndb_restore** 实用程序的 `--restore-privilege-tables` 选项已被弃用，但在 NDB 8.0 中仍然受到尊重，并且仍然可以用于将从先前版本的 NDB 集群备份中获取的分布式权限表恢复到运行 NDB 8.0 的集群中。这些表的处理方式如前一段所述。

    共享用户和授权存储在`ndb_sql_metadata`表中，默认情况下，**ndb_restore**在 NDB 8.0 中不会恢复这些内容；您可以指定`--include-stored-grants`选项来执行此操作。

    更多信息请参见第 25.6.13 节，“特权同步和 NDB_STORED_USER”。

+   **INFORMATION_SCHEMA 更改。** 在 Information Schema `FILES`表中关于磁盘数据文件信息的显示进行了以下更改：

    +   表空间和日志文件组不再在`FILES`表中表示。（这些构造实际上不是文件。）

    +   每个数据文件现在在`FILES`表中由一行表示。每个撤销日志文件也仅由一行在此表中表示。（以前，在每个数据节点上的每个这些文件的副本都显示为一行。）

    此外，`INFORMATION_SCHEMA`表现在为 MySQL Cluster 表填充表空间统计信息。（Bug #27167728）

+   **使用 ndb_perror 获取错误信息。** 已删除了**perror**的已弃用`--ndb`选项。请改用**ndb_perror**来从`NDB`错误代码获取错误消息信息。（Bug #81704, Bug #81705, Bug #23523926, Bug #23523957）

+   **条件推送增强。** 以前，条件推送仅限于将条件推送到引用来自与条件所推送的表相同的列值的谓词项。在 NDB 8.0 中，取消了此限制，使得可以从查询计划中较早的表中引用列值。NDB 8.0 支持比较列表达式的连接，以及在同一表中比较列。要比较的列和列表达式必须完全相同类型；这意味着它们在适用这些属性时也必须具有相同的符号、长度、字符集、精度和比例。在 NDB 8.0.27 之前，被推送的条件不能是推送连接的一部分，当解除此限制时。

    下推更大部分条件允许数据节点过滤更多行，从而减少**mysqld**在连接处理期间必须处理的行数。这些增强的另一个好处是过滤可以在 LDM 线程中并行执行，而不是在 SQL 节点上的单个 mysqld 进程中执行；这有可能显著提高查询性能。

    继续适用于被比较的列值之间的类型兼容性的现有规则（参见第 10.2.1.5 节，“引擎条件下推优化”）。

    **外连接和半连接的下推。** 在 NDB 8.0.20 中完成的工作允许许多外连接和半连接（不仅仅是使用主键或唯一键查找的连接）被下推到数据节点（参见第 10.2.1.5 节，“引擎条件下推优化”）。

    现在可以下推的扫描外连接包括符合以下条件的连接：

    +   表中没有未下推的条件

    +   在同一连接嵌套中的其他表或依赖它的上层连接嵌套中没有未下推的条件

    +   同一连接嵌套中的所有其他表，或依赖它的上层连接嵌套，也被下推

    如果半连接使用索引扫描，并且符合刚才提到的被下推外连接的条件，并且使用`firstMatch`策略，那么现在可以将其下推（参见第 10.2.2.1 节，“使用半连接转换优化 IN 和 EXISTS 子查询谓词”）。

    这些额外的改进是在 NDB 8.0.21 中完成的：

    +   通过转换`NOT EXISTS`和`NOT IN`查询而由 MySQL 优化器生成的反连接（参见第 10.2.2.1 节，“使用半连接转换优化 IN 和 EXISTS 子查询谓词”）可以被`NDB`下推到数据节点。

        当表中没有未下推的条件，并且查询满足外连接下推必须满足的任何其他条件时，可以执行此操作。

    +   `NDB`在尝试从附加的表中检索任何行之前，会尝试识别和评估一个非依赖标量子查询。当它能够这样做时，获得的值将作为下推条件的一部分使用，而不是使用提供值的子查询。

    从 NDB 8.0.27 开始，作为下推查询的一部分下推的条件现在可以引用同一下推查询中祖先表的列，但必须满足以下条件：

    +   推送的条件可能包括任何比较运算符`<`，`<=`，`>`，`>=`，`=`，和`<>`。

    +   被比较的值必须是相同类型，包括长度、精度和比例。

    +   `NULL`处理根据 ISO SQL 标准指定的比较语义执行；任何与`NULL`的比较都返回`NULL`。

    考虑使用此处所示语句创建的表：

    ```sql
    CREATE TABLE t (
        x INT PRIMARY KEY, 
        y INT
    ) ENGINE=NDB;
    ```

    查询语句如`SELECT * FROM t AS m JOIN t AS n ON m.x >= n.y`现在可以使用引擎条件推送优化来推送条件列`y`。

    当无法推送连接时，`EXPLAIN`应提供原因或原因。

    更多信息请参见 Section 10.2.1.5, “引擎条件推送优化”。

    NDB API 方法`branch_col_eq_param()`，`branch_col_ne_param()`，`branch_col_lt_param()`，`branch_col_le_param()`，`branch_col_gt_param()`，和`branch_col_ge_param()`在 NDB 8.0.27 中作为这项工作的一部分添加。这些`NdbInterpretedCode`可用于比较列值与参数值。

    此外，`NdbScanFilter::cmp_param()`，也在 NDB 8.0.27 中添加，使得可以定义列值和参数值之间的比较，用于执行扫描。

+   **最大行大小增加。** NDB 8.0 将`NDBCLUSTER`表中可存储的最大字节数从 14000 增加到 30000 字节。

    `BLOB`或`TEXT`列继续使用此总数的 264 字节，与以前一样。

    `NDB`表的固定宽度列的最大偏移量为 8188 字节；这与之前的版本保持不变。

    有关更多信息，请参见第 25.2.7.5 节，“NDB 集群中数据库对象相关的限制”。

+   **ndb_mgm SHOW 命令和单用户模式。** 在 NDB 8.0 中，当集群处于单用户模式时，管理客户端的 `SHOW` 命令的输出指示在此模式下哪个 API 或 SQL 节点具有独占访问权限。

+   **在线列重命名。** 现在可以在线重命名`NDB`表的列，使用`ALGORITHM=INPLACE`。有关更多信息，请参见第 25.6.12 节，“NDB 集群中 ALTER TABLE 的在线操作”。

+   **改进的 ndb_mgmd 启动时间。** 在 NDB 8.0 中，管理节点守护程序的启动时间已经得到显著改善，具体体现在以下方面：

    +   由于将`ndb_mgmd`以前用于处理配置数据中节点属性的列表数据结构替换为哈希表，管理服务器的整体启动时间已经减少了 6 倍或更多。

    +   此外，在集群配置文件中使用的数据和 SQL 节点主机名未出现在管理服务器的`hosts`文件中时，**ndb_mgmd** 的启动时间可能比以前短多达 20 倍。

+   **NDB API 增强。** `NdbScanFilter::cmp()` 和 `NdbInterpretedCode` 的几种比较方法现在可以用于比较表列值。受影响的`NdbInterpretedCode`方法在此列出：

    +   `branch_col_eq()`

    +   `branch_col_ge()`

    +   `branch_col_gt()`

    +   `branch_col_le()`

    +   `branch_col_lt()`

    +   `branch_col_ne()`

    对于上述列出的所有方法，要比较的表列值必须完全匹配类型，包括长度、精度、有符号性、比例、字符集和排序规则（如适用）。

    有关更多信息，请参见各个 API 方法的描述。

+   **离线多线程索引构建。** 现在可以指定一组核心用于执行离线多线程构建有序索引的 I/O 线程，而不是执行正常的 I/O 任务，如文件 I/O，压缩或解压缩。在这种情况下，“离线”指的是在父表未被写入时执行有序索引构建；这种构建发生在`NDB`集群执行节点或系统重启时，或作为使用**ndb_restore** `--rebuild-indexes`从备份还原集群的一部分时。

    此外，离线索引构建工作的默认行为已修改为使用所有可用于**ndbmtd**")的核心，而不仅限于为 I/O 线程保留的核心。这样做可以提高重启和还原时间以及性能、可用性和用户体验。

    此增强功能的实现如下：

    1.  `BuildIndexThreads`的默认值从 0 更改为 128。这意味着离线有序索引构建现在默认是多线程的。

    1.  `TwoPassInitialNodeRestartCopy`的默认值从`false`更改为`true`。这意味着初始节点重启首先将所有数据从“活动”节点复制到正在启动的节点，而不创建任何索引，然后离线构建有序索引，然后再次将其数据与活动节点同步，即在两次同步之间进行离线索引构建。这使得初始节点重启的行为更像节点的正常重启，并减少了构建索引所需的时间。

    1.  为`ThreadConfig`配置参数定义了一个新的线程类型（`idxbld`），以允许将离线索引构建线程锁定到特定的 CPU。

    此外，`NDB`现在通过这两个标准区分可访问`ThreadConfig`的线程类型：

    1.  线程是否是执行线程。`main`，`ldm`，`recv`，`rep`，`tc`和`send`类型的线程是执行线程；`io`，`watchdog`和`idxbld`类型的线程不是。

    1.  线程分配给特定任务是永久的还是临时的。目前除了`idxbld`之外的所有线程类型都是永久的。

    要获取更多信息，请参阅手册中指定参数的描述。（Bug #25835748, Bug #26928111）

+   **logbuffers 表备份过程信息。** 在执行 NDB 备份时，`ndbinfo.logbuffers` 表现在显示每个数据节点上备份过程中缓冲区使用情况的信息。这通过反映两种新的日志类型的行来实现，除了 `REDO` 和 `DD-UNDO`。其中一行具有日志类型 `BACKUP-DATA`，显示备份期间用于将片段复制到备份文件的数据缓冲区的使用量。另一行具有日志类型 `BACKUP-LOG`，显示备份期间用于记录备份开始后所做更改的日志缓冲区的使用量。在集群中的每个数据节点的 `logbuffers` 表中显示这两种 `log_type` 行。只有在 NDB 备份正在进行时，表中才存在具有这两种日志类型的行。 (Bug #25822988)

+   **Windows 上的 ndbinfo.processes 表。** 在 Windows 平台上由 `RESTART` 用于生成和重新启动 **mysqld** 的监视进程的进程 ID 现在在 `processes` 表中显示为 `angel_pid`。

+   **字符串哈希改进。** 在 NDB 8.0 之前，所有字符串哈希都是基于首先将字符串转换为规范化形式，然后对结果的二进制图像进行 MD5 哈希。由于以下原因，这可能会导致一些性能问题：

    +   规范化字符串始终会被填充到其完整长度。对于 `VARCHAR`，这通常涉及添加比原始字符串中的字符更多的空格。

    +   字符串库未针对这种空格填充进行优化，在某些情况下增加了相当大的开销。

    +   填充语义在字符集之间有所不同，其中一些字符集没有填充到其完整长度。

    +   转换后的字符串可能会变得非常庞大，即使没有空格填充；一些 Unicode 9.0 的排序规则可以将单个代码点转换为 100 字节甚至更多的字符数据。

    +   后续的 MD5 哈希主要是通过填充空格来实现的，并且效率不是特别高，可能会通过刷新 L1 缓存的大部分内容导致额外的性能损失。

    排序规则提供了自己的哈希函数，直接对字符串进行哈希而不是首先创建规范化字符串。此外，对于 Unicode 9.0 排序规则，哈希是在不填充的情况下计算的。`NDB` 现在在哈希一个被识别为使用 Unicode 9.0 排序规则的字符串时利用这个内置函数。

    由于对于其他排序规则，存在着已经在转换后的字符串上进行哈希分区的现有数据库，`NDB`继续采用先前用于对这些字符串进行哈希的方法，以保持兼容性。（Bug #89590，Bug #89604，Bug #89609，Bug #27515000，Bug #27523758，Bug #27522732）

+   **RESET MASTER 更改。** 因为 MySQL 服务器现在使用全局读锁执行`RESET MASTER`，所以当与 NDB Cluster 一起使用时，此语句的行为在以下两个方面发生了变化：

    +   不再保证是同步的；也就是说，现在可能会出现在发出`RESET MASTER`之前立即进行的读取操作可能要等到二进制日志被轮换后才被记录。

    +   现在无论语句是在写入二进制日志的相同 SQL 节点上执行，还是在同一集群中的不同 SQL 节点上执行，它的行为都完全相同。

    注意

    `SHOW BINLOG EVENTS`，`FLUSH LOGS`，以及大多数数据定义语句继续像以前的`NDB`版本一样以同步方式运行。

+   **ndb_restore 选项用法。** 在调用**ndb_restore**时，现在需要同时使用`--nodeid`和`--backupid`选项。

+   **ndb_log_bin 默认值。** NDB 8.0 将`ndb_log_bin`系统变量的默认值从`TRUE`更改为`FALSE`。

+   **动态事务资源分配。** 现在，事务协调器中的资源分配是使用动态内存池进行的。这意味着资源分配由数据节点配置参数决定，例如`MaxDMLOperationsPerTransaction`、`MaxNoOfConcurrentIndexOperations`、`MaxNoOfConcurrentOperations`、`MaxNoOfConcurrentScans`、`MaxNoOfConcurrentTransactions`、`MaxNoOfFiredTriggers`、`MaxNoOfLocalScans`和`TransactionBufferMemory`现在以这样的方式进行，即如果每个参数所代表的负载都在所有这些资源的目标负载范围内，那么其他这些资源可以被限制，以使总资源不超过可用资源。

    作为这项工作的一部分，已添加了几个新的数据节点参数，用于控制`DBTC`中的事务资源，列在这里：

    +   `ReservedConcurrentIndexOperations`

    +   `ReservedConcurrentOperations`

    +   `ReservedConcurrentScans`

    +   `ReservedConcurrentTransactions`

    +   `ReservedFiredTriggers`

    +   `ReservedLocalScans`

    +   `ReservedTransactionBufferMemory`.

    有关刚才列出的参数的更多信息，请参阅其描述。

+   **每个数据节点使用多个本地数据管理器进行备份。** `NDB` 现在可以在每个数据节点上并行执行备份，使用多个本地数据管理器（LDMs）。 （以前，备份是在数据节点之间并行进行的，但在数据节点进程内部始终是串行的。）在 **ndb_mgm** 客户端的 `START BACKUP` 命令中不需要特殊的语法来启用此功能，但所有数据节点必须使用多个 LDMs。 这意味着数据节点必须运行 **ndbmtd**（**ndbd** 是单线程的，因此始终只有一个 LDM）并且在进行备份之前必须配置为使用多个 LDMs；您可以通过为多线程数据节点配置参数 `MaxNoOfExecutionThreads` 或 `ThreadConfig` 之一选择适当的设置来实现这一点。

    使用多个 LDMs 进行备份会在 `BACKUP/BACKUP-*backup_id*/` 目录下创建子目录，每个 LDM 一个。**ndb_restore** 现在会自动检测这些子目录，如果存在，则尝试并行恢复备份；详细信息请参见 第 25.5.23.3 节，“从并行备份中恢复”。 （单线程备份将像以前的 `NDB` 版本一样恢复。）还可以通过修改通常的恢复过程，使用先前版本的 NDB 集群的 **ndb_restore** 二进制文件来恢复并行备份；第 25.5.23.3.2 节，“串行恢复并行备份” 提供了如何执行此操作的信息。

    您可以通过在集群的全局配置文件（`config.ini`）的 `[ndbd default]` 部分为所有数据节点将 `EnableMultithreadedBackup` 数据节点参数设置为 0 来强制创建单线程备份。

+   **二进制配置文件增强。** NDB 8.0 使用了管理服务器二进制配置文件的新格式。以前，集群配置文件中最多可以出现 16381 个部分；现在，部分的最大数量为 4G。这旨在支持比以前更大数量的节点在集群中。

    升级到新格式相对无缝，并且很少需要手动干预，因为管理服务器仍然可以无问题地读取旧格式。从 NDB 8.0 降级到旧版本的 NDB 集群软件需要手动删除任何二进制配置文件，或者使用`--initial`选项启动旧管理服务器二进制文件。

    欲了解更多信息，请参阅第 25.3.7 节，“NDB 集群升级和降级”。

+   **增加数据节点数量。** NDB 8.0 将每个集群支持的数据节点最大数量增加到 144（以前为 48）。数据节点现在可以使用范围为 1 到 144 的节点 ID。

    以前，管理节点的推荐节点 ID 为 49 和 50。这些仍然支持用作管理节点，但将它们用作此类节点会限制数据节点的最大数量为 142；因此，现在建议将节点 ID 145 和 146 用于管理节点。

    作为这项工作的一部分，用于数据节点`sysfile`的格式已更新为版本 2。该文件记录了每个节点的最后全局检查点索引、重启状态和节点组成员资格等信息（参见 NDB 集群数据节点文件系统目录）。

+   **RedoOverCommitCounter 和 RedoOverCommitLimit 更改。** 由于将它们设置为 0 的语义存在歧义，因此数据节点配置参数`RedoOverCommitCounter`和`RedoOverCommitLimit`的最小值已增加到 1。

+   **ndb_autoincrement_prefetch_sz 更改。** `ndb_autoincrement_prefetch_sz`服务器系统变量的默认值增加到 512。

+   **参数最大值和默认值的更改。** NDB 8.0 对配置参数的最大值和默认值进行了以下更改：

    +   `DataMemory`的最大值增加到 16TB。

    +   `DiskPageBufferMemory`的最大值也增加到 16TB。

    +   默认值`StringMemory`已增加至 25%。

    +   `LcpScanProgressTimeout`的默认值已增加至 180 秒。

+   **磁盘数据检查点改进。** NDB Cluster 8.0 提供了许多新的增强功能，有助于在使用非易失性内存设备（如固态硬盘和 NVMe 规范的设备）时减少磁盘数据表和表空间的检查点延迟。这些改进包括以下内容：

    +   避免检查点磁盘写入的突发

    +   加速当重做日志或撤销日志变满时的磁盘数据表空间检查点

    +   在必要时平衡磁盘和内存检查点之间的检查点

    +   保护磁盘设备免受过载，以确保在高负载下低延迟

    作为这项工作的一部分，已添加了两个数据节点配置参数。`MaxDiskDataLatency`设定了磁盘访问允许的延迟上限，并导致超过此时间长度的事务被中止。`DiskDataUsingSameDisk`使得可以利用将磁盘数据表空间放置在不同磁盘上，从而增加这些表空间的检查点执行速度。

    此外，`ndbinfo`数据库中的三个新表提供有关磁盘数据性能的信息：

    +   `diskstat`表报告过去一秒钟对磁盘数据表空间的写入

    +   `diskstats_1sec`表报告过去 20 秒对磁盘数据表空间的写入

    +   `pgman_time_track_stats`表报告与磁盘数据表空间相关的磁盘操作的延迟

+   **内存分配和 TransactionMemory。** 新的`TransactionMemory`参数简化了数据节点内存用于事务的分配，作为汇总事务和本地数据管理器（LDM）内存的工作的一部分。该参数旨在取代已被弃用的几个旧事务内存参数。

    事务内存现在可以通过以下三种方式之一设置：

    +   几个配置参数与`TransactionMemory`不兼容。如果设置了其中任何一个，就无法设置`TransactionMemory`（请参阅与 TransactionMemory 不兼容的参数），并且数据节点的事务内存将如同在 NDB 8.0 之前一样确定。

        注意

        尝试在`config.ini`文件中同时设置`TransactionMemory`和任何这些参数会阻止管理服务器启动。

    +   如果设置了`TransactionMemory`，则该值用于确定事务内存。如果已设置了前面提到的任何不兼容参数，则无法设置`TransactionMemory`。

    +   如果没有设置任何不兼容的参数，并且也没有设置`TransactionMemory`，则事务内存由`NDB`设置。

    更多信息，请参阅`TransactionMemory`的描述，以及第 25.4.3.13 节，“数据节点内存管理”。

+   **支持额外的片段副本。** NDB 8.0 将在生产中支持的片段副本的最大数量从两个增加到四个。（以前，可以将`NoOfReplicas`设置为 3 或 4，但这在测试中并未得到官方支持或验证。）

+   **分片恢复。** 从 NDB 8.0.20 开始，可以将备份分成大致相等的部分（片段），并使用两个新选项并行恢复这些片段，这两个选项已经在**ndb_restore**中实现：

    +   `--num-slices`确定备份应分成的片段数。

    +   `--slice-id`提供要由当前实例的**ndb_restore**恢复的片段的 ID。

    这使得可以使用多个实例的**ndb_restore**并行恢复备份的子集，可能减少执行恢复操作所需的时间。

    更多信息，请参阅**ndb_restore**的`--num-slices`选项的描述。

+   **从任何片段副本读取已启用。** 对于所有`NDB`表，默认启用从任何片段副本读取。这意味着`ndb_read_backup`系统变量的默认值现在为 ON，并且在创建新的`NDB`表时，`NDB_TABLE`注释选项`READ_BACKUP`的值为 1。启用从任何片段副本读取显著提高了从`NDB`表读取的性能，对写入的影响很小。

    欲了解更多信息，请参阅`ndb_read_backup`系统变量的描述，以及 Section 15.1.20.12，“设置 NDB 注释选项”。

+   **ndb_blob_tool 增强。** 从 NDB 8.0.20 开始，**ndb_blob_tool**实用程序可以检测缺失的 blob 部分，其中存在内联部分，并用正确长度的占位符 blob 部分（由空格字符组成）替换这些部分。要检查是否存在缺失的 blob 部分，请使用此程序的`--check-missing`选项。要用占位符替换任何缺失的 blob 部分，请使用`--add-missing`选项。

    欲了解更多信息，请参阅 Section 25.5.6，“ndb_blob_tool — 检查和修复 NDB Cluster 表的 BLOB 和 TEXT 列”。

+   **ndbinfo 版本控制。** `NDB` 8.0.20 及更高版本支持`ndbinfo`表的版本控制，并在内部维护其表的当前定义。在启动时，`NDB`会将其支持的`ndbinfo`版本与数据字典中存储的版本进行比较。如果版本不同，`NDB`会删除任何旧的`ndbinfo`表，并使用当前定义重新创建它们。

+   **对 Fedora Linux 的支持。** 从 NDB 8.0.20 开始，Fedora Linux 是 NDB Cluster Community 版本的支持平台，并可以使用 Oracle 提供的 RPM 包进行安装。这些可以从[NDB Cluster 下载页面](https://dev.mysql.com/downloads/cluster/)获取。

+   **NDB 程序—NDBT 依赖项移除。** 已移除了许多`NDB`实用程序对`NDBT`库的依赖。该库在开发中内部使用，对于正常使用不需要；将其包含在这些程序中可能会导致测试时出现不希望的问题。

    受影响的程序列在此处，以及移除依赖项的`NDB`版本：

    +   **ndb_restore**

    +   **ndb_delete_all**

    +   **ndb_show_tables** (NDB 8.0.20)

    +   **ndb_waiter** (NDB 8.0.20)

    这一变化对用户的主要影响是，这些程序在运行完成后不再打印`NDBT_ProgramExit - *`status`*`。依赖于这种行为的应用程序在升级到指定版本时应该进行更新。

+   **外键和大小写。** `NDB` 使用外键名称的定义大小写存储这些名称。以前，当`lower_case_table_names`系统变量的值设置为 0 时，它对外键名称进行区分大小写比较，就像在`SELECT`和其他 SQL 语句中使用的名称与存储的名称一样。从 NDB 8.0.20 开始，无论`lower_case_table_names`的值如何，这样的比较现在总是以不区分大小写的方式进行。

+   **多个传输器。** NDB 8.0.20 引入了支持多个传输器，用于处理数据节点之间的节点间通信。这有助于集群中每个节点组的更新操作速率更高，并有助于避免使用单个套接字进行节点间通信时系统或其他限制所施加的约束。

    默认情况下，`NDB`现在根据本地数据管理（LDM）线程数或事务协调器（TC）线程数中较大的那个数来使用一定数量的传输器。默认情况下，传输器的数量等于这个数的一半。虽然默认情况下对大多数工作负载表现良好，但可以通过设置`NodeGroupTransporters`数据节点配置参数（也在 NDB 8.0.20 中引入）来调整每个节点组使用的传输器数量，最大值为 LDM 线程数或 TC 线程数中较大的那个数。将其设置为 0 会导致传输器的数量与 LDM 线程数相同。

+   **ndb_restore：主键模式更改。** NDB 8.0.21（及更高版本）在使用**ndb_restore**恢复 `NDB` 本机备份时，支持源表和目标表的不同主键定义，当使用`--allow-pk-changes`选项运行时。 支持增加和减少构成原始主键的列数。

    当主键使用额外列扩展时，添加的任何列必须定义为`NOT NULL`，并且在进行备份时，这些列中的任何值都不得更改。 因为一些应用程序在更新行时会设置所有列的值，无论实际上是否更改了所有值，这可能会导致恢复操作失败，即使要添加到主键的列中没有值发生更改。 您可以使用 NDB 8.0.21 中还添加的`--ignore-extended-pk-updates`选项覆盖此行为；在这种情况下，您必须确保不更改任何这样的值。

    无论这一列是否仍然是表的一部分，都可以从表的主键中删除一列。

    要获取更多信息，请参阅**ndb_restore**选项`--allow-pk-changes`的描述。

+   **使用 ndb_restore 合并备份。** 在某些情况下，可能希望将最初存储在不同 NDB 集群实例（都使用相同模式）中的数据合并到单个目标 NDB 集群中。 当使用**ndb_mgm**客户端创建的备份（请参阅第 25.6.8.2 节，“使用 NDB 集群管理客户端创建备份”）并使用**ndb_restore**进行恢复时，现在支持使用 NDB 8.0.21 中添加的`--remap-column`选项以及`--restore-data`（可能还需要或希望的其他兼容选项）。 `--remap-column` 可用于处理源集群之间主键和唯一键值重叠的情况，并且在目标集群中不重叠是必要的，以及保留表之间的其他关系，如外键。

    `--remap-column`的参数是一个字符串，格式为`*`db`*.*`tbl`*.*`col`*:*`fn`*:*`args`*`，其中*`db`*，*`tbl`*和*`col`*分别是数据库，表和列的名称，*`fn`*是重新映射函数的名称，*`args`*是一个或多个*`fn`*的参数。没有默认值。只支持`offset`作为函数名称，*`args`*是要在从备份插入目标表时应用到列值的整数偏移量。此列必须是`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")或`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")之一；偏移值的允许范围与该类型的有符号版本相同（如果需要，这允许偏移为负）。

    新选项可以在同一次调用**ndb_restore**中多次使用，这样您可以重新映射同一表的多个列，不同表或两者的新值。偏移值不必对所有选项实例相同。

    此外，还为**ndb_desc**提供了两个新选项，从 NDB 8.0.21 开始：

    +   `--auto-inc`（简写形式`-a`）：如果表具有`AUTO_INCREMENT`列，则在输出中包含下一个自增值。

    +   `--context`（简写形式`-x`）：提供有关表的额外信息，包括模式，数据库名称，表名称和内部 ID。

    有关更多信息和示例，请参阅`--remap-column`选项的描述。

+   **发送线程改进。** 从 NDB 8.0.20 开始，每个发送线程现在处理发送到一组传输器的发送，并且每个块线程现在只辅助一个发送线程，导致更多的发送线程，从而提高性能和数据节点的可伸缩性。

+   **使用 SpinMethod 进行自适应自旋控制。** 在支持的平台上设置自适应 CPU 自旋的简单接口，使用`SpinMethod`数据节点参数。该参数（在 NDB 8.0.20 中添加，从 NDB 8.0.24 开始生效）有四个设置，分别用于静态自旋、基于成本的自适应自旋、优化延迟的自适应自旋以及针对每个线程都有自己 CPU 的数据库机器进行优化的自适应自旋。这些设置中的每一个都会使数据节点使用一组预定值来设置一个或多个自旋参数，从而实现自适应自旋，设置自旋时间和自旋开销，适用于特定场景，从而无需为常见用例直接设置这些值。

    为了对自旋行为进行微调，还可以直接设置这些和其他自旋参数，使用现有的`SchedulerSpinTimer`数据节点配置参数以及在**ndb_mgm**客户端中的以下`DUMP`命令：

    +   `DUMP 104000 (SetSchedulerSpinTimerAll)`: 设置所有线程的自旋时间

    +   `DUMP 104001 (SetSchedulerSpinTimerThread)`: 设置指定线程的自旋时间

    +   `DUMP 104002 (SetAllowedSpinOverhead)`: 设置自旋开销，即允许获得 1 单位延迟所需的 CPU 时间单位数

    +   `DUMP 104003 (SetSpintimePerCall)`: 设置调用的自旋时间

    +   `DUMP 104004 (EnableAdaptiveSpinning)`: 启用或禁用自适应自旋

    NDB 8.0.20 还添加了一个新的 TCP 配置参数`TcpSpinTime`，用于设置给定 TCP 连接的自旋时间。

    **ndb_top** 工具也得到增强，以提供每个线程的自旋时间信息。

    欲了解更多信息，请参阅`SpinMethod`参数的描述，列出的`DUMP`命令以及 Section 25.5.29, “ndb_top — 查看 NDB 线程的 CPU 使用信息”。

+   **磁盘数据和集群重启。** 从 NDB 8.0.21 开始，集群的初始重启会强制删除所有磁盘数据对象，如表空间和日志文件组，包括与这些对象相关联的任何数据文件和撤销日志文件。

    更多信息请参见第 25.6.11 节，“NDB 集群磁盘数据表”。

+   **磁盘数据范围分配。** 从 NDB 8.0.20 开始，在数据文件中分配范围是在给定表空间中使用的所有数据文件之间以循环方式进行的。这有望改善在使用多个存储设备进行磁盘数据存储时数据的分布。

    更多信息请参见第 25.6.11.1 节，“NDB 集群磁盘数据对象”。

+   **--ndb-log-fail-terminate 选项。** 从 NDB 8.0.21 开始，您可以通过使用 `--ndb-log-fail-terminate` 选项启动 **mysqld**，使 SQL 节点在无法完全记录所有行事件时终止。

+   **AllowUnresolvedHostNames 参数。** 默认情况下，管理节点在无法解析全局配置文件中存在的主机名时拒绝启动，这在某些环境（如 Kubernetes）中可能会有问题。从 NDB 8.0.22 开始，可以通过在集群全局配置文件（`config.ini` 文件）的 `[tcp default]` 部分中将 `AllowUnresolvedHostNames` 设置为 `true` 来覆盖此行为。这样做会将此类错误视为警告，并允许 **ndb_mgmd** 继续启动

+   **Blob 写入性能增强。** NDB 8.0.22 实现了一些改进，允许在同一行中修改多个 blob 列或在同一语句中修改包含 blob 列的多行时更有效地进行批处理，从而减少 SQL 或其他 API 节点与数据节点之间在应用这些修改时所需的往返次数。因此，许多 `INSERT`、`UPDATE` 和 `DELETE` 语句的性能可以得到改善。这里列出了一些这样的语句示例，其中 *`table`* 是包含一个或多个 Blob 列的 `NDB` 表：

    +   `INSERT INTO *`table`* VALUES ROW(1, *`blob_value1`*, *`blob_value2`*, ...)`，即插入包含一个或多个 Blob 列的一行数据

    +   `INSERT INTO *`table`* VALUES ROW(1, *`blob_value1`*), ROW(2, *`blob_value2`*), ROW(3, *`blob_value3`*), ...`，即插入包含一个或多个 Blob 列的多行数据

    +   `UPDATE *`table`* SET *`blob_column1`* = *`blob_value1`*, *`blob_column2`* = *`blob_value2`*, ...`

    +   `UPDATE *`table`* SET *`blob_column`* = *`blob_value`* WHERE *`primary_key_column`* in (*`value_list`*)`，其中主键列不是 Blob 类型

    +   `DELETE FROM *`table`* WHERE *`primary_key_column`* = *`value`*`，其中主键列不是 Blob 类型

    +   `DELETE FROM *`table`* WHERE *`primary_key_column`* IN (*`value_list`*)`，其中主键列不是 Blob 类型

    其他 SQL 语句也可能从这些改进中受益。这些包括 `LOAD DATA INFILE` 和 `CREATE TABLE ... SELECT ...`。此外，`ALTER TABLE *`table`* ENGINE = NDB`，其中 *`table`* 在执行语句之前使用的存储引擎不是 `NDB`，也可能执行得更有效率。

    这种改进适用于影响 MySQL 类型 `BLOB`、`MEDIUMBLOB`、`LONGBLOB`、`TEXT`、`MEDIUMTEXT` 和 `LONGTEXT` 列的语句。仅更新 `TINYBLOB` 或 `TINYTEXT` 列（或两种类型）的语句不受此工作的影响，不应期望其性能发生变化。

    由于某些 SQL 语句需要扫描表格 Blob 列，这会破坏批处理，因此这种改进对某些 SQL 语句的性能没有明显提升。这些语句包括以下类型：

    +   [`SELECT FROM *`table`* [WHERE *`key_column`* IN (*`blob_value_list`*)]`](select.html "15.2.13 SELECT Statement")，其中通过匹配使用 Blob 类型的主键或唯一键列来选择行

    +   `UPDATE *`table`* SET *`blob_column`* = *`blob_value`* WHERE *`condition`*`，使用一个不依赖于唯一值的 *`condition`*

    +   `DELETE FROM *`table`* WHERE *`condition`*` 用于删除包含一个或多个 Blob 列的行，使用一个不依赖于唯一值的 *`condition`*

    +   在执行语句之前已经使用 `NDB` 存储引擎的表上执行复制 `ALTER TABLE` 语句，并且在执行语句之前或之后（或两者都有）表的行包含一个或多个 Blob 列

    为了充分利用这一改进，您可能希望增加用于 **mysqld** 的 `--ndb-batch-size` 和 `--ndb-blob-write-batch-bytes` 选项的值，以最小化修改 blob 所需的往返次数。对于复制，还建议启用 `slave_allow_batching` 系统变量，以最小化副本集群应用时代事务所需的往返次数。

    注意

    从 NDB 8.0.30 版本开始，您还应该使用 `ndb_replica_batch_size` 替代 `--ndb-batch-size`，以及 `ndb_replica_blob_write_batch_bytes` 而不是 `--ndb-blob-write-batch-bytes`。有关这些变量的描述以及更多信息，请参见这些变量的描述，以及 第 25.7.5 节，“为复制准备 NDB 集群”。

+   **Node.js 更新。** 从 NDB 8.0.22 版本开始，Node.js 的 `NDB` 适配器使用的是版本 12.18.3，并且现在仅支持该版本（或更高版本的 Node.js）。

+   **加密备份。** NDB 8.0.22 版本新增了对使用 AES-256-CBC 加密的备份文件的支持；这旨在防止未经授权的人员从已被访问的备份中恢复数据。当备份数据加密时，会由用户提供的密码进行保护。密码可以是任何由可打印 ASCII 字符范围内除了 `!`, `'`, `"`, `$`, `%`, `\`, 和 `^` 之外的最多 256 个字符组成的字符串。对于加密任何给定 NDB 集群备份所使用的密码的保留必须由用户或应用程序执行；`NDB` 不保存密码。密码可以为空，尽管这并不推荐。

    在进行 NDB 集群备份时，可以使用管理客户端 `START BACKUP` 命令，并通过 `ENCRYPT PASSWORD=*`password`*` 进行加密。MGM API 的用户也可以通过调用 `ndb_mgm_start_backup4()` 来启动加密备份。

    你可以使用**ndbxfrm**实用程序对现有的备份文件进行加密，该实用程序已添加到 NDB Cluster 发行版中的 8.0.22 版本中；该程序还可用于解密加密的备份文件。此外，**ndbxfrm**可以使用与 NDB Cluster 在设置`CompressedBackup`配置参数为 1 时创建备份时所使用的相同方法来压缩备份文件和解压缩压缩备份文件。

    要从加密备份中恢复，请使用**ndb_restore**与选项`--decrypt`和`--backup-password`。这两个选项是必需的，以及任何其他在未加密情况下恢复相同备份所需的选项。**ndb_print_backup_file**和**ndbxfrm**也可以分别使用`-P` *`password`*和`--decrypt-password=*`password`*`读取加密文件。

    在提供密码以及加密或解密选项的所有情况下，密码必须用引号括起来；您可以使用单引号或双引号来界定密码。

    从 NDB 8.0.24 开始，这里列出的几个`NDB`程序还支持从标准输入输入密码，类似于在与**mysql**客户端交互登录时使用`--password`选项（不包括在命令行中输入密码）的方式：

    +   对于**ndb_restore**和**ndb_print_backup_file**，`--backup-password-from-stdin`选项使得可以以安全的方式输入密码，类似于**mysql**客户端的`--password`选项。对于**ndb_restore**，与`--decrypt`选项一起使用；对于**ndb_print_backup_file**，使用该选项代替 `-P` 选项。

    +   对于**ndb_mgm**，选项`--backup-password-from-stdin`与[`--execute "START BACKUP [*`选项`*]"`](mysql-cluster-programs-ndb-mgm.html#option_ndb_mgm_execute)一起支持从系统 shell 启动集群备份。

    +   两个**ndbxfrm**选项，`--encrypt-password-from-stdin`和`--decrypt-password-from-stdin`，在使用该程序加密或解密备份文件时会导致类似的行为。

    有关刚列出的程序的更多信息，请参阅其描述。

    从 NDB 8.0.22 开始，还可以通过在集群全局配置文件的 `[ndbd default]` 部分中设置`RequireEncryptedBackup=1`来强制备份加密。这样做时，**ndb_mgm**客户端将拒绝任何未加密的备份尝试。

    从 NDB 8.0.24 开始，您可以通过使用`--encrypt-backup`启动**ndb_mgm**来使其在创建备份时使用加密。在这种情况下，如果未提供密码，则在调用`START BACKUP`时会提示用户输入密码。

+   **IPv6 支持。** 从 NDB 8.0.22 开始，IPv6 寻址支持与管理和数据节点的连接；这包括管理和数据节点与 SQL 节点之间的连接。在配置集群时，可以使用数字 IPv6 地址、解析为 IPv6 地址的主机名或两者兼用。

    要使 IPv6 寻址正常工作，部署集群的操作平台和网络必须支持 IPv6。与使用 IPv4 寻址时一样，主机名解析为 IPv6 地址必须由操作平台提供。

    在 Linux 平台上运行 NDB 8.0.22 及更高版本时已知的问题是，即使没有使用 IPv6 地址，操作系统内核也需要提供 IPv6 支持。这个问题在 NDB 8.0.34 及更高版本中已修复，如果不打算使用 IPv6 寻址，则可以安全地在 Linux 内核中禁用 IPv6 支持（Bug #33324817，Bug #33870642）。

    IPv4 addressing continues to be supported by `NDB`. Using IPv4 and IPv6 addresses concurrently is not recommended, but can be made to work in the following cases:

    +   当管理节点配置为 IPv6，数据节点配置为 IPv4 地址在`config.ini`文件中时：如果未使用`--bind-address`与**mgmd**一起使用，并且数据节点启动时使用`--ndb-connectstring`设置为管理节点的 IPv4 地址，则可以正常工作。

    +   当管理节点配置为 IPv4，数据节点配置为 IPv6 地址在`config.ini`文件中时：类似于另一种情况，如果未传递`--bind-address`给**mgmd**，并且数据节点启动时使用`--ndb-connectstring`设置为管理节点的 IPv6 地址，则可以正常工作。

    这些情况之所以有效，是因为**ndb_mgmd**默认不绑定到任何 IP 地址。

    要从不支持 IPv6 寻址的`NDB`版本升级到支持 IPv6 寻址的版本，前提是网络支持 IPv4 和 IPv6，首先执行软件升级；完成后，可以使用 IPv6 地址更新`config.ini`文件中使用的 IPv4 地址。之后，为使配置更改生效并使集群开始使用 IPv6 地址，需要对集群进行系统重启。

+   **自动安装程序弃用和移除。** MySQL NDB Cluster 自动安装程序基于 Web 的安装工具（**ndb_setup.py**）在 NDB 8.0.22 中已被弃用，并在 NDB 8.0.23 及更高版本中被移除。不再受支持。

+   **ndbmemcache 弃用和移除。** `ndbmemcache`不再受支持。`ndbmemcache`在 NDB 8.0.22 中已被弃用，并在 NDB 8.0.23 中被移除。

+   **ndbinfo 备份 ID 表。** NDB 8.0.24 向`ndbinfo`信息数据库添加了一个`backup_id`表。这旨在作为通过使用**ndb_select_all**来转储内部`SYSTAB_0`表的内容获取此信息的替代方法，后者容易出错且执行时间过长。

    这个表有一列和一行，包含使用`START BACKUP`管理客户端命令对集群进行的最新备份的 ID。如果找不到此集群的备份，则表中包含一个列值为`0`的单行。

+   **表分区增强。** NDB 8.0.23 引入了一种新的处理表分区和片段的方法，可以独立于重做日志部分的数量确定给定数据节点的本地数据管理器（LDMs）的数量。这意味着现在 LDMs 的数量可以高度变化。当`ClassicFragmentation`数据节点配置参数设置为`false`时，`NDB`可以使用这种方法；在这种情况下，不再使用 LDMs 的数量来确定每个数据节点为表创建多少个分区，而是由`PartitionsPerNode`参数的值（也在 NDB 8.0.23 中引入）来确定这个数量，这个数量也用于计算表使用的片段数量。

    当`ClassicFragmentation`具有其默认值`true`时，将使用传统方法来确定表应具有的片段数，即使用 LDM 数。

    更多信息，请参阅之前引用的新参数的描述，在多线程配置参数（ndbmtd）中。

+   **术语更新。** 为了与 MySQL 8.0.21 和 NDB 8.0.21 中开始的工作保持一致，NDB 8.0.23 实施了一些术语上的更改，列在此处：

    +   系统变量`ndb_slave_conflict_role`现已弃用。它被`ndb_conflict_role`替代。

    +   许多`NDB`状态变量已被弃用。这些变量及其替代项在下表中列出：

        **表 25.1 已弃用的 NDB 状态变量及其替代项**

        | 已弃用变量 | 替代项 |
        | --- | --- |
        | `Ndb_api_adaptive_send_deferred_count_slave` | `Ndb_api_adaptive_send_deferred_count_replica` |
        | `Ndb_api_adaptive_send_forced_count_slave` | `Ndb_api_adaptive_send_forced_count_replica` |
        | `Ndb_api_adaptive_send_unforced_count_slave` | `Ndb_api_adaptive_send_unforced_count_replica` |
        | `Ndb_api_bytes_received_count_slave` | `Ndb_api_bytes_received_count_replica` |
        | `Ndb_api_bytes_sent_count_slave` | `Ndb_api_bytes_sent_count_replica` |
        | `Ndb_api_pk_op_count_slave` | `Ndb_api_pk_op_count_replica` |
        | `Ndb_api_pruned_scan_count_slave` | `Ndb_api_pruned_scan_count_replica` |
        | `Ndb_api_range_scan_count_slave` | `Ndb_api_range_scan_count_replica` |
        | `Ndb_api_read_row_count_slave` | `Ndb_api_read_row_count_replica` |
        | `Ndb_api_scan_batch_count_slave` | `Ndb_api_scan_batch_count_replica` |
        | `Ndb_api_table_scan_count_slave` | `Ndb_api_table_scan_count_replica` |
        | `Ndb_api_trans_abort_count_slave` | `Ndb_api_trans_abort_count_replica` |
        | `Ndb_api_trans_close_count_slave` | `Ndb_api_trans_close_count_replica` |
        | `Ndb_api_trans_commit_count_slave` | `Ndb_api_trans_commit_count_replica` |
        | `Ndb_api_trans_local_read_row_count_slave` | `Ndb_api_trans_local_read_row_count_replica` |
        | `Ndb_api_trans_start_count_slave` | `Ndb_api_trans_start_count_replica` |
        | `Ndb_api_uk_op_count_slave` | `Ndb_api_uk_op_count_replica` |
        | `Ndb_api_wait_exec_complete_count_slave` | `Ndb_api_wait_exec_complete_count_replica` |
        | `Ndb_api_wait_meta_request_count_slave` | `Ndb_api_wait_meta_request_count_replica` |
        | `Ndb_api_wait_nanos_count_slave` | `Ndb_api_wait_nanos_count_replica` |
        | `Ndb_api_wait_scan_result_count_slave` | `Ndb_api_wait_scan_result_count_replica` |
        | `Ndb_slave_max_replicated_epoch` | `Ndb_replica_max_replicated_epoch` |
        | 弃用变量 | 替换 |

        弃用的状态变量仍然显示在`SHOW STATUS`的输出中，但应用程序应尽快更新，不再依赖于它们，因为它们在未来的发布系列中的可用性不能保证。

    +   先前在`ndbinfo` `ndbinfo.table_distribution_status`表的`tab_copy_status`列中显示的`ADD_TABLE_MASTER`和`ADD_TABLE_SLAVE`已被弃用。分别由`ADD_TABLE_COORDINATOR`和`ADD_TABLE_PARTICIPANT`替代。

    +   一些`NDB`客户端和实用程序的`--help`输出，如**ndb_restore**已经修改。

+   **ThreadConfig 增强。** 从 NDB 8.0.23 开始，`ThreadConfig`参数的可配置性已经扩展，增加了两种新的线程类型，列在这里：

    +   `query`: 查询线程仅处理`READ COMMITTED`查询。查询线程还充当恢复线程。查询线程的数量必须是 LDM 线程数量的 0、1、2 或 3 倍。 0（默认值，除非启用`ThreadConfig`或`AutomaticThreadConfig`）会导致 LDM 表现为 NDB 8.0.23 之前的行为。

    +   `recover`: 恢复线程从本地检查点检索数据。指定为恢复线程的恢复线程永远不会充当查询线程。

    还可以以两种方式组合现有的`main`和`rep`线程：

    +   通过将这两个参数中的一个设置为 0 来将其合并为一个线程。这样做时，生成的合并线程在`ndbinfo.threads`表中显示为`main_rep`。

    +   通过将`ldm`和`tc`都设置为 0，并将`recv`设置为 1，与`recv`线程一起。在这种情况下，组合线程被命名为`main_rep_recv`。

    此外，许多现有线程类型的最大数量已增加。包括查询线程和恢复线程在内的新最大值在此处列出：

    +   LDM：332

    +   查询：332

    +   恢复：332

    +   TC：128

    +   接收：64

    +   发送：64

    +   主要：2

    其他线程类型的最大值保持不变。

    此外，作业缓冲区在使用超过 32 个块线程时，`NDB`现在使用互斥锁来保护。虽然这可能会导致性能略微下降（在大多数情况下为 1 到 2％），但也显着减少了非常大配置所需的内存量。例如，在 NDB 8.0.23 之前，使用 2 GB 作业缓冲区内存的 64 个线程设置，现在在 NDB 8.0.23 及更高版本中只需要约 1 GB。在我们的测试中，这导致非常复杂查询的执行整体改善约 5％。

    欲了解更多信息，请参阅`ThreadConfig`参数和`ndbinfo.threads`表的描述。

+   **ThreadConfig 线程计数更改。** 由于 NDB 8.0.30 中的工作结果，需要在`ThreadConfig`值字符串中明确包含`main`、`rep`、`recv`和`ldm`的值，以及在此后的 NDB 集群版本中。此外，必须明确设置`count=0`以表示不使用的每个线程类型（`main`、`rep`或`ldm`），并且为复制线程（`rep`）设置`count=1`还需要为`main`设置`count=1`。

    这些更改可能对使用此参数的 NDB 集群的升级产生重大影响；有关更多信息，请参阅第 25.3.7 节，“NDB 集群升级和降级”。

+   **ndbmtd 线程自动配置。** 从 NDB 8.0.23 开始，可以使用 **ndbmtd**") 配置参数 `AutomaticThreadConfig` 来实现多线程数据节点的自动线程配置。当此参数设置为 1 时，`NDB` 根据应用程序可用的处理器数量自动设置线程分配，适用于所有支持线程类型，包括前一项中描述的新的 `query` 和 `recover` 线程类型。如果系统不限制处理器数量，您可以通过设置 `NumCPUs`（也在 NDB 8.0.23 中添加）来限制处理器数量。否则，自动线程配置支持最多 1024 个处理器。

    无论在 `config.ini` 中设置了 `ThreadConfig` 或 `MaxNoOfExecutionThreads` 的值如何，自动线程配置都会发生；这意味着不需要设置这两个参数。

    此外，NDB 8.0.23 实现了许多新的 `ndbinfo` 信息数据库表，提供有关硬件和 CPU 可用性以及 `NDB` 数据节点的 CPU 使用情况的信息。这些表列在此处：

    +   `cpudata`

    +   `cpudata_1sec`

    +   `cpudata_20sec`

    +   `cpudata_50ms`

    +   `cpuinfo`

    +   `hwinfo`

    这些表中的一些在 NDB Cluster 支持的每个平台上都不可用；有关更多信息，请参阅它们的各自描述。

+   **NDB 数据库对象的分层视图。** `dict_obj_tree` 表是在 NDB 8.0.24 中添加到 `ndbinfo` 信息数据库中的，可以提供许多 `NDB` 数据库对象的分层和树状视图，包括以下内容：

    +   表和相关索引

    +   表空间和相关数据文件

    +   日志文件组和相关的撤销日志文件

    有关更多信息和示例，请参见 第 25.6.16.25 节，“ndbinfo dict_obj_tree 表”。

+   **索引统计增强。** NDB 8.0.24 实现了以下改进，用于计算索引统计信息：

    +   以前仅从一个片段收集索引统计信息；现在已更改为将此外推扩展到其他片段。

    +   用于非常小的表的算法已经改进，例如那些具有非常少行且结果被丢弃的表，因此对于这些表的估计应该比以前更准确。

    从 NDB 8.0.27 开始，默认情况下索引统计表会自动创建和更新，`IndexStatAutoCreate` 和 `IndexStatAutoUpdate` 都默认为 `1`（启用），而不是 `0`（禁用），因此不再需要运行 `ANALYZE TABLE` 来更新统计信息。

    有关更多信息，请参见 第 25.6.15 节，“NDB API 统计计数器和变量”。

+   **在恢复操作期间 NULL 和 NOT NULL 之间的转换。** 从 NDB 8.0.26 开始，**ndb_restore** 可以支持将 `NULL` 列恢复为 `NOT NULL`，反之亦然，使用以下列出的选项：

    +   要将 `NULL` 列恢复为 `NOT NULL`，请使用 `--lossy-conversions` 选项。

        最初声明为 `NULL` 的列不得包含任何 `NULL` 行；如果包含，**ndb_restore** 将以错误退出。

    +   要将 `NOT NULL` 列恢复为 `NULL`，请使用 `--promote-attributes` 选项。

    有关指定的 **ndb_restore** 选项的描述，请参见更多信息。

+   **NdbScanFilter 的 SQL 兼容 NULL 比较模式。** 传统上，在涉及 `NULL` 的比较时，`NdbScanFilter` 将 `NULL` 视为等于 `NULL`（因此认为 `NULL == NULL` 为 `TRUE`）。这与 SQL 标准规定不同，SQL 标准要求任何与 `NULL` 的比较都返回 `NULL`，包括 `NULL == NULL`。

    以前，NDB API 应用程序无法覆盖此行为；从 NDB 8.0.26 开始，您可以在创建扫描过滤器之前调用 `NdbScanFilter::setSqlCmpSemantics()` 来实现。这样做会导致下一个 `NdbScanFilter` 对象在其生命周期内执行的所有比较操作都采用符合 SQL 标准的 `NULL` 比较。您必须为每个应使用符合 SQL 标准比较的 `NdbScanFilter` 对象调用该方法。

    欲了解更多信息，请参阅 NdbScanFilter::setSqlCmpSemantics()。

+   **NDB API .FRM 文件方法的弃用。** MySQL 8.0 和 NDB 8.0 不再使用 `.FRM` 文件存储表元数据。因此，NDB API 方法 `getFrmData()`，`getFrmLength()`，和 `setFrm()` 在 NDB 8.0.27 中已被弃用，并可能在未来的版本中移除。要读取和写入表元数据，请改用 `getExtraMetadata()` 和 `setExtraMetadata()`。

+   **IPv4 或 IPv6 寻址偏好。** NDB 8.0.26 添加了 `PreferIPVersion` 配置参数，用于控制 DNS 解析的寻址偏好。IPv4（`PreferIPVersion=4`）是默认设置。由于 NDB 中的配置检索要求所有 TCP 连接的偏好设置相同，因此您应该仅在集群全局配置文件（`config.ini`）的 `[tcp default]` 部分中设置它。

    有关更多信息，请参阅 Section 25.4.3.10, “NDB Cluster TCP/IP Connections”。

+   **日志增强。** 以前，NDB Cluster 数据节点和管理节点日志的分析可能会受到不同日志消息使用不同格式以及不是所有日志消息都包含时间戳的影响。这些问题部分是由于日志记录是通过多种不同机制执行的，如函数 `printf`，`fprintf`，`ndbout` 和 `ndbout_c`，`<<` 操作符的重载等。

    我们通过标准化 `EventLogger` 机制来解决这些问题，该机制已经存在于 `NDB` 中，并且每条日志消息以 `YYYY-MM-DD HH:MM:SS` 格式的时间戳开头。

    有关 NDB Cluster 事件日志和 `EventLogger` 日志消息格式的更多信息，请参见 第 25.6.3 节，“NDB Cluster 生成的事件报告”。

+   **复制 ALTER TABLE 改进。** 从 NDB 8.0.27 开始，在 `NDB` 表上执行复制的 `ALTER TABLE` 会比较执行复制前后源表的片段提交计数。这使得执行此语句的 SQL 节点可以确定是否有任何并发写入活动对正在更改的表进行了操作；如果有，SQL 节点可以终止操作。

    当检测到对正在更改的表进行并发写入时，`ALTER TABLE` 语句将被拒绝，并显示错误消息 Detected change to data in source table during copying ALTER TABLE. Alter aborted to avoid inconsistency (`ER_TABLE_DEF_CHANGED`)。停止更改操作，而不是允许其继续进行并发写入，可以帮助防止数据丢失或损坏。

+   **ndbinfo index_stats 表。** NDB 8.0.28 添加了 `index_stats` 表，提供关于 NDB 索引统计信息的基本信息。它主要用于内部测试，但可能作为 **ndb_index_stat** 的补充。

+   **ndb_import --table 选项。** 在 NDB 8.0.28 之前，**ndb_import** 总是将从 CSV 文件中读取的数据导入到一个表中，该表的名称是根据正在读取的文件的名称派生而来的。NDB 8.0.28 添加了一个 `--table` 选项（简写形式：`-t`）用于指定目标表的名称，并覆盖先前的行为。

    **ndb_import** 的默认行为��然是使用输入文件的基本名称作为目标表的名称。

+   **ndb_import --missing-ai-column 选项。** 从 NDB 8.0.29 开始，**ndb_import** 可以使用在该版本中引入的 `--missing-ai-column` 选项导入包含 `AUTO_INCREMENT` 列的 CSV 文件中的空值数据。该选项可用于包含此类列的一个或多个表。

    为使此选项生效，CSV 文件中的 `AUTO_INCREMENT` 列不得包含任何值。否则，导入操作无法继续。

+   **ndb_import 和空行。** **ndb_import** 一直会拒绝在传入的 CSV 文件中遇到的任何空行。NDB 8.0.30 添加了支持，可以将空行导入到单个列中，前提是可以将空值转换为列值。

+   **ndb_restore --with-apply-status 选项。** 从 NDB 8.0.29 开始，可以使用 **ndb_restore** 和该版本中添加的 `--with-apply-status` 选项从 `NDB` 备份中恢复 `ndb_apply_status` 表。要使用此选项，调用 **ndb_restore** 时还必须使用 `--restore-data`。

    `--with-apply-status` 会恢复除了具有 `server_id = 0` 的行之外的所有 `ndb_apply_status` 表的行；要恢复此行，请使用 `--restore-epoch`。有关更多信息，请参阅 ndb_apply_status 表，以及 `--with-apply-status` 选项的描述。

+   **具有缺失索引的表的 SQL 访问。** 在 NDB 8.0.29 之前，当用户查询尝试打开具有缺失或损坏索引的 `NDB` 表时，MySQL 服务器会引发 `NDB` 错误 `4243`（索引未找到）。当约束违规或缺失数据使得无法在 `NDB` 表上恢复索引时，会出现这种情况，并且使用 **ndb_restore** 的 `--disable-indexes` 用于在没有索引的情况下恢复数据。

    从 NDB 8.0.29 开始，针对一个具有缺失索引的 `NDB` 表的 SQL 查询，如果查询不使用任何缺失的索引，则成功。否则，查询将被拒绝，并显示 `ER_NOT_KEYFILE`。在这种情况下，您可以使用 `ALTER TABLE ... ALTER INDEX ... INVISIBLE` 来阻止 MySQL 优化器尝试使用该索引，或者使用适当的 SQL 语句删除该索引（然后可能重新创建）。

+   **NDB API List::clear() 方法。** NDB API `Dictionary` 方法 `listEvents()`, `listIndexes()` 和 `listObjects()` 每个都需要一个空的 `List` 对象的引用。以前，使用任何这些方法重用现有的 `List` 都会因为这个原因出现问题。NDB 8.0.29 通过实现一个 `clear()` 方法来简化这个过程，该方法会从列表中删除所有数据。

    作为这项工作的一部分，`List` 类析构函数现在在从列表中删除任何元素或属性之前调用 `List::clear()`。

+   **NDB 字典表在 ndbinfo 中。** NDB 8.0.29 在 `ndbinfo` 数据库中引入了几个新表，提供了以前需要使用 **ndb_desc**, **ndb_select_all** 和其他 **NDB** 实用程序程序来获取的来自 `NdbDictionary` 的信息。

    这些表中有两个实际上是视图。`hash_maps` 表提供了 `NDB` 使用的哈希映射的信息；`files` 表显示了关于在磁盘上存储数据的文件的信息（参见 第 25.6.11 节，“NDB 集群磁盘数据表”）。

    NDB 8.0.29 中添加的剩余六个 `ndbinfo` 表都是基本表。这些表不是隐藏的，也不使用 `ndb$` 前缀命名。这些表在此列出，包括每个表中表示的对象的描述：

    +   `blobs`: 用于存储 `BLOB` 和 `TEXT` 列的可变大小部分的 Blob 表

    +   `dictionary_columns`: `NDB` 表的列

    +   `dictionary_tables`: `NDB` 表

    +   `events`: NDB API 中的事件订阅

    +   `foreign_keys`: `NDB` 表上的外键

    +   `index_columns`：`NDB` 表上的索引

    NDB 8.0.29 还对 `ndbinfo` 存储引擎的主键实现进行了更改，以提高与 `NdbDictionary` 的兼容性。

+   **ndbcluster 插件和性能模式。** 从 NDB 8.0.29 开始，`ndbcluster` 插件线程显示在性能模式 `threads` 和 `setup_threads` 中，可以获取有关这些线程性能的信息。在 `performance_schema` 表中公开的三个线程列在此处：

    +   `ndb_binlog`：二进制日志线程

    +   `ndb_index_stat`：索引统计线程

    +   `ndb_metadata`：元数据线程

    有关更多信息和示例，请参阅 ndbcluster 插件线程。

    在 NDB 8.0.30 及更高版本中，事务批处理内存使用情况可在性能模式 `memory_summary_by_thread_by_event_name` 和 `setup_instruments` 中以 `memory/ndbcluster/Thd_ndb::batch_mem_root` 的形式显示。您可以使用这些信息来查看事务使用了多少内存。有关更多信息，请参阅 事务内存使用。

+   **可配置的 blob 内联大小。** 从 NDB 8.0.30 开始，可以在 `CREATE TABLE` 或 `ALTER TABLE` 中设置 blob 列的内联大小。NDB Cluster 支持的最大内联大小为 29980 字节。

    有关更多信息和示例，请参阅 NDB_COLUMN 选项，以及 字符串类型存储要求。

+   **replica_allow_batching 默认启用。** 复制写批处理极大地提高了 NDB 集群复制性能，特别是在复制 blob 类型列（`TEXT`、`BLOB` 和 `JSON`）时，因此通常在使用 NDB 集群进行复制时应启用。因此，从 NDB 8.0.30 开始，默认启用了`replica_allow_batching`系统变量，并将其设置为`OFF`会引发警告。

+   **冲突解析插入操作支持。** 在 NDB 8.0.30 之前，仅有两种可用于解决主键冲突的策略，用作更新和删除操作的函数为 `NDB$MAX()` 和 `NDB$MAX_DELETE_WIN()`。这两者对写操作没有任何影响，除非具有与先前写入相同主键的写操作总是被拒绝，并且仅在没有具有相同主键的操作存在时才被接受和应用。NDB 8.0.30 引入了两个新的冲突解析函数 `NDB$MAX_INS()")` 和 `NDB$MAX_DEL_WIN_INS()")`，用于处理插入操作之间的主键冲突。这些函数处理冲突写入如下：

    1.  如果没有冲突写入，应用此操作（与 `NDB$MAX()` 相同）。

    1.  否则，应用“最大时间戳获胜”冲突解析，如下所示：

        1.  如果传入写入的时间戳大于冲突写入的时间戳，则应用传入操作。

        1.  如果传入写入的时间戳*不*大，则拒绝传入写入操作。

    对于冲突的更新和删除操作，`NDB$MAX_INS()` 的行为类似于 `NDB$MAX()")`，而 `NDB$MAX_DEL_WIN_INS()` 的行为与 `NDB$MAX_DELETE_WIN()")` 相同。

    此增强功能提供了在处理冲突的复制写操作时配置冲突检测的支持，因此具有更高时间戳列值的复制`INSERT`会被幂等地应用，而具有较低时间戳列值的复制`INSERT`会被拒绝。

    与其他冲突解决函数一样，拒绝的操作可以选择记录在异常表中；拒绝的操作会增加一个计数器（状态变量`Ndb_conflict_fn_max` 用于“最大时间戳获胜”，以及`Ndb_conflict_fn_old` 用于“相同时间戳获胜”）。

    欲了解更多信息，请参阅新冲突解决函数的描述，以及第 25.7.12 节，“NDB 集群复制冲突解决”。

+   **复制应用程序批处理大小控制。** 以前，写入到副本 NDB 集群时使用的批处理大小由 `--ndb-batch-size` 控制，而写入 blob 数据到副本时使用的批处理大小由 `ndb-blob-write-batch-bytes` 确定。这种安排的一个问题是，副本使用这些变量的全局值，这意味着更改其中一个变量对于副本也会影响所有其他会话使用的值。此外，不可能为这些值设置专门适用于副本的不同默认值，副本的默认值应该比其他会话的默认值更高。

    NDB 8.0.30 添加了两个新的系统变量，这些变量专门用于复制应用程序。`ndb_replica_batch_size` 现在控制用于复制应用程序的批处理大小，而 `ndb_replica_blob_write_batch_bytes` 变量现在确定用于在复制上执行批量 blob 写入的 blob 写入批处理大小。

    此更改应该改善使用默认设置的 MySQL NDB 集群复制的行为，并允许用户微调 NDB 复制性能，而不会影响用户线程，例如执行 SQL 查询处理的线程。

    欲了解更多信息，请参阅新变量的描述。另请参阅第 25.7.5 节，“准备 NDB 集群进行复制”。

+   **二进制日志事务压缩。** NDB 8.0.31 增加了对使用`ZSTD`压缩的压缩事务的二进制日志的支持。要启用此功能，请将此版本中引入的`ndb_log_transaction_compression`系统变量设置为`ON`。使用的压缩级别可以通过`ndb_log_transaction_compression_level_zstd`系统变量进行控制，该系统变量也在该版本中添加；默认压缩级别为 3。

    尽管`binlog_transaction_compression`和`binlog_transaction_compression_level_zstd`服务器系统变量对`NDB`表的二进制日志记录没有影响，但使用`--binlog-transaction-compression=ON`启动**mysqld**会自动启用`ndb_log_transaction_compression`。在服务器启动完成后，你可以在 MySQL 客户端会话中使用`SET @@global.ndb_log_transaction_compression=OFF`来禁用它。

    查看`ndb_log_transaction_compression`的描述以及第 7.4.4.5 节，“二进制日志事务压缩”，以获取更多信息。

+   **NDB 复制：多线程应用程序。** 从 NDB 8.0.33 开始，NDB Cluster 复制支持副本服务器上的 MySQL 多线程应用程序（MTA）（以及`replica_parallel_workers`的非零值），这使得副本可以并行应用二进制日志事务，从而提高吞吐量。（有关 MySQL 服务器中多线程应用程序的更多信息，请参阅第 19.2.3 节，“复制线程”。）

    在副本上启用此功能需要源数据库使用`--ndb-log-transaction-dependency`参数设置为`ON`（此选项也在 NDB 8.0.33 中实现）。此外，源数据库还需要将`binlog_transaction_dependency_tracking`设置为`WRITESET`。另外，你必须确保副本上的`replica_parallel_workers`的值大于 1，从而确保副本使用多个工作线程。

    有关更多信息和要求，请参见第 25.7.11 节，“使用多线程应用程序复制的 NDB 集群复制”。

+   **构建选项的更改。** NDB 8.0.31 对用于构建 MySQL 集群的 CMake 选项进行了以下更改。

    +   `WITH_NDBCLUSTER`选项已被弃用，`WITH_PLUGIN_NDBCLUSTER`已被移除。

    +   要从源代码构建 MySQL 集群，请使用新添加的`WITH_NDB`选项。

    +   `WITH_NDBCLUSTER_STORAGE_ENGINE`仍然受支持，但对于大多数构建而言不再需要。

    有关更多信息，请参见用于编译 NDB 集群的 CMake 选项。

+   **文件系统加密。** 透明数据加密（TDE）通过加密在静止状态下的`NDB`数据，即所有持久化到磁盘的`NDB`表数据和日志文件。这旨在防止在未经授权访问 NDB 集群数据文件（如表空间文件或日志）后恢复数据。

    加密由数据节点上的 NDB 文件系统层（`NDBFS`）透明实现；数据在读取和写入文件时进行加密和解密，`NDBFS`内部客户端块像正常操作文件一样。

    `NDBFS`可以从用户提供的密码直接透明加密文件，但将单个文件的加密和解密与用户提供的密码分离可能有助于提高效率、可用性、安全性和灵活性。请参见第 25.6.14.2 节，“NDB 文件系统加密实现”。

    TDE 使用两种类型的密钥。一个秘密密钥用于加密存储在磁盘上的实际数据和日志文件（包括 LCP、重做、撤销和表空间文件）。然后使用主密钥加密秘密密钥。

    数据节点配置参数`EncryptedFileSystem`，从 NDB 8.0.29 开始提供，设置为`1`时，强制对存储表数据的文件进行加密。这包括 LCP 数据文件、重做日志文件、表空间文件和撤销日志文件。

    在启动或重新启动每个数据节点时，还需要为其提供密码，使用`--filesystem-password`或`--filesystem-password-from-stdin`选项之一。请参见第 25.6.14.1 节，“NDB 文件系统加密设置和使用”。此密码使用相同格式，并受到与用于加密`NDB`备份的密码相同的约束（有关加密`NDB`备份的详细信息，请参见**ndb_restore** `--backup-password`选项的描述）。

    只有使用`NDB`存储引擎的表才会受到此功能的加密影响；参见第 25.6.14.3 节，“NDB 文件系统加密限制”。其他表，如用于`NDB`模式分发、复制和二进制日志的表，通常使用`InnoDB`；参见第 17.13 节，“InnoDB 数据静止时加密”。有关二进制日志文件加密的信息，请参见第 19.3.2 节，“加密二进制日志文件和中继日志文件”。

    由`NDB`进程生成或使用的文件，如操作系统日志、崩溃日志和核心转储，不会被加密。由`NDB`使用但不包含任何用户表数据的文件也不会被加密；这些文件包括 LCP 控制文件、模式文件和系统文件（参见 NDB 集群数据节点文件系统）。管理服务器配置缓存也不会被加密。

    此外，NDB 8.0.31 添加了一个新的实用程序**ndb_secretsfile_reader**，用于从秘密文件（`S0.sysfile`）中提取密钥信息。

    此增强功能建立在 NDB 8.0.22 中实现加密`NDB`备份的工作基础之上。有关加密备份的更多信息，请参见`RequireEncryptedBackup`配置参数的描述，以及第 25.6.8.2 节，“使用 NDB 集群管理客户端创建备份”。

+   **移除不必要的程序选项。** 在 NDB Cluster 8.0.31 版本中，删除了一些“垃圾”命令行选项，这些选项从未被实现过。以下列出了被删除选项和程序：

    +   `--ndb-optimized-node-selection`:

        **ndbd**, **ndbmtd**, **ndb_mgm**, **ndb_delete_all**, **ndb_desc**, **ndb_drop_index**, **ndb_drop_table**, **ndb_show_table**, **ndb_blob_tool**, **ndb_config**, **ndb_index_stat**, **ndb_move_data**, **ndbinfo_select_all**, **ndb_select_count**

    +   `--character-sets-dir`:

        **ndb_mgm**, **ndb_mgmd**, **ndb_config**, **ndb_delete_all**, **ndb_desc**, **ndb_drop_index**, **ndb_drop_table**, **ndb_show_table**, **ndb_blob_tool**, **ndb_config**, **ndb_index_stat**, **ndb_move_data**, **ndbinfo_select_all**, **ndb_select_count**, **ndb_waiter**

    +   `--core-file`:

        **ndb_mgm**, **ndb_mgmd**, **ndb_config**, **ndb_delete_all**, **ndb_desc**, **ndb_drop_index**, **ndb_drop_table**, **ndb_show_table**, **ndb_blob_tool**, **ndb_config**, **ndb_index_stat**, **ndb_move_data**, **ndbinfo_select_all**, **ndb_select_count**, **ndb_waiter**

    +   `--connect-retries` 和 `--connect-retry-delay`：

        **ndb_mgmd**

    +   `--ndb-nodeid`：

        **ndb_config**

    更多信息，请参阅第 25.5 节，“NDB 集群程序”中相关程序和选项的描述。

+   **读取配置缓存文件。** 从 NDB 8.0.32 开始，可以使用该版本引入的 `--config-binary-file` 选项，通过**ndb_mgmd**创建的二进制配置缓存文件。这可以简化确定给定配置文件中的设置是否已应用于集群的过程，或在 `config.ini` 文件在某种方式受损或丢失后，从二进制缓存中恢复设置。

    有关更多信息和示例，请参阅第 25.5.7 节，“ndb_config — 提取 NDB 集群配置信息”中对此选项的描述。

+   **ndbinfo transporter_details 表。** 这个`ndbinfo`表提供了关于 NDB 集群中使用的各个传输器的信息。在 NDB 8.0.37 中添加，与 `ndbinfo` `transporters`表类似。

    有关更多信息，请参阅第 25.6.16.64 节，“ndbinfo transporter_details 表”。

MySQL Cluster Manager 支持 NDB 集群 8.0。MySQL Cluster Manager 具有先进的命令行界面，可以简化许多复杂的 NDB 集群管理任务。有关更多信息，请参阅 MySQL Cluster Manager 8.0.36 用户手册。
