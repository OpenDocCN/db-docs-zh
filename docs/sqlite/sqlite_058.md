# 使用 sqlite3_unlock_notify() API

> 原文：[`sqlite.com/unlock_notify.html`](https://sqlite.com/unlock_notify.html)

```sql
/* This example uses the pthreads API */
#include <pthread.h>

/*
** A pointer to an instance of this structure is passed as the user-context
** pointer when registering for an unlock-notify callback.
*/
typedef struct UnlockNotification UnlockNotification;
struct UnlockNotification {
  int fired;                         /* True after unlock event has occurred */
  pthread_cond_t cond;               /* Condition variable to wait on */
  pthread_mutex_t mutex;             /* Mutex to protect structure */
};

/*
** This function is an unlock-notify callback registered with SQLite.
*/
static void unlock_notify_cb(void **apArg, int nArg){
  int i;
  for(i=0; i<nArg; i++){
    UnlockNotification *p = (UnlockNotification *)apArg[i];
    pthread_mutex_lock(&p->mutex);
    p->fired = 1;
    pthread_cond_signal(&p->cond);
    pthread_mutex_unlock(&p->mutex);
  }
}

/*
** This function assumes that an SQLite API call (either sqlite3_prepare_v2() 
** or sqlite3_step()) has just returned SQLITE_LOCKED. The argument is the
** associated database connection.
**
** This function calls sqlite3_unlock_notify() to register for an 
** unlock-notify callback, then blocks until that callback is delivered 
** and returns SQLITE_OK. The caller should then retry the failed operation.
**
** Or, if sqlite3_unlock_notify() indicates that to block would deadlock 
** the system, then this function returns SQLITE_LOCKED immediately. In 
** this case the caller should not retry the operation and should roll 
** back the current transaction (if any).
*/
static int wait_for_unlock_notify(sqlite3 *db){
  int rc;
  UnlockNotification un;

  /* Initialize the UnlockNotification structure. */
  un.fired = 0;
  pthread_mutex_init(&un.mutex, 0);
  pthread_cond_init(&un.cond, 0);

  /* Register for an unlock-notify callback. */
  rc = sqlite3_unlock_notify(db, unlock_notify_cb, (void *)&un);
  assert( rc==SQLITE_LOCKED || rc==SQLITE_OK );

  /* The call to sqlite3_unlock_notify() always returns either SQLITE_LOCKED 
 ** or SQLITE_OK. 
 **
 ** If SQLITE_LOCKED was returned, then the system is deadlocked. In this
 ** case this function needs to return SQLITE_LOCKED to the caller so 
 ** that the current transaction can be rolled back. Otherwise, block
 ** until the unlock-notify callback is invoked, then return SQLITE_OK.
  */
  if( rc==SQLITE_OK ){
    pthread_mutex_lock(&un.mutex);
    if( !un.fired ){
      pthread_cond_wait(&un.cond, &un.mutex);
    }
    pthread_mutex_unlock(&un.mutex);
  }

  /* Destroy the mutex and condition variables. */
  pthread_cond_destroy(&un.cond);
  pthread_mutex_destroy(&un.mutex);

  return rc;
}

/*
** This function is a wrapper around the SQLite function sqlite3_step().
** It functions in the same way as step(), except that if a required
** shared-cache lock cannot be obtained, this function may block waiting for
** the lock to become available. In this scenario the normal API step()
** function always returns SQLITE_LOCKED.
**
** If this function returns SQLITE_LOCKED, the caller should rollback
** the current transaction (if any) and try again later. Otherwise, the
** system may become deadlocked.
*/
int sqlite3_blocking_step(sqlite3_stmt *pStmt){
  int rc;
  while( SQLITE_LOCKED==(rc = sqlite3_step(pStmt)) ){
    rc = wait_for_unlock_notify(sqlite3_db_handle(pStmt));
    if( rc!=SQLITE_OK ) break;
    sqlite3_reset(pStmt);
  }
  return rc;
}

/*
** This function is a wrapper around the SQLite function sqlite3_prepare_v2().
** It functions in the same way as prepare_v2(), except that if a required
** shared-cache lock cannot be obtained, this function may block waiting for
** the lock to become available. In this scenario the normal API prepare_v2()
** function always returns SQLITE_LOCKED.
**
** If this function returns SQLITE_LOCKED, the caller should rollback
** the current transaction (if any) and try again later. Otherwise, the
** system may become deadlocked.
*/
int sqlite3_blocking_prepare_v2(
  sqlite3 *db,              /* Database handle. */
  const char *zSql,         /* UTF-8 encoded SQL statement. */
  int nSql,                 /* Length of zSql in bytes. */
  sqlite3_stmt **ppStmt,    /* OUT: A pointer to the prepared statement */
  const char **pz           /* OUT: End of parsed string */
){
  int rc;
  while( SQLITE_LOCKED==(rc = sqlite3_prepare_v2(db, zSql, nSql, ppStmt, pz)) ){
    rc = wait_for_unlock_notify(db);
    if( rc!=SQLITE_OK ) break;
  }
  return rc;
}

```

当两个或多个连接以共享缓存模式访问同一数据库时，对各个表的读写（共享和排他）锁用于确保并发执行的事务保持隔离。在写入表之前，必须获得该表的写（排他）锁。在读取之前，必须获得读（共享）锁。连接在结束事务时释放所有持有的表锁。如果连接无法获得所需的锁，则对 sqlite3_step() 的调用将返回 SQLITE_LOCKED。

虽然不常见，调用 sqlite3_prepare() 或 sqlite3_prepare_v2() 也可能因为无法在每个附加数据库的 sqlite_schema 表 上获取读锁而返回 SQLITE_LOCKED。这些 API 需要读取 sqlite_schema 表中包含的模式数据，以便将 SQL 语句编译为 sqlite3_stmt* 对象。

本文介绍了一种使用 SQLite sqlite3_unlock_notify() 接口的技术，使得对 sqlite3_step() 和 sqlite3_prepare_v2() 的调用在所需锁可用之前将阻塞，而不是立即返回 SQLITE_LOCKED。如果左侧提供的 sqlite3_blocking_step() 或 sqlite3_blocking_prepare_v2() 函数返回 SQLITE_LOCKED，则表明阻塞会导致系统死锁。

如果库编译时定义了预处理符号 SQLITE_ENABLE_UNLOCK_NOTIFY，则可以使用 sqlite3_unlock_notify() API。这篇文章没有替代阅读完整的 API 文档！

sqlite3_unlock_notify() 接口适用于每个 数据库连接 都有专门线程的系统。实现中没有任何东西阻止单个线程运行多个数据库连接。然而，sqlite3_unlock_notify() 接口一次只能处理一个连接，因此这里呈现的锁定解决逻辑仅适用于每个线程的单个数据库连接。

**sqlite3_unlock_notify() API**

在调用 sqlite3_step() 或 sqlite3_prepare_v2() 后返回 SQLITE_LOCKED 之后，可以调用 sqlite3_unlock_notify() API 注册解锁通知回调函数。解锁通知回调函数由 SQLite 在持有阻止调用 sqlite3_step() 或 sqlite3_prepare_v2() 成功的数据库连接完成其事务并释放所有锁之后调用。例如，如果调用 sqlite3_step() 是尝试从表 X 读取数据，并且某些其他连接 Y 正在持有表 X 的写锁，则 sqlite3_step() 将返回 SQLITE_LOCKED。如果随后调用了 sqlite3_unlock_notify()，则在连接 Y 的事务结束后将调用解锁通知回调函数。在本例中解锁通知回调函数正在等待的连接，即连接 Y，被称为“阻塞连接”。

如果尝试向数据库表写入的 sqlite3_step() 调用返回 SQLITE_LOCKED，则可能有多个其他连接正在持有所涉及的数据库表的读锁。在这种情况下，SQLite 简单地任意选择其中一个其他连接，并在该连接的事务完成时发出解锁通知回调。无论 sqlite3_step() 调用被一个还是多个连接阻止，当发出相应的解锁通知回调时，并不保证所需的锁可用，只是可能可用。

当发出解锁通知回调时，它是从与阻止连接相关联的 sqlite3_step()（或 sqlite3_close()）调用中发出的。从解锁通知回调中调用任何 sqlite3_XXX() API 函数都是非法的。预期的使用是解锁通知回调将信号发送到其他等待的线程或安排稍后执行的某些操作。

sqlite3_blocking_step() 函数使用的算法如下所示：

1.  对所提供的语句句柄调用 sqlite3_step()。如果调用返回的结果不是 SQLITE_LOCKED，则将此值返回给调用者。否则，继续。

1.  在与提供的语句句柄相关联的数据库连接句柄上调用 sqlite3_unlock_notify() 注册解锁通知回调函数。如果调用 unlock_notify() 返回 SQLITE_LOCKED，则将此值返回给调用者。

1.  阻塞直到另一个线程调用解锁通知回调。

1.  对语句句柄调用 sqlite3_reset()。由于 SQLITE_LOCKED 错误只可能发生在第一次调用 sqlite3_step() 时（不可能一次调用 sqlite3_step() 返回 SQLITE_ROW，然后下一个返回 SQLITE_LOCKED），因此此时可以重置语句句柄而不会影响调用者视角下的查询结果。如果此时不调用 sqlite3_reset()，则下一次调用 sqlite3_step() 将返回 SQLITE_MISUSE。

1.  返回到步骤 1。

sqlite3_blocking_prepare_v2() 函数使用的算法类似，不同之处在于省略了步骤 4（重置语句句柄）。

**写入器饥饿**

多个连接可以同时持有读锁。如果许多线程正在获取重叠的读锁，可能至少有一个线程始终持有读锁。然后，等待写锁的表将永远等待。这种情况称为 "写入器饥饿"。

SQLite 帮助应用程序避免写入器饥饿。在尝试获取表的写锁失败后（因为一个或多个其他连接持有读锁），在共享缓存上所有尝试打开新事务的尝试都失败，直到以下之一为真：

+   当前的写入程序结束其事务，或者

+   共享缓存上的打开读事务数减少到零。

尝试打开新读事务失败，将 SQLITE_LOCKED 返回给调用方。如果调用方随后调用 sqlite3_unlock_notify() 来注册解锁通知回调函数，则阻塞连接是当前在共享缓存上有打开写事务的连接。这样可以防止写入器饥饿，因为如果不能打开新的读事务，并且假设所有现有的读事务最终会结束，写入器最终将有机会获取所需的写锁。

**pthread API**

在 wait_for_unlock_notify() 调用 sqlite3_unlock_notify() 时，阻塞连接可能已经完成了阻止 sqlite3_step() 或 sqlite3_prepare_v2() 调用成功的事务。在这种情况下，解锁通知回调函数会立即调用，即在 sqlite3_unlock_notify() 返回之前。或者，在调用 sqlite3_unlock_notify() 之后，但在线程开始等待异步信号之前，解锁通知回调函数可能由第二个线程调用。

如何处理这种潜在的竞争条件取决于应用程序使用的线程和同步原语接口。本示例使用 pthreads，这是现代类 UNIX 系统（包括 Linux）提供的接口。

pthreads 接口提供了 pthread_cond_wait() 函数。此函数允许调用者同时释放互斥锁并开始等待异步信号。使用此函数、"fired" 标志和互斥锁，可以消除上述描述的竞争条件：

当调用解锁通知回调函数时，可能在调用 sqlite3_unlock_notify() 的线程开始等待异步信号之前，执行以下操作：

1.  获得互斥锁。

1.  将 "fired" 标志设置为 true。

1.  尝试向等待的线程发出信号。

1.  释放互斥锁。

当 wait_for_unlock_notify() 线程准备好开始等待解锁通知回调函数到达时，它：

1.  获取互斥锁。

1.  检查是否设置了“fired”标志。如果是，则解锁通知回调已经被调用。释放互斥锁并继续。

1.  原子性地释放互斥锁并开始等待异步信号。当信号到达时，继续执行。

这种方式，当`wait_for_unlock_notify()`线程开始阻塞时，无论解锁通知回调函数是否已被调用或正在被调用都无关紧要。

**可能的增强**

本文中的代码至少可以通过两种方式改进：

+   它可以管理线程优先级。

+   它可以处理在删除表或索引时可能发生的 SQLITE_LOCKED 特殊情况。

即使 sqlite3_unlock_notify()函数只允许调用者指定单个用户上下文指针，解锁通知回调函数会传递一个这样的上下文指针数组。这是因为如果阻塞连接结束其事务时，如果有多个注册了调用同一个 C 函数的解锁通知，上下文指针会被整理成数组并发出单个回调。如果每个线程被分配一个优先级，那么不像这个实现所做的随机顺序信号线程，高优先级线程可以在低优先级线程之前被通知。

如果执行了“DROP TABLE”或“DROP INDEX” SQL 命令，并且同一数据库连接当前有一个或多个正在执行的 SELECT 语句，则会返回 SQLITE_LOCKED。如果在这种情况下调用 sqlite3_unlock_notify()，则指定的回调函数将立即被调用。重新尝试“DROP TABLE”或“DROP INDEX”语句将返回另一个 SQLITE_LOCKED 错误。在左侧显示的 sqlite3_blocking_step()实现中，这可能导致无限循环。

调用方可以通过扩展错误代码区分这种特殊的“DROP TABLE|INDEX”情况和其他情况。在适当调用 sqlite3_unlock_notify()的情况下，扩展错误代码是 SQLITE_LOCKED_SHAREDCACHE。否则，在“DROP TABLE|INDEX”情况下，仅仅是普通的 SQLITE_LOCKED。另一种解决方法可能是限制单个查询的重试次数（比如 100 次）。尽管这可能不如人意，但这种情况并不经常发生。
