> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-trx-info.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-trx-info.html)

#### 30.4.5.17 The ps_thread_trx_info() Function

返回一个包含有关给定线程信息的 JSON 对象。该信息包括当前事务以及它已经执行的语句，从性能模式`events_transactions_current`和`events_statements_history`表中获取的。（这些表的消费者必须启用才能在 JSON 对象中获取完整数据。）

如果输出超过截断长度（默认为 65535），将返回一个 JSON 错误对象，例如：

```sql
{ "error": "Trx info truncated: Row 6 was cut by GROUP_CONCAT()" }
```

在函数执行期间引发的其他警告和异常会返回类似的错误对象。

##### 参数

+   `in_thread_id BIGINT UNSIGNED`：要返回事务信息的线程 ID。该值应与某些性能模式`threads`表行的`THREAD_ID`列匹配。

##### 配置选项

`ps_thread_trx_info()` Function") 操作可以使用以下配置选项或它们对应的用户定义变量进行修改（参见第 30.4.2.1 节，“sys_config 表”）：

+   `ps_thread_trx_info.max_length`，`@sys.ps_thread_trx_info.max_length`

    输出的最大长度。默认为 65535。

##### 返回值

一个`LONGTEXT`值。

##### 示例

```sql
mysql> SELECT sys.ps_thread_trx_info(48)\G
*************************** 1\. row ***************************
sys.ps_thread_trx_info(48): [
  {
    "time": "790.70 us",
    "state": "COMMITTED",
    "mode": "READ WRITE",
    "autocommitted": "NO",
    "gtid": "AUTOMATIC",
    "isolation": "REPEATABLE READ",
    "statements_executed": [
      {
        "sql_text": "INSERT INTO info VALUES (1, \'foo\')",
        "time": "471.02 us",
        "schema": "trx",
        "rows_examined": 0,
        "rows_affected": 1,
        "rows_sent": 0,
        "tmp_tables": 0,
        "tmp_disk_tables": 0,
        "sort_rows": 0,
        "sort_merge_passes": 0
      },
      {
        "sql_text": "COMMIT",
        "time": "254.42 us",
        "schema": "trx",
        "rows_examined": 0,
        "rows_affected": 0,
        "rows_sent": 0,
        "tmp_tables": 0,
        "tmp_disk_tables": 0,
        "sort_rows": 0,
        "sort_merge_passes": 0
      }
    ]
  },
  {
    "time": "426.20 us",
    "state": "COMMITTED",
    "mode": "READ WRITE",
    "autocommitted": "NO",
    "gtid": "AUTOMATIC",
    "isolation": "REPEATABLE READ",
    "statements_executed": [
      {
        "sql_text": "INSERT INTO info VALUES (2, \'bar\')",
        "time": "107.33 us",
        "schema": "trx",
        "rows_examined": 0,
        "rows_affected": 1,
        "rows_sent": 0,
        "tmp_tables": 0,
        "tmp_disk_tables": 0,
        "sort_rows": 0,
        "sort_merge_passes": 0
      },
      {
        "sql_text": "COMMIT",
        "time": "213.23 us",
        "schema": "trx",
        "rows_examined": 0,
        "rows_affected": 0,
        "rows_sent": 0,
        "tmp_tables": 0,
        "tmp_disk_tables": 0,
        "sort_rows": 0,
        "sort_merge_passes": 0
      }
    ]
  }
]
```
