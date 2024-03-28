> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-resource-group.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-resource-group.html)

#### 15.7.2.1 ALTER RESOURCE GROUP Statement

```sql
ALTER RESOURCE GROUP *group_name*
    [VCPU [=] *vcpu_spec* [, *vcpu_spec*] ...]
    [THREAD_PRIORITY [=] *N*]
    [ENABLE|DISABLE [FORCE]]

*vcpu_spec*: {*N* | *M* - *N*}
```

`ALTER RESOURCE GROUP` 用于资源组管理（参见 Section 7.1.16, “Resource Groups”）。此语句更改现有资源组的可修改属性。它需要`RESOURCE_GROUP_ADMIN` 权限。

*`group_name`* 标识要更改的资源组。如果该组不存在，则会出现错误。

可以使用`ALTER RESOURCE GROUP`修改 CPU 亲和性、优先级以及组是否启用的属性。这些属性的指定方式与`CREATE RESOURCE GROUP`中描述的方式相同（参见 Section 15.7.2.2, “CREATE RESOURCE GROUP Statement”）。只有指定的属性会被更改，未指定的属性保留其当前值。

`FORCE` 修饰符与 `DISABLE` 一起使用。如果资源组有任何线程分配给它，则确定语句的行为：

+   如果未给出 `FORCE`，则组中的现有线程将继续运行直到终止，但新线程不能分配给该组。

+   如果给出 `FORCE`，则组中的现有线程将移动到各自的默认组（系统线程到 `SYS_default`，用户线程到 `USR_default`）。

名称和类型属性在组创建时设置，之后不能使用`ALTER RESOURCE GROUP`进行修改。

示例：

+   更改组 CPU 亲和性：

    ```sql
    ALTER RESOURCE GROUP rg1 VCPU = 0-63;
    ```

+   更改组线程优先级：

    ```sql
    ALTER RESOURCE GROUP rg2 THREAD_PRIORITY = 5;
    ```

+   禁用一个组，将任何分配给它的线程移动到默认组：

    ```sql
    ALTER RESOURCE GROUP rg3 DISABLE FORCE;
    ```

资源组管理是在发生的服务器上本地的。`ALTER RESOURCE GROUP` 语句不会写入二进制日志，也不会被复制。
