# 6.10 MySQL 中的 Unix 信号处理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/unix-signal-response.html`](https://dev.mysql.com/doc/refman/8.0/en/unix-signal-response.html)

在 Unix 和类 Unix 系统上，一个进程可以接收由 `root` 系统帐户或拥有该进程的系统帐户发送的信号。信号可以使用 **kill** 命令发送。一些命令解释器将某些按键序列与信号关联起来，例如 **Control+C** 发送 `SIGINT` 信号。本节描述了 MySQL 服务器和客户端程序对信号的响应。

+   服务器对信号的响应

+   客户端对信号的响应

### 服务器对信号的响应

**mysqld** 对信号的响应如下：

+   `SIGTERM` 导致服务器关闭。这类似于执行 `SHUTDOWN` 语句而无需连接到服务器（对于关闭需要具有 `SHUTDOWN` 权限的帐户）。

+   `SIGHUP` 导致服务器重新加载授权表并刷新表、日志、线程缓存和主机缓存。这些操作类似于各种形式的 `FLUSH` 语句。发送信号使得刷新操作可以在不连接到服务器的情况下执行，这需要一个具有足够权限的 MySQL 帐户。在 MySQL 8.0.20 之前，服务器还会向错误日志写入一个具有以下格式的状态报告：

    ```sql
    Status information:

    Current dir: /var/mysql/data/
    Running threads: 4  Stack size: 262144
    Current locks:
    lock: 0x7f742c02c0e0:

    lock: 0x2cee2a20:
    :
    lock: 0x207a080:

    Key caches:
    default
    Buffer_size:       8388608
    Block_size:           1024
    Division_limit:        100
    Age_limit:             300
    blocks used:             4
    not flushed:             0
    w_requests:              0
    writes:                  0
    r_requests:              8
    reads:                   4

    handler status:
    read_key:           13
    read_next:           4
    read_rnd             0
    read_first:         13
    write:               1
    delete               0
    update:              0

    Table status:
    Opened tables:        121
    Open tables:          114
    Open files:            18
    Open streams:           0

    Memory status:
    <malloc version="1">
    <heap nr="0">
    <sizes>
      <size from="17" to="32" total="32" count="1"/>
      <size from="33" to="48" total="96" count="2"/>
      <size from="33" to="33" total="33" count="1"/>
      <size from="97" to="97" total="6014" count="62"/>
      <size from="113" to="113" total="904" count="8"/>
      <size from="193" to="193" total="193" count="1"/>
      <size from="241" to="241" total="241" count="1"/>
      <size from="609" to="609" total="609" count="1"/>
      <size from="16369" to="16369" total="49107" count="3"/>
      <size from="24529" to="24529" total="98116" count="4"/>
      <size from="32689" to="32689" total="32689" count="1"/>
      <unsorted from="241" to="7505" total="7746" count="2"/>
    </sizes>
    <total type="fast" count="3" size="128"/>
    <total type="rest" count="84" size="195652"/>
    <system type="current" size="690774016"/>
    <system type="max" size="690774016"/>
    <aspace type="total" size="690774016"/>
    <aspace type="mprotect" size="690774016"/>
    </heap>
    :
    <total type="fast" count="85" size="5520"/>
    <total type="rest" count="116" size="316820"/>
    <total type="mmap" count="82" size="939954176"/>
    <system type="current" size="695717888"/>
    <system type="max" size="695717888"/>
    <aspace type="total" size="695717888"/>
    <aspace type="mprotect" size="695717888"/>
    </malloc>

    Events status:
    LLA = Last Locked At  LUA = Last Unlocked At
    WOC = Waiting On Condition  DL = Data Locked

    Event scheduler status:
    State      : INITIALIZED
    Thread id  : 0
    LLA        : n/a:0
    LUA        : n/a:0
    WOC        : NO
    Workers    : 0
    Executed   : 0
    Data locked: NO

    Event queue status:
    Element count   : 0
    Data locked     : NO
    Attempting lock : NO
    LLA             : init_queue:95
    LUA             : init_queue:103
    WOC             : NO
    Next activation : never
    ```

+   从 MySQL 8.0.19 开始，`SIGUSR1` 导致服务器刷新错误日志、一般查询日志和慢查询日志。`SIGUSR1` 的一个用途是实现日志轮换而无需连接到服务器，这需要一个具有足够权限的 MySQL 帐户。有关日志轮换的信息，请参见 Section 7.4.6, “Server Log Maintenance”。

    服务器对 `SIGUSR1` 的响应是对 `SIGHUP` 的响应的一个子集，使得 `SIGUSR1` 可以被用作一个更“轻量级”的信号，刷新某些日志而不带有其他 `SIGHUP` 的效果，比如刷新线程和主机缓存以及向错误日志写入状态报告。

+   `SIGINT` 通常被服务器忽略。使用 `--gdb` 选项启动服务器会为 `SIGINT` 安装一个中断处理程序，用于调试目的。参见 Section 7.9.1.4, “Debugging mysqld under gdb”。

### 客户端对信号的响应

MySQL 客户端程序对信号的响应如下：

+   **mysql**客户端将`SIGINT`（通常是键入**Control+C**的结果）解释为中断当前语句的指令（如果有的话），或者取消任何部分输入行。可以使用`--sigint-ignore`选项来忽略`SIGINT`信号以禁用此行为。

+   使用 MySQL 客户端库的客户端程序默认会阻止`SIGPIPE`信号。有以下变化可能：

    +   客户端可以安装自己的`SIGPIPE`处理程序以覆盖默认行为。请参阅编写 C API 多线程客户端程序。

    +   客户端可以通过在连接时向`mysql_real_connect()`指定`CLIENT_IGNORE_SIGPIPE`选项来防止安装`SIGPIPE`处理程序。请参阅 mysql_real_connect()。
