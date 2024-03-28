> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-maximum-consensus.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions-for-maximum-consensus.html)

#### 14.18.1.3 检查和配置组的最大共识实例的函数

以下函数使您能够检查和配置组可以并行执行的最大共识实例数。

+   `group_replication_get_write_concurrency()`

    检查组可以并行执行的最大共识实例数。

    语法：

    ```sql
    INT group_replication_get_write_concurrency()
    ```

    此函数没有参数。

    返回值：

    当前为组设置的最大共识实例数。

    示例：

    ```sql
    SELECT group_replication_get_write_concurrency()
    ```

    欲了解更多信息，请参阅第 20.5.1.3 节，“使用组复制组写共识”。

+   `group_replication_set_write_concurrency()`

    配置组可以并行执行的最大共识实例数。需要`GROUP_REPLICATION_ADMIN`权限才能使用此函数。

    语法：

    ```sql
    STRING group_replication_set_write_concurrency(*instances*)
    ```

    参数：

    +   *`members`*: 设置组可以并行执行的最大共识实例数。默认值为 10，有效值为 10 到 200 之间的整数。

    返回值：

    任何导致的错误都会以字符串形式返回。

    示例：

    ```sql
    SELECT group_replication_set_write_concurrency(*instances*);
    ```

    欲了解更多信息，请参阅第 20.5.1.3 节，“使用组复制组写共识”。
