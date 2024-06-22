# 使用 SQLite 在线备份 API

> 原文：[`sqlite.com/backup.html`](https://sqlite.com/backup.html)

历史上，使用以下方法创建 SQLite 数据库的备份（副本）：

1.  使用 SQLite API（即 shell 工具）在数据库文件上建立共享锁。

1.  使用外部工具（例如 Unix 的'cp'实用程序或 DOS 的'copy'命令）复制数据库文件。

1.  在第一步获取的数据库文件上放弃共享锁。

这一过程在许多场景中表现良好，通常非常快速。然而，这种技术有以下缺点：

+   在创建备份时，任何希望向数据库文件写入的数据库客户端必须等待共享锁被放弃。

+   不能用于从或向内存数据库复制数据。

+   如果在复制数据库文件时发生断电或操作系统故障，系统恢复后备份数据库可能会损坏。

创建在线备份 API 旨在解决这些问题。在线备份 API 允许将一个数据库的内容复制到另一个数据库文件中，替换目标数据库的任何原始内容。复制操作可以是增量的，这样源数据库在复制期间不需要锁定，只在实际读取时需要短暂锁定。这使得在创建在线数据库备份时，其他数据库用户可以继续而无需过多延迟。

完成备份调用序列的效果是使目标成为源数据库在复制开始时的位完全相同的副本。 （目标变成“快照”。）

在这里文档化了在线备份 API。本页的其余部分包含两个 C 语言示例，说明了 API 的常见用途及其讨论。阅读这些示例并不代替阅读 API 文档！

更新：在 SQLite 版本 3.27.0（2019-02-07）中引入的 VACUUM INTO 命令可以作为备份 API 的替代方法。

## 示例 1：加载和保存内存数据库

```sql
/*
** This function is used to load the contents of a database file on disk 
** into the "main" database of open database connection pInMemory, or
** to save the current contents of the database opened by pInMemory into
** a database file on disk. pInMemory is probably an in-memory database, 
** but this function will also work fine if it is not.
**
** Parameter zFilename points to a nul-terminated string containing the
** name of the database file on disk to load from or save to. If parameter
** isSave is non-zero, then the contents of the file zFilename are 
** overwritten with the contents of the database opened by pInMemory. If
** parameter isSave is zero, then the contents of the database opened by
** pInMemory are replaced by data loaded from the file zFilename.
**
** If the operation is successful, SQLITE_OK is returned. Otherwise, if
** an error occurs, an SQLite error code is returned.
*/
int loadOrSaveDb(sqlite3 *pInMemory, const char *zFilename, int isSave){
  int rc;                   /* Function return code */
  sqlite3 *pFile;           /* Database connection opened on zFilename */
  sqlite3_backup *pBackup;  /* Backup object used to copy data */
  sqlite3 *pTo;             /* Database to copy to (pFile or pInMemory) */
  sqlite3 *pFrom;           /* Database to copy from (pFile or pInMemory) */

  /* Open the database file identified by zFilename. Exit early if this fails
  ** for any reason. */
  rc = sqlite3_open(zFilename, &pFile);
  if( rc==SQLITE_OK ){

    /* If this is a 'load' operation (isSave==0), then data is copied
    ** from the database file just opened to database pInMemory. 
    ** Otherwise, if this is a 'save' operation (isSave==1), then data
    ** is copied from pInMemory to pFile.  Set the variables pFrom and
    ** pTo accordingly. */
    pFrom = (isSave ? pInMemory : pFile);
    pTo   = (isSave ? pFile     : pInMemory);

    /* Set up the backup procedure to copy from the "main" database of 
    ** connection pFile to the main database of connection pInMemory.
    ** If something goes wrong, pBackup will be set to NULL and an error
    ** code and message left in connection pTo.
    **
    ** If the backup object is successfully created, call backup_step()
    ** to copy data from pFile to pInMemory. Then call backup_finish()
    ** to release resources associated with the pBackup object.  If an
    ** error occurred, then an error code and message will be left in
    ** connection pTo. If no error occurred, then the error code belonging
    ** to pTo is set to SQLITE_OK.
    */
    pBackup = sqlite3_backup_init(pTo, "main", pFrom, "main");
    if( pBackup ){
      (void)sqlite3_backup_step(pBackup, -1);
      (void)sqlite3_backup_finish(pBackup);
    }
    rc = sqlite3_errcode(pTo);
  }

  /* Close the database connection opened on database file zFilename
  ** and return the result of this function. */
  (void)sqlite3_close(pFile);
  return rc;
}

```

右侧的 C 函数演示了备份 API 的最简单和最常见的用法之一：将内存数据库的内容加载并保存到磁盘上的文件中。在此示例中，备份 API 的使用如下：

1.  调用函数 sqlite3_backup_init()以创建一个 sqlite3_backup 对象，用于在两个数据库之间复制数据（从文件到内存数据库，或反之）。

1.  调用带有参数`-1`的函数 sqlite3_backup_step()以将整个源数据库复制到目标数据库。

1.  调用 sqlite3_backup_finish()来清理由 sqlite3_backup_init()分配的资源。

**错误处理**

如果三个主要的备份 API 例程中发生错误，则错误代码和消息会附加到目标数据库连接。此外，如果 sqlite3_backup_step()遇到错误，则 sqlite3_backup_step()调用本身和随后的 sqlite3_backup_finish()调用都将返回错误代码。因此，sqlite3_backup_finish()调用不会覆盖 sqlite3_backup_step()存储在目标数据库连接中的错误代码。此功能在示例代码中用于减少所需的错误处理量。忽略 sqlite3_backup_step()和 sqlite3_backup_finish()调用的返回值，并从目标数据库连接中收集指示复制操作成功或失败的错误代码。

**可能的增强**

至少可以通过两种方式增强此函数的实现：

1.  在数据库文件 zFilename 上无法获取锁（SQLITE_BUSY 错误）可能需要处理，

1.  数据库 pInMemory 和 zFilename 的页面大小不同的情况可能需要更好地处理。

由于数据库 zFilename 是磁盘上的文件，可能会被另一个进程外部访问。这意味着当调用 sqlite3_backup_step()试图从中读取或写入数据时，可能会无法获得所需的文件锁。如果这种情况发生，该实现将失败，并立即返回 SQLITE_BUSY。解决方法是在打开它时使用 sqlite3_busy_handler()或 sqlite3_busy_timeout()为数据库连接 pFile 注册一个繁忙处理程序回调或超时。如果立即无法获取所需的锁，sqlite3_backup_step()会像 sqlite3_step()或 sqlite3_exec()一样使用任何已注册的繁忙处理程序回调或超时。

通常，在目标内容被覆盖之前，源数据库和目标数据库的页面大小是否不同并不重要。目标数据库的页面大小仅作为备份操作的一部分进行更改。唯一的例外是目标数据库恰好是内存数据库的情况。在这种情况下，如果备份操作开始时页面大小不同，则操作将以 SQLITE_READONLY 错误失败。不幸的是，在使用函数 loadOrSaveDb() 将数据库图像从文件加载到内存数据库时，可能会发生这种情况。

然而，如果内存数据库 pInMemory 刚刚被打开（因此完全为空）并且在传递给函数 loadOrSaveDb() 之前，那么仍然可以使用 SQLite 的 "PRAGMA page_size" 命令来更改其页面大小。函数 loadOrSaveDb() 可以检测到这种情况，并尝试将内存数据库的页面大小设置为数据库 zFilename 的页面大小，然后调用在线备份 API 函数。

## 示例 2：正在运行的数据库的在线备份

```sql
/*
** Perform an online backup of database pDb to the database file named
** by zFilename. This function copies 5 database pages from pDb to
** zFilename, then unlocks pDb and sleeps for 250 ms, then repeats the
** process until the entire database is backed up.
** 
** The third argument passed to this function must be a pointer to a progress
** function. After each set of 5 pages is backed up, the progress function
** is invoked with two integer parameters: the number of pages left to
** copy, and the total number of pages in the source file. This information
** may be used, for example, to update a GUI progress bar.
**
** While this function is running, another thread may use the database pDb, or
** another process may access the underlying database file via a separate 
** connection.
**
** If the backup process is successfully completed, SQLITE_OK is returned.
** Otherwise, if an error occurs, an SQLite error code is returned.
*/
int backupDb(
  sqlite3 *pDb,               /* Database to back up */
  const char *zFilename,      /* Name of file to back up to */
  void(*xProgress)(int, int)  /* Progress function to invoke */ 
){
  int rc;                     /* Function return code */
  sqlite3 *pFile;             /* Database connection opened on zFilename */
  sqlite3_backup *pBackup;    /* Backup handle used to copy data */

  /* Open the database file identified by zFilename. */
  rc = sqlite3_open(zFilename, &pFile);
  if( rc==SQLITE_OK ){

    /* Open the sqlite3_backup object used to accomplish the transfer */
    pBackup = sqlite3_backup_init(pFile, "main", pDb, "main");
    if( pBackup ){

      /* Each iteration of this loop copies 5 database pages from database
      ** pDb to the backup database. If the return value of backup_step()
      ** indicates that there are still further pages to copy, sleep for
      ** 250 ms before repeating. */
      do {
        rc = sqlite3_backup_step(pBackup, 5);
        xProgress(
            sqlite3_backup_remaining(pBackup),
            sqlite3_backup_pagecount(pBackup)
        );
        if( rc==SQLITE_OK || rc==SQLITE_BUSY || rc==SQLITE_LOCKED ){
          sqlite3_sleep(250);
        }
      } while( rc==SQLITE_OK || rc==SQLITE_BUSY || rc==SQLITE_LOCKED );

      /* Release resources allocated by backup_init(). */
      (void)sqlite3_backup_finish(pBackup);
    }
    rc = sqlite3_errcode(pFile);
  }

  /* Close the database connection opened on database file zFilename
  ** and return the result of this function. */
  (void)sqlite3_close(pFile);
  return rc;
}

```

前面示例中呈现的函数通过一次调用 sqlite3_backup_step() 将整个源数据库复制。这需要在操作期间保持对源数据库文件的读锁，阻止任何其他数据库用户写入数据库。它还通过复制过程中持有与数据库 pInMemory 关联的互斥体，阻止任何其他线程使用它。本节中的 C 函数旨在由后台线程或进程调用，用于创建在线数据库备份，通过以下方法避免这些问题：

1.  调用函数 sqlite3_backup_init() 创建一个 sqlite3_backup 对象，用于将数据库 pDb 的数据复制到由 zFilename 标识的备份数据库文件中。

1.  函数 sqlite3_backup_step() 被调用，参数为 5，将数据库 pDb 的 5 个页面复制到备份数据库（文件 zFilename）。

1.  如果数据库 pDb 还有更多页面需要复制，则函数将休眠 250 毫秒（使用 sqlite3_sleep() 实用工具），然后返回到步骤 2。

1.  调用函数 sqlite3_backup_finish() 清理由 sqlite3_backup_init() 分配的资源。

**文件和数据库连接锁定**

在上述步骤 3 中的 250 毫秒休眠期间，不会持有数据库文件的读锁，并且不会持有与 pDb 关联的互斥体。这允许其他线程使用 数据库连接 pDb 和其他连接来写入底层数据库。

如果另一个线程或进程在此函数休眠时向源数据库写入，则 SQLite 会检测到并且通常会在下次调用 sqlite3_backup_step() 时重新启动备份过程。这条规则有一个例外：如果源数据库不是内存数据库，并且写入是在与备份操作相同的进程内部使用相同的数据库句柄（pDb）执行的，则目标数据库（使用连接 pFile 打开的数据库）会自动与源数据库一起更新。然后可以在 sqlite3_sleep() 调用返回后继续备份过程，就好像什么都没有发生一样。

无论备份过程是否因中途对源数据库的写入而重新启动，用户可以确信，当备份操作完成时，备份数据库包含的是原始数据库的一致且最新的快照。然而：

+   通过内存源数据库写入，或者通过使用与 pDb 不同的数据库连接的外部进程或线程写入基于文件的源数据库，比使用 pDb 写入基于文件的源数据库显著更昂贵（因为在前两种情况下必须重新启动整个备份操作）。

+   如果备份过程频繁重新启动，它可能永远不会完成，并且 backupDb() 函数可能永远不会返回。

**backup_remaining() 和 backup_pagecount()**

backupDb() 函数使用 sqlite3_backup_remaining() 和 sqlite3_backup_pagecount() 函数通过用户提供的 xProgress() 回调报告其进度。函数 sqlite3_backup_remaining() 返回剩余要复制的页数，sqlite3_backup_pagecount() 返回源数据库中的总页数（在这种情况下是由 pDb 打开的数据库）。因此，可以计算出进程的完成百分比：

完成度 = 100% * (pagecount() - remaining()) / pagecount()

sqlite3_backup_remaining() 和 sqlite3_backup_pagecount() API 报告的是由前一个对 sqlite3_backup_step() 的调用存储的值，它们实际上并不检查源数据库文件。这意味着如果在 sqlite3_backup_step() 返回后但在 sqlite3_backup_remaining() 和 sqlite3_backup_pagecount() 返回的值被使用之前，源数据库被另一个线程或进程写入，那么这些值可能是技术上不正确的。通常情况下这不是问题。
