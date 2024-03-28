# 28.3.24 INFORMATION_SCHEMA PROFILING 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-profiling-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-profiling-table.html)

[`PROFILING`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-profiling-table.html) 表提供语句分析信息。其内容对应于 `SHOW PROFILE` 和 `SHOW PROFILES` 语句产生的信息（参见 Section 15.7.7.30, “SHOW PROFILE Statement”）。除非将 `profiling` 会话变量设置为 1，否则该表为空。

注意

该表已被弃用；预计在未来的 MySQL 版本中将被移除。请改用 Performance Schema；参见 Section 29.19.1, “Query Profiling Using Performance Schema”。

[`PROFILING`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-profiling-table.html) 表具有以下列：

+   `QUERY_ID`

    数字语句标识符。

+   `SEQ`

    表示具有相同 `QUERY_ID` 值的行的显示顺序的序列号。

+   `STATE`

    应用于行测量的受监视状态。

+   `DURATION`

    语句执行在给定状态中保持的时间，以秒为单位。

+   `CPU_USER`, `CPU_SYSTEM`

    用户和系统 CPU 使用时间，以秒为单位。

+   `CONTEXT_VOLUNTARY`, `CONTEXT_INVOLUNTARY`

    发生了多少次自愿和非自愿的上下文切换。

+   `BLOCK_OPS_IN`, `BLOCK_OPS_OUT`

    块输入和输出操作的数量。

+   `MESSAGES_SENT`, `MESSAGES_RECEIVED`

    发送和接收的通信消息数量。

+   `PAGE_FAULTS_MAJOR`, `PAGE_FAULTS_MINOR`

    主要和次要页面错误的数量。

+   `SWAPS`

    发生了多少次交换。

+   `SOURCE_FUNCTION`, `SOURCE_FILE`, 和 `SOURCE_LINE`

    表示在源代码中执行的受监视状态的位置。

#### 注意

+   [`PROFILING`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-profiling-table.html) 是一个非标准的 `INFORMATION_SCHEMA` 表。

也可以从 `SHOW PROFILE` 和 `SHOW PROFILES` 语句中获取分析信息。参见 Section 15.7.7.30, “SHOW PROFILE Statement”。例如，以下查询是等效的：

```sql
SHOW PROFILE FOR QUERY 2;

SELECT STATE, FORMAT(DURATION, 6) AS DURATION
FROM INFORMATION_SCHEMA.PROFILING
WHERE QUERY_ID = 2 ORDER BY SEQ;
```
