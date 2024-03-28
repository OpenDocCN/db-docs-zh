> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-resource-group.html`](https://dev.mysql.com/doc/refman/8.0/en/set-resource-group.html)

#### 15.7.2.4 SET RESOURCE GROUP Statement

```sql
SET RESOURCE GROUP *group_name*
    [FOR *thread_id* [, *thread_id*] ...]
```

`SET RESOURCE GROUP` 用于资源组管理（参见 第 7.1.16 节，“资源组”）。此语句将线程分配给资源组。它需要 `RESOURCE_GROUP_ADMIN` 或 `RESOURCE_GROUP_USER` 权限。

*`group_name`* 标识要分配的资源组。任何 *`thread_id`* 值表示要分配给该组的线程。线程 ID 可以从性能模式 `threads` 表中确定。如果资源组或任何命名线程 ID 不存在，则会出现错误。

没有 `FOR` 子句时，该语句将当前会话的当前线程分配给资源组。

使用命名线程 ID 的 `FOR` 子句时，该语句将这些线程分配给资源组。

尝试将系统线程分配给用户资源组或用户线程分配给系统资源组时，会发出警告。

示例：

+   将当前会话线程分配给一个组：

    ```sql
    SET RESOURCE GROUP rg1;
    ```

+   将命名线程分配给一个组：

    ```sql
    SET RESOURCE GROUP rg2 FOR 14, 78, 4;
    ```

资源组管理是局限于发生在其上的服务器的。`SET RESOURCE GROUP` 语句不会写入二进制日志，也不会被复制。

一个替代 `SET RESOURCE GROUP` 的方法是 `RESOURCE_GROUP` 优化器提示，它将单个语句分配给资源组。参见 第 10.9.3 节，“优化器提示”。
