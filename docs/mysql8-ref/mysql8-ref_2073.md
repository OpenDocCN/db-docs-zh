> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-metadata-locks-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-metadata-locks-table.html)

#### 29.12.13.3 元数据锁表

MySQL 使用元数据锁定来管理对数据库对象的并发访问，并确保数据一致性；参见第 10.11.4 节，“元数据锁定”。元数据锁定不仅适用于表，还适用于模式、存储程序（过程、函数、触发器、计划事件）、表空间、使用`GET_LOCK()`函数获取的用户锁（参见第 14.14 节，“锁定函数”）以及使用第 7.6.9.1 节，“锁定服务”中描述的锁定服务获取的锁。

性能模式通过`metadata_locks`表公开元数据锁信息：

+   已被授予的锁（显示哪些会话拥有哪些当前元数据锁）。

+   已请求但尚未授予的锁（显示哪些会话正在等待哪些元数据锁）。

+   已被死锁检测器杀死的锁请求。

+   已超时并正在等待请求会话的锁请求被丢弃的锁请求。

此信息使您能够了解会话之间的元数据锁依赖关系。您不仅可以看到会话正在等待哪个锁，还可以看到哪个会话当前持有该锁。

`metadata_locks`表是只读的，不能更新。默认情况下，它是自动调整大小的；要配置表大小，请在服务器启动时设置`performance_schema_max_metadata_locks`系统变量。

元数据锁仪表化使用默认启用的`wait/lock/metadata/sql/mdl`工具。

要在服务器启动时控制元数据锁仪表化状态，请在您的`my.cnf`文件中使用以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='wait/lock/metadata/sql/mdl=ON'
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='wait/lock/metadata/sql/mdl=OFF'
    ```

要在运行时控制元数据锁仪表化状态，请更新`setup_instruments`表：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES', TIMED = 'YES'
    WHERE NAME = 'wait/lock/metadata/sql/mdl';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO', TIMED = 'NO'
    WHERE NAME = 'wait/lock/metadata/sql/mdl';
    ```

性能模式维护`metadata_locks`表内容如下，使用`LOCK_STATUS`列指示每个锁的状态：

+   当请求元数据锁并立即获得时，将插入状态为`GRANTED`的行。

+   当请求元数据锁并且未立即获得时，将插入状态为`PENDING`的行。

+   当先前请求的元数据锁被授予时，其行状态将更新为`GRANTED`。

+   当释放元数据锁时，其行将被删除。

+   当挂起的锁请求被死锁检测器取消以打破死锁（`ER_LOCK_DEADLOCK`），其行状态从`PENDING`更新为`VICTIM`。

+   当挂起的锁请求超时（`ER_LOCK_WAIT_TIMEOUT`），其行状态从`PENDING`更新为`TIMEOUT`。

+   当授予锁或挂起的锁请求被终止时，其行状态从`GRANTED`或`PENDING`更新为`KILLED`。

+   `VICTIM`、`TIMEOUT`和`KILLED`状态值简洁明了，表示锁行即将被删除。

+   `PRE_ACQUIRE_NOTIFY`和`POST_RELEASE_NOTIFY`状态值简洁明了，表示元数据锁子系统在进入锁获取操作或离开锁释放操作时通知感兴趣的存储引擎。

`metadata_locks`表具有以下列：

+   `OBJECT_TYPE`

    元数据锁子系统中使用的锁类型。该值是`GLOBAL`、`SCHEMA`、`TABLE`、`FUNCTION`、`PROCEDURE`、`TRIGGER`（当前未使用）、`EVENT`、`COMMIT`、`USER LEVEL LOCK`、`TABLESPACE`、`BACKUP LOCK`或`LOCKING SERVICE`之一。

    `USER LEVEL LOCK`的值表示使用`GET_LOCK()`获取的锁。`LOCKING SERVICE`的值表示使用第 7.6.9.1 节，“锁定服务”中描述的锁定服务获取的锁。

+   `OBJECT_SCHEMA`

    包含对象的模式。

+   `OBJECT_NAME`

    工具化对象的名称。

+   `OBJECT_INSTANCE_BEGIN`

    工具化对象在内存中的地址。

+   `LOCK_TYPE`

    元数据锁子系统的锁类型。该值是`INTENTION_EXCLUSIVE`、`SHARED`、`SHARED_HIGH_PRIO`、`SHARED_READ`、`SHARED_WRITE`、`SHARED_UPGRADABLE`、`SHARED_NO_WRITE`、`SHARED_NO_READ_WRITE`或`EXCLUSIVE`之一。

+   `LOCK_DURATION`

    元数据锁子系统的锁持续时间。该值是`STATEMENT`、`TRANSACTION`或`EXPLICIT`之一。`STATEMENT`和`TRANSACTION`值表示在语句或事务结束时隐式释放的锁，`EXPLICIT`值表示在语句或事务结束时仍然存在的锁，并通过显式操作释放，例如使用`FLUSH TABLES WITH READ LOCK`获取的全局锁。

+   `LOCK_STATUS`

    元数据锁子系统的锁状态。该值是`PENDING`、`GRANTED`、`VICTIM`、`TIMEOUT`、`KILLED`、`PRE_ACQUIRE_NOTIFY`或`POST_RELEASE_NOTIFY`之一。性能模式根据前面描述的情况分配这些值。

+   `SOURCE`

    包含产生事件的有仪器代码的源文件的名称和仪器发生的文件中的行号。这使您可以检查源代码以确定涉及的确切代码。

+   `OWNER_THREAD_ID`

    请求元数据锁的线程。

+   `OWNER_EVENT_ID`

    请求元数据锁的事件。

`metadata_locks` 表具有以下索引:

+   在 (`OBJECT_INSTANCE_BEGIN`) 上的主键

+   在 (`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`) 上的索引

+   在 (`OWNER_THREAD_ID`, `OWNER_EVENT_ID`) 上的索引

`TRUNCATE TABLE` 不允许用于 `metadata_locks` 表。
