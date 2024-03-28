> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-stack.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-thread-stack.html)

#### 30.4.5.16 ps_thread_stack() 函数

返回给定线程 ID 的性能模式中所有语句、阶段和事件的 JSON 格式堆栈。

##### 参数

+   `in_thread_id BIGINT`: 要跟踪的线程的 ID。该值应与某些性能模式`threads`表行的`THREAD_ID`列匹配。

+   `in_verbose BOOLEAN`: 是否在事件中包含`file:lineno`信息。

##### 返回值

一个`LONGTEXT CHARACTER SET latin1`值。

##### 示例

```sql
mysql> SELECT sys.ps_thread_stack(37, FALSE) AS thread_stack\G
*************************** 1\. row ***************************
thread_stack: {"rankdir": "LR","nodesep": "0.10",
"stack_created": "2014-02-19 13:39:03", "mysql_version": "8.0.2-dmr-debug-log",
"mysql_user": "root@localhost","events": [{"nesting_event_id": "0",
"event_id": "10", "timer_wait": 256.35, "event_info": "sql/select",
"wait_info": "select @@version_comment limit 1\nerrors: 0\nwarnings: 0\nlock time:
...
```
