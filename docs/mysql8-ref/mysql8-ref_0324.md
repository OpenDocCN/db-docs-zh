> 原文：[`dev.mysql.com/doc/refman/8.0/en/locking-service.html`](https://dev.mysql.com/doc/refman/8.0/en/locking-service.html)

#### 7.6.9.1 锁定服务

MySQL 发行版提供了一个可在两个级别访问的锁定接口：

+   在 SQL 级别上，作为一组可加载函数，每个函数映射到对服务例程的调用。

+   作为 C 语言接口，可作为服务器插件或可加载函数的插件服务调用。

有关插件服务的一般信息，请参见 Section 7.6.9, “MySQL Plugin Services”。有关可加载函数的一般信息，请参见添加可加载函数。

锁定接口具有以下特点：

+   锁具有三个属性：锁命名空间、锁名称和锁模式：

    +   锁由命名空间和锁名称的组合标识。命名空间使不同应用程序可以在不发生冲突的情况下使用相同的锁名称，方法是在不同的命名空间中创建锁。例如，如果应用程序 A 和 B 使用命名空间`ns1`和`ns2`，则每个应用程序可以使用锁名称`lock1`和`lock2`而不会干扰另一个应用程序。

    +   锁模式为读或写。读锁是共享的：如果一个会话对给定的锁标识符有读锁定，则其他会话可以对相同标识符获取读锁定。写锁是排他的：如果一个会话对给定的锁标识符有写锁定，则其他会话无法对相同标识符获取读或写锁定。

+   命名空间和锁名称必须为非`NULL`、非空，并且最大长度为 64 个字符。如果命名空间或锁名称指定为`NULL`、空字符串或长度超过 64 个字符，则会导致`ER_LOCKING_SERVICE_WRONG_NAME`错误。

+   锁定接口将命名空间和锁名称视为二进制字符串，因此比较区分大小写。

+   锁定接口提供了获取锁定和释放锁定的函数。调用这些函数不需要特殊权限。权限检查是调用应用程序的责任。

+   如果锁定不可立即获得，可以等待锁定。锁定获取调用需要一个整数超时值，指示在放弃之前等待多少秒以获取锁定。如果超时到达而未成功获取锁定，则会发生`ER_LOCKING_SERVICE_TIMEOUT`错误。如果超时为 0，则不会等待，如果无法立即获取锁定，则调用会产生错误。

+   锁定接口检测不同会话中的锁获取调用之间的死锁。在这种情况下，锁定服务选择一个调用者，并以`ER_LOCKING_SERVICE_DEADLOCK`错误终止其锁获取请求。此错误不会导致事务回滚。在死锁情况下选择会话时，锁定服务更喜欢持有读锁的会话而不是持有写锁的会话。

+   一个会话可以通过单个锁获取调用获取多个锁。对于给定的调用，锁获取是原子的：如果所有锁都被获取，则调用成功。如果任何锁的获取失败，则调用不会获取任何锁并失败，通常会出现`ER_LOCKING_SERVICE_TIMEOUT`或`ER_LOCKING_SERVICE_DEADLOCK`错误。

+   一个会话可以为相同的锁标识符（命名空间和锁名称组合）获取多个锁。这些锁实例可以是读锁、写锁或两者的混合。

+   在会话中获取的锁通过显式调用释放锁函数释放，或者在会话正常或异常终止时隐式释放。当事务提交或回滚时不会释放锁。

+   在一个会话中，释放给定命名空间的所有锁时会一起释放。

锁定服务提供的接口与`GET_LOCK()`及相关 SQL 函数提供的接口不同（请参见第 14.14 节，“锁定函数”）。例如，`GET_LOCK()`不实现命名空间，并且仅提供排他锁，而不是不同的读锁和写锁。

##### 7.6.9.1.1 锁定服务 C 接口

本节描述如何使用锁定服务的 C 语言接口。要使用函数接口，请参见第 7.6.9.1.2 节，“锁定服务函数接口”有关锁定服务接口的一般特性，请参见第 7.6.9.1 节，“锁定服务”有关插件服务的一般信息，请参见第 7.6.9 节，“MySQL 插件服务”。

使用锁定服务的源文件应包含此头文件：

```sql
#include <mysql/service_locking.h>
```

要获取一个或多个锁，请调用此函数：

```sql
int mysql_acquire_locking_service_locks(MYSQL_THD opaque_thd,
                                        const char* lock_namespace,
                                        const char**lock_names,
                                        size_t lock_num,
                                        enum enum_locking_service_lock_type lock_type,
                                        unsigned long lock_timeout);
```

参数的含义如下：

+   `opaque_thd`: 一个线程句柄。如果指定为`NULL`，则使用当前线程的句柄。

+   `lock_namespace`: 一个以空字符结尾的字符串，表示锁命名空间。

+   `lock_names`: 一个以空字符结尾的字符串数组，提供要获取的锁的名称。

+   `lock_num`: `lock_names`数组中名称的数量。

+   `lock_type`: 锁定模式，可以是`LOCKING_SERVICE_READ`或`LOCKING_SERVICE_WRITE`，分别用于获取读锁或写锁。

+   `lock_timeout`: 等待获取锁的秒数，超时放弃。

要释放为给定命名空间获取的锁，请调用此函数：

```sql
int mysql_release_locking_service_locks(MYSQL_THD opaque_thd,
                                        const char* lock_namespace);
```

参数的含义如下：

+   `opaque_thd`: 一个线程句柄。如果指定为`NULL`，则使用当前线程的句柄。

+   `lock_namespace`: 一个以空字符结尾的字符串，表示锁定命名空间。

通过性能模式可以在 SQL 级别监视由锁定服务获取或等待的锁。详情请参见锁定服务监视。

##### 7.6.9.1.2 锁定服务函数接口

本节描述了如何使用其可加载函数提供的锁定服务接口。要使用 C 语言接口，请参见 Section 7.6.9.1.1, “锁定服务 C 接口”。有关锁定服务接口的一般特性，请参见 Section 7.6.9.1, “锁定服务”。有关可加载函数的一般信息，请参见添加可加载函数。

+   安装或卸载锁定服务函数接口

+   使用锁定服务函数接口

+   锁定服务监视

+   锁定服务接口函数参考

###### 安装或卸载锁定服务函数接口

描述在 Section 7.6.9.1.1, “锁定服务 C 接口”中的锁定服务例程不需要安装，因为它们已经内置在服务器中。但映射到服务例程调用的可加载函数不是这样的：这些函数必须在使用之前安装。本节描述了如何进行安装。有关可加载函数安装的一般信息，请参见 Section 7.7.1, “安装和卸载可加载函数”。

锁定服务函数实现在一个插件库文件中，该文件位于由`plugin_dir`系统变量命名的目录中。文件基本名称为`locking_service`。文件名后缀因平台而异（例如，Unix 和类 Unix 系统为`.so`，Windows 为`.dll`）。

要安装锁定服务函数，请使用`CREATE FUNCTION`语句，并根据需要调整`.so`后缀以适配您的平台：

```sql
CREATE FUNCTION service_get_read_locks RETURNS INT
  SONAME 'locking_service.so';
CREATE FUNCTION service_get_write_locks RETURNS INT
  SONAME 'locking_service.so';
CREATE FUNCTION service_release_locks RETURNS INT
  SONAME 'locking_service.so';
```

如果在复制源服务器上使用这些函数，请在所有副本服务器上安装它们，以避免复制问题。

一旦安装，函数将一直保持安装状态，直到卸载。要删除它们，请使用`DROP FUNCTION`语句：

```sql
DROP FUNCTION service_get_read_locks;
DROP FUNCTION service_get_write_locks;
DROP FUNCTION service_release_locks;
```

###### 使用锁定服务函数接口

在使用锁定服务函数之前，请根据提供的说明安装它们，详情请参阅安装或卸载锁定服务函数接口。

要获取一个或多个读锁，请调用此函数：

```sql
mysql> SELECT service_get_read_locks('mynamespace', 'rlock1', 'rlock2', 10);
+---------------------------------------------------------------+
| service_get_read_locks('mynamespace', 'rlock1', 'rlock2', 10) |
+---------------------------------------------------------------+
|                                                             1 |
+---------------------------------------------------------------+
```

第一个参数是锁定命名空间。最后一个参数是一个整数超时，指示在放弃之前等待多少秒才能获取锁。中间的参数是锁定名称。

对于刚刚显示的示例，该函数获取具有锁定标识符`(mynamespace, rlock1)`和`(mynamespace, rlock2)`的锁。

要获取写锁而不是读锁，请调用此函数：

```sql
mysql> SELECT service_get_write_locks('mynamespace', 'wlock1', 'wlock2', 10);
+----------------------------------------------------------------+
| service_get_write_locks('mynamespace', 'wlock1', 'wlock2', 10) |
+----------------------------------------------------------------+
|                                                              1 |
+----------------------------------------------------------------+
```

在这种情况下，锁定标识符为`(mynamespace, wlock1)`和`(mynamespace, wlock2)`。

要释放命名空间的所有锁，请使用此函数：

```sql
mysql> SELECT service_release_locks('mynamespace');
+--------------------------------------+
| service_release_locks('mynamespace') |
+--------------------------------------+
|                                    1 |
+--------------------------------------+
```

每个锁定函数成功返回非零值。如果函数失败，将会发生错误。例如，由于锁定名称不能为空，会发生以下错误：

```sql
mysql> SELECT service_get_read_locks('mynamespace', '', 10);
ERROR 3131 (42000): Incorrect locking service lock name ''.
```

一个会话可以为相同的锁标识符获取多个锁。只要不同会话没有对标识符的写锁，该会话可以获取任意数量的读锁或写锁。对于标识符的每个锁请求都会获取一个新锁。以下语句获取具有相同标识符的三个写锁，然后获取相同标识符的三个读锁：

```sql
SELECT service_get_write_locks('ns', 'lock1', 'lock1', 'lock1', 0);
SELECT service_get_read_locks('ns', 'lock1', 'lock1', 'lock1', 0);
```

如果此时检查性能模式`metadata_locks`表，您应该发现会话持有六个具有相同`(ns, lock1)`标识符的不同锁。（详情请参阅锁定服务监控。）

因为会话至少持有一个对`(ns, lock1)`的写锁，其他会话无法为其获取读锁或写锁。如果会话仅持有标识符的读锁，其他会话可以获取其读锁，但无法获取写锁。

单个锁获取调用的锁是原子获取的，但原子性不适用于跨调用。因此，对于以下语句，其中每行结果集调用一次 `service_get_write_locks()`，每个单独调用的原子性保持，但对于整个语句不保持：

```sql
SELECT service_get_write_locks('ns', 'lock1', 'lock2', 0) FROM t1 WHERE ... ;
```

注意

因为锁定服务为给定锁标识符的每个成功请求返回一个单独的锁，所以一个语句可能获取大量锁。例如：

```sql
INSERT INTO ... SELECT service_get_write_locks('ns', t1.col_name, 0) FROM t1;
```

这些类型的语句可能会产生某些不良影响。例如，如果语句在中途失败并回滚，则在失败点之前获取的锁仍然存在。如果意图是要求插入的行与获取的锁对应，那么这个意图就无法实现。此外，如果重要的是按照特定顺序授予锁，请注意结果集顺序可能会因优化器选择的执行计划而有所不同。因此，最好限制每个语句对单个锁获取调用。

###### 锁定服务监控

锁定服务是使用 MySQL Server 元数据锁框架实现的，因此您可以通过检查性能模式 `metadata_locks` 表来监视锁定服务获取或等待的锁。

首先，启用元数据锁仪器：

```sql
mysql> UPDATE performance_schema.setup_instruments SET ENABLED = 'YES'
 -> WHERE NAME = 'wait/lock/metadata/sql/mdl';
```

然后获取一些锁并检查 `metadata_locks` 表的内容：

```sql
mysql> SELECT service_get_write_locks('mynamespace', 'lock1', 0);
+----------------------------------------------------+
| service_get_write_locks('mynamespace', 'lock1', 0) |
+----------------------------------------------------+
|                                                  1 |
+----------------------------------------------------+
mysql> SELECT service_get_read_locks('mynamespace', 'lock2', 0);
+---------------------------------------------------+
| service_get_read_locks('mynamespace', 'lock2', 0) |
+---------------------------------------------------+
|                                                 1 |
+---------------------------------------------------+
mysql> SELECT OBJECT_TYPE, OBJECT_SCHEMA, OBJECT_NAME, LOCK_TYPE, LOCK_STATUS
 -> FROM performance_schema.metadata_locks
 -> WHERE OBJECT_TYPE = 'LOCKING SERVICE'\G
*************************** 1\. row ***************************
  OBJECT_TYPE: LOCKING SERVICE
OBJECT_SCHEMA: mynamespace
  OBJECT_NAME: lock1
    LOCK_TYPE: EXCLUSIVE
  LOCK_STATUS: GRANTED
*************************** 2\. row ***************************
  OBJECT_TYPE: LOCKING SERVICE
OBJECT_SCHEMA: mynamespace
  OBJECT_NAME: lock2
    LOCK_TYPE: SHARED
  LOCK_STATUS: GRANTED
```

锁定服务锁的 `OBJECT_TYPE` 值为 `LOCKING SERVICE`。这与例如使用 `GET_LOCK()` 函数获取的锁不同，后者的 `OBJECT_TYPE` 为 `USER LEVEL LOCK`。

锁定命名空间、名称和模式出现在 `OBJECT_SCHEMA`、`OBJECT_NAME` 和 `LOCK_TYPE` 列中。读取和写入锁的 `LOCK_TYPE` 值分别为 `SHARED` 和 `EXCLUSIVE`。

`LOCK_STATUS` 值为 `GRANTED` 表示已获取锁，`PENDING` 表示正在等待锁。如果一个会话持有写锁，另一个会话正在尝试获取具有相同标识符的锁，则可以期望看到 `PENDING`。

###### 锁定服务接口函数参考

锁定服务的 SQL 接口实现了本节中描述的可加载函数。有关使用示例，请参见 使用锁定服务函数接口。

这些函数具有以下特征：

+   返回值为非零表示成功。否则，将发生错误。

+   命名空间和锁名称必须为非`NULL`、非空，并且最大长度为 64 个字符。

+   超时值必须是整数，表示在放弃并产生错误之前等待获取锁的秒数。如果超时为 0，则不会等待，如果无法立即获取锁，则函数会产生错误。

这些锁定服务函数可用：

+   [`service_get_read_locks(*`namespace`*, *`lock_name`*[, *`lock_name`*] ..., *`timeout`*)`](locking-service.html#function_service-get-read-locks)

    使用给定的锁名称在给定的命名空间中获取一个或多个读（共享）锁，在给定的超时值内未获取到锁时会超时报错。

+   [`service_get_write_locks(*`namespace`*, *`lock_name`*[, *`lock_name`*] ..., *`timeout`*)`](locking-service.html#function_service-get-write-locks)

    使用给定的锁名称在给定的命名空间中获取一个或多个写（独占）锁，在给定的超时值内未获取到锁时会超时报错。

+   `service_release_locks(*`namespace`*)`

    对于给定的命名空间，释放当前会话中使用`service_get_read_locks()`和`service_get_write_locks()`获取的所有锁。

    在命名空间中没有锁也不是错误。
