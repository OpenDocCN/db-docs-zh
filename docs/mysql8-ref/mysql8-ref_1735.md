> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-using-management-client.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-using-management-client.html)

#### 25.6.8.2 使用 NDB 集群管理客户端创建备份

在开始备份之前，请确保集群已正确配置以执行备份。（参见第 25.6.8.3 节，“NDB 集群备份配置”.）

`START BACKUP` 命令用于创建备份，其语法如下所示：

```sql
START BACKUP [*backup_id*]
    [*encryption_option*]
    [*wait_option*]
    [*snapshot_option*]

*encryption_option*:
ENCRYPT [PASSWORD=*password*]

*password*:
{'*password_string*' | "*password_string*"}

*wait_option*:
WAIT {STARTED | COMPLETED} | NOWAIT

*snapshot_option*:
SNAPSHOTSTART | SNAPSHOTEND
```

连续备份会自动按顺序识别，因此*`backup_id`*，一个大于或等于 1 的整数，是可选的；如果省略，则使用下一个可用值。如果使用现有的*`backup_id`*值，则备份将失败，并显示错误消息备份失败：文件已存在。如果使用*`backup_id`*，则必须紧跟在`START BACKUP`关键字之后，在使用任何其他选项之前。

在 NDB 8.0.22 及更高版本中，`START BACKUP` 支持使用 `ENCRYPT PASSWORD=*`password`*` 创建加密备份。*`password`* 必须满足以下所有要求：

