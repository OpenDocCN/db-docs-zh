> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-stack-trace.html`](https://dev.mysql.com/doc/refman/8.0/en/using-stack-trace.html)

#### 7.9.1.5 使用堆栈跟踪

在一些操作系统上，如果**mysqld**意外死机，错误日志中会包含堆栈跟踪。您可以使用这个来找出**mysqld**死机的地方（也许是为什么）。参见 Section 7.4.2，“错误日志”。要获得堆栈跟踪，您不能使用 gcc 的`-fomit-frame-pointer`选项来编译**mysqld**。参见 Section 7.9.1.1，“为调试编译 MySQL”。

错误日志中的堆栈跟踪看起来像这样：

```sql
mysqld got signal 11;
Attempting backtrace. You can use the following information
to find out where mysqld died. If you see no messages after
this, something went terribly wrong...

stack_bottom = 0x41fd0110 thread_stack 0x40000
mysqld(my_print_stacktrace+0x32)[0x9da402]
mysqld(handle_segfault+0x28a)[0x6648e9]
/lib/libpthread.so.0[0x7f1a5af000f0]
/lib/libc.so.6(strcmp+0x2)[0x7f1a5a10f0f2]
mysqld(_Z21check_change_passwordP3THDPKcS2_Pcj+0x7c)[0x7412cb]
mysqld(_ZN16set_var_password5checkEP3THD+0xd0)[0x688354]
mysqld(_Z17sql_set_variablesP3THDP4ListI12set_var_baseE+0x68)[0x688494]
mysqld(_Z21mysql_execute_commandP3THD+0x41a0)[0x67a170]
mysqld(_Z11mysql_parseP3THDPKcjPS2_+0x282)[0x67f0ad]
mysqld(_Z16dispatch_command19enum_server_commandP3THDPcj+0xbb7[0x67fdf8]
mysqld(_Z10do_commandP3THD+0x24d)[0x6811b6]
mysqld(handle_one_connection+0x11c)[0x66e05e]
```

如果跟踪的函数名解析失败，跟踪将包含较少信息：

```sql
mysqld got signal 11;
Attempting backtrace. You can use the following information
to find out where mysqld died. If you see no messages after
this, something went terribly wrong...

stack_bottom = 0x41fd0110 thread_stack 0x40000
[0x9da402]
[0x6648e9]
[0x7f1a5af000f0]
[0x7f1a5a10f0f2]
[0x7412cb]
[0x688354]
[0x688494]
[0x67a170]
[0x67f0ad]
[0x67fdf8]
[0x6811b6]
[0x66e05e]
```

较新版本的`glibc`堆栈跟踪函数也会打印相对于对象的地址。在基于`glibc`的系统（Linux）上，插件意外退出的跟踪看起来像这样：

```sql
plugin/auth/auth_test_plugin.so(+0x9a6)[0x7ff4d11c29a6]
```

要将相对地址(`+0x9a6`)翻译成文件名和行号，请使用以下命令：

```sql
$> addr2line -fie auth_test_plugin.so 0x9a6
auth_test_plugin
mysql-trunk/plugin/auth/test_plugin.c:65
```

**addr2line**实用程序是 Linux 上`binutils`软件包的一部分。

在 Solaris 上，该过程类似。Solaris 的`printstack()`已经打印了相对地址：

```sql
plugin/auth/auth_test_plugin.so:0x1510
```

要进行翻译，请使用以下命令：

```sql
$> gaddr2line -fie auth_test_plugin.so 0x1510
mysql-trunk/plugin/auth/test_plugin.c:88
```

Windows 已经打印了地址、函数名和行号：

```sql
000007FEF07E10A4 auth_test_plugin.dll!auth_test_plugin()[test_plugin.c:72]
```
