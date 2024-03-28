# 9.6.3 如何修复 MyISAM 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-repair.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-repair.html)

本节讨论了如何在`MyISAM`表（扩展名为`.MYI`和`.MYD`）上使用**myisamchk**。

您还可以使用`CHECK TABLE`和`REPAIR TABLE`语句来检查和修复`MyISAM`表。请参阅 Section 15.7.3.2, “CHECK TABLE Statement”和 Section 15.7.3.5, “REPAIR TABLE Statement”。

损坏表的症状包括意外中止的查询和可观察到的错误，例如：

+   找不到文件`*`tbl_name`*.MYI`（错误代码：*`nnn`*）

+   文件意外结束

+   记录文件已崩溃

+   从表处理程序得到错误*`nnn`*

要获取有关错误的更多信息，请运行**perror** *`nnn`*，其中*`nnn`*是错误编号。以下示例显示如何使用**perror**查找指示表存在问题的最常见错误编号的含义：

```sql
$> perror 126 127 132 134 135 136 141 144 145
MySQL error code 126 = Index file is crashed
MySQL error code 127 = Record-file is crashed
MySQL error code 132 = Old database file
MySQL error code 134 = Record was already deleted (or record file crashed)
MySQL error code 135 = No more room in record file
MySQL error code 136 = No more room in index file
MySQL error code 141 = Duplicate unique key or constraint on write or update
MySQL error code 144 = Table is crashed and last repair failed
MySQL error code 145 = Table was marked as crashed and should be repaired
```

请注意，错误 135（记录文件中没有更多空间）和错误 136（索引文件中没有更多空间）不是简单修复的错误。在这种情况下，您必须使用`ALTER TABLE`来增加`MAX_ROWS`和`AVG_ROW_LENGTH`表选项值：

```sql
ALTER TABLE *tbl_name* MAX_ROWS=*xxx* AVG_ROW_LENGTH=*yyy*;
```

如果您不知道当前的表选项值，请使用`SHOW CREATE TABLE`。

对于其他错误，您必须修复您的表。**myisamchk**通常可以检测和修复大多数出现的问题。

修复过程包括最多三个阶段，这里进行描述。在开始之前，您应该切换到数据库目录并检查表文件的权限。在 Unix 上，请确保它们可被**mysqld**运行的用户（以及您，因为您需要访问正在检查的文件）读取。如果您需要修改文件，它们也必须可被您写入。

本节适用于表检查失败的情况（例如 Section 9.6.2, “How to Check MyISAM Tables for Errors”中描述的情况），或者您想要使用**myisamchk**提供的扩展功能。

用于表维护的**myisamchk**选项在第 6.6.4 节“myisamchk — MyISAM Table-Maintenance Utility”中有描述。**myisamchk**还有一些变量，您可以设置以控制可能改善性能的内存分配。请参阅第 6.6.4.6 节“myisamchk 内存使用”。

如果要从命令行修复表，必须先停止**mysqld**服务器。请注意，当在远程服务器上执行**mysqladmin shutdown**时，**mysqld**服务器在**mysqladmin**返回后仍可用一段时间，直到所有语句处理停止并所有索引更改已刷新到磁盘。

**第一阶段：检查您的表**

运行**myisamchk *.MYI**或**myisamchk -e *.MYI**如果时间充裕。使用`-s`（静默）选项以抑制不必要的信息。

如果**mysqld**服务器已停止，应使用`--update-state`选项告诉**myisamchk**将表标记为“已检查”。

只需修复**myisamchk**报告错误的那些表。对于这样的表，继续进行第二阶段。

如果在检查时遇到意外错误（例如`内存不足`错误），或者**myisamchk**崩溃，请转到第三阶段。

**第二阶段：简单安全修复**

首先尝试**myisamchk -r -q *`tbl_name`***（`-r -q`表示“快速恢复模式”）。这将尝试修复索引文件而不触及数据文件。如果数据文件包含应有的所有内容，并且删除链接指向数据文件内的正确位置，则应该可以工作，并且表已修复。开始修复下一个表。否则，请使用以下过程：

1.  在继续之前备份数据文件。

1.  使用**myisamchk -r *`tbl_name`***（`-r`表示“恢复模式”）。这将从数据文件中删除不正确的行和已删除的行，并重建索引文件。

1.  如果前面的步骤失败，请使用**myisamchk --safe-recover *`tbl_name`***。安全恢复模式使用一种旧的恢复方法，处理一些常规恢复模式无法处理的情况（但速度较慢）。

注意

如果您希望修复操作更快完成，应将`sort_buffer_size`和`key_buffer_size`变量的值分别设置为可用内存的约 25%，当运行**myisamchk**时。

如果在修复过程中遇到意外错误（如`内存不足`错误），或者**myisamchk**崩溃，请进入第三阶段。

**第三阶段：困难修复**

只有在索引文件中的第一个 16KB 块被破坏或包含不正确信息，或者索引文件丢失时，才应该达到这个阶段。在这种情况下，需要创建一个新的索引文件。操作如下：

1.  将数据文件移至安全位置。

1.  使用表描述文件创建新的（空）数据和索引文件：

    ```sql
    $> mysql *db_name*
    ```

    ```sql
    mysql> SET autocommit=1;
    mysql> TRUNCATE TABLE *tbl_name*;
    mysql> quit
    ```

1.  将旧数据文件复制回新创建的数据文件。（不要只是将旧文件移回新文件。您希望保留一份副本以防出现问题。）

重要提示

如果您正在使用复制功能，应在执行上述过程之前停止复制，因为这涉及文件系统操作，而 MySQL 不会记录这些操作。

返回到第二阶段。**myisamchk -r -q**应该可以工作。（这不应该是一个无限循环。）

您还可以使用`REPAIR TABLE *`tbl_name`* USE_FRM` SQL 语句，该语句会自动执行整个过程。由于服务器在使用`REPAIR TABLE`时会完成所有工作，因此不会出现工具与服务器之间的意外交互。请参阅 Section 15.7.3.5, “REPAIR TABLE Statement”。
