> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-group-state-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-group-state-table.html)

#### 29.12.16.1 The tp_thread_group_state Table

注意

此处描述的性能模式表在 MySQL 8.0.14 版本中可用。在 MySQL 8.0.14 之前，请改用相应的 `INFORMATION_SCHEMA` 表；参见 Section 28.5.2, “The INFORMATION_SCHEMA TP_THREAD_GROUP_STATE Table”。

[`tp_thread_group_state`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-group-state-table.html "29.12.16.1 The tp_thread_group_state Table") 表中每个线程组都有一行。每行提供有关组的当前状态的信息。

[`tp_thread_group_state`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-tp-thread-group-state-table.html "29.12.16.1 The tp_thread_group_state Table") 表具有以下列：

+   `TP_GROUP_ID`

    线程组 ID。这是表中的唯一键。

+   `CONSUMER THREADS`

    消费者线程的数量。如果活动线程变得停滞或阻塞，最多有一个线程准备开始执行。

+   `RESERVE_THREADS`

    保留状态中的线程数。这意味着直到需要唤醒新线程且没有消费者线程时才会启动它们。当线程组创建的线程多于正常操作所需的线程时，大多数线程都会进入此状态。通常，线程组需要额外的线程一小段时间，然后一段时间内不再需要它们。在这种情况下，它们进入保留状态，并保持直到再次需要。它们占用一些��外的内存资源，但不占用额外的计算资源。

+   `CONNECT_THREAD_COUNT`

    处理或等待处理连接初始化和身份验证的线程数。每个线程组最多可以有四个连接线程；这些线程在一段不活动时间后会过期。

+   `CONNECTION_COUNT`

    使用此线程组的连接数。

+   `QUEUED_QUERIES`

    高优先级队列中等待的语句数量。

+   `QUEUED_TRANSACTIONS`

    低优先级队列中等待的语句数量。这些是尚未开始的事务的初始语句，因此它们也代表排队的事务。

+   `STALL_LIMIT`

    线程组的[`thread_pool_stall_limit`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_thread_pool_stall_limit) 系统变量的值。这对于所有线程组都是相同的值。

+   `PRIO_KICKUP_TIMER`

    线程组的[`thread_pool_prio_kickup_timer`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_thread_pool_prio_kickup_timer) 系统变量的值。这对于所有线程组都是相同的值。

+   `ALGORITHM`

    线程组的[`thread_pool_algorithm`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_thread_pool_algorithm) 系统变量的值。这对于所有线程组都是相同的值。

+   `线程计数`

    作为线程组的一部分启动的线程数。

+   `活动线程计数`

    在执行语句的活动线程数。

+   `停滞线程计数`

    线程组中停滞语句的数量。停滞的语句可能正在执行，但从线程池的角度来看，它们停滞不前。长时间运行的语句很快就会进入这个类别。

+   `等待线程编号`

    如果有一个线程处理线程组中语句的轮询，这指定了该线程组内的线程编号。这个线程可能正在执行语句。

+   `最老的排队`

    最老的排队语句等待执行的毫秒数。

+   `线程组中的最大线程 ID`

    线程组中线程的最大线程 ID。当从`tp_thread_state`表中选择线程时，这与`MAX(TP_THREAD_NUMBER)`相同。也就是说，这两个查询是等效的：

    ```sql
    SELECT TP_GROUP_ID, MAX_THREAD_IDS_IN_GROUP
    FROM tp_thread_group_state;

    SELECT TP_GROUP_ID, MAX(TP_THREAD_NUMBER)
    FROM tp_thread_state GROUP BY TP_GROUP_ID;
    ```

`tp_thread_group_state`表具有以下索引：

+   (`TP_GROUP_ID`)上的唯一索引

不允许对`tp_thread_group_state`表进行`TRUNCATE TABLE`。
