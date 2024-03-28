> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-new-primary.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-new-primary.html)

#### 14.18.1.1 配置组复制主服务器的函数

以下函数使您能够设置单一主复制组的成员接管为主服务器。当前主服务器变为只读辅助服务器，指定的组成员成为读写主服务器。该函数可用于在单一主模式下运行的复制组的任何成员。此函数替换了通常的主服务器选举过程；有关更多信息，请参见 Section 20.5.1.1, “Changing the Primary”。

如果标准源到副本复制通道在现有主成员上运行，除了 Group Replication 通道之外，您必须在更改主成员之前停止该复制通道。您可以使用性能模式表`replication_group_members`中的`MEMBER_ROLE`列，或`group_replication_primary_member`状态变量来识别当前主成员。

组正在等待的任何未提交事务必须在操作完成之前提交、回滚或终止。在 MySQL 8.0.29 之前，该函数会等待现有主服务器上的所有活动事务结束，包括在使用该函数后启动的传入事务。从 MySQL 8.0.29 开始，您可以为在使用该函数时运行的事务指定超时。要使超时起作用，组的所有成员必须运行在 MySQL 8.0.29 或更高版本。

当超时到期时，对于尚未达到提交阶段的任何事务，客户端会断开会话，以阻止事务继续进行。已达到提交阶段的事务允许完成。设置超时还会阻止从那时起在主服务器上启动的新事务。明确定义的事务（使用`START TRANSACTION`或`BEGIN`语句）也会受到超时、断开连接和传入事务阻止的影响，即使它们不修改任何数据。为了允许在函数操作时检查主服务器，允许执行不修改数据的单个语句，如一致性规则下允许的查询中列出的语句。

+   `group_replication_set_as_primary()`

    指定组中的特定成员作为新的主服务器，覆盖任何选举过程。

    语法：

    ```sql
    STRING group_replication_set_as_primary(*member_uuid*[, *timeout*])
    ```

    参数：

    +   *`member_uuid`*: 一个包含您希望成为新主服务器的组成员的 UUID 的字符串。

    +   *`timeout`*: 一个整数，指定在使用该函数时在现有主服务器上运行的事务的超时时间（以秒为单位）。您可以设置超时时间从 0 秒（立即）到 3600 秒（60 分钟）。当您设置超时时，从那时起，主服务器上将无法启动新的事务。超时没有默认设置，因此如果您不设置它，等待时间没有上限，并且在此期间可以启动新的事务。此选项从 MySQL 8.0.29 版本开始提供。

    返回值：

    一个包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT group_replication_set_as_primary(‘00371d66-3c45-11ea-804b-080027337932’, 300);
    ```

    欲了解更多信息，请参阅 Section 20.5.1.1, “Changing the Primary”。
