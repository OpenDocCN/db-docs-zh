# 自定义构建 SQLite 或将 SQLite 移植到新操作系统

> 原文：[`sqlite.com/custombuild.html`](https://sqlite.com/custombuild.html)

## 1.0 介绍

对于大多数应用程序，构建 SQLite 的推荐方法是使用汇编代码文件**sqlite3.c**及其对应的头文件**sqlite3.h**。sqlite3.c 代码文件应该可以在任何 Unix、Windows 系统上编译和运行，而不需要任何修改或特殊的编译器选项。大多数应用程序可以简单地将 sqlite3.c 文件与构成应用程序的其他 C 代码文件一起包含在一起，同时编译它们，就能够得到可工作且配置良好的 SQLite 版本。

> *大多数应用程序在默认配置下和没有特殊的编译时配置下都能够很好地使用 SQLite。大多数开发人员应该能够完全忽略此文档，简单地从汇编构建 SQLite，而无需任何特殊知识或特殊操作。*

然而，高度调优和专门应用程序可能希望或需要用适合应用程序需求的替代实现替换 SQLite 的一些内置系统接口。SQLite 被设计为在编译时很容易重新配置，以满足个别项目的特定需求。SQLite 的编译时配置选项包括：

+   用替代实现替换内置的互斥子系统。

+   完全禁用所有互斥体，用于单线程应用程序。

+   重新配置内存分配子系统，使用标准库中 malloc()实现之外的内存分配器。

+   重新调整内存分配子系统，使其根本不调用 malloc()，而是在 SQLite 启动时使用固定大小的内存缓冲区满足所有内存请求。

+   用另一种设计替换文件系统接口。换句话说，使用完全不同的系统调用集合覆盖 SQLite 用于与磁盘通信的所有系统调用。

+   覆盖其他操作系统接口，如获取 Zulu 或本地时间的调用。

一般来说，SQLite 中有三个单独的子系统可以在编译时进行修改或重写。互斥锁子系统用于串行访问在多线程间共享的 SQLite 资源。内存分配子系统用于分配 SQLite 对象和数据库缓存所需的内存。最后，虚拟文件系统子系统用于提供 SQLite 与底层操作系统以及文件系统之间的可移植接口。我们称这三个子系统为 SQLite 的“接口”子系统。

我们强调大多数应用程序通过使用 SQLite 内置默认实现来达到良好的服务。鼓励开发者尽可能使用默认内置实现，并在编译 SQLite 时不使用任何特殊的编译时选项或参数。然而，一些高度专业化的应用程序可能会从替换或修改这些内置 SQLite 接口子系统中的一个或多个中受益。或者，如果 SQLite 用于非 Unix（Linux 或 Mac OS X）、Windows（Win32 或 WinCE）或 OS/2 操作系统，则 SQLite 内置的接口子系统将不起作用，并且应用程序需要提供适用于目标平台的替代实现。

## 2.0 配置或替换互斥锁子系统

在多线程环境中，SQLite 使用互斥锁（mutexes）来串行访问共享资源。互斥锁子系统仅适用于从多个线程访问 SQLite 的应用程序。对于单线程应用程序或仅从单个线程调用 SQLite 的应用程序，可以通过重新编译并使用以下选项完全禁用互斥锁子系统：

> ```sql
> -DSQLITE_THREADSAFE=0
> 
> ```

互斥锁是廉价的，但并非免费，因此当完全禁用互斥锁时，性能将会更好。生成的库文件大小也会稍微减小。在编译时禁用互斥锁是推荐的优化方式，适用于有意义的应用程序。

当作为共享库使用 SQLite 时，应用程序可以使用 sqlite3_threadsafe() API 测试互斥锁是否已禁用。在运行时链接到 SQLite 并从多个线程使用 SQLite 的应用程序应该检查此 API，以确保它们没有意外地链接到已禁用互斥锁的 SQLite 库版本。当然，单线程应用程序无论 SQLite 是否配置为线程安全都将正常工作，尽管在使用禁用互斥锁的 SQLite 版本时会稍微快一些。

SQLite 互斥锁也可以使用 sqlite3_config() 接口在运行时禁用。要完全禁用所有互斥锁，应用程序可以调用：

> ```sql
> sqlite3_config(SQLITE_CONFIG_SINGLETHREAD);
> 
> ```

在运行时禁用互斥锁不如在编译时禁用有效，因为 SQLite 仍然必须在可能需要互斥锁的每个点上测试一个布尔变量以查看互斥锁是否已启用或禁用。但是，在运行时禁用互斥锁仍具有性能优势。

对于小心管理线程的多线程应用程序，SQLite 支持一个中间互斥体对齐的替代运行时配置，介于不使用任何互斥体和默认情况下互斥体一切的中间位置之间。

> ```sql
> sqlite3_config(SQLITE_CONFIG_MULTITHREAD);
> sqlite3_config(SQLITE_CONFIG_MEMSTATUS, 0);
> 
> ```

这里有两个独立的配置更改，可以同时使用也可以分开使用。SQLITE_CONFIG_MULTITHREAD 设置禁用互斥体，用于序列化对数据库连接对象和预编译语句对象的访问。使用此设置，应用程序可以自由地从多个线程中使用 SQLite，但必须确保没有两个线程同时尝试访问相同的数据库连接或任何与同一数据库连接关联的预编译语句。两个线程可以同时使用 SQLite，但必须使用独立的数据库连接。第二个 SQLITE_CONFIG_MEMSTATUS 设置禁用 SQLite 中跟踪所有未解决内存分配请求总大小的机制。这样可以避免每次调用 sqlite3_malloc()和 sqlite3_free()时使用互斥体，从而节省大量互斥体操作。但禁用内存统计机制的一个后果是，sqlite3_memory_used()、sqlite3_memory_highwater() 和 sqlite3_soft_heap_limit64() 接口将停止工作。

SQLite 在 Unix 上使用 pthreads 实现其互斥功能，并且 SQLite 需要递归互斥。大多数现代 pthread 实现支持递归互斥，但并非所有实现都支持。对于不支持递归互斥的系统，建议应用程序仅在单线程模式下运行。如果这不可能，SQLite 提供了一个替代的递归互斥实现，建立在 pthreads 的标准“快速”互斥之上。只要 pthread_equal() 是原子的，并且处理器具有一致的数据缓存，这种替代递归互斥实现就应该能正常工作。可以通过以下编译器命令行开关启用这种替代递归互斥实现：

> ```sql
> -DSQLITE_HOMEGROWN_RECURSIVE_MUTEX=1
> 
> ```

当将 SQLite 移植到新操作系统时，通常需要完全替换内置的互斥子系统，使用围绕新操作系统的互斥原语构建的替代方案。这可以通过使用以下选项编译 SQLite 来实现：

> ```sql
> -DSQLITE_MUTEX_APPDEF=1
> 
> ```

当 SQLite 使用 SQLITE_MUTEX_APPDEF=1 选项编译时，它完全省略了其 互斥原语函数 的实现。但是 SQLite 库仍然尝试在必要时调用这些函数，因此应用程序必须自行实现 互斥原语函数，并将它们与 SQLite 链接在一起。

## 3.0 配置或替换内存分配子系统

默认情况下，SQLite 从标准库的 malloc()/free() 实现获取对象和缓存所需的内存。还在进行实验性内存分配器的工作，这些分配器在应用程序启动时将所有内存请求满足为单个固定内存缓冲区。有关这些实验性内存分配器的额外信息将在本文档的未来修订版中提供。

SQLite 支持应用程序在运行时通过填充一个 sqlite3_mem_methods 对象的实例来指定替代内存分配器的能力，并使用 sqlite3_config()接口注册新的替代实现。例如：

> ```sql
> sqlite3_config(SQLITE_CONFIG_MALLOC, &my_malloc_implementation);
> 
> ```

SQLite 会复制 sqlite3_mem_methods 对象的内容，因此该对象可以在 sqlite3_config()调用返回后进行修改。

## 4.0 添加新的虚拟文件系统

自 版本 3.5.0（2007-09-04）起，SQLite 支持称为 virtual file system 或“VFS”的接口。这个对象有点名不副实，因为它实际上是整个底层操作系统的接口，而不仅仅是文件系统。

VFS 接口的一个有趣特性是 SQLite 可以同时支持多个 VFS。每个数据库连接在首次使用 sqlite3_open_v2()打开连接时必须选择一个单一的 VFS 供其使用。但如果一个进程包含多个数据库连接，则每个连接可以选择不同的 VFS。可以使用 sqlite3_vfs_register()接口在运行时添加 VFS。

在 Unix、Windows 和 OS/2 上的 SQLite 默认构建包含适合目标平台的 VFS。其他操作系统上的 SQLite 构建默认不包含 VFS，但应用程序可以在运行时注册一个或多个。

## 5.0 将 SQLite 移植到新操作系统

为了将 SQLite 移植到一个新的操作系统 - 一个默认不支持的操作系统 - 应用程序必须提供...

+   工作中的互斥子系统（但仅在多线程时），

+   工作中的内存分配子系统（假设其标准库中缺少 malloc()），和

+   一个工作中的 VFS 实现。

所有这些内容可以在单独的辅助 C 代码文件中提供，然后与存储的 "sqlite3.c" 代码文件链接，以生成适用于目标操作系统的工作 SQLite 构建。除了替代互斥锁和内存分配子系统以及新的 VFS 外，辅助 C 代码文件还应包含以下两个例程的实现：

+   sqlite3_os_init()

+   sqlite3_os_end()

"sqlite3.c" 代码文件包含适用于 Unix、Windows 和 OS/2 的默认 VFS 和 sqlite3_initialize() 及 sqlite3_shutdown() 函数的实现。要在编译 sqlite3.c 时防止加载其中一个默认组件，需要添加以下编译时选项：

> ```sql
> -DSQLITE_OS_OTHER=1
> 
> ```

SQLite 核心会尽早调用 sqlite3_initialize()。辅助 C 代码文件可以包含 sqlite3_initialize() 的实现，注册适当的 VFS，并可能初始化替代互斥系统（如果需要互斥锁）或执行任何所需的内存分配子系统初始化。SQLite 核心从不调用 sqlite3_shutdown()，但它是官方 SQLite API 的一部分，并且在使用 -DSQLITE_OS_OTHER=1 编译时不提供其他方式，因此辅助 C 代码文件应为完整性可能提供它。
