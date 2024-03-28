> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-optimize-tablespace-page-allocation.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-optimize-tablespace-page-allocation.html)

#### 17.6.3.8 优化 Linux 上的表空间空间分配

截至 MySQL 8.0.22，您可以优化`InnoDB`在 Linux 上为文件表和通用表空间分配空间的方式。默认情况下，当需要额外空间时，`InnoDB`会为表空间分配页面，并在这些页面上物理写入 NULL。如果频繁分配新页面，此行为可能会影响性能。从 MySQL 8.0.22 开始，您可以在 Linux 系统上禁用`innodb_extend_and_initialize`以避免在新分配的表空间页面上物理写入 NULL。当禁用`innodb_extend_and_initialize`时，空间将使用`posix_fallocate()`调用分配给表空间文件，而无需物理写入 NULL。

当使用`posix_fallocate()`调用分配页面时，默认情况下扩展大小较小，通常一次只分配几个页面，这可能导致碎片化并增加随机 I/O。为避免此问题，在启用`posix_fallocate()`调用时增加表空间扩展大小。可以使用`AUTOEXTEND_SIZE`选项将表空间扩展大小增加到 4GB。更多信息，请参见第 17.6.3.9 节，“表空间 AUTOEXTEND_SIZE 配置”。

`InnoDB`在分配新表空间页面之前写入重做日志记录。如果页面分配操作中断，则在恢复期间从重做日志记录中重放该操作。 （从重做日志记录中重放的页面分配操作会在新分配的页面上物理写入 NULL。）无论`innodb_extend_and_initialize`设置如何，都会在分配页面之前写入重做日志记录。

在非 Linux 系统和 Windows 上，`InnoDB`将新页面分配给表空间并在这些页面上物理写入 NULL，这是默认行为。在这些系统上尝试禁用`innodb_extend_and_initialize`将返回以下错误：

在此平台上不支持更改 innodb_extend_and_initialize。回退到默认设置。
