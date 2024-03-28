# 29.19.1 使用性能模式进行查询分析

> 译文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-query-profiling.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-query-profiling.html)

以下示例演示了如何使用性能模式语句事件和阶段事件检索与`SHOW PROFILES`和`SHOW PROFILE`语句提供的性能信息类似的数据。

`setup_actors`表可用于通过主机、用户或帐户限制历史事件的收集，以减少运行时开销和历史表中收集的数据量。示例的第一步显示了如何将历史事件的收集限制为特定用户。

性能模式以皮秒（万亿分之一秒）显示事件计时器信息，以将时间数据标准化为标准单位。在以下示例中，`TIMER_WAIT`值除以 1000000000000 以以秒为单位显示数据。值还被截断为 6 位小数以以与`SHOW PROFILES`和`SHOW PROFILE`语句相同的格式显示数据。

1.  将历史事件的收集限制为运行查询的用户。默认情况下，`setup_actors`配置为允许监视和历史事件收集所有前台线程：

    ```sql
    mysql> SELECT * FROM performance_schema.setup_actors;
    +------+------+------+---------+---------+
    | HOST | USER | ROLE | ENABLED | HISTORY |
    +------+------+------+---------+---------+
    | %    | %    | %    | YES     | YES     |
    +------+------+------+---------+---------+
    ```

    更新`setup_actors`表中的默认行，禁用所有前台线程的历史事件收集和监视，并插入一个新行，以启用运行查询的用户的监视和历史事件收集：

    ```sql
    mysql> UPDATE performance_schema.setup_actors
           SET ENABLED = 'NO', HISTORY = 'NO'
           WHERE HOST = '%' AND USER = '%';

    mysql> INSERT INTO performance_schema.setup_actors
           (HOST,USER,ROLE,ENABLED,HISTORY)
           VALUES('localhost','test_user','%','YES','YES');
    ```

    `setup_actors`表中的数据现在应类似于以下内容：

    ```sql
    mysql> SELECT * FROM performance_schema.setup_actors;
    +-----------+-----------+------+---------+---------+
    | HOST      | USER      | ROLE | ENABLED | HISTORY |
    +-----------+-----------+------+---------+---------+
    | %         | %         | %    | NO      | NO      |
    | localhost | test_user | %    | YES     | YES     |
    +-----------+-----------+------+---------+---------+
    ```

1.  确保通过更新`setup_instruments`表启用语句和阶段仪器。一些仪器可能已默认启用。

    ```sql
    mysql> UPDATE performance_schema.setup_instruments
           SET ENABLED = 'YES', TIMED = 'YES'
           WHERE NAME LIKE '%statement/%';

    mysql> UPDATE performance_schema.setup_instruments
           SET ENABLED = 'YES', TIMED = 'YES'
           WHERE NAME LIKE '%stage/%';
    ```

1.  确保`events_statements_*`和`events_stages_*`消费者已启用。一些消费者可能已默认启用。

    ```sql
    mysql> UPDATE performance_schema.setup_consumers
           SET ENABLED = 'YES'
           WHERE NAME LIKE '%events_statements_%';

    mysql> UPDATE performance_schema.setup_consumers
           SET ENABLED = 'YES'
           WHERE NAME LIKE '%events_stages_%';
    ```

1.  在您正在监视的用户帐户下，运行要分析的语句。例如：

    ```sql
    mysql> SELECT * FROM employees.employees WHERE emp_no = 10001;
    +--------+------------+------------+-----------+--------+------------+
    | emp_no | birth_date | first_name | last_name | gender | hire_date |
    +--------+------------+------------+-----------+--------+------------+
    |  10001 | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
    +--------+------------+------------+-----------+--------+------------+
    ```

1.  通过查询`events_statements_history_long`表来识别语句的`EVENT_ID`。这一步类似于运行`SHOW PROFILES`来识别`Query_ID`。以下查询产生类似于`SHOW PROFILES`的输出：

    ```sql
    mysql> SELECT EVENT_ID, TRUNCATE(TIMER_WAIT/1000000000000,6) as Duration, SQL_TEXT
           FROM performance_schema.events_statements_history_long WHERE SQL_TEXT like '%10001%';
    +----------+----------+--------------------------------------------------------+
    | event_id | duration | sql_text                                               |
    +----------+----------+--------------------------------------------------------+
    |       31 | 0.028310 | SELECT * FROM employees.employees WHERE emp_no = 10001 |
    +----------+----------+--------------------------------------------------------+
    ```

1.  查询`events_stages_history_long`表以检索语句的阶段事件。阶段通过事件嵌套与语句相关联。每个阶段事件记录都有一个包含父语句的`EVENT_ID`的`NESTING_EVENT_ID`列。

    ```sql
    mysql> SELECT event_name AS Stage, TRUNCATE(TIMER_WAIT/1000000000000,6) AS Duration
           FROM performance_schema.events_stages_history_long WHERE NESTING_EVENT_ID=31;
    +--------------------------------+----------+
    | Stage                          | Duration |
    +--------------------------------+----------+
    | stage/sql/starting             | 0.000080 |
    | stage/sql/checking permissions | 0.000005 |
    | stage/sql/Opening tables       | 0.027759 |
    | stage/sql/init                 | 0.000052 |
    | stage/sql/System lock          | 0.000009 |
    | stage/sql/optimizing           | 0.000006 |
    | stage/sql/statistics           | 0.000082 |
    | stage/sql/preparing            | 0.000008 |
    | stage/sql/executing            | 0.000000 |
    | stage/sql/Sending data         | 0.000017 |
    | stage/sql/end                  | 0.000001 |
    | stage/sql/query end            | 0.000004 |
    | stage/sql/closing tables       | 0.000006 |
    | stage/sql/freeing items        | 0.000272 |
    | stage/sql/cleaning up          | 0.000001 |
    +--------------------------------+----------+
    ```
