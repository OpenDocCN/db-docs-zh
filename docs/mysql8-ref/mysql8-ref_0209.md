> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-hexdump.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-hexdump.html)

#### 6.6.9.1 mysqlbinlog 十六进制转储格式

`--hexdump` 选项会导致**mysqlbinlog**生成二进制日志内容的十六进制转储：

```sql
mysqlbinlog --hexdump source-bin.000001
```

十六进制输出由以`#`开头的注释行组成，因此对于前述命令，输出可能如下所示：

```sql
/*!40019 SET @@SESSION.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
# at 4
#051024 17:24:13 server id 1  end_log_pos 98
# Position  Timestamp   Type   Master ID        Size      Master Pos    Flags
# 00000004 9d fc 5c 43   0f   01 00 00 00   5e 00 00 00   62 00 00 00   00 00
# 00000017 04 00 35 2e 30 2e 31 35  2d 64 65 62 75 67 2d 6c |..5.0.15.debug.l|
# 00000027 6f 67 00 00 00 00 00 00  00 00 00 00 00 00 00 00 |og..............|
# 00000037 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00 |................|
# 00000047 00 00 00 00 9d fc 5c 43  13 38 0d 00 08 00 12 00 |.......C.8......|
# 00000057 04 04 04 04 12 00 00 4b  00 04 1a                |.......K...|
#       Start: binlog v 4, server v 5.0.15-debug-log created 051024 17:24:13
#       at startup
ROLLBACK;
```

目前的十六进制转储输出包含以下列表中的元素。此格式可能会更改。有关二进制日志格式的更多信息，请参阅 MySQL Internals: 二进制日志。

+   `Position`: 日志文件中的字节位置。

+   `Timestamp`: 事件时间戳。在示例中，`'9d fc 5c 43'` 是十六进制表示中的`'051024 17:24:13'`。

+   `Type`: 事件类型代码。

+   `Master ID`: 创建事件的复制源服务器的服务器 ID。

+   `Size`: 事件的字节大小。

+   `Master Pos`: 原始源日志文件中下一个事件的位置。

+   `Flags`: 事件标志值。
