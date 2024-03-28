> 原文：[`dev.mysql.com/doc/refman/8.0/en/timezone-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/timezone-problems.html)

#### B.3.3.7 时区问题

如果`SELECT NOW()`返回的值是 UTC 时间而不是本地时间，您需要告诉服务器当前的时区。如果`UNIX_TIMESTAMP()`返回错误的值，也是一样的。这应该针对服务器运行的环境进行设置（例如，在**mysqld_safe**或**mysql.server**中）。请参阅第 6.9 节，“环境变量”。

您可以通过在启动**mysqld**之前设置`TZ`环境变量，或者使用`--timezone=*`timezone_name`*`选项来为服务器设置时区。

允许的`--timezone`或`TZ`的值取决于系统。请查阅您的操作系统文档，了解可接受的数值。
