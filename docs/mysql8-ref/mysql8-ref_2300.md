> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-lost-connection.html`](https://dev.mysql.com/doc/refman/8.0/en/error-lost-connection.html)

#### B.3.2.3 与 MySQL 服务器的连接丢失

这个错误消息有三种可能的原因。

通常表示网络连接出现问题，如果经常出现此错误，应检查网络状况。如果错误消息中包含“during query”，那么你可能正在经历这种情况。

有时，“during query”形式发生在数百万行作为一个或多个查询的一部分被发送时。如果你知道正在发生这种情况，你应该尝试将`net_read_timeout`从默认的 30 秒增加到 60 秒或更长时间，足以完成数据传输。

更少见的情况是，当客户端尝试与服务器进行初始连接时可能会发生这种情况。在这种情况下，如果你的`connect_timeout`值仅设置为几秒钟，你可以通过将其增加到十秒钟来解决问题，如果你的距离很远或连接速度很慢，可能需要增加更多时间。你可以通过使用`SHOW GLOBAL STATUS LIKE 'Aborted_connects'`来确定是否遇到了这种不太常见的原因。每次服务器中止初始连接尝试时，该值会增加一次。你可能会看到“reading authorization packet”作为错误消息的一部分；如果是这样，这也表明这是你需要的解决方案。

如果造成这个问题的原因不是上述描述的任何一种，那么你可能遇到了一个与`BLOB`值大于`max_allowed_packet`有关的问题，这可能会导致某些客户端出现此错误。有时你可能会看到一个`ER_NET_PACKET_TOO_LARGE`错误，这证实了你需要增加`max_allowed_packet`。
