> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-monitoring.html)

#### 7.6.7.10 监控克隆操作

本节描述了监控克隆操作的选项。

+   使用性能模式克隆表监控克隆操作

+   使用性能模式阶段事件监控克隆操作

+   使用性能模式克隆工具监控克隆操作

+   Com_clone 状态变量

##### 使用性能模式克隆表监控克隆操作

克隆操作可能需要一些时间才能完成，这取决于数据量和与数据传输相关的其他因素。您可以使用`clone_status` 和 `clone_progress` 性能模式表在接收 MySQL 服务器实例上监视克隆操作的状态和进度。

注意

`clone_status` 和 `clone_progress` 性能模式表仅可用于监视接收 MySQL 服务器实例上的克隆操作。要监视捐赠 MySQL 服务器实例上的克隆操作，请使用克隆阶段事件，如使用性能模式阶段事件监控克隆操作中所述。

+   `clone_status` 表提供当前或最近执行的克隆操作的状态。克隆操作有四种可能的状态：`未开始`、`进行中`、`已完成` 和 `失败`。

+   `clone_progress` 表提供当前或最近执行的克隆操作的进度信息，按阶段划分。克隆操作的阶段包括 `DROP DATA`、`FILE COPY`、`PAGE_COPY`、`REDO_COPY`、`FILE_SYNC`、`RESTART` 和 `RECOVERY`。

访问性能模式克隆表需要在性能模式上具有`SELECT`和`EXECUTE`权限。

要检查克隆操作的状态：

1.  连接到接收方 MySQL 服务器实例。

1.  查询`clone_status`表：

    ```sql
    mysql> SELECT STATE FROM performance_schema.clone_status;
    +-----------+
    | STATE     |
    +-----------+
    | Completed |
    +-----------+
    ```

如果在克隆操作期间发生故障，您可以查询`clone_status`表以获取错误信息：

```sql
mysql> SELECT STATE, ERROR_NO, ERROR_MESSAGE FROM performance_schema.clone_status;
+-----------+----------+---------------+
| STATE     | ERROR_NO | ERROR_MESSAGE |
+-----------+----------+---------------+
| Failed    |      xxx | "xxxxxxxxxxx" |
+-----------+----------+---------------+
```

要查看克隆操作的每个阶段的详细信息：

1.  连接到接收方 MySQL 服务器实例。

1.  查询`clone_progress`表。例如，以下查询提供了克隆操作每个阶段的状态和结束时间数据：

    ```sql
    mysql> SELECT STAGE, STATE, END_TIME FROM performance_schema.clone_progress;
    +-----------+-----------+----------------------------+
    | stage     | state     | end_time                   |
    +-----------+-----------+----------------------------+
    | DROP DATA | Completed | 2019-01-27 22:45:43.141261 |
    | FILE COPY | Completed | 2019-01-27 22:45:44.457572 |
    | PAGE COPY | Completed | 2019-01-27 22:45:44.577330 |
    | REDO COPY | Completed | 2019-01-27 22:45:44.679570 |
    | FILE SYNC | Completed | 2019-01-27 22:45:44.918547 |
    | RESTART   | Completed | 2019-01-27 22:45:48.583565 |
    | RECOVERY  | Completed | 2019-01-27 22:45:49.626595 |
    +-----------+-----------+----------------------------+
    ```

    要监视其他克隆状态和进度数据点，请参考第 29.12.19 节，“性能模式克隆表”。

##### 使用性能模式阶段事件监视克隆操作

克隆操作可能需要一些时间才能完成，这取决于数据量和与数据传输相关的其他因素。有三个阶段事件用于监视克隆操作的进度。每个阶段事件报告 ``WORK_COMPLETED`` 和 ``WORK_ESTIMATED`` 值。随着操作的进行，报告的值会进行修订。

可以在捐赠方或接收方 MySQL 服务器实例上使用此方法监视克隆操作。

按发生顺序，克隆操作阶段事件包括：

+   `stage/innodb/clone (file copy)`: 表示克隆操作的文件复制阶段的进度。``WORK_ESTIMATED`` 和 ``WORK_COMPLETED`` 单位为文件块。在文件复制阶段开始时，就已知要传输的文件数量，并且根据文件数量估算出块的数量。`WORK_ESTIMATED` 设置为估计文件块的数量。每发送一个块后，`WORK_COMPLETED` 都会更新。

