> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-histogram-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-histogram-summary-tables.html)

#### 29.12.20.4 语句直方图摘要表

性能模式维护语句事件摘要表，其中包含有关语句延迟的最小、最大和平均信息（参见第 29.12.20.3 节，“语句摘要表”）。这些表允许对系统性能进行高级评估。为了允许在更细粒度级别进行评估，性能模式还收集语句延迟的直方图数据。这些直方图提供了对延迟分布的额外见解。

第 29.12.6 节，“性能模式语句事件表”描述了语句摘要基于的事件。查看该讨论以获取有关语句事件内容、当前和历史语句事件表以及如何控制语句事件收集（默认情况下部分禁用）的信息。

示例语句直方图信息：

```sql
mysql> SELECT *
       FROM performance_schema.events_statements_histogram_by_digest
       WHERE SCHEMA_NAME = 'mydb' AND DIGEST = 'bb3f69453119b2d7b3ae40673a9d4c7c'
       AND COUNT_BUCKET > 0 ORDER BY BUCKET_NUMBER\G
*************************** 1\. row ***************************
           SCHEMA_NAME: mydb
                DIGEST: bb3f69453119b2d7b3ae40673a9d4c7c
         BUCKET_NUMBER: 42
      BUCKET_TIMER_LOW: 66069344
     BUCKET_TIMER_HIGH: 69183097
          COUNT_BUCKET: 1
COUNT_BUCKET_AND_LOWER: 1
       BUCKET_QUANTILE: 0.058824
*************************** 2\. row ***************************
           SCHEMA_NAME: mydb
                DIGEST: bb3f69453119b2d7b3ae40673a9d4c7c
         BUCKET_NUMBER: 43
      BUCKET_TIMER_LOW: 69183097
     BUCKET_TIMER_HIGH: 72443596
          COUNT_BUCKET: 1
COUNT_BUCKET_AND_LOWER: 2
       BUCKET_QUANTILE: 0.117647
*************************** 3\. row ***************************
           SCHEMA_NAME: mydb
                DIGEST: bb3f69453119b2d7b3ae40673a9d4c7c
         BUCKET_NUMBER: 44
      BUCKET_TIMER_LOW: 72443596
     BUCKET_TIMER_HIGH: 75857757
          COUNT_BUCKET: 2
COUNT_BUCKET_AND_LOWER: 4
       BUCKET_QUANTILE: 0.235294
*************************** 4\. row ***************************
           SCHEMA_NAME: mydb
                DIGEST: bb3f69453119b2d7b3ae40673a9d4c7c
         BUCKET_NUMBER: 45
      BUCKET_TIMER_LOW: 75857757
     BUCKET_TIMER_HIGH: 79432823
          COUNT_BUCKET: 6
COUNT_BUCKET_AND_LOWER: 10
       BUCKET_QUANTILE: 0.625000
...
```

例如，在第 3 行，这些值表明 23.52%的查询在 75.86 微秒以下运行：

```sql
BUCKET_TIMER_HIGH: 75857757
  BUCKET_QUANTILE: 0.235294
```

在第 4 行，这些值表明 62.50%的查询在 79.44 微秒以下运行：

```sql
BUCKET_TIMER_HIGH: 79432823
  BUCKET_QUANTILE: 0.625000
```

每个语句直方图摘要表都有一个或多个分组列，指示表如何聚合事件：

+   `events_statements_histogram_by_digest`有`SCHEMA_NAME`、`DIGEST`和`BUCKET_NUMBER`列：

    +   `SCHEMA_NAME`和`DIGEST`列在`events_statements_summary_by_digest`表中标识语句摘要行。

    +   具有相同`SCHEMA_NAME`和`DIGEST`值的`events_statements_histogram_by_digest`行构成该模式/摘要组合的直方图。

    +   在给定的直方图中，`BUCKET_NUMBER`列指示桶号。

+   `events_statements_histogram_global`有一个`BUCKET_NUMBER`列。该表在全局范围内汇总模式名称和摘要值的延迟，使用单个直方图。`BUCKET_NUMBER`列指示此全局直方图中的桶号。

直方图由*`N`*个桶组成，每一行代表一个桶，桶号由`BUCKET_NUMBER`列指示。桶号从 0 开始。

每个语句直方图摘要表都有这些包含聚合值的摘要列：

+   `BUCKET_TIMER_LOW`, `BUCKET_TIMER_HIGH`

    一个桶计算具有在`BUCKET_TIMER_LOW`和`BUCKET_TIMER_HIGH`之间测量的皮秒延迟的语句：

    +   第一个桶（`BUCKET_NUMBER` = 0）的`BUCKET_TIMER_LOW`值为 0。

    +   桶（`BUCKET_NUMBER` = *`k`*）的`BUCKET_TIMER_LOW`值与前一个桶（`BUCKET_NUMBER` = *`k`*−1）的`BUCKET_TIMER_HIGH`值相同

    +   最后一个桶是一个捕获所有超过直方图中前一个桶的延迟的语句的桶。

+   `COUNT_BUCKET`

    在从`BUCKET_TIMER_LOW`到`BUCKET_TIMER_HIGH`之间具有延迟的语句数量。

+   `COUNT_BUCKET_AND_LOWER`

    在从 0 到`BUCKET_TIMER_HIGH`之间具有延迟的语句数量。

+   `BUCKET_QUANTILE`

    陷入此桶或更低桶的语句比例。根据定义，此比例对应于`COUNT_BUCKET_AND_LOWER / SUM(COUNT_BUCKET)`，并显示为一个便利列。

语句直方图摘要表具有以下索引：

+   `events_statements_histogram_by_digest`:

    +   在(`SCHEMA_NAME`, `DIGEST`, `BUCKET_NUMBER`)上的唯一索引

+   `events_statements_histogram_global`:

    +   在(`BUCKET_NUMBER`)上的主键

允许对语句直方图摘要表进行`TRUNCATE TABLE`。截断将`COUNT_BUCKET`和`COUNT_BUCKET_AND_LOWER`列设置为 0。

此外，截断`events_statements_summary_by_digest` 隐式截断了`events_statements_histogram_by_digest` ，截断`events_statements_summary_global_by_event_name` 隐式截断了`events_statements_histogram_global`。
