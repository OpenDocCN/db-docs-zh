# 1\. 合并与单独源文件

> 原文：[`sqlite.com/howtocompile.html`](https://sqlite.com/howtocompile.html)

<title>如何编译 SQLite</title>

## 概述

SQLite 是 ANSI-C 源代码，在实用之前必须编译成机器代码。本文指南介绍了编译 SQLite 的各种方法。

本文并不提供编译 SQLite 的逐步指南，因为每个开发环境都不同，这样做会很困难。相反，本文描述和阐述了 SQLite 编译背后的原理。提供了典型的编译命令作为示例，期望应用程序开发者可以借此作为开发自定义编译过程的指导。换句话说，本文提供了思路和见解，而非成品解决方案。

SQLite 由 100 多个 C 代码文件和分布在多个目录中的脚本构建而成。SQLite 的实现是纯 ANSI-C，但许多 C 语言源代码文件在最终 SQLite 库被合并之前，要经过辅助 C 程序、AWK、SED 和 TCL 脚本的生成或转换。构建必要的 C 程序，以及转换和/或创建 SQLite 的 C 语言源代码是一个复杂的过程。

为了简化问题，SQLite 也作为预打包的合并源代码文件**sqlite3.c**提供。合并是一个包含整个 SQLite 库的单个 ANSI-C 代码文件。合并文件更易于处理。所有内容都包含在一个代码文件中，因此很容易将其添加到较大的 C 或 C++程序的源树中。所有代码生成和转换步骤都已完成，因此不需要配置和编译辅助 C 程序，也不需要运行脚本。而且，由于整个库包含在单个翻译单元中，编译器能够进行更高级的优化，从而实现 5%至 10%的性能提升。因此，推荐对所有应用程序使用合并源文件（**sqlite3.c**）。

> *推荐所有应用程序使用合并。*

直接从单独的源代码文件编译 SQLite 当然是可能的，但并不推荐。对于某些特定应用程序，可能需要修改构建过程，这是无法仅通过从网站下载的预构建的合并源文件完成的。对于这些情况，建议使用定制的合并文件构建（如下所述 howtocompile.html#amal）并使用它。换句话说，即使项目需要从单独的源文件开始构建 SQLite，仍建议使用合并源文件作为中间步骤。

# 2\. 编译命令行界面

命令行界面的构建需要三个源文件：

+   **sqlite3.c**: SQLite 汇编源文件

+   **sqlite3.h**: 伴随 sqlite3.c 的头文件，定义了 SQLite 的 C 语言接口。

+   **shell.c**: 命令行界面程序本身。这是包含**main()**例程定义和循环的 C 源代码文件，用于提示用户输入并将该输入传递给 SQLite 数据库引擎进行处理。

所有这三个源文件都包含在合并 tarball 中，可以从下载页面下载。

要构建 CLI，只需将这三个文件放在同一个目录中，并将它们一起编译。使用 MSVC：

> ```sql
> cl shell.c sqlite3.c -Fesqlite3.exe
> 
> ```

在 Unix 系统上（或在 Windows 上使用 cygwin 或 mingw+msys），命令通常看起来像这样：

> ```sql
> gcc shell.c sqlite3.c -lpthread -ldl -lm -o sqlite3
> 
> ```

pthreads 库用于使 SQLite 线程安全。但由于 CLI 是单线程的，我们可以指示 SQLite 以非线程安全模式构建，从而省略 pthreads 库：

> ```sql
> gcc -DSQLITE_THREADSAFE=0 shell.c sqlite3.c -ldl -lm -o sqlite3
> 
> ```

需要-ldl 库来支持动态加载，sqlite3_load_extension()接口和 load_extension() SQL 函数。如果不需要这些功能，则可以使用 SQLITE_OMIT_LOAD_EXTENSION 编译时选项来省略它们：

> ```sql
> gcc -DSQLITE_THREADSAFE=0 -DSQLITE_OMIT_LOAD_EXTENSION shell.c sqlite3.c -o sqlite3
> 
> ```

可能希望提供其他编译时选项，如

+   -DSQLITE_ENABLE_FTS4 或-DSQLITE_ENABLE_FTS5 用于全文搜索，

+   -DSQLITE_ENABLE_RTREE 用于 R*Tree 搜索引擎扩展，

+   -DSQLITE_ENABLE_DBSTAT_VTAB 用于 dbstat 虚拟表，或者

+   -DSQLITE_ENABLE_MATH_FUNCTIONS 用于扩展数学函数。

若要在 EXPLAIN 列表中看到额外的注释，请添加-DSQLITE_ENABLE_EXPLAIN_COMMENTS 选项。添加-DHAVE_READLINE 以及-lreadline 和-lncurses 库以获取命令行编辑支持。您可能还希望指定一些编译器优化开关。（SQLite 网站提供的可下载的预编译 CLI 使用了"-Os"。）这里有无数种可能的变化。编译完整特性的 shell 的命令可能如下所示：

> ```sql
> gcc -Os -I. -DSQLITE_THREADSAFE=0 -DSQLITE_ENABLE_FTS4 \
>    -DSQLITE_ENABLE_FTS5 -DSQLITE_ENABLE_JSON1 \
>    -DSQLITE_ENABLE_RTREE -DSQLITE_ENABLE_EXPLAIN_COMMENTS \
>    -DHAVE_READLINE \
>    shell.c sqlite3.c -ldl -lm -lreadline -lncurses -o sqlite3
> 
> ```

关键点在于：构建 CLI 包括将两个 C 语言文件编译在一起。**shell.c**文件包含入口点和用户输入循环的定义，而 SQLite 汇编**sqlite3.c**包含了 SQLite 库的完整实现。

# 3\. 编译 TCL 接口

SQLite 对 TCL 的接口是一个小模块，它被添加到常规的汇编中。结果是一个新的汇编源文件称为“**tclsqlite3.c**”。这个单一的源文件就足以生成一个可以加载到标准 [tclsh](http://wiki.tcl-lang.org/2541) 或 [wish](http://wiki.tcl-lang.org/2364) 中使用 [TCL load 命令](http://wiki.tcl-lang.org/9830) 的共享库，或者生成一个内置 SQLite 的独立 tclsh。TCL 汇编的一个副本已包含在 下载页面 上作为 TEA tarball 中的一个文件。

要在 Linux 上生成可 TCL 加载的 SQLite 库，以下命令足以：

> ```sql
> gcc -o libtclsqlite3.so -shared tclsqlite3.c -lpthread -ldl -ltcl
> 
> ```

不幸的是，为 Mac OS X 和 Windows 构建共享库并不那么简单。对于这些平台，最好使用包含在 TEA tarball 中的配置脚本和 makefile。

要生成一个静态链接到 SQLite 的独立 tclsh，使用以下编译器调用：

> ```sql
> gcc -DTCLSH=1 tclsqlite3.c -ltcl -lpthread -ldl -lz -lm
> 
> ```

这里的技巧在于 -DTCLSH=1 选项。SQLite 的 TCL 接口模块包括一个 **main()** 程序，在编译时使用 -DTCLSH=1 编译时初始化 TCL 解释器并进入命令行循环。上述命令适用于 Linux 和 Mac OS X，尽管可能需要根据平台和 TCL 版本调整库选项。

# 4\. 构建汇编

提供在 下载页面 上供应的 SQLite 汇编版本通常对大多数用户来说是足够的。然而，一些项目可能希望或需要构建自己的汇编。构建自定义汇编的常见原因是为了使用某些 编译时选项 来自定义 SQLite 库。记住，SQLite 汇编包含大量由辅助程序和脚本生成的 C 代码。许多编译时选项影响这些生成的代码，并且必须在组装汇编之前提供给代码生成器。必须传递给代码生成器的编译时选项集合可以因 SQLite 的不同版本而异，但在撰写本文时（约 SQLite 3.6.20，2009-11-04），代码生成器必须知道的选项集包括：

+   SQLITE_ENABLE_UPDATE_DELETE_LIMIT

+   SQLITE_OMIT_ALTERTABLE

+   SQLITE_OMIT_ANALYZE

+   SQLITE_OMIT_ATTACH

+   SQLITE_OMIT_AUTOINCREMENT

+   SQLITE_OMIT_CAST

+   SQLITE_OMIT_COMPOUND_SELECT

+   SQLITE_OMIT_EXPLAIN

+   SQLITE_OMIT_FOREIGN_KEY

+   SQLITE_OMIT_PRAGMA

+   SQLITE_OMIT_REINDEX

+   SQLITE_OMIT_SUBQUERY

+   SQLITE_OMIT_TEMPDB

+   SQLITE_OMIT_TRIGGER

+   SQLITE_OMIT_VACUUM

+   SQLITE_OMIT_VIEW

+   SQLITE_OMIT_VIRTUALTABLE

要构建自定义的集成，首先在 Unix 或类 Unix 开发平台上下载原始的单个源文件。确保获取原始的源文件而不是“预处理源文件”。可以从下载页面或直接从[配置管理系统](https://www.sqlite.org/src)获取完整的原始源文件集合。

假设 SQLite 源代码树存储在名为 "sqlite" 的目录中。计划在并行目录中构建集成，例如名为 "bld"。首先通过在 SQLite 源代码树顶部运行配置脚本或复制顶部源代码树中的模板 Makefile 之一来构建适当的 Makefile。然后手动编辑此 Makefile 以包括所需的编译时选项。最后运行：

> ```sql
> make sqlite3.c
> 
> ```

或者在使用 MSVC 的 Windows 上：

> ```sql
> nmake /f Makefile.msc sqlite3.c
> 
> ```

"sqlite3.c" 的 make 目标将自动构建常规的 "**sqlite3.c**" 集成源文件，其头文件 "**sqlite3.h**"，以及包含 TCL 接口的 "**tclsqlite3.c**" 集成源文件。然后，可以将所需的文件复制到项目目录中，并根据上述步骤进行编译。

# 5\. 构建 Windows DLL

要构建用于 Windows 的 SQLite DLL，请首先获取适当的集成源代码文件，sqlite3.c 和 sqlite3.h。可以从[SQLite 网站](https://www.sqlite.org/download.html)下载，或者按照上面显示的方式自定义生成。

使用工作目录中的源代码文件，可以使用以下命令在 MSVC 中生成 DLL：

> ```sql
> cl sqlite3.c -link -dll -out:sqlite3.dll
> 
> ```

上述命令应该从 MSVC 本机工具命令提示符中运行。如果在您的机器上安装了 MSVC，可能会有多个版本的此命令提示符，用于 x86 的本机构建，以及可能还用于交叉编译到 ARM。根据所需的 DLL 使用适当的命令提示符。

如果使用 MinGW 编译器，命令行是这样的：

> ```sql
> gcc -shared sqlite3.c -o sqlite3.dll
> 
> ```

注意，MinGW 只生成 32 位的 DLL。有一个单独的 MinGW64 项目可以用来生成 64 位的 DLL。假设命令行语法类似。还要注意，最近版本的 MSVC 生成的 DLL 在 Windows XP 及更早版本的 Windows 上无法工作。因此，为了生成的 DLL 的最大兼容性，建议使用 MinGW。一个好的经验法则是使用 MinGW 生成 32 位的 DLL，使用 MSVC 生成 64 位的 DLL。

在大多数情况下，您将希望根据您的应用程序补充上述基本命令，使用适合您的应用程序的编译时选项。常用的编译时选项包括：

+   **-Os** - 优化大小。尽可能使 DLL 文件尽量小。

+   **-O2** - 优化以提升速度。这会通过展开循环和内联函数来增加 DLL 的大小。

+   **-DSQLITE_ENABLE_FTS4** - 在 SQLite 中包括全文搜索引擎代码。

+   **-DSQLITE_ENABLE_RTREE** - 包括 R-Tree 扩展。

+   **-DSQLITE_ENABLE_COLUMN_METADATA** - 启用一些额外的 API，这些 API 被一些常见系统（包括 Ruby-on-Rails）所需。