+   `stage/innodb/clone (page copy)`: 表示克隆操作的页面复制阶段的进度。`WORK_ESTIMATED` 和 `WORK_COMPLETED` 单位为页面。一旦文件复制阶段完成，就会知道要传输的页面数量，并且将 `WORK_ESTIMATED` 设置为此值。每发送一个页面后，`WORK_COMPLETED` 都会更新。

+   `stage/innodb/clone (redo copy)`: 表示克隆操作的重做复制阶段的进度。`WORK_ESTIMATED` 和 `WORK_COMPLETED` 单位为重做块。一旦页面复制阶段完成，就会知道要传输的重做块数量，并且 `WORK_ESTIMATED` 设置为此值。每发送一个块后，`WORK_COMPLETED` 都会更新。

以下示例演示了如何启用`stage/innodb/clone%`事件仪器和相关的消费者表来监视克隆操作。有关性能模式阶段事件仪器和相关消费者的信息，请参见第 29.12.5 节，“性能模式阶段事件表”。

1.  启用`stage/innodb/clone%`仪器：

    ```sql
    mysql> UPDATE performance_schema.setup_instruments SET ENABLED = 'YES'
           WHERE NAME LIKE 'stage/innodb/clone%';
    ```

1.  启用阶段事件消费者表，包括`events_stages_current`、`events_stages_history`和`events_stages_history_long`。

    ```sql
    mysql> UPDATE performance_schema.setup_consumers SET ENABLED = 'YES'
           WHERE NAME LIKE '%stages%';
    ```

1.  运行克隆操作。在此示例中，将本地数据目录克隆到名为`cloned_dir`的目录。

    ```sql
    mysql> CLONE LOCAL DATA DIRECTORY = '/path/to/cloned_dir';
    ```

1.  通过查询性能模式`events_stages_current`表来检查克隆操作的进度。显示的阶段事件取决于正在进行的克隆阶段。`WORK_COMPLETED`列显示已完成的工作。`WORK_ESTIMATED`列显示总共需要的工作量。

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM performance_schema.events_stages_current
           WHERE EVENT_NAME LIKE 'stage/innodb/clone%';
    +--------------------------------+----------------+----------------+
    | EVENT_NAME                     | WORK_COMPLETED | WORK_ESTIMATED |
    +--------------------------------+----------------+----------------+
    | stage/innodb/clone (redo copy) |              1 |              1 |
    +--------------------------------+----------------+----------------+
    ```

    如果克隆操作已经完成，`events_stages_current`表将返回一个空集。在这种情况下，您可以检查`events_stages_history`表以查看已完成操作的事件数据。例如：

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM events_stages_history
           WHERE EVENT_NAME LIKE 'stage/innodb/clone%';
    +--------------------------------+----------------+----------------+
    | EVENT_NAME                     | WORK_COMPLETED | WORK_ESTIMATED |
    +--------------------------------+----------------+----------------+
    | stage/innodb/clone (file copy) |            301 |            301 |
    | stage/innodb/clone (page copy) |              0 |              0 |
    | stage/innodb/clone (redo copy) |              1 |              1 |
    +--------------------------------+----------------+----------------+
    ```

##### 使用性能模式克隆仪器监视克隆操作

性能模式提供了用于高级性能监控克隆操作的仪器。要查看可用的克隆仪器，并发出以下查询：

```sql
mysql> SELECT NAME,ENABLED FROM performance_schema.setup_instruments
       WHERE NAME LIKE '%clone%';
+---------------------------------------------------+---------+
| NAME                                              | ENABLED |
+---------------------------------------------------+---------+
| wait/synch/mutex/innodb/clone_snapshot_mutex      | NO      |
| wait/synch/mutex/innodb/clone_sys_mutex           | NO      |
| wait/synch/mutex/innodb/clone_task_mutex          | NO      |
| wait/synch/mutex/group_rpl/LOCK_clone_donor_list  | NO      |
| wait/synch/mutex/group_rpl/LOCK_clone_handler_run | NO      |
| wait/synch/mutex/group_rpl/LOCK_clone_query       | NO      |
| wait/synch/mutex/group_rpl/LOCK_clone_read_mode   | NO      |
| wait/synch/cond/group_rpl/COND_clone_handler_run  | NO      |
| wait/io/file/innodb/innodb_clone_file             | YES     |
| stage/innodb/clone (file copy)                    | YES     |
| stage/innodb/clone (redo copy)                    | YES     |
| stage/innodb/clone (page copy)                    | YES     |
| statement/abstract/clone                          | YES     |
| statement/clone/local                             | YES     |
| statement/clone/client                            | YES     |
| statement/clone/server                            | YES     |
| memory/innodb/clone                               | YES     |
| memory/clone/data                                 | YES     |
+---------------------------------------------------+---------+
```

