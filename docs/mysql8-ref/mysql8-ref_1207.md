> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-in-core-file.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-in-core-file.html)

#### 17.8.3.7 排除核心文件中的缓冲池页面

核心文件记录了运行进程的状态和内存映像。由于缓冲池位于主内存中，并且运行进程的内存映像被转储到核心文件中，当**mysqld**进程死机时，具有大缓冲池的系统可能会产生大型核心文件。

大型核心文件可能会带来许多问题，包括编写它们所需的时间、它们占用的磁盘空间以及传输大文件所面临的挑战。

为了减小核心文件大小，您可以禁用`innodb_buffer_pool_in_core_file`变量，以在核心转储中省略缓冲池页面。`innodb_buffer_pool_in_core_file`变量在 MySQL 8.0.14 中引入，并默认启用。

如果您担心将数据库页面转储到可能在组织内外共享用于调试目的的核心文件中，从安全角度考虑��除缓冲池页面可能也是值得的。

注意

当**mysqld**进程死机时，访问缓冲池页面中的数据可能在某些调试场景中很有益。如果不确定是否包含或排除缓冲池页面，请咨询 MySQL 支持。

禁用`innodb_buffer_pool_in_core_file`仅在启用`core_file`变量且操作系统支持`MADV_DONTDUMP`非 POSIX 扩展到[madvise()](http://man7.org/linux/man-pages/man2/madvise.2.html)系统调用时生效，该扩展在 Linux 3.4 及更高版本中受支持。`MADV_DONTDUMP`扩展导致指定范围内的页面被排除在核心转储之外。

假设操作系统支持`MADV_DONTDUMP`扩展，请使用`--core-file`和`--innodb-buffer-pool-in-core-file=OFF`选项启动服务器，以生成不包含缓冲池页面的核心文件。

```sql
$> mysqld --core-file --innodb-buffer-pool-in-core-file=OFF
```

`core_file`变量是只读的，默认情况下禁用。通过在启动时指定`--core-file`选项来启用它。`innodb_buffer_pool_in_core_file`变量是动态的。它可以在启动时指定，也可以使用`SET`语句在运行时配置。

```sql
mysql> SET GLOBAL innodb_buffer_pool_in_core_file=OFF;
```

如果禁用了`innodb_buffer_pool_in_core_file`变量，但操作系统不支持`MADV_DONTDUMP`，或者`madvise()`失败，则会向 MySQL 服务器错误日志写入警告，并禁用`core_file`变量以防止意外包含缓冲池页的核心文件写入。如果只读的`core_file`变量被禁用，则必须重新启动服务器才能再次启用它。

以下表格显示了确定是否生成核心文件以及是否包含缓冲池页的配置和`MADV_DONTDUMP`支持方案。

**表 17.4 核心文件配置方案**

| `core_file`变量 | `innodb_buffer_pool_in_core_file`变量 | madvise() MADV_DONTDUMP 支持 | 结果 |
| --- | --- | --- | --- |
| 关（默认） | 与结果无关 | 与结果无关 | 不生成核心文件 |
| 开 | 开（默认） | 与结果无关 | 核心文件生成时包含缓冲池页 |
| 开 | 关 | 是 | 生成的核心文件不包含缓冲池页 |
| 开 | 关 | 否 | 不生成核心文件，`core_file`被禁用，并且向服务器错误日志写入警告 |

通过禁用`innodb_buffer_pool_in_core_file`变量来减小核心文件大小取决于缓冲池的大小，但也受到`InnoDB`页面大小的影响。较小的页面大小意味着相同数据量需要更多的页面，而更多的页面意味着更多的页面元数据。以下表格提供了对于具有不同页面大小的 1GB 缓冲池可能看到的大小减小示例。

**表 17.5 包含和不包含缓冲池页的核心文件大小**

| `innodb_page_size`设置 | 包含缓冲池页（`innodb_buffer_pool_in_core_file=ON`） | 不包含缓冲池页（`innodb_buffer_pool_in_core_file=OFF`） |
| --- | --- | --- |
| 4KB | 2.1GB | 0.9GB |
| 64KB | 1.7GB | 0.7GB |
