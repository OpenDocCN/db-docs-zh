# 7.9.4 `DBUG` 包

> 原文：[`dev.mysql.com/doc/refman/8.0/en/dbug-package.html`](https://dev.mysql.com/doc/refman/8.0/en/dbug-package.html)

MySQL 服务器和大多数 MySQL 客户端都是使用由 Fred Fish 最初创建的 `DBUG` 包编译的。当您为调试配置 MySQL 时，此包使得可以获得程序正在执行的跟踪文件。参见 Section 7.9.1.2, “Creating Trace Files”。

本节总结了在使用带有调试支持的 MySQL 程序时可以在命令行上指定的调试选项的参数值。

`DBUG` 包可以通过使用 `--debug[=*`debug_options`*]` 或 `-# [*`debug_options`*]` 选项来调用程序。如果您在不指定 *`debug_options`* 值的情况下指定 `--debug` 或 `-#` 选项，大多数 MySQL 程序会使用默认值。在 Unix 上，服务器默认值为 `d:t:i:o,/tmp/mysqld.trace`，在 Windows 上为 `d:t:i:O,\mysqld.trace`。这个默认值的效果是：

+   `d`: 启用所有调试宏的输出

+   `t`: 跟踪函数调用和退出

+   `i`: 在输出行中添加 PID

+   `o,/tmp/mysqld.trace`, `O,\mysqld.trace`: 设置调试输出文件。

大多数客户端程序使用默认的 *`debug_options`* 值为 `d:t:o,/tmp/*`program_name`*.trace`，无论平台如何。

以下是一些示例调试控制字符串，如在 shell 命令行上指定的方式：

```sql
--debug=d:t
--debug=d:f,main,subr1:F:L:t,20
--debug=d,input,output,files:n
--debug=d:t:i:O,\\mysqld.trace
```

对于 **mysqld**，还可以通过设置 `debug` 系统变量在运行时更改 `DBUG` 设置。此变量具有全局和会话值：

```sql
mysql> SET GLOBAL debug = '*debug_options*';
mysql> SET SESSION debug = '*debug_options*';
```

更改全局 `debug` 值需要足够设置全局系统变量的权限。更改会话 `debug` 值需要足够设置受限会话系统变量的权限。参见 Section 7.1.9.1, “System Variable Privileges”。

*`debug_options`* 值是一个以冒号分隔的字段序列：

```sql
field_1:field_2:...:field_*N*
```

值内的每个字段由一个必需的标志字符组成，可选地前面跟着一个 `+` 或 `-` 字符，并可选地后跟着一个逗号分隔的修饰符列表：

```sql
[+|-]flag[,modifier,modifier,...,modifier]
```

以下表描述了允许的标志字符。未识别的标志字符会被静默忽略。

| 标志 | 描述 |
| --- | --- |
| `d` | 启用当前状态下 `DBUG_*`XXX`*` 宏的输出。可以跟随一个关键字列表，仅为具有该关键字的 `DBUG` 宏启用输出。空关键字列表为所有宏启用输出。在 MySQL 中，常见的调试宏关键字包括 `enter`、`exit`、`error`、`warning`、`info` 和 `loop`。 |
| `D` | 每个调试器输出行后延迟。参数是延迟时间，以秒为单位，取决于机器能力。例如，`D,20`指定了两秒的延迟。 |
| `f` | 将调试、跟踪和性能分析限制为命名函数列表。空列表启用所有函数。仍然必须给出适当的`d`或`t`标志；此标志仅限制它们的操作（如果它们已启用）。 |
| `F` | 为每行调试或跟踪输出标识源文件名。 |
| `i` | 为每行调试或跟踪输出标识具有 PID 或线程 ID 的进程。 |
| `L` | 为每行调试或跟踪输出标识源文件行号。 |
| `n` | 为每行调试或跟踪输出打印当前函数嵌套深度。 |
| `N` | 为每行调试输出编号。 |
| `o` | 重定向调试器输出流到指定文件。默认输出为`stderr`。 |
| `O` | 类似于`o`，但文件在每次写入之间真正刷新。在需要时，文件在每次写入之间关闭并重新打开。 |
| `a` | 类似于`o`，但用于追加打开。 |
| `A` | 类似于`O`，但用于追加打开。 |
| `p` | 将调试器操作限制为指定的进程。必须使用`DBUG_PROCESS`宏标识进程，并且匹配列表中的一个进程才会发生调试器操作。 |
| `P` | 为每行调试或跟踪输出打印当前进程名称。 |
| `r` | 推送新状态时，不继承先前状态的函数嵌套级别。当输出从左边距开始时很有用。 |
| `t` | 启用函数调用/退出跟踪行。可以跟随一个列表（仅包含一个修饰符），给出一个数字最大跟踪级别，超过该级别，不会发生任何输出，无论是调试还是跟踪宏。默认是编译时选项。 |
| `T` | 为每行输出打印当前时间戳。 |
| 标志 | 描述 |

前导的`+`或`-`字符和修饰符列表用于像`d`或`f`这样的标志字符，可以为所有适用的修饰符启用调试操作或仅为其中一些启用：

+   没有前导的`+`或`-`，标志值将准确设置为给定的修饰符列表。

+   以前导的`+`或`-`，列表中的修饰符将被添加到或从当前修饰符列表中减去。

以下示例展示了`d`标志的工作原理。空的`d`列表启用了所有调试宏的输出。非空列表仅启用列表中的宏关键字的输出。

这些语句将`d`值设置为给定的修饰符列表：

```sql
mysql> SET debug = 'd';
mysql> SELECT @@debug;
+---------+
| @@debug |
+---------+
| d       |
+---------+
mysql> SET debug = 'd,error,warning';
mysql> SELECT @@debug;
+-----------------+
| @@debug         |
+-----------------+
| d,error,warning |
+-----------------+
```

前导的`+`或`-`会增加或减去当前`d`值：

```sql
mysql> SET debug = '+d,loop';
mysql> SELECT @@debug;
+----------------------+
| @@debug              |
+----------------------+
| d,error,warning,loop |
+----------------------+

mysql> SET debug = '-d,error,loop';
mysql> SELECT @@debug;
+-----------+
| @@debug   |
+-----------+
| d,warning |
+-----------+
```

添加到“启用所有宏”会导致没有变化：

```sql
mysql> SET debug = 'd';
mysql> SELECT @@debug;
+---------+
| @@debug |
+---------+
| d       |
+---------+

mysql> SET debug = '+d,loop';
mysql> SELECT @@debug;
+---------+
| @@debug |
+---------+
| d       |
+---------+
```

禁用所有已启用的宏会完全禁用`d`标志：

```sql
mysql> SET debug = 'd,error,loop';
mysql> SELECT @@debug;
+--------------+
| @@debug      |
+--------------+
| d,error,loop |
+--------------+

mysql> SET debug = '-d,error,loop';
mysql> SELECT @@debug;
+---------+
| @@debug |
+---------+
|         |
+---------+
```
