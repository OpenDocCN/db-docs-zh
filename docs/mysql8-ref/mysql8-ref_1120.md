> 原文：[`dev.mysql.com/doc/refman/8.0/en/binlog.html`](https://dev.mysql.com/doc/refman/8.0/en/binlog.html)

#### 15.7.8.1 BINLOG Statement

```sql
BINLOG '*str*'
```

`BINLOG` 是一个内部使用的语句。它由**mysqlbinlog**程序生成，作为二进制日志文件中某些事件的可打印表示。（参见 Section 6.6.9, “mysqlbinlog — Utility for Processing Binary Log Files”.）`'*`str`*'`值是一个 base 64 编码的字符串，服务器解码以确定相应事件指示的数据更改。

在应用**mysqlbinlog**输出时执行`BINLOG`语句，用户帐户需要`BINLOG_ADMIN`权限（或已弃用的`SUPER`权限），或者`REPLICATION_APPLIER`权限加上适当的权限来执行每个日志事件。

这个语句只能执行格式描述事件和行事件。
