> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-metrics.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-metrics.html)

#### 30.4.3.21 指标视图

此视图总结了 MySQL 服务器的指标，显示变量名称、数值、类型以及是否启用。默认情况下，按变量类型和名称排序。

`metrics`视图包括以下信息：

+   来自性能模式`global_status`表的全局状态变量

+   来自`INFORMATION_SCHEMA` `INNODB_METRICS`表的`InnoDB`指标

+   基于性能模式内存仪表化的当前和总内存分配

+   当前时间（人类可读和 Unix 时间戳格式）

在`global_status`和`INNODB_METRICS`表之间存在一些信息重复，`metrics`视图消除了这种重复。

`metrics`视图具有以下列：

+   `变量名称`

    指标名称。指标类型确定了名称的来源：

    +   对于全局状态变量：`global_status`表的`VARIABLE_NAME`列

    +   对于`InnoDB`指标：`INNODB_METRICS`表的`NAME`列

    +   对于其他指标：视图提供的描述性字符串

+   `Variable_value`

    指标数值。指标类型确定了数值的来源：

    +   对于全局状态变量：`global_status`表的`VARIABLE_VALUE`列

    +   对于`InnoDB`指标：`INNODB_METRICS`表的`COUNT`列

    +   对于内存指标：性能模式`memory_summary_global_by_event_name`表中的相关列

    +   对于当前时间：`NOW(3)`或`UNIX_TIMESTAMP(NOW(3))`的值

+   `类型`

    指标类型：

    +   对于全局状态变量：`全局状态`

    +   对于`InnoDB`指标：`InnoDB 指标 - %`，其中`%`由`INNODB_METRICS`表的`SUBSYSTEM`列的值替换而来

    +   对于内存指标：`性能模式`

    +   对于当前时间：`系统时间`

+   `已启用`

    指标是否已启用：

    +   对于全局状态变量：`是`

    +   对于`InnoDB`指标：如果`INNODB_METRICS`表的`STATUS`列为`enabled`，则为`是`，否则为`否`

    +   对于内存指标：`否`，`是`或`部分`（目前，`部分`仅出现在内存指标中，表示并非所有`memory/%`工具都已启用；性能模式内存工具始终已启用）

    +   对于当前时间：`是`
