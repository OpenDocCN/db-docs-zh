# 一个 SQLite 的异步 I/O 模块

> 原文：[`sqlite.com/asyncvfs.html`](https://sqlite.com/asyncvfs.html)

* * *

**注意：** WAL 模式与 PRAGMA synchronous 设置为 NORMAL 可以避免在事务提交期间调用 fsync()，并仅在 checkpoint 操作期间调用 fsync()。使用 WAL 模式 很大程度上减少了对此异步 I/O 模块的需求。因此，该模块不再受支持。尽管 SQLite 源代码树中仍然存在该源代码，但它不是任何标准构建的一部分，也不再维护。此文档保留供历史参考之用。

* * *

通常情况下，当 SQLite 写入数据库文件时，它会等待写操作完成后才将控制返回给调用应用程序。由于写入文件系统通常比 CPU 绑定操作慢得多，这可能成为性能瓶颈。异步 I/O 后端是一个扩展，使 SQLite 使用后台运行的单独线程处理所有写请求。尽管这不会减少整体系统资源（CPU、磁盘带宽等），但它确实允许 SQLite 即使在写入数据库时也能快速地将控制返回给调用者。

## 1.0 功能

使用异步 I/O，写请求由后台运行的单独线程处理。这意味着发起数据库写入的线程不必等待（有时很慢的）磁盘 I/O 完成。尽管实际上写入是以通常的慢速度在后台进行的，但看起来写入非常快速。

异步 I/O 看起来响应更快，但代价是失去了持久性属性。使用 SQLite 的默认 I/O 后端时，一旦写入完成，您就知道您写入的信息已经安全地存储在磁盘上。但使用异步 I/O 则不然。如果程序崩溃或在异步写线程完成之前发生断电，则数据库更改可能永远不会写入磁盘，数据库的下一个用户可能看不到您的更改。

异步 I/O 会失去持久性，但仍保留 ACID 的其他部分：原子性、一致性和隔离性。许多应用程序在没有持久性的情况下也能正常运行。

### 1.1 工作原理

异步 I/O 的工作原理是创建一个 SQLite VFS 对象，并使用 sqlite3_vfs_register() 将其注册。通过此 VFS 打开的文件进行写入（使用 vfs xWrite() 方法）时，数据并不直接写入磁盘，而是放置在“写入队列”中，由后台线程处理。

当使用异步虚拟文件系统打开文件并通过 vfs xRead() 方法读取时，数据会从磁盘文件和写入队列中读取，因此从 vfs 读者的角度来看，xWrite() 似乎已经完成。

异步 I/O VFS 是通过调用 API 函数 sqlite3async_initialize() 和 sqlite3async_shutdown() 来注册（和取消注册）的。有关详细信息，请参见下文的“编译和使用”部分。

### 1.2 限制

为了获得关于异步 IO 主要思想的经验，此实现故意保持简单。未来可能会添加额外的功能。

例如，如当前实现的那样，如果写入以超过后台写入线程的 I/O 能力的稳定流进行，挂起的写操作队列将无限增长。如果这种情况持续足够长的时间，主机系统可能会耗尽内存。一个更复杂的模块可以跟踪挂起写入的数量，并在挂起的写操作队列增长过大时停止接受新的写入请求。

### 1.3 锁定和并发

使用此实现的异步 IO 的单个进程内的多个连接可以同时访问单个数据库文件。从用户的角度来看，如果所有连接来自单个进程，则使用“普通”SQLite 和使用异步后端的 SQLite 之间的并发性没有区别。

如果启用文件锁定（默认启用），那么来自多个进程的连接也可以读取和写入数据库文件。然而，并发性会如下所示减少：

+   当使用异步 IO 的连接开始数据库事务时，数据库会立即被锁定。然而，锁定直到所有相关的写入队列操作都被刷新到磁盘后才会释放。这意味着（例如），数据库在执行"COMMIT"或"ROLLBACK"命令后可能会保持锁定一段时间。

+   如果使用异步 IO 的应用程序快速执行事务，其他数据库用户可能会被有效地锁定在数据库外。这是因为当执行 BEGIN 时，会立即建立数据库锁定。但是当相应的 COMMIT 或 ROLLBACK 发生时，直到写入队列的相关部分被刷新后才会释放锁定。因此，如果在写入队列被刷新之前的 BEGIN 之后紧跟着 COMMIT，数据库就永远不会解锁，从而阻止其他进程访问数据库。

可以使用 sqlite3async_control() API（见下文）在运行时禁用文件锁定。当使用 NFS 或其他网络文件系统时，避免了与服务器建立文件锁定所需的同步往返可能会提高性能。然而，如果在禁用文件锁定时多个连接试图访问同一数据库文件，则应用程序可能会崩溃，并且数据库可能会损坏。

## 2.0 编译和使用

异步 IO 扩展由一个 C 代码文件（sqlite3async.c）和一个头文件（sqlite3async.h）组成，位于 SQLite 源代码树的[`ext/async/`子文件夹](https://www.sqlite.org/src/dir?name=ext/async)中，定义了应用程序用来激活和控制模块功能的 C API。

要使用异步 IO 扩展，请将 sqlite3async.c 编译为使用 SQLite 的应用程序的一部分。然后使用 sqlite3async.h 中定义的 API 初始化和配置模块。

异步 IO VFS API 在 sqlite3async.h 的注释中有详细描述。通常使用该 API 包括以下步骤：

1.  通过调用 sqlite3async_initialize()函数向 SQLite 注册异步 IO VFS。

1.  创建一个后台线程执行写操作，并调用 sqlite3async_run()。

1.  使用普通的 SQLite API 通过异步 IO VFS 读取和写入数据库。

有关详细信息，请参考[sqlite3async.h 头文件](https://www.sqlite.org/src/finfo?name=ext/async/sqlite3async.h)中的注释。

## 3.0 迁移

目前，异步 IO 扩展与 win32 系统和支持 pthreads 接口的系统兼容，包括 Mac OS X、Linux 和其他各种 Unix 系统。

要将异步 IO 扩展移植到另一个平台，用户必须为新平台实现互斥锁和条件变量原语。目前没有外部可用的接口来实现这一点，但是相对容易地修改 sqlite3async.c 中的代码以包含新平台的并发原语。在 sqlite3async.c 中搜索"PORTING FUNCTIONS"注释字符串以获取详细信息。然后实现以下每个函数的新版本：

> ```sql
> static void async_mutex_enter(int eMutex);
> static void async_mutex_leave(int eMutex);
> static void async_cond_wait(int eCond, int eMutex);
> static void async_cond_signal(int eCond);
> static void async_sched_yield(void);
> 
> ```

上述每个函数所需的功能在 sqlite3async.c 的注释中有描述。
