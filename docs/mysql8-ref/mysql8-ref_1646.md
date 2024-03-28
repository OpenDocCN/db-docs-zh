> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-binary.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-binary.html)

#### 25.3.2.1 从二进制发行版在 Windows 上安装 NDB Cluster

本节描述了在 Windows 上使用 Oracle 提供的二进制“无需安装” NDB Cluster 发行版进行基本安装的过程，使用本节开头概述的相同 4 节点设置（参见 第 25.3 节，“NDB Cluster 安装”），如下表所示：

**表 25.7 示例集群中节点的网络地址**

| 节点 | IP 地址 |
| --- | --- |
| 管理节点 (**mgmd**) | 198.51.100.10 |
| SQL 节点 (**mysqld**) | 198.51.100.20 |
| 数据节点 "A" (**ndbd**) | 198.51.100.30 |
| 数据节点 "B" (**ndbd**) | 198.51.100.40 |

与其他平台一样，运行 SQL 节点的 NDB Cluster 主机计算机必须安装 MySQL Server 二进制文件 (**mysqld.exe**)。您还应该在此主机上安装 MySQL 客户端 (**mysql.exe**)。对于管理节点和数据节点，不需要安装 MySQL Server 二进制文件；但是，每个管理节点都需要管理服务器守护程序 (**ndb_mgmd.exe**)；每个数据节点都需要数据节点守护程序 (**ndbd.exe** 或 **ndbmtd.exe**"))。在本示例中，我们将 **ndbd.exe** 称为数据节点可执行文件，但您也可以安装 **ndbmtd.exe**")，这是该程序的多线程版本，方式完全相同。您还应该在管理服务器主机上安装管理客户端 (**ndb_mgm.exe**)。本节涵盖了为每种类型的 NDB Cluster 节点安装正确的 Windows 二进制文件所需的步骤。

注意

与其他 Windows 程序一样，NDB Cluster 可执行文件的文件扩展名为`.exe`。但是，在从命令行调用这些程序时，不需要包含`.exe`扩展名。因此，在本文档中，我们通常简称这些程序为**mysqld**、**mysql**、**ndb_mgmd**等。您应该明白，无论我们是指**mysqld**还是**mysqld.exe**，两个名称都表示相同的东西（MySQL 服务器程序）。

