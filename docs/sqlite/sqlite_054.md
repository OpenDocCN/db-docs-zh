# 1\. 概述

> 原文：[`sqlite.com/threadsafe.html`](https://sqlite.com/threadsafe.html)

SQLite 支持三种不同的线程模式：

1.  **单线程**。在此模式下，所有互斥锁都被禁用，SQLite 不能在多个线程中同时使用。

1.  **多线程**。在此模式下，SQLite 可以安全地被多个线程使用，前提是单个数据库连接或任何派生自数据库连接的对象，如 prepared statement，不会同时在两个或更多线程中使用。

1.  **串行**。在串行模式下，可以安全地从多个线程调用影响或使用任何 SQLite 数据库连接或任何此类数据库连接派生的对象的 API 调用。对单个对象的影响与如果这些 API 调用都从单个线程以相同顺序进行的情况相同。名称“串行”来自于 SQLite 使用互斥锁来串行化对每个对象的访问。

线程模式可以在编译时（从源代码编译 SQLite 库时）、启动时（初始化打算使用 SQLite 的应用程序时）或运行时（创建新的 SQLite 数据库连接时）选择。一般来说，运行时会覆盖启动时，启动时会覆盖编译时。但是，一旦选择了单线程模式，就无法再覆盖。

默认模式是串行的。

# 2\. 编译时选择线程模式

使用 SQLITE_THREADSAFE 编译时参数来选择线程模式。如果没有 SQLITE_THREADSAFE 编译时参数存在，则使用串行模式。可以通过-DSQLITE_THREADSAFE=1 显式表示这一点。使用-DSQLITE_THREADSAFE=0 时，线程模式为单线程。使用-DSQLITE_THREADSAFE=2 时，线程模式为多线程。

sqlite3_threadsafe() 接口的返回值是编译时设置的 SQLITE_THREADSAFE 的值。它不反映通过 sqlite3_config()接口或通过作为 sqlite3_open_v2()第三个参数给出的标志在运行时所做的线程模式更改。

如果在编译时选择了单线程模式，则从构建中省略了关键的互斥逻辑，因此无法在启动时或运行时启用多线程或串行模式。

# 3\. 线程模式的启动时间选择

假设编译时的线程模式不是单线程，则可以在初始化期间使用 sqlite3_config()接口更改线程模式。SQLITE_CONFIG_SINGLETHREAD 选项将 SQLite 置于单线程模式，SQLITE_CONFIG_MULTITHREAD 选项设置多线程模式，SQLITE_CONFIG_SERIALIZED 选项设置串行化模式。

# 4\. 运行时选择线程模式

如果在编译时或启动时未选择单线程模式，则可以将单个数据库连接创建为多线程或串行化模式。无法将单个数据库连接降级为单线程模式，如果编译时或启动时模式为单线程，则也无法将单个数据库连接升级为其他模式。

数据库连接的线程模式由作为第三个参数给出的标志决定，这些标志是传递给 sqlite3_open_v2()的。SQLITE_OPEN_NOMUTEX 标志导致数据库连接处于多线程模式，而 SQLITE_OPEN_FULLMUTEX 标志则将连接置于串行化模式。如果未指定任何标志，或者使用 sqlite3_open()或 sqlite3_open16()而非 sqlite3_open_v2()，则使用由编译时和启动时设置确定的默认模式。
