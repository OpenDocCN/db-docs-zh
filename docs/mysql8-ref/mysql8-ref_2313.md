> 原��：[`dev.mysql.com/doc/refman/8.0/en/not-enough-file-handles.html`](https://dev.mysql.com/doc/refman/8.0/en/not-enough-file-handles.html)

#### B.3.2.16 文件未找到及类似错误

如果你遇到 `ERROR '*`file_name`*' not found (errno: 23)`，`Can't open file: *`file_name`* (errno: 24)`，或者从 MySQL 得到任何带有 `errno 23` 或 `errno 24` 的错误，这意味着你为 MySQL 服务器分配的文件描述符不够。你可以使用 **perror** 实用程序来获取关于错误编号的描述：

```sql
$> perror 23
OS error code  23:  File table overflow
$> perror 24
OS error code  24:  Too many open files
$> perror 11
OS error code  11:  Resource temporarily unavailable
```

这里的问题是 **mysqld** 尝试同时保持打开太多文件。你可以告诉 **mysqld** 要么不要一次打开那么多文件，要么增加可用于 **mysqld** 的文件描述符数量。

要告诉 **mysqld** 一次打开较少的文件，你可以通过减少 `table_open_cache` 系统变量的值来使表缓存更小（默认值为 64）。这可能无法完全防止文件描述符用尽，因为在某些情况下，服务器可能会尝试临时扩展缓存大小，如 10.4.3.1 “MySQL 如何打开和关闭表” 中所述。减少 `max_connections` 的值也会减少打开文件的数量（默认值为 100）。

要更改可用于 **mysqld** 的文件描述符数量，你可以使用 `--open-files-limit` 选项来设置 **mysqld_safe** 或设置 `open_files_limit` 系统变量。参见 7.1.8 “Server System Variables”。设置这些值的最简单方法是向你的选项文件添加一个选项。参见 6.2.2.2 “Using Option Files”。如果你有一个不支持设置文件打开限制的旧版本 **mysqld**，你可以编辑 **mysqld_safe** 脚本。脚本中有一行被注释掉的 **ulimit -n 256**。你可以去掉 `#` 字符来取消注释这行，并将数字 `256` 更改为要提供给 **mysqld** 的文件描述符数量。

`--open-files-limit` 和 **ulimit** 可以增加文件描述符的数量，但只能增加到操作系统强加的限制。还有一个“硬”限制，只有在以 `root` 身份启动 **mysqld_safe** 或 **mysqld** 时才能覆盖（请记住，在这种情况下，您还需要使用 `--user` 选项启动服务器，以便在启动后不继续以 `root` 运行）。如果您需要增加操作系统对每个进程可用文件描述符的限制，请查阅系统文档。

注意

如果您使用 **tcsh** shell，**ulimit** 不起作用！当您请求当前限制时，**tcsh** 还会报告不正确的值。在这种情况下，您应该使用 **sh** 启动 **mysqld_safe**。
