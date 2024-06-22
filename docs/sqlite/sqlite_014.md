# SQLite 的 C 语言接口规范

> 原文：[`sqlite.com/c3ref/intro.html`](https://sqlite.com/c3ref/intro.html)

这些页面旨在提供精确和详细的规范说明。有关教程介绍，请参见：

+   SQLite 五分钟快速入门 或者

+   SQLite C/C++接口介绍

相同的内容也可作为一个 单个大的 HTML 文件 查看。

SQLite 接口元素可以分为三类：

1.  **对象列表。** 这是 SQLite 库使用的所有抽象对象和数据类型的列表。总共有几十个对象，但最重要的两个对象是：数据库连接对象 sqlite3，以及预编译语句对象 sqlite3_stmt。

1.  **常量列表。** 这是 SQLite 使用的数值常量列表，由 sqlite3.h 头文件中的 #defines 表示。这些常量包括从各种接口返回的数值 结果代码（例如：SQLITE_OK）或者传入函数以控制行为的标志（例如：SQLITE_OPEN_READONLY）。

1.  **函数列表。** 这是所有在 对象 上操作并使用和/或返回 常量 的所有函数和方法的列表。有很多函数，但大多数应用只使用少数几个。
