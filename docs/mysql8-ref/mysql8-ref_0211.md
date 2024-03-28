> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-backup.html)

#### 6.6.9.3 使用 mysqlbinlog 备份二进制日志文件

默认情况下，**mysqlbinlog** 读取二进制日志文件并以文本格式显示其内容。这使您能够更轻松地检查文件中的事件，并重新执行它们（例如，通过将输出用作 **mysql** 的输入）。**mysqlbinlog** 可以直接从本地文件系统读取日志文件，或者使用 `--read-from-remote-server` 选项，它可以连接到服务器并从该服务器请求二进制日志内容。**mysqlbinlog** 将文本输出写入其标准输出，或者如果给定了 `--result-file=*`file_name`*` 选项，则写入到指定文件名的文件中。

+   mysqlbinlog 备份功能

+   mysqlbinlog 备份选项

+   静态和实时备份

+   输出文件命名

+   示例：使用 mysqldump + mysqlbinlog 进行备份和恢复

+   mysqlbinlog 备份限制

##### mysqlbinlog 备份功能

**mysqlbinlog**可以读取二进制日志文件并写入包含相同内容的新文件，即以二进制格式而不是文本格式。这种能力使您可以轻松地以原始格式备份二进制日志。**mysqlbinlog**可以进行静态备份，备份一组日志文件并在到达最后一个文件的末尾时停止。它还可以进行连续（“实时”）备份，在到达最后一个日志文件的末尾时保持与服务器的连接，并继续复制生成的新事件。在连续备份操作中，**mysqlbinlog**会一直运行，直到连接结束（例如，服务器退出）或**mysqlbinlog**被强制终止。当连接结束时，**mysqlbinlog**不会等待并重试连接，不像复制服务器。要在服务器重新启动后继续实时备份，您还必须重新启动**mysqlbinlog**。

重要提示

**mysqlbinlog**可以备份加密和未加密的二进制日志文件。然而，使用**mysqlbinlog**生成的加密二进制日志文件的副本以未加密格式存储。

##### mysqlbinlog 备份选项

二进制日志备份要求您至少使用两个选项调用**mysqlbinlog**：

+   `--read-from-remote-server`（或`-R`）选项告诉**mysqlbinlog**连接到服务器并请求其二进制日志。（这类似于复制服务器连接到其复制源服务器。）

+   `--raw`选项告诉**mysqlbinlog**写入原始（二进制）输出，而不是文本输出。

除了`--read-from-remote-server`之外，通常还需要指定其他选项：`--host`指示服务器运行的位置，您可能还需要指定连接选项，如`--user`和`--password`。

与`--raw`一起使用的其他几个选项很有用：

+   `--stop-never`：在到达最后一个日志文件并继续读取新事件后，保持与服务器的连接。

+   `--connection-server-id=*`id`*`：连接到服务器时**mysqlbinlog**报告的服务器 ID。当使用`--stop-never`时，默认报告的服务器 ID 为 1。如果这与复制服务器或另一个**mysqlbinlog**进程的 ID 发生冲突，请使用`--connection-server-id`指定替代服务器 ID。参见第 6.6.9.4 节，“指定 mysqlbinlog 服务器 ID”。

+   `--result-file`：输出文件名的前缀，稍后会描述。

##### 静态和实时备份

使用**mysqlbinlog**备份服务器的二进制日志文件时，必须指定服务器上实际存在的文件名。如果不知道文件名，连接到服务器并使用`SHOW BINARY LOGS`语句查看当前的文件名。假设该语句产生以下输出：

```sql
mysql> SHOW BINARY LOGS;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000130 |     27459 | No        |
| binlog.000131 |     13719 | No        |
| binlog.000132 |     43268 | No        |
+---------------+-----------+-----------+
```

有了这些信息，您可以使用**mysqlbinlog**将二进制日志备份到当前目录，如下所示（每个命令单独一行输入）：

