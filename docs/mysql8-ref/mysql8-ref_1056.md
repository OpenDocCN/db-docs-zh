> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-resource-group.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-resource-group.html)

#### 15.7.2.3 删除资源组语句

```sql
DROP RESOURCE GROUP *group_name* [FORCE]
```

`DROP RESOURCE GROUP` 用于资源组管理（参见第 7.1.16 节，“资源组”）。此语句删除一个资源组。它需要`RESOURCE_GROUP_ADMIN` 权限。

*`group_name`* 标识要删除的资源组。如果该组不存在，则会出现错误。

`FORCE` 修饰符确定资源组有任何线程分配时语句的行为：

+   如果未给出 `FORCE` 并且任何线程被分配到该组，则会出现错误。

+   如果给出 `FORCE`，则组中的现有线程将移动到各自的默认组（系统线程到 `SYS_default`，用户线程到 `USR_default`）。

示例：

+   删除一个组，如果该组包含任何线程则失败：

    ```sql
    DROP RESOURCE GROUP rg1;
    ```

+   删除一个组并将现有线程移动到默认组：

    ```sql
    DROP RESOURCE GROUP rg2 FORCE;
    ```

资源组管理是发生在其上的服务器本地的。`DROP RESOURCE GROUP` 语句不会写入二进制日志，也不会被复制。
