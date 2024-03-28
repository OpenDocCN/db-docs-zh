> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-preload-buffer-pool.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-preload-buffer-pool.html)

#### 17.8.3.6 保存和恢复缓冲池状态

为了在服务器重新启动后减少热身期，`InnoDB` 在服务器关闭时保存每个缓冲池中最近使用的页面的一定百分比，并在服务器启动时恢复这些页面。存储的最近使用页面的百分比由 `innodb_buffer_pool_dump_pct` 配置选项定义。

重启繁忙的服务器后，通常会有一个热身期，吞吐量稳步增加，因为缓冲池中的磁盘页被重新加载到内存中（当相同数据被查询、更新等时）。在启动时恢复缓冲池的能力通过重新加载重启前在缓冲池中的磁盘页，缩短了热身期，而不是等待 DML 操作访问相应的行。此外，I/O 请求可以批量执行，使整体 I/O 更快。页面加载在后台进行，不会延迟数据库启动。

除了在关闭时保存缓冲池状态并在启动时恢复它之外，您还可以在服务器运行时的任何时候保存和恢复缓冲池状态。例如，在稳定的工作负载下达到稳定吞吐量后，您可以保存缓冲池的状态。您还可以在运行报告或将数据页带入缓冲池的维护作业后，或在运行其他非典型工作负载后，恢复先前的缓冲池状态。

即使缓冲池的大小可能达到几个千兆字节，但相比之下，`InnoDB` 保存到磁盘的缓冲池数据非常小。只保存用于定位适当页面的表空间 ID 和页面 ID。此信息来自 `INNODB_BUFFER_PAGE_LRU` `INFORMATION_SCHEMA` 表。默认情况下，表空间 ID 和页面 ID 数据保存在名为 `ib_buffer_pool` 的文件中，该文件保存在 `InnoDB` 数据目录中。文件名和位置可以使用 `innodb_buffer_pool_filename` 配置参数进行修改。

因为数据像常规数据库操作一样在缓冲池中缓存并被淘汰，所以如果磁盘页最近被更新，或者 DML 操作涉及尚未加载的数据，都不会有问题。加载机制会跳过不再存在的请求页面。

底层机制涉及一个后台线程，用于执行转储和加载操作。

来自压缩表的磁盘页面以压缩形式加载到缓冲池中。在进行 DML 操作期间访问页面内容时，页面将像往常一样解压缩。因为解压缩页面是一个消耗 CPU 的过程，所以为了并发性能更高，最好在连接线程中执行操作，而不是在执行缓冲池恢复操作的单个线程中执行操作。

有关保存和恢复缓冲池状态的操作描述在以下主题中：

+   配置缓冲池页面转储百分比

+   在关闭时保存缓冲池状态并在启动时恢复

+   在线保存和恢复缓冲池状态

+   显示缓冲池转储进度

+   显示缓冲池加载进度

+   中止缓冲池加载操作

+   使用性能模式监视缓冲池加载进度

##### 配置缓冲池页面转储百分比

在从缓冲池转储页面之前，您可以通过设置`innodb_buffer_pool_dump_pct`选项来配置要转储的最近使用的缓冲池页面的百分比。如果您计划在服务器运行时转储缓冲池页面，可以动态配置该选项：

```sql
SET GLOBAL innodb_buffer_pool_dump_pct=40;
```

如果您计划在服务器关闭时转储缓冲池页面，请在配置文件中设置`innodb_buffer_pool_dump_pct`。

```sql
[mysqld]
innodb_buffer_pool_dump_pct=40
```

`innodb_buffer_pool_dump_pct`的默认值为 25（转储最近使用的页面的 25%）。

##### 在关闭时保存缓冲池状态并在启动时恢复

在关闭服务器之前，请在关闭服务器之前发出以下语句以保存缓冲池状态：

```sql
SET GLOBAL innodb_buffer_pool_dump_at_shutdown=ON;
```

`innodb_buffer_pool_dump_at_shutdown`默认启用。

在服务器启动时恢复缓冲池状态，请在启动服务器时指定`--innodb-buffer-pool-load-at-startup`选项：

```sql
mysqld --innodb-buffer-pool-load-at-startup=ON;
```

`innodb_buffer_pool_load_at_startup`默认启用。

##### 在线保存和恢复缓冲池状态

要在 MySQL 服务器运行时保存缓冲池状态，请执行以下语句：

```sql
SET GLOBAL innodb_buffer_pool_dump_now=ON;
```

