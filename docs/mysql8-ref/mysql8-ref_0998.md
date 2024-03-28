> 原文：[`dev.mysql.com/doc/refman/8.0/en/start-slave.html`](https://dev.mysql.com/doc/refman/8.0/en/start-slave.html)

#### 15.4.2.7 启动从库语句

```sql
START {SLAVE | REPLICA} [*thread_types*] [*until_option*] [*connection_options*] [*channel_option*]

*thread_types*:
    [*thread_type* [, *thread_type*] ... ]

*thread_type*:
    IO_THREAD | SQL_THREAD

*until_option*:
    UNTIL {   {SQL_BEFORE_GTIDS | SQL_AFTER_GTIDS} = *gtid_set*
          |   MASTER_LOG_FILE = '*log_name*', MASTER_LOG_POS = *log_pos*
          |   SOURCE_LOG_FILE = '*log_name*', SOURCE_LOG_POS = *log_pos*
          |   RELAY_LOG_FILE = '*log_name*', RELAY_LOG_POS = *log_pos*
          |   SQL_AFTER_MTS_GAPS  }

*connection_options*:
    [USER='*user_name*'] [PASSWORD='*user_pass*'] [DEFAULT_AUTH='*plugin_name*'] [PLUGIN_DIR='*plugin_dir*']

*channel_option*:
    FOR CHANNEL *channel*

*gtid_set*:
    *uuid_set* [, *uuid_set*] ...
    | ''

*uuid_set*:
    *uuid*:*interval*[:*interval*]...

*uuid*:
    *hhhhhhhh*-*hhhh*-*hhhh*-*hhhh*-*hhhhhhhhhhhh*

*h*:
    [0-9,A-F]

*interval*:
    *n*[-*n*]

    (*n* >= 1)
```

启动复制线程。从 MySQL 8.0.22 开始，`START SLAVE` 已被弃用，应改用别名 `START REPLICA`。该语句的工作方式与以前相同，只是语句及其输出所使用的术语已更改。使用时，两个版本的语句都会更新相同的状态变量。请参阅 `START REPLICA` 的文档以了解该语句的描述。