+   使用除 `!`、`'`、`"`、`$`、`%`、`\` 和 `^` 之外的任何可打印 ASCII 字符。

+   长度不超过 256 个字符

+   用单引号或双引号括起来

当使用 `ENCRYPT PASSWORD='*`password`*'` 时，由每个数据节点写入的备份数据记录和日志文件会使用从用户提供的*`password`*和随机生成的盐派生的密钥进行加密，使用 PBKDF2-SHA256 算法的密钥派生函数（KDF）生成用于该文件的对称加密密钥。此函数的形式如下所示：

```sql
key = KDF(*random_salt*, *password*)
```

然后使用生成的密钥使用 AES 256 CBC 内联加密备份数据，并对备份文件集（使用生成的密钥）使用对称加密。

注意

NDB 集群*永远*不会保存用户提供的密码或生成的加密密钥。

从 NDB 8.0.24 开始，可以从*`encryption_option`*中省略 `PASSWORD` 选项。在这种情况下，管理客户端会提示用户输入密码。

可以使用 `PASSWORD` 设置空密码（`''` 或 `""`），但不建议这样做。

可以使用以下任一命令解密加密备份：

+   **ndb_restore** `--decrypt` `--backup-password=*`password`*`

+   **ndbxfrm** `--decrypt-password=*`password`*` *`input_file`* *`output_file`*

+   **ndb_print_backup_file** `-P` *`password`* *`file_name`*

NDB 8.0.24 及更高版本支持以下附加命令：

+   **ndb_restore** `--decrypt` `--backup-password-from-stdin`

+   **ndbxfrm** `--decrypt-password-from-stdin` *`input_file`* *`output_file`*

+   **ndb_print_backup_file** `--backup-password=*`password`*` *`file_name`*

+   **ndb_print_backup_file** `--backup-password-from-stdin` *`file_name`*

+   **ndb_mgm** `--backup-password-from-stdin` `--execute "START BACKUP ..."`

查看这些程序的描述以获取更多信息，例如可能需要的其他选项。

*`wait_option`* 可用于确定在发出 `START BACKUP` 命令后何时将控制权返回给管理客户端，如下列表所示：

+   如果指定了 `NOWAIT`，管理客户端会立即显示提示，如下所示：

    ```sql
    ndb_mgm> START BACKUP NOWAIT
    ndb_mgm>
    ```

    在这种情况下，管理客户端可以在打印备份过程的进度信息的同时使用。

+   使用 `WAIT STARTED`，管理客户端会等到备份开始后再将控制权返回给用户，如下所示：

    ```sql
    ndb_mgm> START BACKUP WAIT STARTED
    Waiting for started, this may take several minutes
    Node 2: Backup 3 started from node 1
    ndb_mgm>
    ```

+   **`WAIT COMPLETED`** 导致管理客户端在备份过程完成之前等待，然后将控制权返回给用户。

`WAIT COMPLETED` 是默认选项。

*`snapshot_option`*可用于确定备份是否与发出`START BACKUP`命令时的集群状态匹配，或者与备份完成时的状态匹配。`SNAPSHOTSTART`导致备份与备份开始时的集群状态匹配；`SNAPSHOTEND`导致备份反映备份完成时的集群状态。`SNAPSHOTEND`是默认值，并与以前的 NDB Cluster 版本中找到的行为相匹配。

注意

如果在`START BACKUP`中使用`SNAPSHOTSTART`选项，并且启用了`CompressedBackup`参数，则只有数据和控制文件会被压缩，日志文件不会被压缩。

如果同时使用*`wait_option`*和*`snapshot_option`*，它们可以以任何顺序指定。例如，假设不存在 ID 为 4 的现有备份，则以下所有命令都是有效的：

```sql
START BACKUP WAIT STARTED SNAPSHOTSTART
START BACKUP SNAPSHOTSTART WAIT STARTED
START BACKUP 4 WAIT COMPLETED SNAPSHOTSTART
START BACKUP SNAPSHOTEND WAIT COMPLETED
START BACKUP 4 NOWAIT SNAPSHOTSTART
```

创建备份的过程包括以下步骤：

1.  启动管理客户端（**ndb_mgm**），如果尚未运行。

1.  执行**`START BACKUP`**命令。这将产生几行输出，指示备份的进度，如下所示：

    ```sql
    ndb_mgm> START BACKUP
    Waiting for completed, this may take several minutes
    Node 2: Backup 1 started from node 1
    Node 2: Backup 1 started from node 1 completed
     StartGCP: 177 StopGCP: 180
     #Records: 7362 #LogRecords: 0
     Data: 453648 bytes Log: 0 bytes
    ndb_mgm>
    ```

1.  当备份开始时，管理客户端显示此消息：

    ```sql
    Backup *backup_id* started from node *node_id*
    ```

    *`backup_id`*是此特定备份的唯一标识符。如果未配置其他方式保存此标识符，则将其保存在集群日志中。*`node_id`*是协调备份与数据节点的管理服务器的标识符。在备份过程的这一点上，集群已经接收并处理了备份请求。这并不意味着备份已经完成。这种情况的示例如下：

    ```sql
    Node 2: Backup 1 started from node 1
    ```

1.  管理客户端通过类似于这样的消息指示备份已经开始：

    ```sql
    Backup *backup_id* started from node *node_id* completed
    ```

    与指示备份已经开始的通知一样，*`backup_id`*是此特定备份的唯一标识符，而*`node_id`*是协调备份与数据节点的管理服务器的节点 ID。此输出还附带其他信息，包括相关的全局检查点、备份的记录数以及数据的大小，如下所示：

    ```sql
    Node 2: Backup 1 started from node 1 completed
     StartGCP: 177 StopGCP: 180
     #Records: 7362 #LogRecords: 0
     Data: 453648 bytes Log: 0 bytes
    ```

也可以通过在系统 shell 中调用**ndb_mgm**并使用`-e`或`--execute`选项来执行备份，如下例所示：

```sql
$> ndb_mgm -e "START BACKUP 6 WAIT COMPLETED SNAPSHOTSTART"
```

在这种方式下使用`START BACKUP`时，必须指定备份 ID。

集群备份默认创建在每个数据节点的`BACKUP`子目录中。这可以通过`DataDir`中的一个或多个数据节点，或者在`config.ini`文件中使用`BackupDataDir`配置参数来覆盖。为具有给定*`backup_id`*的备份创建的备份文件存储在备份目录中名为`BACKUP-*backup_id*`的子目录中。

**取消备份。** 要取消或中止已经进行中的备份，请执行以下步骤：

1.  启动管理客户端。

1.  执行此命令：

    ```sql
    ndb_mgm> ABORT BACKUP *backup_id*
    ```

    数字*`backup_id`*是在备份启动时管理客户端的响应中包含的备份标识符（在消息`Backup *backup_id* started from node *management_node_id*`中）。

1.  管理客户端使用`Abort of backup *backup_id* ordered`确认中止请求。

    注意

    此时，管理客户端尚未收到集群数据节点对此请求的响应，备份实际上尚未被中止。

1.  备份被中止后，管理客户端以类似于以下所示的方式报告此事实：

    ```sql
    Node 1: Backup 3 started from 5 has been aborted.
      Error: 1321 - Backup aborted by user request: Permanent error: User defined error
    Node 3: Backup 3 started from 5 has been aborted.
      Error: 1323 - 1323: Permanent error: Internal error
    Node 2: Backup 3 started from 5 has been aborted.
      Error: 1323 - 1323: Permanent error: Internal error
    Node 4: Backup 3 started from 5 has been aborted.
      Error: 1323 - 1323: Permanent error: Internal error
    ```

    在此示例中，我们展示了一个具有 4 个数据节点的集群的示例输出，要中止的备份的序列号为`3`，并且集群管理客户端连接的管理节点具有节点 ID`5`。完成中止备份的第一个节点报告中止的原因是由于用户的请求。（其余节点报告备份由于未指定的内部错误而中止。）

    注意

    不能保证集群节点以任何特定顺序响应`ABORT BACKUP`命令。

    `备份 *backup_id* 从节点 *management_node_id* 开始已中止`消息意味着备份已终止，并且与此备份相关的所有文件已从集群文件系统中删除。

也可以使用以下命令从系统 shell 中中止正在进行的备份：

```sql
$> ndb_mgm -e "ABORT BACKUP *backup_id*"
```

注意

如果在发出`ABORT BACKUP`时没有正在运行 ID 为*`backup_id`*的备份，管理客户端不会做出响应，也不会在集群日志中指示发送了无效的中止命令。
