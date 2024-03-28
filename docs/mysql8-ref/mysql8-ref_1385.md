> 原文：[`dev.mysql.com/doc/refman/8.0/en/channels-naming-conventions.html`](https://dev.mysql.com/doc/refman/8.0/en/channels-naming-conventions.html)

#### 19.2.2.4 复制通道命名约定

本节描述了复制通道对命名约定的影响。

每个复制通道都有一个唯一的名称，其最大长度为 64 个字符，且不区分大小写。由于通道名称用于复制的应用程序元数据存储库表中，因此这些名称的字符集始终为 UTF-8。虽然通常可以自由选择通道的任何名称，但以下名称已保留：

+   `group_replication_applier`

+   `group_replication_recovery`

你为复制通道选择的名称也会影响多源复制的中继日志文件和索引文件的命名。每个通道的中继日志文件和索引文件的命名格式为`*`relay_log_basename`*-*`channel`*.xxxxxx`，其中*`relay_log_basename`*是使用`relay_log`系统变量指定的基本名称，*`channel`*是记录到该文件的通道的名称。如果未指定`relay_log`系统变量，则将使用默认文件名，该文件名还包括通道的名称。
