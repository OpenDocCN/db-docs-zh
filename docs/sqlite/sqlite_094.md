# 调试提示

> 原文：[`sqlite.com/debugging.html`](https://sqlite.com/debugging.html)

以下是 SQLite 开发者用来跟踪、检查和理解核心 SQLite 库行为的随机技术集合。

这些技术旨在帮助理解核心 SQLite 库本身，而不是仅仅使用 SQLite 的应用程序。

1.  **在命令行 shell 上使用“.eqp full”选项**

    当你有一个正在调试或尝试理解的 SQL 脚本时，通常可以在命令行 shell 中使用“.eqp full”设置运行它。当“.eqp”设置为 FULL 时，shell 会自动显示每个命令在实际运行之前的 EXPLAIN 和 EXPLAIN QUERY PLAN 输出。

    为了增强可读性，还可以设置“.echo on”，这样输出将包含原始的 SQL 文本。

    新的“.eqp trace”命令与“.eqp full”做的一切相同，并且还打开 VDBE 跟踪。

1.  **使用编译时选项来启用调试功能。**

    建议的编译时选项包括：

    +   -DSQLITE_DEBUG

    +   -DSQLITE_ENABLE_EXPLAIN_COMMENTS

    +   -DSQLITE_ENABLE_TREETRACE

    +   -DSQLITE_ENABLE_WHERETRACE

SQLITE_ENABLE_TREETRACE 和 SQLITE_ENABLE_WHERETRACE 选项在编译时选项文档中未记录，因为它们不受官方支持。它们的作用是在命令行 shell 中激活“.treetrace”和“.wheretrace”命令，这些命令提供用于生成 SELECT 和 DML 语句以及 WHERE 子句的代码的低级跟踪输出。

1.  **从调试器中调用 sqlite3ShowExpr() 和类似函数。**

    当使用 SQLITE_DEBUG 编译时，SQLite 包括一些例程，可以将各种内部抽象语法树结构打印为 ASCII 艺术图表。这在调试过程中非常有用，可以帮助理解 SQLite 正在处理的变量。以下例程可用：

    +   void sqlite3ShowExpr(const Expr*);

    +   void sqlite3ShowExprList(const ExprList*);

    +   void sqlite3ShowIdList(const IdList*);

    +   void sqlite3ShowSrcList(const SrcList*);

    +   void sqlite3ShowSelect(const Select*);

    +   void sqlite3ShowWith(const With*);

    +   void sqlite3ShowUpsert(const Upsert*);

    +   void sqlite3ShowTrigger(const Trigger*);

    +   void sqlite3ShowTriggerList(const Trigger*);

    +   void sqlite3ShowTriggerStep(const TriggerStep*);

    +   void sqlite3ShowTriggerStepList(const TriggerStep*);

    +   void sqlite3ShowWindow(const Window*);

    +   void sqlite3ShowWinFunc(const Window*);

    这些例程不是 API，可能会更改。仅用于交互式调试。

1.  **在 test_addoptrace 上设置断点**

    在调试 bytecode 生成器时，了解特定操作码的生成位置通常很有用。要轻松找到这一点，请在调试器中运行脚本。在 "test_addoptrace" 例程上设置断点。然后运行 "PRAGMA vdbe_addoptrace=ON;"，然后运行相应的 SQL 语句。每个操作码将显示为附加到 VDBE 程序时，并且断点将立即触发。逐步执行直到达到操作码，然后向后查看堆栈，以查看生成它的位置和方式。

    只有在使用 SQLITE_DEBUG 编译时才有效。

1.  **使用 ".treetrace" 和 ".wheretrace" shell 命令**

    当命令行 shell 和核心 SQLite 库都编译了 SQLITE_DEBUG、SQLITE_ENABLE_TREETRACE 和 SQLITE_ENABLE_WHERETRACE 时，shell 具有两个命令用于打开代码生成器中最复杂部分的调试设施 - 分别是处理 SELECT 语句和 WHERE 子句的逻辑。".treetrace"和".wheretrace"命令各自接受一个十六进制的数值参数。每个位开启不同的调试部分。通常使用"0xfff"和"0xff"的值。使用参数"0"可以关闭所有跟踪输出。

1.  **使用".breakpoint"命令**

    在 CLI 中，“.breakpoint”命令仅调用名为“test_breakpoint()”的过程，这是一个空操作。

    如果你有一个脚本，想要在脚本中间某个时刻开始调试，只需在 gdb（或其他调试器）中的 test_breakpoint()函数设置断点，并在想要停止的地方添加".breakpoint"命令。当到达第一个断点时，设置任何你需要的额外断点和变量追踪。

1.  **禁用看得见的内存分配器**

    当查找内存分配问题（内存泄漏、释放后使用错误、缓冲区溢出等）时，有时候禁用看得见的内存分配器然后在 valgrind 或 MSAN 或其他堆内存调试工具下运行测试是有用的。可以使用 SQLITE_CONFIG_LOOKASIDE 接口在启动时禁用看得见的内存分配器。如果命令行 shell 以"--lookaside 0 0"命令行选项启动，将使用该接口禁用看得见的内存分配器。
