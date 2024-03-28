> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-filters-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-filters-table.html)

#### 29.12.11.10 复制 _applier_filters 表

此表显示在此副本上配置的复制通道特定过滤器。每行提供有关复制通道配置的过滤器类型的信息。`replication_applier_filters`表具有以下列：

+   `CHANNEL_NAME`

    已配置复制过滤器的复制通道的名称。

+   `FILTER_NAME`

    已为此复制通道配置的复制过滤器类型。

+   `FILTER_RULE`

    使用`--replicate-*`命令选项或`CHANGE REPLICATION FILTER`配置的复制过滤器类型的规则。

+   `CONFIGURED_BY`

    用于配置复制过滤器的方法，可以是以下之一：

    +   `CHANGE_REPLICATION_FILTER`通过使用`CHANGE REPLICATION FILTER`语句配置的全局复制过滤器。

    +   `STARTUP_OPTIONS`通过使用`--replicate-*`选项配置的全局复制过滤器。

    +   通过使用`CHANGE REPLICATION FILTER FOR CHANNEL`语句配置的特定通道复制过滤器。

    +   `STARTUP_OPTIONS_FOR_CHANNEL`通过使用`--replicate-*`选项配置的特定通道复制过滤器。

+   `ACTIVE_SINCE`

    配置复制过滤器的时间戳。

+   `COUNTER`

    自配置以来复制过滤器已使用的次数。
