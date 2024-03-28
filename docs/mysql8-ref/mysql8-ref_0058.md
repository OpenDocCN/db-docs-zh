# 2.3.7 Windows 平台限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-restrictions.html)

在 Windows 平台上使用 MySQL 时，有以下限制：

+   **进程内存**

    在 Windows 32 位平台上，默认情况下无法在单个进程中使用超过 2GB 的 RAM，包括 MySQL。这是因为 Windows 32 位的物理地址限制为 4GB，而 Windows 的默认设置是在内核（2GB）和用户/应用程序（2GB）之间分割虚拟地址空间。

    一些 Windows 版本在启动时设置以通过减少内核应用程序来启用更大的应用程序。或者，要使用超过 2GB，请使用 64 位版本的 Windows。

+   **文件系统别名**

    在使用 `MyISAM` 表时，不能在 Windows 中使用别名链接到另一个卷上的数据文件，然后再链接回主 MySQL `datadir` 位置。

    这个功能通常用于将数据和索引文件移动到 RAID 或其他快速解决方案。

+   **有限的端口数量**

    Windows 系统有大约 4,000 个端口可用于客户端连接，当端口上的连接关闭后，需要两到四分钟才能重新使用该端口。在客户端与服务器之间以高速率连接和断开的情况下，可能会在关闭的端口再次可用之前用尽所有可用端口。如果发生这种情况，MySQL 服务器看起来无响应，尽管它正在运行。端口也可能被机器上运行的其他应用程序使用，这种情况下，可供 MySQL 使用的端口数量较低。

    有关此问题的更多信息，请参见 [`support.microsoft.com/kb/196271`](https://support.microsoft.com/kb/196271)。

+   **`DATA DIRECTORY` 和 `INDEX DIRECTORY`**

    `CREATE TABLE` 语句的 `DATA DIRECTORY` 子句仅支持 Windows 上的 `InnoDB` 表，如 Section 17.6.1.2, “Creating Tables Externally” 中所述。对于 `MyISAM` 和其他存储引擎，在 Windows 和其他具有非功能性 `realpath()` 调用的平台上，`CREATE TABLE` 的 `DATA DIRECTORY` 和 `INDEX DIRECTORY` 子句将被忽略。

+   **`DROP DATABASE`**

    你不能删除另一个会话正在使用的数据库。

+   **不区分大小写的名称**

    Windows 系统上的文件名不区分大小写，因此 MySQL 数据库和表名在 Windows 上也不区分大小写。唯一的限制是数据库和表名必须在给定语句中始终使用相同的大小写。参见 Section 11.2.3, “Identifier Case Sensitivity”。

+   **目录和文件名**

    在 Windows 上，MySQL 服务器仅支持与当前 ANSI 代码页兼容的目录和文件名。例如，以下日文目录名在西方区域设置（代码页 1252）中无法使用：

    ```sql
    datadir="C:/私たちのプロジェクトのデータ"
    ```

    相同的限制也适用于 SQL 语句中引用的目录和文件名，比如`LOAD DATA`中的数据文件路径名。

+   **路径名分隔符 `\` 字符**

    Windows 中的路径名组件由 `\` 字符分隔，这也是 MySQL 中的转义字符。如果你正在使用`LOAD DATA`或`SELECT ... INTO OUTFILE`，请使用带有 `/` 字符的 Unix 风格文件名：

    ```sql
    mysql> LOAD DATA INFILE 'C:/tmp/skr.txt' INTO TABLE skr;
    mysql> SELECT * INTO OUTFILE 'C:/tmp/skr.txt' FROM skr;
    ```

    或者，你必须将 `\` 字符加倍：

    ```sql
    mysql> LOAD DATA INFILE 'C:\\tmp\\skr.txt' INTO TABLE skr;
    mysql> SELECT * INTO OUTFILE 'C:\\tmp\\skr.txt' FROM skr;
    ```

+   **管道问题**

    从 Windows 命令行提示符中，管道无法可靠工作。如果管道包含字符 `^Z` / `CHAR(24)`，Windows 认为已经遇到文件结尾并中止程序。

    当你尝试应用二进制日志时，这主要是一个问题：

    ```sql
    C:\> mysqlbinlog *binary_log_file* | mysql --user=root
    ```

    如果你在应用日志时遇到问题，并怀疑是因为 `^Z` / `CHAR(24)` 字符，你可以使用以下解决方法：

    ```sql
    C:\> mysqlbinlog *binary_log_file* --result-file=/tmp/bin.sql
    C:\> mysql --user=root --execute "source /tmp/bin.sql"
    ```

    后一条命令也可用于可靠地读取可能包含二进制数据的任何 SQL 文件。