###### 等待仪器

性能模式等待仪器跟踪需要时间的事件。克隆等待事件仪器包括：

+   `wait/synch/mutex/innodb/clone_snapshot_mutex`：跟踪克隆快照互斥锁的等待事件，该互斥锁在多个克隆线程之间同步访问动态快照对象（在捐赠者和接收者之间）。

+   `wait/synch/mutex/innodb/clone_sys_mutex`：跟踪克隆系统互斥锁的等待事件。在 MySQL 服务器实例中有一个克隆系统对象。此互斥锁在捐赠者和接收者之间同步访问克隆系统对象。它由克隆线程和其他前台和后台线程获取。

+   `wait/synch/mutex/innodb/clone_task_mutex`：跟踪克隆任务互斥锁的等待事件，用于克隆任务管理。`clone_task_mutex`由克隆线程获取。

+   `wait/io/file/innodb/innodb_clone_file`: 跟踪克隆操作的文件上的所有 I/O 等待操作。

关于监控`InnoDB`互斥等待的信息，请参阅第 17.16.2 节，“使用性能模式监控 InnoDB 互斥等待”。关于一般监控等待事件的信息，请参阅第 29.12.4 节，“性能模式等待事件表”。

###### 阶段工具

性能模式阶段事件跟踪语句执行过程中发生的步骤。克隆阶段事件工具包括：

+   `stage/innodb/clone (file copy)`: 表示克隆操作的文件复制阶段的进度。

+   `stage/innodb/clone (redo copy)`: 表示克隆操作的重做复制阶段的进度。

+   `stage/innodb/clone (page copy)`: 表示克隆操作的页面复制阶段的进度。

关于使用阶段事件监控克隆操作的信息，请参阅使用性能模式阶段事件监控克隆操作。关于一般监控阶段事件的信息，请参阅第 29.12.5 节，“性能模式阶段事件表”。

###### 语句工具

性能模式语句事件跟踪语句执行。当启动克隆操作时，由克隆语句工具跟踪的不同语句类型可能并行执行。您可以在性能模式语句事件表中观察这些语句事件。执行的语句数量取决于`clone_max_concurrency`和`clone_autotune_concurrency`设置。

克隆语句事件工具包括：

+   `statement/abstract/clone`: 跟踪在被分类为本地、客户端或服务器操作类型之前的任何克隆操作的语句事件。

+   `statement/clone/local`: 跟踪本地克隆操作的克隆语句事件；在执行`CLONE LOCAL`语句时生成。

+   `statement/clone/client`: 跟踪发生在接收端 MySQL 服务器实例上的远程克隆语句事件；在接收端执行`CLONE INSTANCE`语句时生成。

+   `statement/clone/server`: 跟踪发生在捐赠 MySQL 服务器实例上的远程克隆语句事件；在接收端执行`CLONE INSTANCE`语句时生成。

欲了解有关监控性能模式语句事件的更多信息，请参见 第 29.12.6 节，“性能模式语句事件表”。

###### 内存工具

性能模式内存工具跟踪内存使用情况。克隆内存使用工具包括：

+   `memory/innodb/clone`: 跟踪 `InnoDB` 为动态快照分配的内存。

+   `memory/clone/data`: 跟踪克隆操作期间克隆插件分配的内存。

欲了解使用性能模式监控内存使用情况的更多信息，请参见 第 29.12.20.10 节，“内存摘要表”。

##### Com_clone 状态变量

`Com_clone` 状态变量提供了 `CLONE` 语句执行次数的计数。

欲了解更多信息，请参考关于 `Com_xxx` 语句计数变量的讨论，见 第 7.1.10 节，“服务器状态变量”。
