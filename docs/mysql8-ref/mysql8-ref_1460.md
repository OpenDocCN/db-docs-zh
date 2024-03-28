> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-timeout.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-timeout.html)

#### 19.5.1.32 复制重试和超时

系统变量`replica_transaction_retries`的全局值（从 MySQL 8.0.26 开始）或`slave_transaction_retries`（在 MySQL 8.0.26 之前）设置了单线程或多线程复制品上应用程序线程在停止之前自动重试失败事务的最大次数。当 SQL 线程由于`InnoDB`死锁而无法执行事务，或者事务的执行时间超过`InnoDB``innodb_lock_wait_timeout`值时，事务会自动重试。如果事务有一个阻止其成功的非临时错误，则不会重试。

默认设置为`replica_transaction_retries`或`slave_transaction_retries`为 10，意味着在出现明显临时错误的失败事务会在停止应用程序线程之前重试 10 次。将该变量设置为 0 会禁用事务的自动重试。在多线程复制中，指定的事务重试次数可以在所有通道的所有应用程序线程上进行。性能模式表`replication_applier_status`显示了每个复制通道上发生的事务重试总数，在`COUNT_TRANSACTIONS_RETRIES`列中。

重试事务的过程可能导致复制品或组复制组成员出现滞后，可以将其配置为单线程或多线程复制品。性能模式表`replication_applier_status_by_worker`显示了单线程或多线程复制品上应用程序线程重试事务的详细信息。这些数据包括时间戳，显示应用程序线程从开始到结束应用最后一个事务所花费的时间（以及当前正在进行的事务何时开始），以及这是在原始来源和直接来源上提交后多长时间。数据还显示了最后一个事务和当前正在进行的事务的重试次数，并使您能够识别导致事务重试的瞬时错误。您可以使用此信息查看事务重试是否导致复制滞后，并调查导致重试的失败的根本原因。
