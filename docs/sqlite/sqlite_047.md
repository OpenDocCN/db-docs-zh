# 在线备份 API。

> 原文：[`sqlite.com/c3ref/backup_finish.html`](https://sqlite.com/c3ref/backup_finish.html)
> 
> ```sql
> sqlite3_backup *sqlite3_backup_init(
>   sqlite3 *pDest,                        /* Destination database handle */
>   const char *zDestName,                 /* Destination database name */
>   sqlite3 *pSource,                      /* Source database handle */
>   const char *zSourceName                /* Source database name */
> );
> int sqlite3_backup_step(sqlite3_backup *p, int nPage);
> int sqlite3_backup_finish(sqlite3_backup *p);
> int sqlite3_backup_remaining(sqlite3_backup *p);
> int sqlite3_backup_pagecount(sqlite3_backup *p);
> 
> ```

备份 API 将一个数据库的内容复制到另一个数据库。它既用于创建数据库备份，也用于将内存数据库复制到或从持久文件中。

参见：使用 SQLite 在线备份 API

SQLite 在整个备份操作期间保持对目标数据库文件的写事务打开。源数据库仅在读取时被读锁定；它并非在整个备份操作期间连续锁定。因此，在备份进行期间，可以在活动源数据库上执行备份，而不会阻止其他数据库连接在备份进行时读取或写入源数据库。

要执行备份操作：

1.  **sqlite3_backup_init()**被调用一次以初始化备份，

1.  **sqlite3_backup_step()**被调用一次或多次来在两个数据库之间传输数据，最终

1.  **sqlite3_backup_finish()**被调用以释放与备份操作相关的所有资源。

每次成功调用 sqlite3_backup_init()都应该有一次调用 sqlite3_backup_finish()。

**sqlite3_backup_init()**

sqlite3_backup_init(D,N,S,M)中的 D 和 N 参数是与目标数据库关联的数据库连接和数据库名称。数据库名称分别是主数据库的"main"，临时数据库的"temp"，或者在 ATTACH 语句中 AS 关键字后指定的附加数据库的名称。传递给 sqlite3_backup_init(D,N,S,M)的 S 和 M 参数标识源数据库的数据库连接和数据库名称。源和目标数据库连接（参数 S 和 D）必须不同，否则 sqlite3_backup_init(D,N,S,M)将因错误而失败。

调用 sqlite3_backup_init()将失败，返回 NULL，如果目标数据库上已经有一个读或读写事务打开。

如果在 sqlite3_backup_init(D,N,S,M)内发生错误，则返回 NULL，并且错误代码和错误消息被存储在目标数据库连接 D 中。可以使用 sqlite3_errcode()，sqlite3_errmsg()和/或 sqlite3_errmsg16()函数检索到调用 sqlite3_backup_init()失败时的错误代码和消息。成功调用 sqlite3_backup_init()将返回指向一个 sqlite3_backup 对象的指针。sqlite3_backup 对象可与 sqlite3_backup_step()和 sqlite3_backup_finish()函数一起用于执行指定的备份操作。

**sqlite3_backup_step()**

函数 sqlite3_backup_step(B,N)将在由 sqlite3_backup 对象 B 指定的源和目标数据库之间复制最多 N 页。如果 N 为负数，则复制所有剩余的源页面。如果 sqlite3_backup_step(B,N)成功地复制了 N 页，并且还有更多页面要复制，则函数返回 SQLITE_OK。如果 sqlite3_backup_step(B,N)成功完成了从源到目标的所有页面的复制，则返回 SQLITE_DONE。如果在运行 sqlite3_backup_step(B,N)时发生错误，则返回一个错误代码。除了 SQLITE_OK 和 SQLITE_DONE 之外，调用 sqlite3_backup_step()可能会返回 SQLITE_READONLY、SQLITE_NOMEM、SQLITE_BUSY、SQLITE_LOCKED 或者 SQLITE_IOERR_XXX 扩展错误代码。

如果 sqlite3_backup_step()返回 SQLITE_READONLY，

1.  目标数据库以只读方式打开，或者

1.  目标数据库正在使用预写式日志记录，并且目标和源页面大小不同，或者

1.  目标数据库是一个内存数据库，并且目标和源页面大小不同。

如果 sqlite3_backup_step()无法获取所需的文件系统锁，则会调用繁忙处理器函数(如果已指定)。如果繁忙处理器在锁可用之前返回非零，则将 SQLITE_BUSY 返回给调用者。在这种情况下，稍后可以重试调用 sqlite3_backup_step()。如果在调用 sqlite3_backup_step()时使用源数据库连接向源数据库写入数据，则会立即返回 SQLITE_LOCKED。同样，在这种情况下，稍后可以重试调用 sqlite3_backup_step()。如果返回 SQLITE_IOERR_XXX、SQLITE_NOMEM 或 SQLITE_READONLY，则没有重试调用 sqlite3_backup_step()的必要。这些错误被认为是致命的。应用程序必须接受备份操作失败，并将备份操作句柄传递给 sqlite3_backup_finish()以释放相关资源。

第一次调用 sqlite3_backup_step() 会获得目标文件的独占锁。直到调用 sqlite3_backup_finish() 或备份操作完成并且 sqlite3_backup_step() 返回 SQLITE_DONE 之前，独占锁才会释放。每次调用 sqlite3_backup_step() 都会在源数据库上获取一个在 sqlite3_backup_step() 调用期间持续的 共享锁。因为在调用 sqlite3_backup_step() 之间源数据库没有被锁定，所以在备份过程中可能会修改源数据库。如果源数据库被外部进程或者使用备份操作所使用的其他数据库连接修改，则下一次调用 sqlite3_backup_step() 将自动重新启动备份。如果源数据库由使用与备份操作相同的数据库连接修改，则备份数据库将同时自动更新。

**sqlite3_backup_finish()**

当 sqlite3_backup_step() 返回 SQLITE_DONE 或者应用程序希望放弃备份操作时，应用程序应该通过将其传递给 sqlite3_backup_finish() 来销毁 sqlite3_backup。sqlite3_backup_finish() 接口释放与 sqlite3_backup 对象关联的所有资源。如果 sqlite3_backup_step() 尚未返回 SQLITE_DONE，则目标数据库上的任何活动写事务都将被回滚。 sqlite3_backup 对象将无效，并且在调用 sqlite3_backup_finish() 后不能再使用。

如果在同一个 sqlite3_backup 对象的任何先前 sqlite3_backup_step() 调用期间发生内存不足条件或 IO 错误，则 sqlite3_backup_finish() 返回相应的 错误代码，如果没有发生 sqlite3_backup_step() 错误，则返回 SQLITE_OK。

sqlite3_backup_step() 返回 SQLITE_BUSY 或者 SQLITE_LOCKED 不是永久错误，不影响 sqlite3_backup_finish() 的返回值。

**sqlite3_backup_remaining() 和 sqlite3_backup_pagecount()**

sqlite3_backup_remaining() 函数返回在最近一次 sqlite3_backup_step() 完成时仍需备份的页面数。sqlite3_backup_pagecount() 函数返回在最近一次 sqlite3_backup_step() 完成时源数据库中的总页面数。这些函数返回的值仅由 sqlite3_backup_step() 更新。如果源数据库以改变源数据库的大小或剩余页面数的方式被修改，sqlite3_backup_pagecount() 和 sqlite3_backup_remaining() 的输出在下一次 sqlite3_backup_step() 之后才会反映这些更改。

**数据库句柄的并发使用**

在备份操作进行或正在初始化时，应用程序可以使用源数据库连接进行其他用途。如果 SQLite 编译和配置以支持线程安全的数据库连接，则源数据库连接可以在其他线程中并发使用。

但是，应用程序必须保证在调用 sqlite3_backup_init() 之后且在对应的 sqlite3_backup_finish() 调用之前，目标数据库连接不能传递给任何其他 API（由任何线程）。SQLite 目前不会检查应用程序是否错误地访问目标数据库连接，因此不会报告错误代码，但操作可能仍会出现故障。在备份正在进行时使用目标数据库连接也可能导致互斥体死锁。

如果在共享缓存模式下运行，则应用程序必须保证在备份运行时不访问目标数据库使用的共享缓存。实际上，这意味着应用程序必须保证备份到的磁盘文件不被进程内的任何连接访问，而不仅仅是传递给 sqlite3_backup_init() 的特定连接。

sqlite3_backup 对象本身是部分线程安全的。多个线程可以安全地同时调用多个并发的 sqlite3_backup_step()。然而，sqlite3_backup_remaining() 和 sqlite3_backup_pagecount() API 在严格意义上并不是线程安全的。如果它们在另一个线程正在调用 sqlite3_backup_step() 的同时被调用，可能会返回无效值。

另请参阅对象、常量和函数列表。
