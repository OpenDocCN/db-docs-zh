# 17.17.3 InnoDB 标准监视器和锁监视器输出

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-standard-monitor.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-standard-monitor.html)

锁监视器与标准监视器相同，只是它包含额外的锁信息。启用周期性输出的任一监视器会打开相同的输出流，但如果启用了锁监视器，则该流会包含额外的信息。例如，如果你启用了标准监视器和锁监视器，那么会打开一个单一的输出流。该流会包含额外的锁信息，直到你禁用锁监视器。

使用`SHOW ENGINE INNODB STATUS`语句生成的标准监视器输出在达到 1MB 时受到限制。此限制不适用于写入服务器标准错误输出（`stderr`）的输出。

示例标准监视器输出：

```sql
mysql> SHOW ENGINE INNODB STATUS\G
*************************** 1\. row ***************************
  Type: InnoDB
  Name:
Status:
=====================================
2018-04-12 15:14:08 0x7f971c063700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 4 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 15 srv_active, 0 srv_shutdown, 1122 srv_idle
srv_master_thread log flush and writes: 0
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 24
OS WAIT ARRAY INFO: signal count 24
RW-shared spins 4, rounds 8, OS waits 4
RW-excl spins 2, rounds 60, OS waits 2
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 2.00 RW-shared, 30.00 RW-excl, 0.00 RW-sx
------------------------
LATEST FOREIGN KEY ERROR
------------------------
2018-04-12 14:57:24 0x7f97a9c91700 Transaction:
TRANSACTION 7717, ACTIVE 0 sec inserting
mysql tables in use 1, locked 1
4 lock struct(s), heap size 1136, 3 row lock(s), undo log entries 3
MySQL thread id 8, OS thread handle 140289365317376, query id 14 localhost root update
INSERT INTO child VALUES (NULL, 1), (NULL, 2), (NULL, 3), (NULL, 4), (NULL, 5), (NULL, 6)
Foreign key constraint fails for table `test`.`child`:
,
  CONSTRAINT `child_ibfk_1` FOREIGN KEY (`parent_id`) REFERENCES `parent` (`id`) ON DELETE
  CASCADE ON UPDATE CASCADE
Trying to add in child table, in index par_ind tuple:
DATA TUPLE: 2 fields;
 0: len 4; hex 80000003; asc     ;;
 1: len 4; hex 80000003; asc     ;;

But in parent table `test`.`parent`, in index PRIMARY,
the closest match we can find is record:
PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000004; asc     ;;
 1: len 6; hex 000000001e19; asc       ;;
 2: len 7; hex 81000001110137; asc       7;;

------------
TRANSACTIONS
------------
Trx id counter 7748
Purge done for trx's n:o < 7747 undo n:o < 0 state: running but idle
History list length 19
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421764459790000, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 7747, ACTIVE 23 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 9, OS thread handle 140286987249408, query id 51 localhost root updating
DELETE FROM t WHERE i = 1
------- TRX HAS BEEN WAITING 23 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 4 page no 4 n bits 72 index GEN_CLUST_INDEX of table `test`.`t`
trx id 7747 lock_mode X waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 6; hex 000000000202; asc       ;;
 1: len 6; hex 000000001e41; asc      A;;
 2: len 7; hex 820000008b0110; asc        ;;
 3: len 4; hex 80000001; asc     ;;

------------------
TABLE LOCK table `test`.`t` trx id 7747 lock mode IX
RECORD LOCKS space id 4 page no 4 n bits 72 index GEN_CLUST_INDEX of table `test`.`t`
trx id 7747 lock_mode X waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 6; hex 000000000202; asc       ;;
 1: len 6; hex 000000001e41; asc      A;;
 2: len 7; hex 820000008b0110; asc        ;;
 3: len 4; hex 80000001; asc     ;;

--------
FILE I/O
--------
I/O thread 0 state: waiting for i/o request (insert buffer thread)
I/O thread 1 state: waiting for i/o request (log thread)
I/O thread 2 state: waiting for i/o request (read thread)
I/O thread 3 state: waiting for i/o request (read thread)
I/O thread 4 state: waiting for i/o request (read thread)
I/O thread 5 state: waiting for i/o request (read thread)
I/O thread 6 state: waiting for i/o request (write thread)
I/O thread 7 state: waiting for i/o request (write thread)
I/O thread 8 state: waiting for i/o request (write thread)
I/O thread 9 state: waiting for i/o request (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
833 OS file reads, 605 OS file writes, 208 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 553253, node heap has 0 buffer(s)
Hash table size 553253, node heap has 1 buffer(s)
Hash table size 553253, node heap has 3 buffer(s)
Hash table size 553253, node heap has 0 buffer(s)
Hash table size 553253, node heap has 0 buffer(s)
Hash table size 553253, node heap has 0 buffer(s)
Hash table size 553253, node heap has 0 buffer(s)
Hash table size 553253, node heap has 0 buffer(s)
0.00 hash searches/s, 0.00 non-hash searches/s
---
LOG
---
Log sequence number          19643450
Log buffer assigned up to    19643450
Log buffer completed up to   19643450
Log written up to            19643450
Log flushed up to            19643450
Added dirty pages up to      19643450
Pages flushed up to          19643450
Last checkpoint at           19643450
129 log i/o's done, 0.00 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 2198863872
Dictionary memory allocated 409606
Buffer pool size   131072
Free buffers       130095
Database pages     973
Old database pages 0
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 810, created 163, written 404
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 973, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   65536
Free buffers       65043
Database pages     491
Old database pages 0
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 411, created 80, written 210
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 491, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 1
Buffer pool size   65536
Free buffers       65052
Database pages     482
Old database pages 0
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 399, created 83, written 194
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 482, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=5772, Main thread ID=140286437054208 , state=sleeping
Number of rows inserted 57, updated 354, deleted 4, read 4421
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================
```