在 MySQL 运行时恢复缓冲池状态，请执行以下语句：

```sql
SET GLOBAL innodb_buffer_pool_load_now=ON;
```

##### 显示缓冲池转储进度

要在将缓冲池状态保存到磁盘时显示进度，请执行以下语句：

```sql
SHOW STATUS LIKE 'Innodb_buffer_pool_dump_status';
```

如果操作尚未开始，则返回“未开始”。如果操作已完成，则打印完成时间（例如 完成于 110505 12:18:02）。如果操作正在进行中，则提供状态信息（例如 正在转储缓冲池 5/7，页码 237/2873）。

##### 显示缓冲池加载进度

要在加载缓冲池时显示进度，请执行以下语句：

```sql
SHOW STATUS LIKE 'Innodb_buffer_pool_load_status';
```

如果操作尚未开始，则返回“未开始”。如果操作已完成，则打印完成时间（例如 完成于 110505 12:23:24）。如果操作正在进行中，则提供状态信息（例如 已加载 123/22301 页）。

##### 中止缓冲池加载操作

要中止缓冲池加载操作，请执行以下语句：

```sql
SET GLOBAL innodb_buffer_pool_load_abort=ON;
```

##### 使用性能模式监视缓冲池加载进度

您可以使用性能模式监视缓冲池加载进度。

以下示例演示了如何启用`stage/innodb/buffer pool load`阶段事件工具和相关的消费者表以监视缓冲池加载进度。

有关本示例中使用的缓冲池转储和加载过程的信息，请参阅第 17.8.3.6 节，“保存和恢复缓冲池状态”。有关性能模式阶段事件工具和相关消费者的信息，请参阅第 29.12.5 节，“性能模式阶段事件表”。

1.  启用`stage/innodb/buffer pool load`工具：

    ```sql
    mysql> UPDATE performance_schema.setup_instruments SET ENABLED = 'YES' 
           WHERE NAME LIKE 'stage/innodb/buffer%';
    ```

1.  启用阶段事件消费者表，包括`events_stages_current`、`events_stages_history`和`events_stages_history_long`。

    ```sql
    mysql> UPDATE performance_schema.setup_consumers SET ENABLED = 'YES' 
           WHERE NAME LIKE '%stages%';
    ```

1.  通过启用`innodb_buffer_pool_dump_now`来转储当前缓冲池状态。

    ```sql
    mysql> SET GLOBAL innodb_buffer_pool_dump_now=ON;
    ```

1.  检查缓冲池转储状态以确保操作已完成。

    ```sql
    mysql> SHOW STATUS LIKE 'Innodb_buffer_pool_dump_status'\G
    *************************** 1\. row ***************************
    Variable_name: Innodb_buffer_pool_dump_status
            Value: Buffer pool(s) dump completed at 150202 16:38:58
    ```

1.  通过启用`innodb_buffer_pool_load_now`来加载缓冲池：

    ```sql
    mysql> SET GLOBAL innodb_buffer_pool_load_now=ON;
    ```

1.  通过查询性能模式`events_stages_current`表，检查缓冲池加载操作的当前状态。`WORK_COMPLETED`列显示加载的缓冲池页面数。`WORK_ESTIMATED`列提供剩余工作的估计，以页面为单位。

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED
           FROM performance_schema.events_stages_current;
    +-------------------------------+----------------+----------------+
    | EVENT_NAME                    | WORK_COMPLETED | WORK_ESTIMATED |
    +-------------------------------+----------------+----------------+
    | stage/innodb/buffer pool load |           5353 |           7167 |
    +-------------------------------+----------------+----------------+
    ```

    如果缓冲池加载操作已完成，则`events_stages_current`表将返回一个空集。在这种情况下，您可以查询`events_stages_history`表查看已完成事件的数据。例如：

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED 
           FROM performance_schema.events_stages_history;
    +-------------------------------+----------------+----------------+
    | EVENT_NAME                    | WORK_COMPLETED | WORK_ESTIMATED |
    +-------------------------------+----------------+----------------+
    | stage/innodb/buffer pool load |           7167 |           7167 |
    +-------------------------------+----------------+----------------+
    ```

注意

在启动时使用`innodb_buffer_pool_load_at_startup`加载缓冲池时，您还可以使用性能模式监视缓冲池加载进度。在这种情况下，必须在启动时启用`stage/innodb/buffer pool load`工具和相关消费者。有关更多信息，请参见第 29.3 节，“性能模式启动配置”。
