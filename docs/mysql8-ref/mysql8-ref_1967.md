# 28.4.21 INFORMATION_SCHEMA INNODB_METRICS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-metrics-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-metrics-table.html)

`INNODB_METRICS`表提供各种`InnoDB`性能信息，补充了`InnoDB`性能模式表的特定关注领域。通过简单的查询，您可以检查系统的整体健康状况。通过更详细的查询，您可以诊断性能瓶颈、资源短缺和应用程序问题等问题。

每个监视器代表`InnoDB`源代码中的一个点，用于收集计数器信息。每个计数器可以启动、停止和重置。您还可以针对它们的共同模块名称执行这些操作的一组计数器。

默认情况下，收集的数据相对较少。要启动、停止和重置计数器，请设置以下系统变量之一`innodb_monitor_enable`、`innodb_monitor_disable`、`innodb_monitor_reset`或`innodb_monitor_reset_all`，使用计数器的名称、模块的名称、使用“%”字符进行名称的通配符匹配，或者特殊关键字`all`。

有关使用信息，请参见第 17.15.6 节，“InnoDB INFORMATION_SCHEMA Metrics Table”。

`INNODB_METRICS`表具有以下列：

+   `NAME`

    计数器的唯一名称。

+   `SUBSYSTEM`

    应用于`InnoDB`的指标方面。

+   `COUNT`

    自计数器启用以来的值。

+   `MAX_COUNT`

    自计数器启用以来的最大值。

+   `MIN_COUNT`

    自计数器启用以来的最小值。

+   `AVG_COUNT`

    自计数器启用以来的平均值。

+   `COUNT_RESET`

    自上次重置以来的计数器值。（`_RESET`列就像秒表上的圈数计数器：您可以测量某个时间间隔内的活动，而累积数据仍然可在`COUNT`、`MAX_COUNT`等中找到。）

+   `MAX_COUNT_RESET`

    自上次重置以来的最大计数器值。

+   `MIN_COUNT_RESET`

    自上次重置以来的最小计数器值。

+   `AVG_COUNT_RESET`

    自上次重置以来的平均计数器值。

+   `TIME_ENABLED`

    上次启动的时间戳。

+   `TIME_DISABLED`

    上次停止的时间戳。

+   `TIME_ELAPSED`

    自计数器启动以来经过的秒数。

+   `TIME_RESET`

    上次重置的时间戳。

+   `STATUS`

    计数器是否仍在运行（`enabled`）或已��止（`disabled`）。

+   `TYPE`

    该项目是累积计数器，还是测量某些资源的当前值。

+   `COMMENT`

    计数器描述。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME='dml_inserts'\G
*************************** 1\. row ***************************
           NAME: dml_inserts
      SUBSYSTEM: dml
          COUNT: 3
      MAX_COUNT: 3
      MIN_COUNT: NULL
      AVG_COUNT: 0.046153846153846156
    COUNT_RESET: 3
MAX_COUNT_RESET: 3
MIN_COUNT_RESET: NULL
AVG_COUNT_RESET: NULL
   TIME_ENABLED: 2014-12-04 14:18:28
  TIME_DISABLED: NULL
   TIME_ELAPSED: 65
     TIME_RESET: NULL
         STATUS: enabled
           TYPE: status_counter
        COMMENT: Number of rows inserted
```

#### 笔记

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   事务计数器`COUNT`的值可能与性能模式`EVENTS_TRANSACTIONS_SUMMARY`表中报告的事务事件数量不同。 `InnoDB`仅计算其执行的事务，而性能模式收集由服务器发起的所有未中止事务的事件，包括空事务。
