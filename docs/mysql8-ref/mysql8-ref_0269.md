> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-syslog.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-syslog.html)

#### 7.4.2.8 将错误日志记录到系统日志

可以让**mysqld**将错误日志写入系统日志（Windows 上的事件日志，Unix 和类 Unix 系统上的`syslog`）。

本节描述如何使用内置过滤器`log_filter_internal`和系统日志接收器`log_sink_syseventlog`配置错误日志记录，以立即生效并在后续服务器启动时生效。有关配置错误日志的一般信息，请参阅 Section 7.4.2.1, “错误日志配置”。

要启用系统日志接收器，首先加载接收器组件，然后修改`log_error_services`的值：

```sql
INSTALL COMPONENT 'file://component_log_sink_syseventlog';
SET PERSIST log_error_services = 'log_filter_internal; log_sink_syseventlog';
```

要在服务器启动时设置`log_error_services`生效，请使用 Section 7.4.2.1, “错误日志配置”中的说明。这些说明也适用于其他错误日志系统变量。

注意

对于 MySQL 8.0 配置，您必须显式启用将错误日志记录到系统日志。这与 MySQL 5.7 及更早版本不同，在 Windows 上默认启用将错误日志记录到系统日志，并且在所有平台上不需要加载组件。

将错误日志记录到系统日志可能需要额外的系统配置。请查阅您平台的系统日志文档。

在 Windows 上，写入应用程序日志中的事件日志错误消息具有以下特征：

+   标记为`Error`、`Warning`和`Note`的条目会被写入事件日志，但不包括来自各个存储引擎的信息声明。

+   事件日志条目的来源是`MySQL`（或`MySQL-*`tag`*`，如果`syseventlog.tag`定义为*`tag`*）。

在 Unix 和类 Unix 系统上，记录到系统日志使用`syslog`。以下系统变量会影响`syslog`消息：

+   `syseventlog.facility`：`syslog`消息的默认设施是`daemon`。设置此变量以指定不同的设施。

+   `syseventlog.include_pid`：是否在每行`syslog`输出中包含服务器进程 ID。

+   `syseventlog.tag`：此变量定义要添加到`syslog`消息中服务器标识符（`mysqld`）的标记。如果定义了标记，则标记将以前导连字符附加到标识符后面。

注意

在 MySQL 8.0.13 之前，请使用`log_syslog_facility`、`log_syslog_include_pid`和`log_syslog_tag`系统变量，而不是`syseventlog.*`xxx`*`变量。

MySQL 使用自定义标签“系统”来表示关于非错误情况的重要系统消息，例如启动、关闭和一些重要设置更改。在不支持自定义标签的日志中，包括 Windows 上的事件日志和 Unix 及类 Unix 系统上的`syslog`，系统消息被分配给信息优先级级别使用的标签。然而，即使 MySQL `log_error_verbosity` 设置通常排除信息级别的消息，这些消息也会被打印到日志中。

当日志接收器必须以“信息”标签而不是“系统”标签回退时，并且日志事件在 MySQL 服务器外进一步处理（例如，通过`syslog`配置进行过滤或转发），这些事件可能默认由次要应用程序处理为“信息”优先级而不是“系统”优先级。
