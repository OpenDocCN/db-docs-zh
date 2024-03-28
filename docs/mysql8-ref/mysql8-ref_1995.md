# 29.4.2 性能模式事件过滤

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-filtering.html)

事件以生产者/消费者方式进行处理：

+   仪器化代码是事件的来源并产生要收集的事件。`setup_instruments`表列出了可以收集事件的仪器，它们是否已启用以及（对于已启用的仪器）是否收集计时信息：

    ```sql
    mysql> SELECT NAME, ENABLED, TIMED
           FROM performance_schema.setup_instruments;
    +---------------------------------------------------+---------+-------+
    | NAME                                              | ENABLED | TIMED |
    +---------------------------------------------------+---------+-------+
    ...
    | wait/synch/mutex/sql/LOCK_global_read_lock        | YES     | YES   |
    | wait/synch/mutex/sql/LOCK_global_system_variables | YES     | YES   |
    | wait/synch/mutex/sql/LOCK_lock_db                 | YES     | YES   |
    | wait/synch/mutex/sql/LOCK_manager                 | YES     | YES   |
    ...
    ```

    `setup_instruments`表提供了对事件生成最基本的控制形式。为了根据被监视的对象或线程类型进一步细化事件生成，可以使用其他表，如第 29.4.3 节“事件预过滤”中所述。

+   性能模式表是事件的目的地并消耗事件。`setup_consumers`表列出了可以发送事件信息的消费者类型以及它们是否已启用：

    ```sql
    mysql> SELECT * FROM performance_schema.setup_consumers;
    +----------------------------------+---------+
    | NAME                             | ENABLED |
    +----------------------------------+---------+
    | events_stages_current            | NO      |
    | events_stages_history            | NO      |
    | events_stages_history_long       | NO      |
    | events_statements_cpu            | NO      |
    | events_statements_current        | YES     |
    | events_statements_history        | YES     |
    | events_statements_history_long   | NO      |
    | events_transactions_current      | YES     |
    | events_transactions_history      | YES     |
    | events_transactions_history_long | NO      |
    | events_waits_current             | NO      |
    | events_waits_history             | NO      |
    | events_waits_history_long        | NO      |
    | global_instrumentation           | YES     |
    | thread_instrumentation           | YES     |
    | statements_digest                | YES     |
    +----------------------------------+---------+
    ```

过滤可以在性能监控的不同阶段进行：

+   **预过滤。** 这是通过修改性能模式配置，以便只从生产者收集某些类型的事件，并且收集的事件仅更新某些消费者。为此，启用或禁用仪器或消费者。预过滤由性能模式执行，并具有适用于所有用户的全局效果。

    使用预过滤的原因：

    +   减少开销。即使启用了所有仪器，性能模式的开销也应该很小，但也许你想进一步减少它。或者你不关心计时事件，想要禁用计时代码以消除计时开销。

    +   为了避免将你不感兴趣的事件填充到当前事件或历史表中。预过滤为启用的仪器类型的行实例留下更多“空间”在这些表中。如果只启用了文件仪器并进行了预过滤，那么不会为非文件仪器收集行。通过后过滤，会收集非文件事件，为文件事件留下较少的行。

    +   避免维护某些类型的事件表。如果禁用了一个消费者，服务器就不会花时间维护该消费者的目的地。例如，如果你不关心事件历史，可以禁用历史表消费者以提高性能。

+   **后过滤。** 这涉及在从性能模式表中选择信息的查询中使用`WHERE`子句，以指定您想要查看哪些可用事件。后过滤是基于每个用户进行的，因为个别用户选择感兴趣的可用事件。

    使用后过滤的原因：

    +   避免为个别用户做出关于哪些事件信息感兴趣的决定。

    +   当事先不知道要施加的预过滤限制时，可以使用性能模式来调查性能问题。

以下各节提供有关预过滤的更多详细信息，并提供有关在过滤操作中命名仪器或消费者的指南。有关编写查询以检索信息（后过滤）的信息，请参见第 29.5 节，“性能模式查询”。
