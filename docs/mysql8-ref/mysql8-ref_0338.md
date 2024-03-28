> 原文：[`dev.mysql.com/doc/refman/8.0/en/compiling-for-debugging.html`](https://dev.mysql.com/doc/refman/8.0/en/compiling-for-debugging.html)

#### 7.9.1.1 为调试编译 MySQL

如果你遇到了一些非常具体的问题，你可以尝试调试 MySQL。为此，你必须使用`-DWITH_DEBUG=1`选项配置 MySQL。你可以通过执行**mysqld --help**来检查 MySQL 是否已启用调试。如果选项中列出了`--debug`，则表示已启用调试。**mysqladmin ver**还会将**mysqld**版本列为**mysql ... --debug**。

如果在使用`-DWITH_DEBUG=1` CMake 选项配置时，**mysqld**停止崩溃，那么你可能已经发现了 MySQL 中的编译器错误或定时错误。在这种情况下，你可以尝试使用`CMAKE_C_FLAGS`和`CMAKE_CXX_FLAGS` CMake 选项添加`-g`，而不使用`-DWITH_DEBUG=1`。如果**mysqld**崩溃，你至少可以使用**gdb**附加到它，或者使用**gdb**在核心文件上找出发生了什么。

当你为调试配置 MySQL 时，会自动启用许多额外的安全检查功能，用于监视**mysqld**的健康状况。如果发现了“意外”情况，会将条目写入`stderr`，而这会被**mysqld_safe**重定向到错误日志！这也意味着，如果你在使用源代码分发时遇到了一些意外问题，你应该首先为 MySQL 配置调试。如果你认为自己发现了一个错误，请按照第 1.5 节“如何报告错误或问题”中的说明操作。

在 Windows MySQL 发行版中，默认情况下，`mysqld.exe`是使用跟踪文件支持编译的。
