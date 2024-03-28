# 27.4.5 事件调度程序状态

> 原文：[`dev.mysql.com/doc/refman/8.0/en/events-status-info.html`](https://dev.mysql.com/doc/refman/8.0/en/events-status-info.html)

事件调度程序会将执行过程中出现错误或警告的事件信息写入 MySQL 服务器的错误日志。参见第 27.4.6 节，“事件调度程序和 MySQL 权限”中的示例。

为了获取有关事件调度程序状态的信息，以进行调试和故障排除，运行**mysqladmin debug**（参见第 6.5.2 节，“mysqladmin — 一个 MySQL 服务器管理程序”）；运行此命令后，服务器的错误日志将包含与事件调度程序相关的输出，类似于这里显示的内容：

```sql
Events status:
LLA = Last Locked At  LUA = Last Unlocked At
WOC = Waiting On Condition  DL = Data Locked

Event scheduler status:
State      : INITIALIZED
Thread id  : 0
LLA        : n/a:0
LUA        : n/a:0
WOC        : NO
Workers    : 0
Executed   : 0
Data locked: NO

Event queue status:
Element count   : 0
Data locked     : NO
Attempting lock : NO
LLA             : init_queue:95
LUA             : init_queue:103
WOC             : NO
Next activation : never
```

在事件调度程序执行的语句中，诊断消息（不仅限于错误，还包括警告）会被写入错误日志，并且在 Windows 上也会写入应用程序事件日志。对于频繁执行的事件，可能会导致许多日志消息。例如，对于`SELECT ... INTO *`var_list`*`语句，如果查询没有返回任何行，会出现带有错误代码 1329 的警告（`没有数据`），并且变量值保持不变。如果查询返回多行，会出现错误 1172（`结果包含多于一行`）。对于任一条件，您可以通过声明条件处理程序来避免警告被记录；参见第 15.6.7.2 节，“DECLARE ... HANDLER 语句”。对于可能检索多行的语句，另一种策略是使用`LIMIT 1`将结果集限制为单行。
