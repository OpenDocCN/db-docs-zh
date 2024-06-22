# 介绍

> 原文：[`sqlite.com/arch.html`](https://sqlite.com/arch.html)

本文描述了 SQLite 库的架构。这里的信息对于那些想要理解或修改 SQLite 内部工作方式的人来说是有用的。

附近的图表显示了 SQLite 的主要组件及其相互作用。下文解释了各个组件的角色。

# 概述

SQLite 的工作原理是将 SQL 文本编译成 字节码，然后使用虚拟机运行该字节码。

sqlite3_prepare_v2() 和相关接口作为编译器，将 SQL 文本转换成字节码。sqlite3_stmt 对象是一个容器，用于实现单个 SQL 语句的单个字节码程序。sqlite3_step() 接口将字节码程序传递给虚拟机，并运行程序，直到完成、形成要返回的结果行、遇到致命错误或被中断。

# 接口

C 语言接口的大部分内容可以在源文件 [main.c](https://sqlite.org/src/file/src/main.c)、[legacy.c](https://sqlite.org/src/file/src/legacy.c) 和 [vdbeapi.c](https://sqlite.org/src/file/src/vdbeapi.c) 中找到，尽管一些例程分散在其他文件中，这些文件可以访问具有文件作用域的数据结构。sqlite3_get_table() 例程实现在 [table.c](https://sqlite.org/src/file/src/table.c) 中。sqlite3_mprintf() 例程位于 [printf.c](https://sqlite.org/src/file/src/printf.c)。sqlite3_complete() 接口位于 [complete.c](https://sqlite.org/src/file/src/complete.c)。TCL 接口 由 [tclsqlite.c](https://sqlite.org/src/file/src/tclsqlite.c) 实现。

为了避免名称冲突，SQLite 库中的所有外部符号都以前缀**sqlite3**开头。那些用于外部使用的符号（换句话说，形成 SQLite API 的那些符号）会添加下划线，因此以**sqlite3_**开头。扩展 API 有时会在下划线之前添加扩展名；例如：**sqlite3rbu_**或**sqlite3session_**。

# 分词器

当要评估包含 SQL 语句的字符串时，首先将其发送到分词器。分词器将 SQL 文本分解为标记，并逐个将这些标记传递给解析器。分词器是在文件<file>tokenize.c.</file>中手工编码的。

注意，在这种设计中，分词器调用解析器。熟悉 YACC 和 BISON 的人可能习惯于反过来做 —— 让解析器调用分词器。然而，让分词器调用解析器更好，因为它可以做到线程安全并且运行更快。

# 解析器

解析器根据其上下文为标记赋予意义。SQLite 的解析器是使用 Lemon 解析生成器生成的。Lemon 完成与 YACC/BISON 相同的工作，但使用了不同的输入语法，减少了错误。Lemon 还生成了可重入和线程安全的解析器。此外，Lemon 定义了非终结符析构函数的概念，以便在遇到语法错误时不会泄漏内存。驱动 Lemon 并定义 SQLite 理解的 SQL 语言的语法文件可以在[parse.y](https://sqlite.org/src/file/src/parse.y)中找到。

由于 Lemon 是一种通常不会出现在开发机器上的程序，Lemon 的完整源代码（只有一个 C 文件）包含在 SQLite 分发的"tool"子目录中。

# 代码生成器

解析器将标记组装成解析树后，代码生成器开始运行，分析解析树并生成执行 SQL 语句工作的字节码。准备好的语句对象是这些字节码的容器。代码生成器中有许多文件，包括：[attach.c](https://sqlite.org/src/file/src/attach.c), [auth.c](https://sqlite.org/src/file/src/auth.c), [build.c](https://sqlite.org/src/file/src/build.c), [delete.c](https://sqlite.org/src/file/src/delete.c), [expr.c](https://sqlite.org/src/file/src/expr.c), [insert.c](https://sqlite.org/src/file/src/insert.c), [pragma.c](https://sqlite.org/src/file/src/pragma.c), [select.c](https://sqlite.org/src/file/src/select.c), [trigger.c](https://sqlite.org/src/file/src/trigger.c), [update.c](https://sqlite.org/src/file/src/update.c), [vacuum.c](https://sqlite.org/src/file/src/vacuum.c), [where.c](https://sqlite.org/src/file/src/where.c), [wherecode.c](https://sqlite.org/src/file/src/wherecode.c) 和 [whereexpr.c](https://sqlite.org/src/file/src/whereexpr.c)。这些文件是大部分复杂操作的地方。[expr.c](https://sqlite.org/src/file/src/expr.c) 处理表达式的代码生成。**where*.c** 处理 SELECT、UPDATE 和 DELETE 语句的 WHERE 子句的代码生成。文件[attach.c](https://sqlite.org/src/file/src/attach.c), [delete.c](https://sqlite.org/src/file/src/delete.c), [insert.c](https://sqlite.org/src/file/src/insert.c), [select.c](https://sqlite.org/src/file/src/select.c), [trigger.c](https://sqlite.org/src/file/src/trigger.c) 和 [update.c](https://sqlite.org/src/file/src/update.c), [vacuum.c](https://sqlite.org/src/file/src/vacuum.c) 处理同名 SQL 语句的代码生成。（这些文件在必要时调用[expr.c](https://sqlite.org/src/file/src/expr.c) 和 [where.c](https://sqlite.org/src/file/src/where.c) 中的例程。）所有其他 SQL 语句均编码在 [build.c](https://sqlite.org/src/file/src/build.c) 中。文件[auth.c](https://sqlite.org/src/file/src/auth.c) 实现了 sqlite3_set_authorizer()的功能。

代码生成器，特别是**where*.c**和[select.c](https://sqlite.org/src/file/src/select.c)中的逻辑，有时被称为查询规划器。对于任何特定的 SQL 语句，可能有数百、数千或数百万种不同的算法来计算答案。查询规划器是一个人工智能，旨在从这些数百万选择中选择最佳算法。

# 字节码引擎

代码生成器创建的字节码程序由虚拟机运行。

虚拟机本身完全包含在一个单独的源文件[vdbe.c](https://sqlite.org/src/file/src/vdbe.c)中。[vdbe.h](https://sqlite.org/src/file/src/vdbe.h)头文件定义了虚拟机与 SQLite 库的其余部分之间的接口，而[vdbeInt.h](https://sqlite.org/src/file/src/vdbeInt.h)定义了虚拟机本身的结构和接口，这些接口是私有的。其他各种**vdbe*.c**文件是虚拟机的辅助程序。[vdbeaux.c](https://sqlite.org/src/file/src/vdbeaux.c)文件包含虚拟机使用的实用程序，以及库的其余部分用来构建 VM 程序的接口模块。[vdbeapi.c](https://sqlite.org/src/file/src/vdbeapi.c)文件包含虚拟机的外部接口，例如 sqlite3_bind_int()和 sqlite3_step()。个别值（字符串、整数、浮点数和 BLOBs）存储在一个名为“Mem”的内部对象中，该对象由[vdbemem.c](https://sqlite.org/src/file/src/vdbemem.c)实现。

SQLite 使用回调到 C 语言例程来实现 SQL 函数。即使是内置的 SQL 函数也是这样实现的。大多数内置的 SQL 函数（例如：[abs()](https://sqlite.org/src/file/src/func.c#abs)，[count()](https://sqlite.org/src/file/src/func.c#count)，[substr()](https://sqlite.org/src/file/src/func.c#substr) 等）可以在 [func.c](https://sqlite.org/src/file/src/func.c) 源文件中找到。日期和时间转换函数可以在 [date.c](https://sqlite.org/src/file/src/date.c) 中找到。一些函数，如 [coalesce()](https://sqlite.org/src/file/src/func.c#coalesce) 和 [typeof()](https://sqlite.org/src/file/src/func.c#typeof)，是由代码生成器直接实现为字节码。

# B 树

SQLite 数据库使用在 [btree.c](https://sqlite.org/src/file/src/btree.c) 源文件中找到的 B 树实现在磁盘上维护。每个表和数据库中的每个索引都使用单独的 B 树。所有 B 树都存储在同一个磁盘文件中。文件格式的详细信息在 [file format](https://sqlite.org/fileformat2.html) 中已经稳定和明确定义，并保证向前兼容。

与 B 树子系统和 SQLite 库的其余部分的接口由头文件 [btree.h](https://sqlite.org/src/file/src/btree.h) 定义。

# 页缓存

B 树模块以固定大小的页面从磁盘请求信息。默认的 [page_size](https://sqlite.org/pragma.html#pragma_page_size) 是 4096 字节，但可以是 512 到 65536 字节之间的任何二的幂。页缓存负责读取、写入和缓存这些页面。页缓存还提供回滚和原子提交抽象，并负责锁定数据库文件。B 树驱动程序从页缓存请求特定页面，并在需要修改页面、提交或回滚更改时通知页缓存。页缓存处理确保请求快速、安全和高效处理的所有混乱细节。

主页缓存的实现在 [pager.c](https://sqlite.org/src/file/src/pager.c) 文件中。WAL 模式 的逻辑在单独的 [wal.c](https://sqlite.org/src/file/src/wal.c) 文件中。内存缓存由 [pcache.c](https://sqlite.org/src/file/src/pcache.c) 和 [pcache1.c](https://sqlite.org/src/file/src/pcache1.c) 文件实现。页面缓存子系统与 SQLite 其余部分之间的接口由头文件 [pager.h](https://sqlite.org/src/file/src/pager.h) 定义。

# 操作系统接口

为了在不同操作系统上提供可移植性，SQLite 使用一个称为 VFS 的抽象对象。每个 VFS 提供了用于打开、读取、写入和关闭磁盘文件的方法，以及用于执行其他 OS 特定任务，如查找当前时间或获取用于初始化内置伪随机数生成器的随机性。SQLite 目前为 unix（在 [os_unix.c](https://sqlite.org/src/file/src/os_unix.c) 文件中）和 Windows（在 [os_win.c](https://sqlite.org/src/file/src/os_win.c) 文件中）提供了 VFS。

# 实用工具

内存分配、大小写不敏感字符串比较例程、便携式文本转数字例程以及其他实用工具位于 [util.c](https://sqlite.org/src/file/src/util.c) 中。解析器使用的符号表由 [hash.c](https://sqlite.org/src/file/src/hash.c) 中的哈希表维护。[utf.c](https://sqlite.org/src/file/src/utf.c) 源文件包含 Unicode 转换子例程。SQLite 在 [printf.c](https://sqlite.org/src/file/src/printf.c) 中有自己的私有实现的 printf()（带有一些扩展），以及在 [random.c](https://sqlite.org/src/file/src/random.c) 中有自己的伪随机数生成器（PRNG）。

# 测试代码

"src/" 文件夹中以 **test** 开头的文件仅用于测试，不包含在库的标准构建中。
