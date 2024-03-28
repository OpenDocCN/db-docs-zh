> 原文：[`dev.mysql.com/doc/refman/8.0/en/thread-pool-elements.html`](https://dev.mysql.com/doc/refman/8.0/en/thread-pool-elements.html)

#### 7.6.3.1 线程池元素

MySQL 企业线程池包括以下元素：

+   一个插件库文件实现了线程池代码的插件以及几个相关的监控表，提供有关线程池操作的信息：

    +   截至 MySQL 8.0.14，监控表是性能模式表；请参见第 29.12.16 节，“性能模式线程池表”。

    +   在 MySQL 8.0.14 之前，监控表是`INFORMATION_SCHEMA`表；请参见第 28.5 节，“INFORMATION_SCHEMA 线程池表”。

        `INFORMATION_SCHEMA`表现在已被弃用；预计它们将在将来的 MySQL 版本中被移除。应用程序应该从`INFORMATION_SCHEMA`表过渡到性能模式表。例如，如果一个应用程序使用此查询：

        ```sql
        SELECT * FROM INFORMATION_SCHEMA.TP_THREAD_STATE;
        ```

        应用程序应该改用此查询：

        ```sql
        SELECT * FROM performance_schema.tp_thread_state;
        ```

    注意

    如果您没有加载所有监控表，一些或所有 MySQL 企业监控器线程池图可能为空。

    要详细了解线程池的工作原理，请参见第 7.6.3.3 节，“线程池操作”。

+   有几个系统变量与线程池相关。当服务器成功加载线程池插件时，`thread_handling`系统变量的值为`loaded-dynamically`。

    其他相关的系统变量由线程池插件实现，并且除非启用，否则不可用。有关使用这些变量的信息，请参见第 7.6.3.3 节，“线程池操作”和第 7.6.3.4 节，“线程池调整”。

+   性能模式具有暴露有关线程池信息的工具，可用于调查操作性能。要识别它们，请使用此查询：

    ```sql
    SELECT * FROM performance_schema.setup_instruments
    WHERE NAME LIKE '%thread_pool%';
    ```

    有关更多信息，请参见第二十九章，“MySQL 性能模式”。