对于使用 Oracle 的`no-install`二进制文件设置 NDB Cluster，安装过程的第一步是从[`dev.mysql.com/downloads/cluster/`](https://dev.mysql.com/downloads/cluster/)下载最新的 NDB Cluster Windows ZIP 二进制存档。此存档的文件名为`mysql-cluster-gpl-*`ver`*-win*`arch`*.zip`，其中*`ver`*是`NDB`存储引擎版本（例如`8.0.34`），*`arch`*是架构（`32`表示 32 位二进制文件，`64`表示 64 位二进制文件）。例如，64 位 Windows 系统的 NDB Cluster 8.0.34 存档名为`mysql-cluster-gpl-8.0.34-win64.zip`。

您可以在 32 位和 64 位 Windows 版本上运行 32 位 NDB Cluster 二进制文件；但是，64 位 NDB Cluster 二进制文件只能在 64 位 Windows 版本上使用。如果您在具有 64 位 CPU 的计算机上使用 32 位 Windows 版本，则必须使用 32 位 NDB Cluster 二进制文件。

为了最大限度地减少需要从互联网下载或在计算机之间复制的文件数量，我们从您打算运行 SQL 节点的计算机开始。

**SQL 节点。** 我们假设您已将存档的副本放在具有 IP 地址 198.51.100.20 的计算机上的目录`C:\Documents and Settings\*`username`*\My Documents\Downloads`中，其中*`username`*是当前用户的名称。（您可以在命令行上使用`ECHO %USERNAME%`获取此名称。）要将 NDB Cluster 可执行文件安装并运行为 Windows 服务，此用户应为`Administrators`组的成员。

从存档中提取所有文件。与 Windows 资源管理器集成的提取向导足以完成此任务。（如果您使用不同的存档程序，请确保它从存档中提取所有文件和目录，并保留存档的目录结构。）当要求输入目标目录时，请输入`C:\`，这将导致提取向导将存档提取到目录`C:\mysql-cluster-gpl-*`ver`*-win*`arch`*`。将此目录重命名为`C:\mysql`。

可以将 NDB Cluster 二进制文件安装到除`C:\mysql\bin`之外的目录；但是，如果这样做，必须相应地修改本过程中显示的路径。特别是，如果 MySQL Server（SQL 节点）二进制文件安装到除`C:\mysql`或`C:\Program Files\MySQL\MySQL Server 8.0`之外的位置，或者如果 SQL 节点的数据目录位于除`C:\mysql\data`或`C:\Program Files\MySQL\MySQL Server 8.0\data`之外的位置，则在启动 SQL 节点时必须在命令行上使用额外的配置选项或将其添加到`my.ini`或`my.cnf`文件中。有关配置 MySQL Server 在非标准位置运行的更多信息，请参见 Section 2.3.4, “Installing MySQL on Microsoft Windows Using a `noinstall` ZIP Archive”。

要使支持 NDB Cluster 的 MySQL Server 作为 NDB Cluster 的一部分运行，必须使用选项`--ndbcluster`和`--ndb-connectstring`启动。虽然可以在命令行上指定这些选项，但通常更方便将它们放在一个选项文件中。为此，请在记事本或其他文本编辑器中创建一个新的文本文件。将以下配置信息输入到此文件中：

```sql
[mysqld]
# Options for mysqld process:
ndbcluster                       # run NDB storage engine
ndb-connectstring=198.51.100.10 # location of management server
```

如果需要，可以添加此 MySQL Server 使用的其他选项（请参见 Section 2.3.4.2, “Creating an Option File”)，但文件必须至少包含所示选项。将此文件��存为`C:\mysql\my.ini`。这完成了 SQL 节点的安装和设置。

**数据节点。** Windows 主机上的 NDB Cluster 数据节点只需要一个可执行文件，即**ndbd.exe**或**ndbmtd.exe**")之一。在本示例中，我们假设您正在使用**ndbd.exe**，但当使用**ndbmtd.exe**")时，相同的说明也适用。在每台计算机上，您希望运行数据节点的计算机（具有 IP 地址 198.51.100.30 和 198.51.100.40），创建目录`C:\mysql`、`C:\mysql\bin`和`C:\mysql\cluster-data`；然后，在您下载并解压`no-install`存档的计算机上，将`ndbd.exe`定位于`C:\mysql\bin`目录。将此文件复制到两个数据节点主机上的`C:\mysql\bin`目录。

要作为 NDB 集群的一部分运行，每个数据节点必须提供管理服务器的地址或主机名。您可以在启动每个数据节点进程时使用 `--ndb-connectstring` 或 `-c` 选项在命令行上提供此信息。然而，通常最好将此信息放在一个选项文件中。为此，请在记事本或其他文本编辑器中创建一个新的文本文件，并输入以下文本：

```sql
[mysql_cluster]
# Options for data node process:
ndb-connectstring=198.51.100.10 # location of management server
```

将此文件保存为 `C:\mysql\my.ini`，保存在数据节点主机上。创建另一个文本文件，其中包含相同的信息，并将其保存为 `C:mysql\my.ini`，保存在另一个数据节点主机上，或者将第一个数据节点主机上的 my.ini 文件复制到第二个数据节点主机上，确保将副本放在第二个数据节点的 `C:\mysql` 目录中。现在，两个数据节点主机已准备好用于 NDB 集群，只剩下安装和配置管理节点。

**管理节点。** 在用于托管 NDB 集群管理节点的计算机上唯一需要的可执行程序是管理服务器程序 **ndb_mgmd.exe**。然而，为了在启动 NDB 集群后对其进行管理，您还应该在与管理服务器相同的计算机上安装 NDB 集群管理客户端程序 **ndb_mgm.exe**。在您下载和提取 `no-install` 存档的计算机上找到这两个程序；这应该是 SQL 节点主机上的目录 `C:\mysql\bin`。在具有 IP 地址 198.51.100.10 的计算机上创建目录 `C:\mysql\bin`，然后将这两个程序复制到该目录。

现在，您应该为 `ndb_mgmd.exe` 创建两个配置文件：

1.  一个本地配置文件，提供特定于管理节点本身的配置数据。通常，此文件只需要提供 NDB 集群全局配置文件的位置（参见项目 2）。

    要创建此文件，请在记事本或其他文本编辑器中启动一个新的文本文件，并输入以下信息：

    ```sql
    [mysql_cluster]
    # Options for management node process
    config-file=C:/mysql/bin/config.ini
    ```

    将此文件保存为文本文件 `C:\mysql\bin\my.ini`。

1.  一个全局配置文件，管理节点可以从中获取管理整个 NDB 集群的配置信息。至少，此文件必须包含 NDB 集群中每个节点的部分，并且管理节点和所有数据节点的 IP 地址或主机名（`HostName` 配置参数）。还建议包括以下附加信息：

    +   任何 SQL 节点的 IP 地址或主机名

    +   分配给每个数据节点的数据内存和索引内存（`DataMemory` 和 `IndexMemory` 配置参数）

    +   使用 `NoOfReplicas` 配置参数来指定分片副本的数量（参见 第 25.2.2 节，“NDB Cluster 节点、节点组、分片副本和分区”）

    +   每个数据节点存储数据和日志文件的目录，以及管理节点保存其日志文件的目录（在这两种情况下，使用 `DataDir` 配置参数）

    使用诸如记事本之类的文本编辑器创建一个新的文本文件，并输入以下信息：

    ```sql
    [ndbd default]
    # Options affecting ndbd processes on all data nodes:
    NoOfReplicas=2 # Number of fragment replicas
    DataDir=C:/mysql/cluster-data # Directory for each data node's data files
     # Forward slashes used in directory path,
     # rather than backslashes. This is correct;
     # see Important note in text
    DataMemory=80M # Memory allocated to data storage
    IndexMemory=18M # Memory allocated to index storage
     # For DataMemory and IndexMemory, we have used the
     # default values. Since the "world" database takes up
     # only about 500KB, this should be more than enough for
     # this example Cluster setup.

    [ndb_mgmd]
    # Management process options:
    HostName=198.51.100.10 # Hostname or IP address of management node
    DataDir=C:/mysql/bin/cluster-logs # Directory for management node log files

    [ndbd]
    # Options for data node "A":
     # (one [ndbd] section per data node)
    HostName=198.51.100.30 # Hostname or IP address

    [ndbd]
    # Options for data node "B":
    HostName=198.51.100.40 # Hostname or IP address

    [mysqld]
    # SQL node options:
    HostName=198.51.100.20 # Hostname or IP address
    ```

    将此文件保存为文本文件 `C:\mysql\bin\config.ini`。

重要提示

在 Windows 上，当在程序选项或 NDB Cluster 使用的配置文件中指定目录路径时，不能使用单个反斜杠字符（`\`）。相反，你必须要么用第二个反斜杠字符（`\\`）转义每个反斜杠字符，要么用正斜杠字符（`/`）替换反斜杠。例如，NDB Cluster `config.ini` 文件中 `[ndb_mgmd]` 部分的以下行不起作用：

```sql
DataDir=C:\mysql\bin\cluster-logs
```

相反，你可以使用以下任一种方式：

```sql
DataDir=C:\\mysql\\bin\\cluster-logs # Escaped backslashes
```

```sql
DataDir=C:/mysql/bin/cluster-logs # Forward slashes
```

为了简洁和易读起见，我们建议在 Windows 上使用 NDB Cluster 程序选项和配置文件中的目录路径时使用正斜杠。
