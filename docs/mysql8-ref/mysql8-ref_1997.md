# 29.4.4 按工具进行预过滤

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-instrument-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-instrument-filtering.html)

`setup_instruments` 表列出了可用的工具：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments;
+---------------------------------------------------+---------+-------+
| NAME                                              | ENABLED | TIMED |
+---------------------------------------------------+---------+-------+
...
| stage/sql/end                                     | NO      | NO    |
| stage/sql/executing                               | NO      | NO    |
| stage/sql/init                                    | NO      | NO    |
| stage/sql/insert                                  | NO      | NO    |
...
| statement/sql/load                                | YES     | YES   |
| statement/sql/grant                               | YES     | YES   |
| statement/sql/check                               | YES     | YES   |
| statement/sql/flush                               | YES     | YES   |
...
| wait/synch/mutex/sql/LOCK_global_read_lock        | YES     | YES   |
| wait/synch/mutex/sql/LOCK_global_system_variables | YES     | YES   |
| wait/synch/mutex/sql/LOCK_lock_db                 | YES     | YES   |
| wait/synch/mutex/sql/LOCK_manager                 | YES     | YES   |
...
| wait/synch/rwlock/sql/LOCK_grant                  | YES     | YES   |
| wait/synch/rwlock/sql/LOGGER::LOCK_logger         | YES     | YES   |
| wait/synch/rwlock/sql/LOCK_sys_init_connect       | YES     | YES   |
| wait/synch/rwlock/sql/LOCK_sys_init_slave         | YES     | YES   |
...
| wait/io/file/sql/binlog                           | YES     | YES   |
| wait/io/file/sql/binlog_index                     | YES     | YES   |
| wait/io/file/sql/casetest                         | YES     | YES   |
| wait/io/file/sql/dbopt                            | YES     | YES   |
...
```

要控制工具是否启用，请将其`ENABLED`列设置为`YES`或`NO`。要配置是否为已启用工具收集计时信息，请将其`TIMED`值设置为`YES`或`NO`。设置`TIMED`列会影响 Performance Schema 表内容，如第 29.4.1 节“性能模式事件计时”中所述。

对大多数`setup_instruments`行的修改立即影响监视。对于某些工具，修改仅在服务器启动时生效；在运行时更改对其没有影响。这主要影响服务器中的互斥体、条件和读写锁，尽管可能还有其他工具也是如此。

`setup_instruments` 表提供了对事件生成最基本的控制形式。为了根据被监视的对象或线程类型进一步细化事件生成，可以使用其他表，如第 29.4.3 节“事件预过滤”中所述。

以下示例演示了对`setup_instruments`表可能的操作。这些更改，像其他预过滤操作一样，会影响所有用户。其中一些查询使用`LIKE`运算符和模式匹配工具名称。有关指定模式以选择工具的更多信息，请参阅第 29.4.9 节“为过滤操作命名工具或消费者”。

+   禁用所有工具：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO';
    ```

    现在不再收集任何事件。

+   禁用所有文件工具，并将它们添加到当前禁用工具集中：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO'
    WHERE NAME LIKE 'wait/io/file/%';
    ```

+   仅禁用文件工具，启用所有其他工具：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = IF(NAME LIKE 'wait/io/file/%', 'NO', 'YES');
    ```

+   启用除了`mysys`库中的工具之外的所有工具：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = CASE WHEN NAME LIKE '%/mysys/%' THEN 'YES' ELSE 'NO' END;
    ```

+   禁用特定工具：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO'
    WHERE NAME = 'wait/synch/mutex/mysys/TMPDIR_mutex';
    ```

+   要切换工具的状态，“翻转”其`ENABLED`值：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = IF(ENABLED = 'YES', 'NO', 'YES')
    WHERE NAME = 'wait/synch/mutex/mysys/TMPDIR_mutex';
    ```

+   禁用所有事件的计时：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET TIMED = 'NO';
    ```
