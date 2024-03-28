> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-solutions-backups-rawdata.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-backups-rawdata.html)

#### 19.4.1.2 从复制中备份原始数据

为了保证复制的文件完整性，在关闭 MySQL 复制服务器时备份原始数据文件应该进行。如果 MySQL 服务器仍在运行，后台任务可能仍在更新数据库文件，特别是涉及具有后台进程的存储引擎，如`InnoDB`。对于`InnoDB`，这些问题应该在崩溃恢复期间得到解决，但由于在备份过程中关闭复制服务器而不影响源执行的能力，可以利用这一功能。

要关闭服务器并备份文件：

1.  关闭复制 MySQL 服务器：

    ```sql
    $> mysqladmin shutdown
    ```

1.  复制数据文件。您可以使用任何适当的复制或存档工具，包括**cp**，**tar**或**WinZip**。例如，假设数据目录位于当前目录下，您可以将整个目录存档如下：

    ```sql
    $> tar cf /tmp/dbbackup.tar ./data
    ```

1.  再次启动 MySQL 服务器。在 Unix 系统下：

    ```sql
    $> mysqld_safe &
    ```

    在 Windows 系统下：

    ```sql
    C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld"
    ```

通常应该备份整个复制 MySQL 服务器的数据目录。如果您希望能够恢复数据并作为复制运行（例如，在复制发生故障时），除了数据外，您还需要复制的连接元数据存储库和应用程序元数据存储库，以及中继日志文件。这些项目在恢复复制数据后需要用于恢复复制。假设表已用于复制的连接元数据存储库和应用程序元数据存储库（请参阅 Section 19.2.4, “Relay Log and Replication Metadata Repositories”），这是 MySQL 8.0 的默认设置，这些表将与数据目录一起备份。如果文件已用于存储库，这已被弃用，您必须单独备份这些文件。如果中继日志文件已放置在与数据目录不同的位置，则必须单独备份中继日志文件。

如果您丢失了中继日志但仍有`relay-log.info`文件，您可以检查它以确定复制 SQL 线程在源二进制日志中执行到哪个程度。然后，您可以使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）与`SOURCE_LOG_FILE` | `MASTER_LOG_FILE`和`SOURCE_LOG_POS` | `MASTER_LOG_POS`选项告诉复制从那一点重新读取二进制日志。这要求源服务器上的二进制日志仍然存在。

如果您的复制品正在复制`LOAD DATA`语句，您还应该备份复制品用于此目的的目录中存在的任何`SQL_LOAD-*`文件。复制品需要这些文件来恢复任何中断的`LOAD DATA`操作的复制。此目录的位置是系统变量`replica_load_tmpdir`（从 MySQL 8.0.26 开始）或`slave_load_tmpdir`（在 MySQL 8.0.26 之前）。如果服务器未使用该变量启动，则目录位置是`tmpdir`系统变量的值。
