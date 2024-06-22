# 从 SQLite 3.4.2 升级到 3.5.0

> 原文：[`sqlite.com/34to35.html`](https://sqlite.com/34to35.html)

SQLite 3.5.0 版本（2007-09-04）引入了一个新的操作系统接口层，与所有先前版本的 SQLite 都不兼容。此外，一些现有接口已经泛化，可以在进程内所有数据库连接中使用，而不仅限于线程内的所有连接。本文的目的是详细描述 3.5.0 版本的变更，以便 SQLite 先前版本的用户可以评估升级到新版本所需的工作量，如果有的话。

## 1.0 变更概述

在这里提供了 SQLite 3.5.0 版本变更的快速概述。后续章节将更详细地描述这些变更。

1.  操作系统接口层已完全重建：

    1.  未记录的**sqlite3_os_switch()**接口已被移除。

    1.  编译时标志**SQLITE_ENABLE_REDEF_IO**不再起作用。现在，I/O 过程总是可重定义的。

    1.  定义了三个新对象用于指定 I/O 过程：[sqlite3_vfs](https://sqlite.com/c3ref/vfs.html)，[sqlite3_file](https://sqlite.com/c3ref/file.html)，和[sqlite3_io_methods](https://sqlite.com/c3ref/io_methods.html)。

    1.  添加了三个新接口用于创建替代操作系统接口：[sqlite3_vfs_register()](https://sqlite.com/c3ref/vfs_find.html)，[sqlite3_vfs_unregister()](https://sqlite.com/c3ref/vfs_find.html)和[sqlite3_vfs_find()](https://sqlite.com/c3ref/vfs_find.html)。

    1.  添加了一个新接口，用于在创建新数据库连接时提供额外的控制：[sqlite3_open_v2()](https://sqlite.com/c3ref/open.html)。传统接口[sqlite3_open()](https://sqlite.com/c3ref/open.html)和[sqlite3_open16()](https://sqlite.com/c3ref/open.html)仍然得到完全支持。

1.  引入于版本 3.3.0 的可选共享缓存和内存管理功能现在可以在同一进程内的多个线程中使用。以前，这些扩展仅适用于单线程内操作的数据库连接。

    1.  sqlite3_enable_shared_cache() 接口现在适用于进程中的所有线程，而不仅限于调用它的那个线程。

    1.  sqlite3_soft_heap_limit() 接口现在适用于进程中的所有线程，而不仅限于调用它的那个线程。

    1.  sqlite3_release_memory() 接口现在尝试减少所有线程中所有数据库连接的内存使用，而不仅限于调用接口的线程中的连接。

    1.  sqlite3_thread_cleanup() 接口现在成为一个空操作。

1.  不再限制同一数据库连接同时被多个线程使用。现在多个线程可以同时使用同一个数据库连接是安全的。

1.  现在有一个编译时选项，允许应用定义替代的 malloc()/free()实现，而无需修改任何核心 SQLite 代码。

1.  现在有一个编译时选项，允许应用定义替代的互斥体实现，而无需修改任何核心 SQLite 代码。

这些变更中，只有 1a 和 2a 到 2c 是在任何正式意义上的不兼容性。但之前对 SQLite 源代码进行过自定义修改（例如为嵌入式硬件添加自定义操作系统层）的用户可能会发现这些变更影响更大。另一方面，这些变更的一个重要目标是，使得在不同操作系统上定制 SQLite 变得更加容易。

## 2.0 操作系统接口层

如果您的系统定义了 SQLite 的自定义操作系统接口，或者如果您正在使用未记录的 **sqlite3_os_switch()** 接口，那么您需要进行修改以升级到 SQLite 版本 3.5.0。乍一看这可能看起来很痛苦。但是仔细查看后，您可能会发现通过新的 SQLite 接口，您的更改变得更小、更容易理解和管理。您的更改现在可能也会与 SQLite 合并版无缝配合工作。您将不再需要对 SQLite 源代码进行任何更改。所有的更改都可以通过应用程序代码进行，并且您可以链接到标准、未修改的 SQLite 合并版。此外，以前未记录的操作系统接口层现在是 SQLite 的官方支持接口。因此，您可以确信这将是一次性的更改，并且您的新后端将在将来的 SQLite 版本中继续工作。

### 2.1 虚拟文件系统对象

SQLite 的新操作系统接口围绕名为 sqlite3_vfs 的对象构建。这里的"vfs"代表"虚拟文件系统"。sqlite3_vfs 对象基本上是一个包含指向实现 SQLite 需要执行的原始磁盘 I/O 操作的函数指针的结构。在本文中，我们经常将 sqlite3_vfs 对象称为"VFS"。

SQLite 能够同时使用多个 VFS。每个单独的数据库连接只与一个 VFS 相关联。但是，如果您有多个数据库连接，则每个连接可以与不同的 VFS 相关联。

总是有一个默认的 VFS。传统接口 sqlite3_open() 和 sqlite3_open16() 总是使用默认的 VFS。新的创建数据库连接接口 sqlite3_open_v2() 允许您通过名称指定要使用的 VFS。

#### 2.1.1 注册新的 VFS 对象

标准的 Unix 或 Windows 上的 SQLite 构建都带有一个名为 "unix" 或 "win32" 的单一 VFS，适当时使用。这个 VFS 也是默认的。因此，如果你在使用旧的打开函数，一切将继续像以前一样运行。改变的是，现在应用程序可以灵活地添加新的 VFS 模块来实现定制的操作系统层。可以使用 sqlite3_vfs_register() API 来告诉 SQLite 关于一个或多个应用程序定义的 VFS 模块：

> ```sql
> int sqlite3_vfs_register(sqlite3_vfs*, int makeDflt);
> 
> ```

应用程序可以在任何时候调用 sqlite3_vfs_register()，尽管需要在使用之前注册 VFS。第一个参数是应用程序准备的自定义 VFS 对象的指针。第二个参数为 true，以使新的 VFS 成为默认 VFS，这样它将被旧的 sqlite3_open() 和 sqlite3_open16() API 使用。如果新的 VFS 不是默认的，则可能需要使用新的 sqlite3_open_v2() API 来使用它。但请注意，如果新的 VFS 是 SQLite 知道的唯一 VFS（如果 SQLite 没有使用通常的默认 VFS 编译，或者使用 sqlite3_vfs_unregister() 移除了预编译的默认 VFS），则无论 sqlite3_vfs_register() 的 makeDflt 参数如何，新的 VFS 都会自动成为默认 VFS。

标准构建包括默认的 "unix" 或 "win32" VFS。但如果使用了 -DOS_OTHER=1 编译时选项，则 SQLite 将没有默认的 VFS。在这种情况下，应用程序在调用 sqlite3_open() 之前必须至少注册一个 VFS。这是嵌入式应用程序应采用的方法。与修改 SQLite 源代码以插入替代 OS 层相比（在以前版本的 SQLite 中执行的操作），更好的做法是使用 -DOS_OTHER=1 选项编译未修改的 SQLite 源文件（最好是汇总文件），然后在创建任何数据库连接之前调用 sqlite3_vfs_register() 定义底层文件系统的接口。

#### 2.1.2 对 VFS 对象的额外控制

sqlite3_vfs_unregister() API 用于从系统中移除现有的 VFS。

> ```sql
> int sqlite3_vfs_unregister(sqlite3_vfs*);
> 
> ```

sqlite3_vfs_find() API 用于按名称查找特定的 VFS。其原型如下：

> ```sql
> sqlite3_vfs *sqlite3_vfs_find(const char *zVfsName);
> 
> ```

参数是所需 VFS 的符号名称。如果参数是 NULL 指针，则返回默认 VFS。该函数返回指向实现 VFS 的 sqlite3_vfs 对象的指针。如果找不到符合搜索条件的对象，则返回 NULL 指针。

#### 2.1.3 修改现有 VFS

一旦注册了 VFS，就不应再修改它。如果需要改变行为，则应注册一个新的 VFS。应用程序可以使用 sqlite3_vfs_find() 定位旧 VFS，将旧 VFS 复制到新的 sqlite3_vfs 对象中，并对新的 VFS 进行所需的修改，然后注销旧 VFS，并注册新的 VFS 替代其位置。即使在注销后，现有的数据库连接仍将继续使用旧的 VFS，但新的数据库连接将使用新的 VFS。

#### 2.1.4 VFS 对象

VFS 对象是以下结构的实例：

> ```sql
> typedef struct sqlite3_vfs sqlite3_vfs;
> struct sqlite3_vfs {
>   int iVersion;            /* Structure version number */
>   int szOsFile;            /* Size of subclassed sqlite3_file */
>   int mxPathname;          /* Maximum file pathname length */
>   sqlite3_vfs *pNext;      /* Next registered VFS */
>   const char *zName;       /* Name of this virtual file system */
>   void *pAppData;          /* Pointer to application-specific data */
>   int (*xOpen)(sqlite3_vfs*, const char *zName, sqlite3_file*,
>                int flags, int *pOutFlags);
>   int (*xDelete)(sqlite3_vfs*, const char *zName, int syncDir);
>   int (*xAccess)(sqlite3_vfs*, const char *zName, int flags);
>   int (*xGetTempName)(sqlite3_vfs*, char *zOut);
>   int (*xFullPathname)(sqlite3_vfs*, const char *zName, char *zOut);
>   void *(*xDlOpen)(sqlite3_vfs*, const char *zFilename);
>   void (*xDlError)(sqlite3_vfs*, int nByte, char *zErrMsg);
>   void *(*xDlSym)(sqlite3_vfs*,void*, const char *zSymbol);
>   void (*xDlClose)(sqlite3_vfs*, void*);
>   int (*xRandomness)(sqlite3_vfs*, int nByte, char *zOut);
>   int (*xSleep)(sqlite3_vfs*, int microseconds);
>   int (*xCurrentTime)(sqlite3_vfs*, double*);
>   /* New fields may be appended in figure versions.  The iVersion
>   ** value will increment whenever this happens. */
> };
> 
> ```

要创建新的 VFS，应用程序需用适当的值填充此结构的实例，然后调用 sqlite3_vfs_register()。

sqlite3_vfs 的`iVersion`字段应为 1，适用于 SQLite 版本 3.5.0。如果我们必须以某种方式修改 VFS 对象，SQLite 未来的版本中这个数字可能会增加。我们希望这种情况永远不会发生，但做了相应的准备。

`szOsFile`字段是定义打开文件的结构（sqlite3_file 对象）的字节大小。稍后会更详细地描述此对象。这里的重点是，每个 VFS 实现都可以定义自己的 sqlite3_file 对象，包含 VFS 实现需要存储的任何信息。然而，SQLite 需要知道这个对象的大小，以便预先分配足够的空间来容纳它。

`mxPathname`字段是此 VFS 可使用的文件路径名的最大长度。有时 SQLite 必须预分配这么大的缓冲区，因此应尽量保持较小。某些文件系统允许巨大的路径名，但实际上路径名很少超过大约 100 字节。您不必将底层文件系统能处理的最长路径名放在这里。您只需放入希望 SQLite 能够处理的最长路径名。在大多数情况下，几百字节是个不错的值。

`pNext`字段由 SQLite 内部使用。具体来说，SQLite 使用此字段来形成注册 VFS 的链表。

`zName`字段是 VFS 的符号名称。这是在寻找 VFS 时，sqlite3_vfs_find()与之比较的名称。

指针`pAppData`在 SQLite 核心中未被使用。该指针可用于存储 VFS 信息可能需要携带的辅助信息。

sqlite3_vfs 对象的其余字段都存储指向实现基本操作的函数指针。我们称这些为“方法”。第一个方法，xOpen，用于打开底层存储介质上的文件。其结果是一个 sqlite3_file 对象。 sqlite3_file 对象本身定义了其他用于读取、写入和关闭文件的方法。以下详细介绍了其他方法。文件名使用 UTF-8 编码。SQLite 将保证传递给 xOpen() 的 zFilename 字符串是由 xFullPathname() 生成的完整路径名，并且该字符串在调用 xClose() 之前是有效且不变的。因此，如果需要记住文件名， sqlite3_file 可以存储指向文件名的指针。传递给 xOpen() 的 flags 参数是 sqlite3_open_v2() 的 flags 参数的副本。如果使用 sqlite3_open() 或 sqlite3_open16()，则 flags 是 SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE。如果 xOpen() 以只读方式打开文件，则设置 *pOutFlags 来包含 SQLITE_OPEN_READONLY。*pOutFlags 中可能设置其他位。SQLite 还将根据正在打开的对象之一向 xOpen() 调用添加以下标志之一：

+   SQLITE_OPEN_MAIN_DB

+   SQLITE_OPEN_MAIN_JOURNAL

+   SQLITE_OPEN_TEMP_DB

+   SQLITE_OPEN_TEMP_JOURNAL

+   SQLITE_OPEN_TRANSIENT_DB

+   SQLITE_OPEN_SUBJOURNAL

+   SQLITE_OPEN_SUPER_JOURNAL

文件 I/O 实现可以使用对象类型标志来改变它处理文件的方式。例如，一个不关心崩溃恢复或回滚的应用程序可能会将日志文件的打开操作设为无操作。对此日志文件的写操作也将是无操作。任何尝试读取日志文件的操作都会返回 SQLITE_IOERR。或者实现可以识别数据库文件将以随机顺序进行页对齐扇区读写，并相应地设置其 I/O 子系统。SQLite 也可能在 xOpen 方法中添加以下标志之一：

+   SQLITE_OPEN_DELETEONCLOSE

+   SQLITE_OPEN_EXCLUSIVE

SQLITE_OPEN_DELETEONCLOSE 标志表示文件在关闭时应删除。这个标志总是为临时数据库、日志文件以及子日志设置。SQLITE_OPEN_EXCLUSIVE 标志表示文件应该以独占方式打开。除了主数据库文件外，所有文件都设置了这个标志。作为第三个参数传递给 xOpen 的 sqlite3_file 结构由调用者分配。xOpen 只是填充它。调用者为 sqlite3_file 结构分配至少 szOsFile 字节。

SQLITE_OPEN_TEMP_DB 数据库与 SQLITE_OPEN_TRANSIENT_DB 数据库的区别在于：SQLITE_OPEN_TEMP_DB 用于显式声明和命名的 TEMP 表（使用 CREATE TEMP TABLE 语法）或用于通过以空字符串作为文件名打开数据库而创建的临时数据库中的命名表。SQLITE_OPEN_TRANSIENT_DB 包含 SQLite 自动创建的用于评估子查询、ORDER BY 或 GROUP BY 子句的数据库表。TEMP_DB 和 TRANSIENT_DB 数据库都是私有的并且会自动删除。TEMP_DB 数据库持续整个数据库连接的时间。TRANSIENT_DB 数据库只持续单个 SQL 语句的时间。

`xDelete` 方法用于删除文件。文件名在第二个参数中给出，文件名将采用 UTF-8 编码。虚拟文件系统（VFS）必须将文件名转换为底层操作系统期望的字符表示。如果 `syncDir` 参数为真，则 `xDelete` 方法在目录内容发生变化后应该一直等待，直到这些变化同步到磁盘，以确保如果在断电后不会再次“出现”该文件。

`xAccess` 方法用于检查文件的访问权限。文件名将采用 UTF-8 编码。`flags` 参数可以是 SQLITE_ACCESS_EXISTS 用于检查文件是否存在，SQLITE_ACCESS_READWRITE 用于检查文件是否可读写，或者 SQLITE_ACCESS_READ 用于检查文件是否至少可读。第二个参数指定的“文件”可能是一个目录名。

方法 `xGetTempName` 用于计算 SQLite 可以使用的临时文件的名称。名称应写入由第二个参数给出的缓冲区。SQLite 将调整该缓冲区的大小以容纳至少 `mxPathname` 字节。生成的文件名应为 UTF-8 编码。为了避免安全问题，生成的临时文件名应包含足够的随机性，以防止攻击者提前猜测临时文件名。

方法 `xFullPathname` 用于将相对路径名转换为完整路径名。结果的完整路径名写入由第三个参数提供的缓冲区。SQLite 将调整输出缓冲区的大小至少为 `mxPathname` 字节。输入和输出名称都应为 UTF-8 编码。

方法 `xDlOpen`、`xDlError`、`xDlSym` 和 `xDlClose` 用于在运行时访问共享库。如果库使用 SQLITE_OMIT_LOAD_EXTENSION 编译，或者从未使用 sqlite3_enable_load_extension() 接口启用动态扩展加载，则可以省略这些方法（将它们的指针设为零）。方法 `xDlOpen` 打开共享库或 DLL 并返回一个句柄指针。如果打开失败，返回 NULL。打开失败时，可以使用方法 `xDlError` 获取文本错误消息。消息写入第三个参数的 `zErrMsg` 缓冲区，该缓冲区长度至少为 `nByte` 字节。方法 `xDlSym` 返回共享库中符号的指针。符号的名称由第二个参数给出。假定为 UTF-8 编码。如果未找到符号，则返回 NULL 指针。方法 `xDlClose` 关闭共享库。

xRandomness 方法仅被用于初始化 SQLite 内的伪随机数生成器（PRNG），且仅在默认的 VFS 上使用。其他 VFS 上的 xRandomness 方法从未被 SQLite 访问过。xRandomness 程序请求将 nByte 字节的随机性写入 zOut。该程序返回实际获取的随机性字节数。所获取的随机性质量将决定内置 SQLite 函数（如 random() 和 randomblob()）生成的随机性质量。SQLite 还使用其 PRNG 来生成临时文件名。在某些平台上（例如：Windows），SQLite 假设临时文件名是唯一的，即使未实际测试碰撞，因此即使不使用 random() 和 randomblob() 函数，拥有高质量的随机性也很重要。

xSleep 方法用于至少挂起给定微秒数的调用线程。该方法用于实现 sqlite3_sleep() 和 sqlite3_busy_timeout() API。在 sqlite3_sleep() 的情况下，总是使用默认 VFS 的 xSleep 方法。如果底层系统没有微秒级分辨率的休眠功能，则休眠时间应向上舍入。xSleep 返回这个向上舍入的值。

xCurrentTime 方法查找当前时间和日期，并将结果写入由第二个参数提供的双精度浮点指针中。时间和日期以协调世界时（UTC）表示，并且是一个分数化的儒略日数。

#### 2.1.5 打开文件对象

打开文件的结果是一个 sqlite3_file 对象的实例。sqlite3_file 对象是一个抽象基类，定义如下：

> ```sql
> typedef struct sqlite3_file sqlite3_file;
> struct sqlite3_file {
>   const struct sqlite3_io_methods *pMethods;
> };
> 
> ```

每个 VFS 实现都会通过在结尾添加额外的字段来继承 sqlite3_file，以保存 VFS 需要了解的关于打开文件的任何信息。存储的信息并不重要，只要结构的总大小不超过 sqlite3_vfs 对象中记录的 szOsFile 值即可。

sqlite3_io_methods 对象是一个包含指向处理文件读取、写入及其他操作方法的指针的结构。该对象定义如下：

> ```sql
> typedef struct sqlite3_io_methods sqlite3_io_methods;
> struct sqlite3_io_methods {
>   int iVersion;
>   int (*xClose)(sqlite3_file*);
>   int (*xRead)(sqlite3_file*, void*, int iAmt, sqlite3_int64 iOfst);
>   int (*xWrite)(sqlite3_file*, const void*, int iAmt, sqlite3_int64 iOfst);
>   int (*xTruncate)(sqlite3_file*, sqlite3_int64 size);
>   int (*xSync)(sqlite3_file*, int flags);
>   int (*xFileSize)(sqlite3_file*, sqlite3_int64 *pSize);
>   int (*xLock)(sqlite3_file*, int);
>   int (*xUnlock)(sqlite3_file*, int);
>   int (*xCheckReservedLock)(sqlite3_file*);
>   int (*xFileControl)(sqlite3_file*, int op, void *pArg);
>   int (*xSectorSize)(sqlite3_file*);
>   int (*xDeviceCharacteristics)(sqlite3_file*);
>   /* Additional methods may be added in future releases */
> };
> 
> ```

sqlite3_io_methods 的 iVersion 字段是为了应对未来的增强而提供的保险措施。对于 SQLite 版本 3.5，iVersion 值应始终为 1。

xClose 方法关闭文件。调用者释放 sqlite3_file 结构的空间。但是，如果 sqlite3_file 包含指向其他分配的内存或资源的指针，则这些分配应该由 xClose 方法释放。

xRead 方法从文件的字节偏移 iOfst 处开始读取 iAmt 字节。读取的数据存储在第二个参数的指针中。如果成功，xRead 返回 SQLITE_OK，如果因为达到文件末尾而未能读取完整的字节数，则返回 SQLITE_IOERR_SHORT_READ，其他任何错误都返回 SQLITE_IOERR_READ。

xWrite 方法从第二个参数开始的偏移量 iOfst 字节处向文件写入 iAmt 字节数据。如果写入前文件的大小小于 iOfst 字节，则 xWrite 应确保在开始写入之前将文件扩展到 iOfst 字节并填充零。xWrite 持续扩展文件以确保在 xWrite 调用结束时文件的大小至少为 iAmt+iOfst 字节。xWrite 方法在成功时返回 SQLITE_OK。如果由于底层存储介质已满而导致写入无法完成，则返回 SQLITE_FULL。对于其他任何错误则返回 SQLITE_IOERR_WRITE。

xTruncate 方法将文件截断为 nByte 字节长度。如果文件已经少于或等于 nByte 字节长度，则此方法不做任何操作。xTruncate 方法在成功时返回 SQLITE_OK，如果出现任何问题则返回 SQLITE_IOERR_TRUNCATE。

xSync 方法用于强制将先前写入的数据从操作系统缓存刷入非易失性存储器。第二个参数通常是 SQLITE_SYNC_NORMAL。如果第二个参数是 SQLITE_SYNC_FULL，则 xSync 方法应确保数据也已通过磁盘控制器的缓存刷入。SQLITE_SYNC_FULL 参数相当于在 Mac OS X 上的 F_FULLSYNC ioctl()。xSync 方法在成功时返回 SQLITE_OK，如果出现任何问题则返回 SQLITE_IOERR_FSYNC。

xFileSize() 方法确定文件当前的字节大小，并将该值写入 *pSize。成功时返回 SQLITE_OK，如果出现问题则返回 SQLITE_IOERR_FSTAT。

`xLock` 和 `xUnlock` 方法用于设置和清除文件锁。SQLite 支持五种文件锁级别，依次为：

+   SQLITE_LOCK_NONE

+   SQLITE_LOCK_SHARED

+   SQLITE_LOCK_RESERVED

+   SQLITE_LOCK_PENDING

+   SQLITE_LOCK_EXCLUSIVE

实现的底层可以支持这些锁定级别的某个子集，只要满足本段的其他要求。锁定级别被指定为`xLock`和`xUnlock`的第二个参数。`xLock`方法将锁定级别增加到指定的级别或更高。`xUnlock`方法将锁定级别降低到不低于指定的级别。SQLITE_LOCK_NONE 表示文件未锁定。SQLITE_LOCK_SHARED 允许读取文件。多个数据库连接可以同时持有 SQLITE_LOCK_SHARED。SQLITE_LOCK_RESERVED 与 SQLITE_LOCK_SHARED 类似，允许读取文件，但是任何时候只能有一个连接持有保留锁。SQLITE_LOCK_PENDING 也允许读取文件。其他连接也可以继续读取文件，但是不允许其他连接将锁从无锁升级为共享锁。SQLITE_LOCK_EXCLUSIVE 允许写文件。只能有一个连接持有排他锁，并且在一个连接持有排他锁时，其他连接不能持有任何锁（除了“无锁”）。`xLock`在成功时返回 SQLITE_OK，如果无法获得锁则返回 SQLITE_BUSY，如果发生其他问题则返回 SQLITE_IOERR_RDLOCK。`xUnlock`方法在成功时返回 SQLITE_OK，在出现问题时返回 SQLITE_IOERR_UNLOCK。

`xCheckReservedLock()`方法用于检查另一个连接或进程是否当前持有文件的保留、挂起或排他锁。它返回 true 或 false。

xFileControl() 方法是一个通用接口，允许自定义 VFS 实现直接控制打开的文件，使用（新的实验性）sqlite3_file_control() 接口。第二个 "op" 参数是一个整数操作码。第三个参数是一个通用指针，预期指向一个可能包含参数或用于写入返回值的结构体指针。xFileControl() 的潜在用途可能包括启用带有超时的阻塞锁功能，更改锁定策略（例如使用点文件锁），查询锁的状态，或者解除过期锁定。SQLite 核心保留了小于 100 的操作码供自身使用。可以查看小于 100 的 操作码列表。定义自定义 xFileControl 方法的应用程序应使用大于 100 的操作码以避免冲突。

xSectorSize 方法返回底层非易失性介质的 "扇区大小"。一个 "扇区" 被定义为可以在不干扰相邻存储的情况下写入的最小存储单元。在磁盘驱动器上，"扇区大小" 直到最近一直为 512 字节，尽管目前有推动将该值增加到 4KiB。SQLite 需要知道扇区大小，以便可以一次写入一个完整的扇区，从而在写入过程中发生断电时避免损坏相邻存储空间。

xDeviceCharacteristics 方法返回一个整数位向量，定义了底层存储介质可能具有的特殊属性，SQLite 可以利用这些属性来提高性能。允许的返回值是以下值的按位 OR 结果：

+   SQLITE_IOCAP_ATOMIC

+   SQLITE_IOCAP_ATOMIC512

+   SQLITE_IOCAP_ATOMIC1K

+   SQLITE_IOCAP_ATOMIC2K

+   SQLITE_IOCAP_ATOMIC4K

+   SQLITE_IOCAP_ATOMIC8K

+   SQLITE_IOCAP_ATOMIC16K

+   SQLITE_IOCAP_ATOMIC32K

+   SQLITE_IOCAP_ATOMIC64K

+   SQLITE_IOCAP_SAFE_APPEND

+   SQLITE_IOCAP_SEQUENTIAL

SQLITE_IOCAP_ATOMIC 位表示对此设备的所有写入是原子的，即要么整个写入发生，要么根本不发生。其他 SQLITE_IOCAP_ATOMIC*nnn*值指示对齐块的写入是原子的，大小由指定的值决定。SQLITE_IOCAP_SAFE_APPEND 意味着在用新数据扩展文件时，会先写入新数据，然后更新文件大小。因此，如果发生电源故障，则不会出现文件可能被随机扩展的情况。SQLITE_IOCAP_SEQUENTIAL 位表示所有写入按照发出的顺序进行，不会被底层文件系统重新排序。

#### 2.1.6 构建新 VFS 的检查表

前面的段落包含大量信息。为了简化为 SQLite 构建新 VFS 的任务，我们提供以下实现检查表：

1.  定义一个适当的 sqlite3_file 对象的子类。

1.  实现 sqlite3_io_methods 对象所需的方法。

1.  创建一个静态和常量的 sqlite3_io_methods 对象，包含指向前一步方法的指针。

1.  实现 xOpen 方法，打开文件并填充一个 sqlite3_file 对象，包括将 pMethods 设置为指向上一步中的 sqlite3_io_methods 对象。

1.  实现 sqlite3_vfs 需要的其他方法。

1.  定义一个静态（但不是常量的）sqlite3_vfs 结构体，其中包含指向 xOpen 方法和其他方法的指针，并包含 iVersion、szOsFile、mxPathname、zName 和 pAppData 的适当值。

1.  实现一个过程，调用 sqlite3_vfs_register()，并传递上一步中的 sqlite3_vfs 结构体的指针。这个过程可能是实现你的 VFS 的源文件中唯一导出的符号。

在你的应用程序中，在打开任何数据库连接之前，在初始化过程中调用上述最后一步实现的过程。

## 3.0 内存分配子系统

从版本 3.5 开始，SQLite 使用 sqlite3_malloc()、sqlite3_free() 和 sqlite3_realloc() 这些例程来获取所有需要的堆内存。这些例程在 SQLite 的先前版本中已存在，但 SQLite 以前绕过了这些例程，使用自己的内存分配器。在 3.5.0 版本中，这一切都发生了变化。

SQLite 源码树实际上包含多个版本的内存分配器。在大多数构建中，默认的高速版本位于 "mem1.c" 源文件中。但如果启用了 SQLITE_MEMDEBUG 标志，则会使用另一个内存分配器，即 "mem2.c" 源文件。mem2.c 分配器实现了大量钩子来进行错误检查，并模拟用于测试目的的内存分配失败。这两个分配器都使用标准 C 库中的 malloc()/free() 实现。

应用程序不需要使用这两种标准内存分配器中的任何一种。如果 SQLite 是用 SQLITE_OMIT_MEMORY_ALLOCATION 编译的，则不提供 sqlite3_malloc()，sqlite3_realloc()和 sqlite3_free()函数的实现。而是链接到 SQLite 的应用程序必须提供这些函数的自己的实现。应用程序提供的内存分配器不需要使用标准 C 库中的 malloc()/free()实现。例如，嵌入式应用程序可以提供一个替代内存分配器，该分配器使用专门为 SQLite 的固定内存池设置的内存。

实现自己内存分配器的应用程序必须提供通常的三个分配函数 sqlite3_malloc()，sqlite3_realloc()和 sqlite3_free()的实现。它们还必须实现第四个函数：

> ```sql
> int sqlite3_memory_alarm(
>   void(*xCallback)(void *pArg, sqlite3_int64 used, int N),
>   void *pArg,
>   sqlite3_int64 iThreshold
> );
> 
> ```

sqlite3_memory_alarm 例程用于在内存分配事件上注册回调。此例程注册或清除一个回调，当分配的内存量超过 iThreshold 时触发。每次只能注册一个回调。每次调用 sqlite3_memory_alarm()都会覆盖先前的回调。通过将 xCallback 设置为 NULL 指针来禁用回调。

回调函数的参数是 pArg 值，当前正在使用的内存量以及引发回调的分配大小。回调函数可能会调用 sqlite3_free()来释放内存空间。回调函数可能会调用 sqlite3_malloc()或 sqlite3_realloc()，但如果调用，递归调用将不会再调用任何其他回调函数。

sqlite3_soft_heap_limit() 接口通过在软堆限制处注册内存警报，并在警报回调中调用 sqlite3_release_memory() 来工作。应用程序不应尝试使用 sqlite3_memory_alarm() 接口，因为这样做会干扰 sqlite3_soft_heap_limit() 模块。此接口仅暴露给应用程序，以便在编译 SQLite 核心时使用 SQLITE_OMIT_MEMORY_ALLOCATION 时提供其自己的替代实现。

SQLite 内置的内存分配器还提供以下额外的接口：

> ```sql
> sqlite3_int64 sqlite3_memory_used(void);
> sqlite3_int64 sqlite3_memory_highwater(int resetFlag);
> 
> ```

应用程序可以使用这些接口来监视 SQLite 正在使用的内存量。sqlite3_memory_used() 例程返回当前正在使用的内存字节数，而 sqlite3_memory_highwater() 则返回最大瞬时内存使用量。这些例程都不包括与内存分配器相关的开销。这些例程是为应用程序提供的。SQLite 本身从不调用它们。因此，如果应用程序提供自己的内存分配子系统，则可以根据需要省略这些接口。

## 4.0 互斥子系统

SQLite 在“同一时间可以在不同线程中使用不同的 SQLite 数据库连接”这个意义上始终是线程安全的。约束条件是不能在两个独立的线程中同时使用同一个数据库连接。SQLite 版本 3.5.0 放宽了这个约束条件。

为了允许多个线程同时使用同一个数据库连接，SQLite 必须大量使用互斥锁。因此，新增了一个新的互斥子系统。互斥子系统具有以下接口：

> ```sql
> sqlite3_mutex *sqlite3_mutex_alloc(int);
> void sqlite3_mutex_free(sqlite3_mutex*);
> void sqlite3_mutex_enter(sqlite3_mutex*);
> int sqlite3_mutex_try(sqlite3_mutex*);
> void sqlite3_mutex_leave(sqlite3_mutex*);
> 
> ```

尽管这些例程是为了 SQLite 核心的使用而存在的，但如果需要的话，应用程序代码也可以自由使用这些例程。互斥体是一个 sqlite3_mutex 对象。sqlite3_mutex_alloc()例程分配一个新的互斥体对象，并返回一个指向它的指针。对于非递归和递归互斥体，sqlite3_mutex_alloc()的参数应该是 SQLITE_MUTEX_FAST 或 SQLITE_MUTEX_RECURSIVE。如果底层系统不提供非递归互斥体，则可以在这种情况下替代为递归互斥体。sqlite3_mutex_alloc()的参数还可以是一个常数，指定几个静态互斥体之一：

+   SQLITE_MUTEX_STATIC_MAIN

+   SQLITE_MUTEX_STATIC_MEM

+   SQLITE_MUTEX_STATIC_MEM2

+   SQLITE_MUTEX_STATIC_PRNG

+   SQLITE_MUTEX_STATIC_LRU

这些静态互斥体被 SQLite 内部保留，不应由应用程序使用。所有静态互斥体均为非递归。

sqlite3_mutex_free()例程应用于释放非静态互斥体。如果将静态互斥体传递给此例程，则行为是未定义的。

sqlite3_mutex_enter()尝试进入互斥体，如果其他线程已经在使用，则会阻塞。sqlite3_mutex_try()尝试进入互斥体，成功时返回 SQLITE_OK，如果另一个线程已经在使用则返回 SQLITE_BUSY。sqlite3_mutex_leave()退出互斥体。互斥体会一直保持到退出的次数与进入的次数相匹配。如果在一个线程没有持有的互斥体上调用 sqlite3_mutex_leave()，则其行为是未定义的。如果对已释放的互斥体调用任何例程，则其行为是未定义的。

SQLite 源代码提供了多种这些 API 的实现，适用于不同的环境。如果 SQLite 编译时使用了 SQLITE_THREADSAFE=0 标志，则提供了一个仅执行空操作的互斥体实现，速度快但不提供真正的互斥。该实现适用于单线程应用程序或者只在单线程中使用 SQLite 的应用程序。其他真实的互斥体实现是基于底层操作系统提供的。

嵌入式应用程序可能希望提供自己的互斥体实现。如果 SQLite 编译时使用了-DSQLITE_MUTEX_APPDEF=1 编译时标志，则 SQLite 核心不提供互斥体子系统，而必须由链接到 SQLite 的应用程序提供一个与上述接口相匹配的互斥体子系统。

## 5.0 其他接口变更

SQLite 版本 3.5.0 改变了一些 API 的行为，从技术上讲是不兼容的。然而，这些 API 很少被使用，即使被使用，很难想象出会破坏任何东西的情况。这些变更实际上使这些接口更加有用和强大。

在版本 3.5.0 之前，sqlite3_enable_shared_cache() API 用于在单个线程中启用或禁用共享缓存功能 - 即调用 sqlite3_enable_shared_cache() 的同一线程。使用共享缓存的数据库连接限制在它们打开的同一线程中运行。从版本 3.5.0 开始，sqlite3_enable_shared_cache() 应用于进程中所有线程的所有数据库连接。现在，在不同线程中运行的数据库连接可以共享缓存。并且使用共享缓存的数据库连接可以从一个线程迁移到另一个线程。

在版本 3.5.0 之前，sqlite3_soft_heap_limit() 设置了单个线程中所有数据库连接的堆内存使用上限。每个线程可以有自己的堆限制。从版本 3.5.0 开始，整个进程只有一个堆限制。这似乎更为严格（只有一个限制而不是多个），但实际上这是大多数用户想要的。

在版本 3.5.0 之前，sqlite3_release_memory() 函数尝试从调用 sqlite3_release_memory() 的同一线程中的所有数据库连接中回收内存。从版本 3.5.0 开始，sqlite3_release_memory() 函数将尝试从所有线程中的所有数据库连接中回收内存。

## 6.0 总结

从 SQLite 3.4.2 版本到 3.5.0 版本的转换是一个重大变化。SQLite 核心中的每个源代码文件都必须进行修改，有些修改非常广泛。这种变化在 C 接口中引入了一些轻微的不兼容性。但我们认为，从 3.4.2 到 3.5.0 的过渡带来的好处远远超过了移植的痛苦。新的 VFS 层现在已经定义明确且稳定，应该简化未来的定制工作。VFS 层、可分离的内存分配器和互斥子系统使得标准的 SQLite 源代码聚合可以在嵌入式项目中无需修改即可使用，大大简化配置管理。由此产生的系统对高度多线程设计更加宽容。
