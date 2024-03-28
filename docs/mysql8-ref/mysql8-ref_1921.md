# 28.3.26 INFORMATION_SCHEMA RESOURCE_GROUPS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-resource-groups-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-resource-groups-table.html)

[`RESOURCE_GROUPS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-resource-groups-table.html) 表提供了关于资源组的信息。有关资源组功能的一般讨论，请参见 Section 7.1.16, “Resource Groups”。

您只能查看您拥有某些权限的列的信息。

[`RESOURCE_GROUPS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-resource-groups-table.html) 表具有以下列：

+   `RESOURCE_GROUP_NAME`

    资源组的名称。

+   `RESOURCE_GROUP_TYPE`

    资源组类型，可以是 `SYSTEM` 或 `USER`。

+   `RESOURCE_GROUP_ENABLED`

    资源组是否启用（1）或禁用（0）；

+   `VCPU_IDS`

    CPU 亲和性；即资源组可以使用的虚拟 CPU 集合。该值是逗号分隔的 CPU 编号或范围的列表。

+   `THREAD_PRIORITY`

    分配给资源组的线程的优先级。优先级范围从 -20（最高优先级）到 19（最低优先级）。系统资源组的优先级范围从 -20 到 0。用户资源组的优先级范围从 0 到 19。
