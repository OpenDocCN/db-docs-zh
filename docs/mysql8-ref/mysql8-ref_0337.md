# 7.9.1 调试 MySQL 服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/debugging-server.html`](https://dev.mysql.com/doc/refman/8.0/en/debugging-server.html)

7.9.1.1 为调试编译 MySQL

7.9.1.2 创建跟踪文件

7.9.1.3 使用 WER 与 PDB 创建 Windows 崩溃转储

7.9.1.4 在 gdb 下调试 mysqld

7.9.1.5 使用堆栈跟踪

7.9.1.6 使用服务器日志查找 mysqld 中错误的原因

7.9.1.7 如果遇到表损坏，制作可重现的测试用例

如果您正在使用 MySQL 中非常新的功能，可以尝试使用 `--skip-new` 选项运行 **mysqld**（该选项禁用所有新的、潜在不安全的功能）。参见 附录 B.3.3.3，“如果 MySQL 一直崩溃怎么办”。

如果 **mysqld** 不想启动，请验证您没有干扰设置的 `my.cnf` 文件！您可以使用 **mysqld --print-defaults** 检查您的 `my.cnf` 参数，并通过使用 **mysqld --no-defaults ...** 来避免使用它们。

如果 **mysqld** 开始占用 CPU 或内存，或者“挂起”，您可以使用 **mysqladmin processlist status** 查看是否有人执行需要很长时间的查询。如果遇到性能问题或新客户端无法连接的问题，建议在某个窗口中运行 **mysqladmin -i10 processlist status**。

命令 **mysqladmin debug** 会将一些关于正在使用的锁、已使用的内存和查询使用情况的信息转储到 MySQL 日志文件中。这可能有助于解决一些问题。即使您没有为调试编译 MySQL，此命令也会提供一些有用的信息！

如果问题是某些表变得越来越慢，您应该尝试使用 `OPTIMIZE TABLE` 或 **myisamchk** 优化表。参见 第七章，*MySQL 服务器管理*。您还应该使用 `EXPLAIN` 检查慢查询。

你还应该阅读本手册中针对可能在你的环境中独特的问题的特定于操作系统的部分。参见第 2.1 节，“一般安装指导”。
