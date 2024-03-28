> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-mode.html)

#### 14.18.1.2 配置组复制模式的函数

以下函数使您能够控制复制组运行的模式，即单一主节点模式或多主节点模式。

+   [`group_replication_switch_to_multi_primary_mode()`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-mode.html#function_group-replication-switch-to-multi-primary-mode)

    将运行在单一主节点模式下的组更改为多主节点模式。必须在运行在单一主节点模式下的复制组的成员上发出。

    语法：

    ```sql
    STRING group_replication_switch_to_multi_primary_mode()
    ```

    此函数没有参数。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT group_replication_switch_to_multi_primary_mode()
    ```

    所有属于该组的成员都变为主节点。

    有关更多信息，请参见第 20.5.1.2 节，“更改组模式”

+   [`group_replication_switch_to_single_primary_mode()`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-mode.html#function_group-replication-switch-to-single-primary-mode)

    将运行在多主节点模式下的组更改为单一主节点模式，无需停止组复制。必须在运行在多主节点模式下的复制组的成员上发出。当您更改为单一主节点模式时，所有组成员上也会禁用严格的一致性检查，如单一主节点模式所需的（`group_replication_enforce_update_everywhere_checks=OFF`）。

    语法：

    ```sql
    STRING group_replication_switch_to_single_primary_mode([*str*])
    ```

    参数：

    +   *`str`*: 包含应成为新单一主节点的组成员的 UUID 的字符串。组的其他成员将成为辅助节点。

    返回值：

    包含操作结果的字符串，例如操作是否成功。

    示例：

    ```sql
    SELECT group_replication_switch_to_single_primary_mode(*member_uuid*);
    ```

    有关更多信息，请参见第 20.5.1.2 节，“更改组模式”
