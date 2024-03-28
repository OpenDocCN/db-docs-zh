# 附录 B 错误消息和常见问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-handling.html`](https://dev.mysql.com/doc/refman/8.0/en/error-handling.html)

**目录**

B.1 错误消息来源和元素

B.2 错误信息接口

B.3 问题和常见错误

B.3.1 如何确定问题的原因

B.3.2 使用 MySQL 程序时的常见错误

B.3.3 管理相关问题

B.3.4 查询相关问题

B.3.5 优化器相关问题

B.3.6 表定义相关问题

B.3.7 MySQL 中已知问题

本附录描述了 MySQL 提供的错误信息类型以及如何获取有关它们的信息。最后一节是用于故障排除。它描述了可能发生的常见问题和错误以及可能的解决方法。

## 其他资源

其他与错误相关的文档包括：

+   配置服务器写入错误日志的位置和方式的信息：第 7.4.2 节，“错误日志”

+   关于用于错误消息的字符集的信息：第 12.6 节，“错误消息字符集”

+   关于错误消息使用的语言的信息：第 12.12 节，“设置错误消息语言”

+   与`InnoDB`相关的错误信息：第 17.21.5 节，“InnoDB 错误处理”

+   与 NDB 集群特定错误相关的信息：NDB 集群 API 错误；另请参阅 NDB API 错误和错误处理，以及 MGM API 错误

+   MySQL 服务器和客户端程序生成的错误消息描述：MySQL 8.0 错误消息参考