+   要对`binlog.000130`到`binlog.000132`进行静态备份，请使用以下任一命令：

    ```sql
    mysqlbinlog --read-from-remote-server --host=*host_name* --raw
      binlog.000130 binlog.000131 binlog.000132

    mysqlbinlog --read-from-remote-server --host=*host_name* --raw
      --to-last-log binlog.000130
    ```

    第一个命令明确指定每个文件名。第二个只命名第一个文件，并使用`--to-last-log`来读取到最后一个文件。这两个命令之间的区别在于，如果服务器在**mysqlbinlog**到达`binlog.000132`的末尾之前打开`binlog.000133`，第一个命令不会读取它，但第二个命令会。

+   要进行实时备份，其中**mysqlbinlog**从`binlog.000130`开始复制现有日志文件，然后保持连接以复制服务器生成的新事件：

    ```sql
    mysqlbinlog --read-from-remote-server --host=*host_name* --raw
      --stop-never binlog.000130
    ```

    使用`--stop-never`，不需要指定`--to-last-log`以读取到最后一个日志文件，因为该选项已经隐含。

##### 输出文件命名

不使用`--raw`，**mysqlbinlog**会生成文本输出，而`--result-file`选项（如果提供）指定了所有输出写入的单个文件的名称。使用`--raw`，**mysqlbinlog**为从服务器传输的每个日志文件写入一个二进制输出文件。默认情况下，**mysqlbinlog**将文件写入当前目录，文件名与原始日志文件相同。要修改输出文件名，请使用`--result-file`选项。与`--raw`一起，`--result-file`选项值被视为修改输出文件名的前缀。

假设服务器当前有名为`binlog.000999`及以上的二进制日志文件。如果使用**mysqlbinlog --raw**备份文件，则`--result-file`选项会生成如下表所示的输出文件名。您可以通过在`--result-file`值前加上目录路径来将文件写入特定目录。如果`--result-file`值仅包含目录名，则该值必须以路径名分隔符字符结尾。如果文件存在，则输出文件将被覆盖。

| `--result-file` 选项 | 输出文件名 |
| --- | --- |
| `--result-file=x` | `xbinlog.000999`及以上 |
| `--result-file=/tmp/` | `/tmp/binlog.000999`及以上 |
| `--result-file=/tmp/x` | `/tmp/xbinlog.000999`及以上 |

##### 示例：备份和恢复的 mysqldump + mysqlbinlog

以下示例描述了一个简单的场景，展示了如何使用**mysqldump**和**mysqlbinlog**一起备份服务器的数据和二进制日志，以及如何使用备份来恢复服务器，如果发生数据丢失。该示例假定服务器运行在主机*`host_name`*上，其第一个二进制日志文件名为`binlog.000999`。每个命令都要单独输入。

使用**mysqlbinlog**进行二进制日志的连续备份：

```sql
mysqlbinlog --read-from-remote-server --host=*host_name* --raw
  --stop-never binlog.000999
```

使用**mysqldump**创建一个转储文件作为服务器数据的快照。使用`--all-databases`，`--events`和`--routines`来备份所有数据，并使用`--master-data=2`在转储文件中包含当前的二进制日志坐标。

```sql
mysqldump --host=*host_name* --all-databases --events --routines --master-data=2> *dump_file*
```

定期执行**mysqldump**命令以根据需要创建更新的快照。

如果发生数据丢失（例如，服务器意外退出），请使用最近的转储文件来恢复数据：

```sql
mysql --host=*host_name* -u root -p < *dump_file*
```

然后使用二进制日志备份来重新执行在转储文件中列出坐标之后编写的事件。假设文件中的坐标看起来像这样：

```sql
-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.001002', MASTER_LOG_POS=27284;
```

如果最近备份的日志文件命名为`binlog.001004`，请像这样重新执行日志事件：

```sql
mysqlbinlog --start-position=27284 binlog.001002 binlog.001003 binlog.001004
  | mysql --host=*host_name* -u root -p
```

您可能会发现将备份文件（转储文件和二进制日志文件）复制到服务器主机上会更容易执行恢复操作，或者如果 MySQL 不允许远程`root`访问。

##### mysqlbinlog 备份限制

使用**mysqlbinlog**的二进制日志备份受到以下限制：

+   **mysqlbinlog**在连接丢失时（例如，服务器重新启动或网络中断）不会自动重新连接到 MySQL 服务器。

+   备份的延迟类似于复制服务器的延迟。