#### 标准监视器输出部分

有关标准监视器报告的每个指标的描述，请参考 Oracle Enterprise Manager for MySQL Database User's Guide 中的指标章节。

+   `状态`

    本部分显示时间戳、监视器名称以及每秒平均值基于的秒数。秒数是当前时间与上次打印`InnoDB`监视器输出之间的经过时间。

+   `后台线程`

    `srv_master_thread`行显示了主后台线程执行的工作。

+   `信号量`

    本部分报告了等待信号量的线程以及线程需要自旋或在互斥量或读写锁信号量上等待的次数的统计信息。大量等待信号量的线程可能是由于磁盘 I/O 或`InnoDB`内部争用问题引起的。争用可能是由于查询的重度并行性或操作系统线程调度问题引起的。在这种情况下，将`innodb_thread_concurrency`系统变量设置为小于默认值可能有所帮助。`每次等待的自旋轮数`行显示了每个 OS 等待互斥量的自旋锁轮数。

    互斥量指标由`SHOW ENGINE INNODB MUTEX`报告。

+   `最新外键错误`

    本部分提供了关于最近的外键约束错误的信息。如果没有发生此类错误，则不会显示。内容包括失败的语句以及有关失败约束的信息以及引用和引用表的信息。

+   `最新检测到的死锁`

    这一部分提供了关于最近死锁的信息。如果没有发生死锁，则不会显示。内容显示了涉及的事务、每个事务试图执行的语句、它们拥有和需要的锁，以及`InnoDB`决定回滚以打破死锁的事务。本节中报告的锁模式在第 17.7.1 节，“InnoDB 锁定”中有解释。

+   `事务`

    如果此部分报告锁等待，您的应用程序可能存在锁争用。输出还可以帮助跟踪事务死锁的原因。

+   `文件 I/O`

    这一部分提供了关于`InnoDB`用于执行各种类型 I/O 的线程的信息。其中前几个专门用于一般的`InnoDB`处理。内容还显示了待处理 I/O 操作的信息和 I/O 性能的统计信息。

    这些线程的数量由`innodb_read_io_threads`和`innodb_write_io_threads`参数控制。请参阅第 17.14 节，“InnoDB 启动选项和系统变量”。

+   `插入缓冲区和自适应哈希索引`

    这一部分显示了`InnoDB`插入缓冲区（也称为更改缓冲区）和自适应哈希索引的状态。

    有关更多信息，请参阅第 17.5.2 节，“更改缓冲区”和第 17.5.3 节，“自适应哈希索引”。

+   `日志`

    这一部分显示了关于`InnoDB`日志的信息。内容包括当前日志序列号、日志已刷新到磁盘的位置，以及`InnoDB`上次进行检查点的位置。 (参见第 17.11.3 节，“InnoDB 检查点”.) 本节还显示了有关待写入和写入性能统计的信息。

+   `缓冲池和内存`

    这一部分为您提供了关于读取和写入页面的统计信息。您可以根据这些数字计算出您的查询当前正在执行多少数据文件 I/O 操作。

    有关缓冲池统计描述，请参阅使用 InnoDB 标准监视器监视缓冲池。有关缓冲池操作的其他信息，请参阅第 17.5.1 节，“缓冲池”。

+   `行操作`

    这一部分展示了主线程正在做的事情，包括每种类型的行操作的数量和性能率。
