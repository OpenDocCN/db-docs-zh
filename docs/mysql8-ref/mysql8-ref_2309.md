> 原文：[`dev.mysql.com/doc/refman/8.0/en/commands-out-of-sync.html`](https://dev.mysql.com/doc/refman/8.0/en/commands-out-of-sync.html)

#### B.3.2.12 命令不同步

如果在客户端代码中出现`Commands out of sync; you can't run this command now`，说明你在错误的顺序中调用客户端函数。

这可能发生在你使用`mysql_use_result()`并在调用`mysql_free_result()`之前尝试执行新查询时。也可能发生在你尝试执行两个返回数据的查询而没有在中间调用`mysql_use_result()`或`mysql_store_result()`。
