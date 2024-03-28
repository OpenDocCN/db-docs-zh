> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-gdb-on-mysqld.html`](https://dev.mysql.com/doc/refman/8.0/en/using-gdb-on-mysqld.html)

#### 7.9.1.4 在 gdb 下调试 mysqld

在大多数系统上，你也可以从 **gdb** 中启动 **mysqld** 以获取更多信息，如果 **mysqld** 崩溃。

在 Linux 上，如果你想要调试 **mysqld** 线程，一些较旧的 **gdb** 版本必须使用 `run --one-thread`。在这种情况下，你一次只能有一个线程处于活动状态。

在 **gdb** 下运行 MySQL 时，NPTL 线程（Linux 上的新线程库）可能会导致问题。一些症状包括：

+   **mysqld** 在启动过程中挂起（在写入 `ready for connections` 之前）。

+   **mysqld** 在调用 `pthread_mutex_lock()` 或 `pthread_mutex_unlock()` 时崩溃。

在这种情况下，在启动 **gdb** 之前，你应该在 shell 中设置以下环境变量：

```sql
LD_ASSUME_KERNEL=2.4.1
export LD_ASSUME_KERNEL
```

在 **gdb** 下运行 **mysqld** 时，你应该使用 `--skip-stack-trace` 禁用堆栈跟踪，以便在 **gdb** 中捕获段错误。

使用 `--gdb` 选项为 **mysqld** 安装一个 `SIGINT` 中断处理程序（用于使用 `^C` 停止 **mysqld** 设置断点），并禁用堆栈跟踪和核心文件处理。

如果你一直在 **gdb** 下进行大量新连接，那么在 **gdb** 中不释放旧线程的内存，会导致在 **gdb** 下调试 MySQL 非常困难。你可以通过将 **mysqld** 的 `thread_cache_size` 设置为 `max_connections` + 1 的值来避免这个问题。在大多数情况下，只需使用 `--thread_cache_size=5'` 就会有很大帮助！

如果在 Linux 上 **mysqld** 因 SIGSEGV 信号而崩溃，你可以使用 `--core-file` 选项启动 **mysqld** 以获取核心转储文件。这个核心文件可以用来生成回溯，帮助你找出 **mysqld** 为何崩溃：

```sql
$> gdb mysqld core
gdb> backtrace full
gdb> quit
```

参见 Section B.3.3.3, “What to Do If MySQL Keeps Crashing”。

如果你在 Linux 上使用 **gdb**，你应该在当前目录中安装一个 `.gdb` 文件，包含以下信息：

```sql
set print sevenbit off
handle SIGUSR1 nostop noprint
handle SIGUSR2 nostop noprint
handle SIGWAITING nostop noprint
handle SIGLWP nostop noprint
handle SIGPIPE nostop
handle SIGALRM nostop
handle SIGHUP nostop
handle SIGTERM nostop noprint
```

这里是一个调试 **mysqld** 的示例：

```sql
$> gdb /usr/local/libexec/mysqld
gdb> run
...
backtrace full # Do this when mysqld crashes
```

将上述输出包含在一个 bug 报告中，你可以按照 Section 1.5, “How to Report Bugs or Problems” 中的说明进行报告。

如果 **mysqld** 卡住了，你可以尝试使用一些系统工具如 `strace` 或 `/usr/proc/bin/pstack` 来查看 **mysqld** 卡在哪里。

```sql
strace /tmp/log libexec/mysqld
```

如果你正在使用 Perl `DBI` 接口，可以通过使用 `trace` 方法或设置 `DBI_TRACE` 环境变量来打开调试信息。
