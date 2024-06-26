# 19.5.2 MySQL 版本之间的复制兼容性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-compatibility.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-compatibility.html)

MySQL 支持从一个发布系列复制到下一个更高的发布系列。例如，您可以从运行 MySQL 5.6 的源复制到运行 MySQL 5.7 的副本，从运行 MySQL 5.7 的源复制到运行 MySQL 8.0 的副本，依此类推。但是，如果源使用在副本上使用的 MySQL 版本中不再支持的语句或行为，可能会在从旧源复制到新副本时遇到困难。例如，MySQL 8.0 不再支持超过 64 个字符的外键名称。

在涉及多个源的复制设置中，不支持使用两个以上的 MySQL 服务器版本，无论源或副本 MySQL 服务器的数量如何。这个限制不仅适用于发布系列，还适用于同一发布系列中的版本号。例如，如果您正在使用链式或循环复制设置，您不能同时使用 MySQL 8.0.22、MySQL 8.0.24 和 MySQL 8.0.28，尽管您可以同时使用这些版本中的任意两个。

重要提示

强烈建议在给定的 MySQL 发布系列中使用最新版本，因为复制（和其他）功能不断得到改进。还建议在 MySQL 发布系列的早期版本的源和副本可用时升级到 GA（生产）版本。

从 MySQL 8.0.14 开始，二进制日志中为每个事务记录了原始提交事务的服务器版本（`original_server_version`），以及在复制拓扑中当前服务器的直接源服务器的服务器版本（`immediate_server_version`）。

从更新的源到旧的副本的复制可能是可能的，但通常不受支持。这是由于许多因素：

+   **二进制日志格式更改。** 二进制日志格式可能会在主要版本发布之间发生变化。虽然我们尽力保持向后兼容性，但这并非总是可能的。源可能还启用了旧副本不理解的可选功能，例如二进制日志事务压缩，导致压缩后的事务有效负载无法在 MySQL 8.0.20 之前的版本中被副本读取。

    这也对升级复制服务器有重大影响；有关更多信息，请参阅第 19.5.3 节，“升级复制拓扑”。

+   有关基于行的复制的更多信息，请参见第 19.2.1 节，“复制格式”。

+   **SQL 不兼容性。** 如果要复制的语句使用源上可用但在副本上不可用的 SQL 功能，并且使用基于语句的复制从较新的源复制到较旧的副本是不允许的。

    然而，如果源和副本都支持基于行的复制，并且没有需要复制的数据定义语句依赖于源上发现但在副本上找不到的 SQL 功能，则可以使用基于行的复制来复制数据修改语句的效果，即使在源上运行的 DDL 在副本上不受支持。

在 MySQL 8.0.26 中，对复制仪器名称进行了不兼容的更改，包括线程阶段的名称，其中包含术语“master”，被更改为“source”，“slave”，被更改为“replica”，以及“mts”（代表“多线程从属”），被更改为“mta”（代表“多线程应用程序”）。使用这些仪器名称的监控工具可能会受到影响。如果不兼容的更改对您产生影响，请将`terminology_use_previous`系统变量设置为`BEFORE_8_0_26`，以使 MySQL Server 使用前面列表中指定对象的旧版本名称。这样可以使依赖于旧名称的监控工具继续工作，直到它们可以更新为使用新名称。

有关潜在复制问题的更多信息，请参见第 19.5.1 节，“复制功能和问题”。
