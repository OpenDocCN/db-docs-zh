> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-relaylog-events.html`](https://dev.mysql.com/doc/refman/8.0/en/show-relaylog-events.html)

#### 15.7.7.32 `SHOW RELAYLOG EVENTS`语句

```sql
SHOW RELAYLOG EVENTS
    [IN '*log_name*']
    [FROM *pos*]
    [LIMIT [*offset*,] *row_count*]
    [*channel_option*]

*channel_option*:
    FOR CHANNEL *channel*
```

显示副本中的中继日志中的事件。如果不指定`'*`log_name`*'`，则显示第一个中继日志。此语句对源没有影响。`SHOW RELAYLOG EVENTS`需要`REPLICATION SLAVE`权限。

`LIMIT`子句的语法与`SELECT`语句相同。请参见 Section 15.2.13，“SELECT Statement”。

注意

发出不带`LIMIT`子句的`SHOW RELAYLOG EVENTS`可能会启动一个非常耗时和资源消耗的过程，因为服务器会将中继日志的完整内容（包括所有已被副本接收的修改数据的语句）返回给客户端。

可选的`FOR CHANNEL *`channel`*`子句使您能够命名语句适用于哪个复制通道。提供`FOR CHANNEL *`channel`*`子句将语句应用于特定的复制通道。如果未命名通道且没有额外通道存在，则该语句适用于默认通道。

在使用多个复制通道时，如果`SHOW RELAYLOG EVENTS`语句没有使用`FOR CHANNEL *`channel`*`子句定义通道，则会生成错误。有关更多信息，请参见 Section 19.2.2，“复制通道”。

`SHOW RELAYLOG EVENTS`为中继日志中的每个事件显示以下字段：

+   `Log_name`

    正在列出的文件的名称。

+   `Pos`

    事件发生的位置。

+   `Event_type`

    描述事件类型的标识符。

+   `Server_id`

    事件发生的服务器的服务器 ID。

+   `End_log_pos`

    此事件在源二进制日志中的`End_log_pos`值。

+   `Info`

    有关事件类型的更详细信息。此信息的格式取决于事件类型。

对于压缩的事务负载，`Transaction_payload_event`首先作为一个单元打印出来，然后解压缩并打印其中的每个事件。

一些涉及设置用户和系统变量的事件不包括在`SHOW RELAYLOG EVENTS`的输出中。要完整覆盖中继日志中的事件，请使用**mysqlbinlog**。
