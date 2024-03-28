> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-initial-start.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-initial-start.html)

#### 25.3.2.3 Windows 上 NDB Cluster 的初始启动

一旦 NDB Cluster 可执行文件和所需的配置文件就位，启动集群的初始步骤只是简单地启动集群中所有节点的 NDB Cluster 可执行文件。每个集群节点进程必须分别在其所在的主机计算机上启动。应首先启动管理节点，然后是数据节点，最后是任何 SQL 节点。

1.  在管理节点主机上，从命令行发出以下命令以启动管理节点进程。输出应类似于所示：

    ```sql
    C:\mysql\bin> ndb_mgmd
    2010-06-23 07:53:34 [MgmtSrvr] INFO -- NDB Cluster Management Server. mysql-8.0.35-ndb-8.0.35
    2010-06-23 07:53:34 [MgmtSrvr] INFO -- Reading cluster configuration from 'config.ini'
    ```

    管理节点进程继续向控制台打印日志输出。这是正常的，因为管理节点不作为 Windows 服务运行。（如果您在类 Unix 平台（如 Linux）上使用 NDB Cluster，则可能会注意到管理节点在 Windows 上的默认行为实际上与其在 Unix 系统上的行为相反，在 Unix 系统上，它默认作为 Unix 守护进程运行。这种行为也适用于在 Windows 上运行的 NDB Cluster 数据节点进程。）因此，请不要关闭运行 **ndb_mgmd.exe** 的窗口；这样会终止管理节点进程。（请参阅 Section 25.3.2.4, “Installing NDB Cluster Processes as Windows Services”，其中我们展示了如何将 NDB Cluster 进程安装并作为 Windows 服务运行。）

    `-f` 选项告诉管理节点在哪里找到全局配置文件（`config.ini`）。此选项的长格式是 `--config-file`。

    重要

    NDB Cluster 管理节点缓存从 `config.ini` 中读取的配置数据；一旦创建了配置缓存，除非被强制执行，否则在后续启动时会忽略 `config.ini` 文件。这意味着，如果管理节点由于此文件中的错误而无法启动，则在纠正错误后，必须让管理节点重新读取 `config.ini`。您可以通过在命令行上使用 `--reload` 或 `--initial` 选项启动 **ndb_mgmd.exe** 来实现这一点。这两个选项中的任何一个都可以刷新配置缓存。

    在管理节点的 `my.ini` 文件中不需要也不建议使用这两个选项。

1.  在每个数据节点主机上，运行此处显示的命令以启动数据节点进程：

    ```sql
    C:\mysql\bin> ndbd
    2010-06-23 07:53:46 [ndbd] INFO -- Configuration fetched from 'localhost:1186', generation: 1
    ```

    在每种情况下，数据节点进程的输出的第一行应类似于前面示例中所示的内容，并且后面跟着其他日志输出行。与管理节点进程一样，这是正常的，因为数据节点不作为 Windows 服务运行。因此，请不要关闭运行数据节点进程的控制台窗口；这样做会终止**ndbd.exe**。（有关更多信息，请参见第 25.3.2.4 节，“将 NDB 集群进程安装为 Windows 服务”。）

1.  尚未启动 SQL 节点；在数据节点完成启动之前，它无法连接到集群，这可能需要一些时间。相反，在管理节点主机上的新控制台窗口中启动 NDB 集群管理客户端**ndb_mgm.exe**，该文件应位于管理节点主机的`C:\mysql\bin`中。（不要尝试在运行**ndb_mgmd.exe**的控制台窗口中重新使用，因为这会终止管理节点。）生成的输出应如下所示：

    ```sql
    C:\mysql\bin> ndb_mgm
    -- NDB Cluster -- Management Client --
    ndb_mgm>
    ```

    当出现提示符`ndb_mgm>`时，这表示管理客户端已准备好接收 NDB 集群管理命令。您可以通过在管理客户端提示符处输入`ALL STATUS`来观察数据节点的启动状态。此命令会导致数据节点启动序列的运行报告，应该看起来像这样：

    ```sql
    ndb_mgm> ALL STATUS
    Connected to Management Server at: localhost:1186
    Node 2: starting (Last completed phase 3) (mysql-8.0.35-ndb-8.0.35)
    Node 3: starting (Last completed phase 3) (mysql-8.0.35-ndb-8.0.35)

    Node 2: starting (Last completed phase 4) (mysql-8.0.35-ndb-8.0.35)
    Node 3: starting (Last completed phase 4) (mysql-8.0.35-ndb-8.0.35)

    Node 2: Started (version 8.0.35)
    Node 3: Started (version 8.0.35)

    ndb_mgm>
    ```

    注意

    在管理客户端中输入的命令不区分大小写；我们使用大写作为这些命令的规范形式，但在输入到**ndb_mgm**客户端时，您不必遵守此约定。有关更多信息，请参见第 25.6.1 节，“NDB 集群管理客户端中的命令”。

    由`ALL STATUS`产生的输出可能会根据数据节点启动速度、您正在使用的 NDB 集群软件的发布版本号以及其他因素而有所不同。重要的是，当您看到两个数据节点都已启动时，您就可以开始启动 SQL 节点了。

    您可以让**ndb_mgm.exe**保持运行；它不会对 NDB Cluster 的性能产生负面影响，并且我们在下一步中使用它来验证 SQL 节点在您启动后是否连接到集群。

1.  在指定为 SQL 节点主机的计算机上，打开控制台窗口并导航到解压 NDB Cluster 二进制文件的目录（如果您遵循我们的示例，这是`C:\mysql\bin`）。

    通过从命令行调用**mysqld.exe**来启动 SQL 节点，如下所示：

    ```sql
    C:\mysql\bin> mysqld --console
    ```

    `--console`选项会导致日志信息写入控制台，在出现问题时可能会有所帮助。（一旦您确信 SQL 节点以令人满意的方式运行，您可以停止它并重新启动，而不使用`--console`选项，以便正常执行日志记录。）

    在管理节点主机上运行管理客户端（**ndb_mgm.exe**）的控制台窗口中，输入`SHOW`命令，应该会产生类似于这里显示的输出：

    ```sql
    ndb_mgm> SHOW
    Connected to Management Server at: localhost:1186
    Cluster Configuration
    ---------------------
    [ndbd(NDB)]     2 node(s)
    id=2    @198.51.100.30  (Version: 8.0.35-ndb-8.0.35, Nodegroup: 0, *)
    id=3    @198.51.100.40  (Version: 8.0.35-ndb-8.0.35, Nodegroup: 0)

    [ndb_mgmd(MGM)] 1 node(s)
    id=1    @198.51.100.10  (Version: 8.0.35-ndb-8.0.35)

    [mysqld(API)]   1 node(s)
    id=4    @198.51.100.20  (Version: 8.0.35-ndb-8.0.35)
    ```

    您还可以使用**mysql**客户端（**mysql.exe**）使用`SHOW ENGINE NDB STATUS`语句验证 SQL 节点是否连接到 NDB Cluster。

现在，您可以准备使用 NDB Cluster 的`NDBCLUSTER`存储引擎来处理数据库对象和数据。有关更多信息和示例，请参见第 25.3.5 节，“具有表和数据的 NDB Cluster 示例”。

您还可以将**ndb_mgmd.exe**、**ndbd.exe**和**ndbmtd.exe**")安装为 Windows 服务。有关如何执行此操作的信息，请参见第 25.3.2.4 节，“将 NDB Cluster 进程安装为 Windows 服务”。
