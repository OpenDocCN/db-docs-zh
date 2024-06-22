# 1\. 概述

> 原文：[`sqlite.com/cintro.html`](https://sqlite.com/cintro.html)

下列两个对象和八种方法构成了 SQLite 接口的基本元素：

+   **sqlite3** → 数据库连接对象。由 sqlite3_open()创建，由 sqlite3_close()销毁。

+   **sqlite3_stmt** → 准备语句对象。由 sqlite3_prepare()创建，由 sqlite3_finalize()销毁。

+   **sqlite3_open()** → 打开到新的或现有 SQLite 数据库的连接。sqlite3 的构造函数。

+   **sqlite3_prepare()** → 将 SQL 文本编译为字节码，以执行查询或更新数据库操作。sqlite3_stmt 的构造函数。

+   **sqlite3_bind()** → 将应用程序数据存储到原始 SQL 的参数中。

+   **sqlite3_step()** → 推进 sqlite3_stmt 至下一个结果行或完成。

+   **sqlite3_column()** → 当前结果行中的列值，对应于 sqlite3_stmt。

+   **sqlite3_finalize()** → sqlite3_stmt 的析构函数。

+   **sqlite3_close()** → sqlite3 的析构函数。

+   **sqlite3_exec()** → 一个包装函数，执行一个或多个 SQL 语句的字符串，包括 sqlite3_prepare()、sqlite3_step()、sqlite3_column()和 sqlite3_finalize()。

# 2\. 引言

SQLite 有超过 225 个 API。但是，大多数 API 都是可选的和非常专业化的，初学者可以忽略它们。核心 API 很小、简单且易于学习。本文总结了核心 API。

单独的文档，SQLite C/C++ 接口，提供了 SQLite 所有 C/C++ API 的详细规格说明。读者一旦理解了 SQLite 的基本操作原理，应将该文档作为参考指南使用。本文仅作介绍，不是 SQLite API 的完整或权威参考。

# 3\. 核心对象和接口

SQL 数据库引擎的主要任务是评估 SQL 语句。为了实现这一点，开发者需要两个对象：

+   数据库连接（database connection）对象：sqlite3

+   准备语句（prepared statement）对象：sqlite3_stmt

严格来说，不需要预编译语句对象，因为可以使用便捷包装接口 sqlite3_exec 或 sqlite3_get_table，这些便捷包装封装和隐藏了预编译语句对象。然而，理解预编译语句对于充分利用 SQLite 是必要的。

数据库连接（sqlite3）和预编译语句对象由下面列出的一小组 C/C++ 接口例程控制。

+   sqlite3_open()

+   sqlite3_prepare()

+   sqlite3_step()

+   sqlite3_column()

+   sqlite3_finalize()

+   sqlite3_close()

注意上面例程列表是概念上的而非实际的。许多这些例程存在多个版本。例如，上面的列表显示了一个名为 sqlite3_open() 的单一例程，实际上有三个稍有不同方式完成相同工作的例程： sqlite3_open(), sqlite3_open16() 和 sqlite3_open_v2()。列表提到了 sqlite3_column()，实际上并没有这样的例程存在。列表中显示的 "sqlite3_column()" 是一组例程的占位符，用于提取各种数据类型的列数据。

以下是核心接口的摘要：

+   **sqlite3_open()**

    此例程打开与 SQLite 数据库文件的连接，并返回一个数据库连接对象。这通常是应用程序进行的第一个 SQLite API 调用，并且是大多数其他 SQLite API 的先决条件。许多 SQLite 接口需要一个数据库连接对象指针作为它们的第一个参数，并可以视为数据库连接对象的方法。此例程是数据库连接对象的构造函数。

+   **sqlite3_prepare()**

    此例程将 SQL 文本转换为预编译语句对象，并返回指向该对象的指针。此接口需要先前调用 sqlite3_open() 创建的数据库连接指针和包含要准备的 SQL 语句的文本字符串。该 API 实际上不评估 SQL 语句，而仅准备 SQL 语句以供评估。

    将每个 SQL 语句视为一个小型计算机程序。sqlite3_prepare() 的目的是将该程序编译为目标代码。预编译语句就是这个目标代码。sqlite3_step() 接口然后运行目标代码以获得结果。

    新应用程序应始终使用 sqlite3_prepare_v2()来替代 sqlite3_prepare()。旧的 sqlite3_prepare()为向后兼容而保留。但是 sqlite3_prepare_v2()提供了一个更好的接口。

+   **sqlite3_step()**

    这个例程用于评估之前由 sqlite3_prepare()接口创建的预编译语句。该语句在第一行结果可用的点被评估。要前进到第二行结果，请再次调用 sqlite3_step()。继续调用 sqlite3_step()，直到语句完成。不返回结果的语句（如：INSERT、UPDATE 或 DELETE 语句）在单次调用 sqlite3_step()中执行完毕。

+   **sqlite3_column()**

    该例程从由 sqlite3_step()评估的预编译语句的当前行中返回单个列。每当 sqlite3_step()停在一个新的结果集行时，可以多次调用此例程来找到该行中所有列的值。

    如上所述，SQLite API 中实际上不存在"sqlite3_column()"函数。相反，我们在这里称之为"sqlite3_column()"的东西是一个整个函数族的占位符，这些函数以各种数据类型从结果集返回值。此族中还有返回结果大小（如果是字符串或 BLOB）和结果集中列数的例程。

    +   sqlite3_column_blob()

    +   sqlite3_column_bytes()

    +   sqlite3_column_bytes16()

    +   sqlite3_column_count()

    +   sqlite3_column_double()

    +   sqlite3_column_int()

    +   sqlite3_column_int64()

    +   sqlite3_column_text()

    +   sqlite3_column_text16()

    +   sqlite3_column_type()

    +   sqlite3_column_value()

+   **sqlite3_finalize()**

    该例程销毁之前由调用 sqlite3_prepare()创建的预编译语句。为了避免内存泄漏，必须使用此例程调用销毁每个预编译语句。

+   **sqlite3_close()**

    此例程关闭之前由调用 sqlite3_open()打开的数据库连接。连接关联的所有预编译语句在关闭连接之前应该被完成。

# 4\. 核心例程和对象的典型用法

应用程序通常会在初始化期间使用 sqlite3_open()创建单个数据库连接。请注意，sqlite3_open()可以用于打开现有的数据库文件或创建并打开新的数据库文件。虽然许多应用程序仅使用单个数据库连接，但应用程序也可以多次调用 sqlite3_open()来打开多个数据库连接，无论是到同一数据库还是不同数据库。有时，多线程应用程序会为每个线程创建单独的数据库连接。请注意，单个数据库连接可以使用 ATTACH SQL 命令访问两个或更多数据库，因此不必为每个数据库文件创建单独的数据库连接。

许多应用程序在关闭时使用 sqlite3_close()销毁它们的数据库连接。例如，使用 SQLite 作为其应用文件格式的应用程序可能会在响应文件/打开菜单操作时打开数据库连接，然后在响应文件/关闭菜单时销毁相应的数据库连接。

要运行 SQL 语句，应用程序遵循以下步骤：

1.  使用 sqlite3_prepare()创建预编译语句。

1.  通过调用 sqlite3_step()一次或多次来评估预编译语句。

1.  对于查询，通过在两次调用 sqlite3_step()之间调用 sqlite3_column()来提取结果。

1.  使用 sqlite3_finalize()销毁预编译语句。

以上内容实际上是有效使用 SQLite 所需了解的全部内容。其余内容均为优化和详细信息。

# 5\. 核心例程周围的便利包装器

sqlite3_exec()接口是一个便利包装器，通过单个函数调用执行以上四个步骤。传入 sqlite3_exec()的回调函数用于处理结果集中的每一行。另一个便利包装器是 sqlite3_get_table()，它也执行以上四个步骤。sqlite3_get_table()接口与 sqlite3_exec()不同之处在于它将查询结果存储在堆内存中，而不是调用回调函数。

需要意识到，sqlite3_exec()和 sqlite3_get_table()都无法执行核心例程无法完成的任何操作。事实上，这些包装器完全基于核心例程实现。

# 6\. 绑定参数并重用预编译语句

在先前的讨论中，假设每个 SQL 语句只准备一次，然后被评估，然后被销毁。然而，SQLite 允许同一个准备语句被多次评估。这通过以下例程实现：

+   sqlite3_reset()

+   sqlite3_bind()

在一个或多个调用 sqlite3_step()之后，准备语句可以通过调用 sqlite3_reset()来重置，以便再次评估。将 sqlite3_reset()视为将准备语句程序倒回到开始。在现有的准备语句上使用 sqlite3_reset()，而不是创建一个新的准备语句，可以避免不必要地调用 sqlite3_prepare()。对于许多 SQL 语句，运行 sqlite3_prepare()所需的时间等于或超过 sqlite3_step()所需的时间。因此，避免调用 sqlite3_prepare()可以显著提高性能。

不常用的是多次评估*完全相同*的 SQL 语句。更常见的情况是要评估类似的语句。例如，您可能希望使用不同的值多次评估 INSERT 语句。或者，您可能希望在 WHERE 子句中使用不同的键多次评估相同的查询。为了适应这一点，SQLite 允许 SQL 语句包含参数，这些参数在被评估之前被“绑定”到值上。这些值稍后可以更改，并且可以使用新值再次评估相同的准备语句。

在查询或数据修改语句中允许参数处于字符串字面量、blob 字面量、数值常量或 NULL 所允许的任何地方。 (DQL 或 DML)（参数不能用于列名或表名，也不能作为约束或默认值的值。 (DDL)）一个参数采用以下形式之一：

+   **?**

+   **?***NNN*

+   **:***AAA*

+   **$***AAA*

+   **@***AAA*

在上述示例中，*NNN*是一个整数值，*AAA*是一个标识符。参数最初具有 NULL 值。在首次调用 sqlite3_step()之前或 sqlite3_reset()后立即，应用程序可以调用 sqlite3_bind()接口将值绑定到参数上。每次调用 sqlite3_bind()都会覆盖同一参数上的先前绑定。

应用程序允许预先准备多个 SQL 语句并根据需要评估它们。没有对准备语句的未完成数量设定任何限制。一些应用程序在启动时多次调用 sqlite3_prepare()来创建它们将来可能需要的所有准备语句。其他应用程序保持最近使用的准备语句的缓存，然后在可用时重用准备语句。另一种方法是只在循环内重用准备语句。

# 7\. Configuring SQLite

默认的 SQLite 配置对大多数应用程序效果很好。但有时开发人员希望调整设置以尝试提升一点性能，或者利用一些不常见的功能。

sqlite3_config() 接口用于为 SQLite 进行全局、进程范围的配置更改。在创建任何数据库连接之前，必须调用 sqlite3_config()接口。sqlite3_config()接口允许程序员执行如下操作：

+   调整 SQLite 的内存分配，包括设置适合安全关键实时嵌入式系统和应用程序定义内存分配器的备用内存分配器。

+   设置进程范围的错误日志。

+   指定应用程序定义的页面缓存。

+   调整互斥体的使用，使其适用于各种线程模型，或者替换应用程序定义的互斥体系统。

在进程范围的配置完成并且创建了数据库连接之后，可以使用 sqlite3_limit()和 sqlite3_db_config()来配置单个数据库连接。

# 8\. Extending SQLite

SQLite 包括可用于扩展其功能的接口。这些例程包括：

+   sqlite3_create_collation()

+   sqlite3_create_function()

+   sqlite3_create_module()

+   sqlite3_vfs_register()

sqlite3_create_collation() 接口用于创建新的用于排序文本的校对顺序。 sqlite3_create_module() 接口用于注册新的虚拟表实现。 sqlite3_vfs_register() 接口创建新的 VFS。

sqlite3_create_function() 接口创建新的 SQL 函数 - 标量或聚合。新函数的实现通常利用以下附加接口：

+   sqlite3_aggregate_context()

+   sqlite3_result()

+   sqlite3_user_data()

+   sqlite3_value()

SQLite 的所有内置 SQL 函数都是使用完全相同的接口创建的。请参阅 SQLite 源代码，特别是[date.c](https://www.sqlite.org/src/doc/trunk/src/date.c)和[func.c](https://www.sqlite.org/src/doc/trunk/src/func.c)源文件以获取示例。

共享库或 DLL 可以作为可加载扩展用于 SQLite。

# 9\. 其他接口

本文仅提及最重要和最常用的 SQLite 接口。SQLite 库包含许多其他实现有用功能的 API，这些功能在此处未描述。SQLite 应用程序编程接口的完整功能列表可在 C/C++接口规范中找到。请参阅该文档以获取关于所有 SQLite 接口的完整和权威信息。
