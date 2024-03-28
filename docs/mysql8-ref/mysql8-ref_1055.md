> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-resource-group.html`](https://dev.mysql.com/doc/refman/8.0/en/create-resource-group.html)

#### 15.7.2.2 创建资源组语句

```sql
CREATE RESOURCE GROUP *group_name*
    TYPE = {SYSTEM|USER}
    [VCPU [=] *vcpu_spec* [, *vcpu_spec*] ...]
    [THREAD_PRIORITY [=] *N*]
    [ENABLE|DISABLE]

*vcpu_spec*: {*N* | *M* - *N*}
```

`CREATE RESOURCE GROUP` 用于资源组管理（参见 Section 7.1.16, “Resource Groups”）。此语句创建一个新的资源组并分配其初始属性值。它需要 `RESOURCE_GROUP_ADMIN` 权限。

*`group_name`* 标识要创建的资源组。如果该组已经存在，则会出现错误。

`TYPE` 属性是必需的。对于系统资源组应为 `SYSTEM`，对于用户资源组应为 `USER`。组类型会影响允许的 `THREAD_PRIORITY` 值，如后面所述。

`VCPU` 属性表示 CPU 亲和性；也就是说，组可以使用的虚拟 CPU 集合：

+   如果没有给定 `VCPU`，资源组没有 CPU 亲和性，可以使用所有可用的 CPU。

+   如果给定了 `VCPU`，则属性值是逗号分隔的 CPU 数字或范围的列表：

    +   每个数字必须是从 0 到 CPU 数量 - 1 的范围内的整数。例如，在具有 64 个 CPU 的系统上，数字的范围可以从 0 到 63。

    +   范围以 *`M`* − *`N`* 的形式给出，其中 *`M`* 小于或等于 *`N`*，并且两个数字都在 CPU 范围内。

    +   如果 CPU 数字是超出允许范围的整数或不是整数，则会出现错误。

示例 `VCPU` 指定器（这些都是等效的）：

```sql
VCPU = 0,1,2,3,9,10
VCPU = 0-3,9-10
VCPU = 9,10,0-3
VCPU = 0,10,1,9,3,2
```

`THREAD_PRIORITY` 属性表示分配给组的线程的优先级：

+   如果没有给定 `THREAD_PRIORITY`，默认优先级为 0。

+   如果给定了 `THREAD_PRIORITY`，则属性值必须在 -20（最高优先级）到 19（最低优先级）的范围内。系统资源组的优先级必须在 -20 到 0 的范围内。用户资源组的优先级必须在 0 到 19 的范围内。使用不同的范围为系统和用户组确保用户线程永远不会比系统线程具有更高的优先级。

`ENABLE` 和 `DISABLE` 指定资源组最初是启用还是禁用。如果没有指定任何一个，那么该组默认是启用的。禁用的组不能分配线程。

示例：

+   创建一个启用的用户组，具有单个 CPU 和最低优先级：

    ```sql
    CREATE RESOURCE GROUP rg1
      TYPE = USER
      VCPU = 0
      THREAD_PRIORITY = 19;
    ```

+   创建一个禁用的系统组，没有 CPU 亲和性（可以使用所有 CPU）和最高优先级：

    ```sql
    CREATE RESOURCE GROUP rg2
      TYPE = SYSTEM
      THREAD_PRIORITY = -20
      DISABLE;
    ```

资源组管理是在发生的服务器上本地的。`CREATE RESOURCE GROUP` 语句不会写入二进制日志，也不会被复制。
