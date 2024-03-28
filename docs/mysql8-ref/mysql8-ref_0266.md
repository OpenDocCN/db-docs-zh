> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-priority-based-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-priority-based-filtering.html)

#### 7.4.2.5 基于优先级的错误日志过滤（log_filter_internal）

`log_filter_internal`日志过滤组件实现了一种基于错误事件优先级和错误代码的简单形式的日志过滤。要影响`log_filter_internal`如何允许或抑制写入错误日志的错误、警告和信息事件，请设置`log_error_verbosity`和`log_error_suppression_list`系统变量。

`log_filter_internal`是内置的并默认启用。如果禁用此过滤器，`log_error_verbosity`和`log_error_suppression_list`将不起作用，因此必须在需要时使用另一个过滤器服务执行过滤（例如，在使用`log_filter_dragnet`时使用单独的过滤规则）。有关过滤器配置的信息，请参见第 7.4.2.1 节“错误日志配置”。

+   详细程度过滤

+   抑制列表过滤

+   详细程度和抑制列表交互

##### 详细程度过滤

事件意图记录到错误日志的优先级为`ERROR`、`WARNING`或`INFORMATION`。`log_error_verbosity`系统变量控制基于哪些优先级允许写入日志的消息的详细程度，如下表所示。

| log_error_verbosity 值 | 允许的消息优先级 |
| --- | --- |
| 1 | `ERROR` |
| 2 | `ERROR`、`WARNING` |
| 3 | `ERROR`、`WARNING`、`INFORMATION` |

如果`log_error_verbosity`为 2 或更高，则服务器会记录关于对基于语句的日志记录不安全的语句的消息。如果值为 3，则服务器会记录有关新连接尝试的中止连接和访问被拒绝的错误。请参见第 B.3.2.9 节“通信错误和中止连接”。

如果使用复制，建议将`log_error_verbosity`值设置为 2 或更高，以获取有关正在发生的情况的更多信息，例如有关网络故障和重新连接的消息。

如果在副本上`log_error_verbosity`为 2 或更高，则副本会将消息打印到错误日志中，以提供有关其状态的信息，例如二进制日志和中继日志的坐标，它开始工作的位置，当它切换到另一个中继日志时，重新连接后等等。

存在一个`SYSTEM`的消息优先级，不受冗长过滤的影响。关于非错误情况的系统消息会被打印到错误日志中，无论`log_error_verbosity`的值是多少。这些消息包括启动和关闭消息，以及一些重要的设置更改。

在 MySQL 错误日志中，系统消息标记为“System”。其他日志接收器可能会或可能不会遵循相同的约定，在生成的日志中，系统消息可能被分配给信息优先级级别使用的标签，例如“Note”或“Information”。如果基于消息标签应用任何额外的过滤或重定向以进行日志记录，系统消息不会覆盖您的过滤器，而是以与其他消息相同的方式处理。

##### 抑制列表过滤

`log_error_suppression_list`系统变量适用于写入错误日志的事件，并指定在出现`WARNING`或`INFORMATION`优先级时要抑制哪些事件。例如，如果某种类型的警告被认为是错误日志中频繁发生但不感兴趣的“噪音”，则可以将其抑制。`log_error_suppression_list`不会抑制具有`ERROR`或`SYSTEM`优先级的消息。

`log_error_suppression_list`的值可以是空字符串以表示无抑制，或者是一个或多个逗号分隔值的列表，指示要抑制的错误代码。错误代码可以用符号形式或数字形式指定。可以使用或不使用`MY-`前缀指定数字代码。数字部分的前导零不重要。允许的代码格式示例：

```sql
ER_SERVER_SHUTDOWN_COMPLETE
MY-000031
000031
MY-31
31
```

为了可读性和可移植性，符号值比数字值更可取。

尽管要抑制的代码可以用符号形式或数字形式表示，但每个代码的数字值必须在允许的范围内：

+   1 至 999：服务器和客户端使用的全局错误代码。

+   10000 及以上：服务器错误代码，意在写入错误日志（不发送给客户端）。

此外，指定的每个错误代码必须实际被 MySQL 使用。尝试指定不在允许范围内或在允许范围内但未被 MySQL 使用的代码会产生错误，并且`log_error_suppression_list`的值保持不变。

有关错误代码范围、每个范围内定义的错误符号和数字的信息，请参见第 B.1 节，“错误消息来源和元素”，以及 MySQL 8.0 错误消息参考。

服务器可以为给定错误代码生成具有不同优先级的消息，因此与`log_error_suppression_list`中列出的错误代码相关联的消息的抑制取决于其优先级。假设变量的值为`'ER_PARSER_TRACE,MY-010001,10002'`。那么`log_error_suppression_list`对这些代码的消息产生以下影响：

+   生成具有`WARNING`或`INFORMATION`优先级的消息会被抑制。

+   生成具有`ERROR`或`SYSTEM`优先级的消息不会被抑制。

##### 详细度和抑制列表的交互

`log_error_verbosity`的效果与`log_error_suppression_list`的效果相结合。考虑使用以下设置启动的服务器：

```sql
[mysqld]
log_error_verbosity=2 # error and warning messages only
log_error_suppression_list='ER_PARSER_TRACE,MY-010001,10002'
```

在这种情况下，`log_error_verbosity`允许具有`ERROR`或`WARNING`优先级的消息，并丢弃具有`INFORMATION`优先级的消息。在未被丢弃的消息中，`log_error_suppression_list`会丢弃具有`WARNING`优先级和任何命名错误代码的消息。

注意

示例中显示的`log_error_verbosity`值为 2，这也是其默认值，因此该变量对`INFORMATION`消息的影响默认情况下如上所述，无需显式设置。如果要使`log_error_suppression_list`影响具有`INFORMATION`优先级的消息，必须将`log_error_verbosity`设置为 3。

考虑使用以下设置启动的服务器：

```sql
[mysqld]
log_error_verbosity=1 # error messages only
```

在这种情况下，`log_error_verbosity` 允许具有`ERROR`优先级的消息，并丢弃具有`WARNING`或`INFORMATION`优先级的消息。设置`log_error_suppression_list` 不起作用，因为它可能抑制的所有错误代码已经由`log_error_verbosity` 设置丢弃了。
