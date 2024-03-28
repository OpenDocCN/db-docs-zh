> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-statement-avg-latency-histogram.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-statement-avg-latency-histogram.html)

#### 30.4.4.21 ps_statement_avg_latency_histogram() 过程

显示跨所有在性能模式中跟踪的标准化语句中的平均延迟值的文本直方图图表`events_statements_summary_by_digest`表。

此过程可用于显示在此 MySQL 实例中运行的语句的延迟分布的非常高级别的图像。

##### 参数

无。

##### 示例

直方图输出以语句单位为单位。例如，在直方图图例中，`* = 2 units` 表示每个 `*` 字符代表 2 个语句。

```sql
mysql> CALL sys.ps_statement_avg_latency_histogram()\G
*************************** 1\. row ***************************
Performance Schema Statement Digest Average Latency Histogram:

  . = 1 unit
  * = 2 units
  # = 3 units

(0 - 66ms)     88  | #############################
(66 - 133ms)   14  | ..............
(133 - 199ms)  4   | ....
(199 - 265ms)  5   | **
(265 - 332ms)  1   | .
(332 - 398ms)  0   |
(398 - 464ms)  1   | .
(464 - 531ms)  0   |
(531 - 597ms)  0   |
(597 - 663ms)  0   |
(663 - 730ms)  0   |
(730 - 796ms)  0   |
(796 - 863ms)  0   |
(863 - 929ms)  0   |
(929 - 995ms)  0   |
(995 - 1062ms) 0   |

  Total Statements: 114; Buckets: 16; Bucket Size: 66 ms;
```
