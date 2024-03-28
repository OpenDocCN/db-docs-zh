> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-config-params.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-config-params.html)

#### 25.6.16.10 ndbinfo config_params 表

`config_params`表是一个静态表，提供了 NDB Cluster 配置参数的名称、内部 ID 号以及其他信息。该表还可与`config_values`表一起使用，以获取有关节点配置参数的实时信息。

`config_params`表包含以下列：

+   `param_number`

    参数的内部 ID 号

+   `param_name`

    参数的名称

+   `param_description`

    参数的简要描述

+   `param_type`

    参数的数据类型

+   `param_default`

    参数的默认值（如果有的话）

+   `param_min`

    参数的最大值（如果有的话）

+   `param_max`

    参数的最小值（如果有的话）

+   `param_mandatory`

    如果参数是必需的，则为 1，否则为 0

+   `param_status`

    目前未使用

##### 注意事项

该表是只读的。

尽管这是一个静态表，但其内容可能因 NDB Cluster 安装而异，因为受支持的参数可能因软件版本、集群硬件配置和其他因素的差异而变化。
