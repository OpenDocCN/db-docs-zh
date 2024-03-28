# 29.14 性能模式命令选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-options.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-options.html)

可以在服务器启动时在命令行或选项文件中指定性能模式参数，以配置性能模式仪器和消费者。在许多情况下也可以进行运行时配置（请参见第 29.4 节“性能模式运行时配置”），但当运行时配置太晚以影响在启动过程中已��初始化的仪器时，必须使用启动配置。

可以使用以下语法在启动时配置性能模式消费者和仪器。有关更多详细信息，请参见第 29.3 节“性能模式启动配置”。

+   `--performance-schema-consumer-*`consumer_name`*=`value``

    配置性能模式消费者。`setup_consumers`表中的消费者名称使用下划线，但对于在启动时设置的消费者，名称中的破折号和下划线是等效的。配置单个消费者的选项将在本节后面详细介绍。

+   `--performance-schema-instrument=*`instrument_name`*=`value``

    配置性能模式仪器。名称可以作为模式给出，以配置与该模式匹配的仪器。

以下项目配置单个消费者：

+   `--performance-schema-consumer-events-stages-current=`value``

    配置`events-stages-current`消费者。

+   `--performance-schema-consumer-events-stages-history=`value``

    配置`events-stages-history`消费者。

+   `--performance-schema-consumer-events-stages-history-long=`value``

    配置`events-stages-history-long`消费者。

+   `--performance-schema-consumer-events-statements-cpu=`value``

    配置`events-statements-cpu`消费者。

+   `--performance-schema-consumer-events-statements-current=`value``

    配置`events-statements-current`消费者。

+   `--performance-schema-consumer-events-statements-history=`value``

    配置`events-statements-history`消费者。

+   `--performance-schema-consumer-events-statements-history-long=`value``

    配置`events-statements-history-long`消费者。

+   `--performance-schema-consumer-events-transactions-current=`value``

    配置性能模式`events-transactions-current`消费者。

+   `--performance-schema-consumer-events-transactions-history=`value``

    配置性能模式`events-transactions-history`消费者。

+   `--performance-schema-consumer-events-transactions-history-long=`value``

    配置性能模式`events-transactions-history-long`消费者。

+   `--performance-schema-consumer-events-waits-current=`value``

    配置`events-waits-current`消费者。

+   `--performance-schema-consumer-events-waits-history=`value``

    配置`events-waits-history`消费者。

+   `--performance-schema-consumer-events-waits-history-long=`value``

    配置`events-waits-history-long`消费者。

+   `--performance-schema-consumer-global-instrumentation=`value``

    配置`global-instrumentation`消费者。

+   `--performance-schema-consumer-statements-digest=`value``

    配置`statements-digest`消费者。

+   `--performance-schema-consumer-thread-instrumentation=`value``

    配置`thread-instrumentation`消费者。
