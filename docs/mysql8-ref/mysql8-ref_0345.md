# 7.9.2 调试 MySQL 客户端

> 原文：[`dev.mysql.com/doc/refman/8.0/en/debugging-client.html`](https://dev.mysql.com/doc/refman/8.0/en/debugging-client.html)

要能够使用集成的调试包调试 MySQL 客户端，您应该使用`-DWITH_DEBUG=1`配置 MySQL。参见 Section 2.8.7, “MySQL Source-Configuration Options”。

在运行客户端之前，您应该设置`MYSQL_DEBUG`环境变量：

```sql
$> MYSQL_DEBUG=d:t:O,/tmp/client.trace
$> export MYSQL_DEBUG
```

这会导致客户端在`/tmp/client.trace`中生成一个跟踪文件。

如果您在自己的客户端代码中遇到问题，您应该尝试连接到服务器，并使用已知可工作的客户端运行您的查询。通过在调试模式下运行**mysql**（假设您已经使用调试编译 MySQL）来执行此操作：

```sql
$> mysql --debug=d:t:O,/tmp/client.trace
```

这将为您提供有用的信息，以防您发送错误报告。请参见 Section 1.5, “How to Report Bugs or Problems”。

如果您的客户端在某些看起来“合法”的代码处崩溃，您应该检查您的`mysql.h`包含文件是否与您的 MySQL 库文件匹配。一个非常常见的错误是在新的 MySQL 库中使用来自旧的 MySQL 安装的旧`mysql.h`文件。
