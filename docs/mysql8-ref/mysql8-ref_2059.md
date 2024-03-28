> 译文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-global-filters-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-global-filters-table.html)

#### 29.12.11.9 复制 _applier_global_filters 表

此表显示在此副本上配置的全局复制过滤器。`replication_applier_global_filters` 表具有以下列：

+   `FILTER_NAME`

    已配置的复制过滤器类型。

+   `FILTER_RULE`

    使用 `--replicate-*` 命令选项或 `CHANGE REPLICATION FILTER` 配置的复制过滤器类型的规则。

+   `CONFIGURED_BY`

    用于配置复制过滤器的方法，可以是以下之一：

    +   `CHANGE_REPLICATION_FILTER` 由使用 `CHANGE REPLICATION FILTER` 语句的全局复制过滤器配置。

    +   `STARTUP_OPTIONS` 由使用 `--replicate-*` 选项的全局复制过滤器配置。

+   `ACTIVE_SINCE`

    复制过滤器配置的时间戳。
