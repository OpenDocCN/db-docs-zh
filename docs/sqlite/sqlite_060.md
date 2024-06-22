# 1\. SQLite 中的 URI 文件名

> 原文：[`sqlite.com/uri.html`](https://sqlite.com/uri.html)

自 version 3.7.7 (2011-06-23)起，可以将 SQLite 数据库文件参数指定为普通文件名或统一资源标识符（URI）。使用 URI 文件名的优点是可以使用 URI 上的查询参数来控制新创建的数据库连接的详细信息。例如，可以使用"vfs="查询参数指定替代 VFS，或者使用"mode=ro"作为查询参数以只读方式打开数据库。

# 2\. 向后兼容性

为了保持对旧应用程序的完全向后兼容性，默认情况下禁用了 URI 文件名功能。可以使用 SQLITE_USE_URI=1 或 SQLITE_USE_URI=0 编译时选项启用或禁用 URI 文件名。可以使用 sqlite3_config(SQLITE_CONFIG_URI,1)或 sqlite3_config(SQLITE_CONFIG_URI,0)配置调用在启动时更改 URI 文件名的编译时设置。无论编译时还是启动时设置如何，可以通过在传递给 sqlite3_open_v2(N,P,F,V)的 F 参数中包含 SQLITE_OPEN_URI 位来为单个数据库连接启用 URI 文件名。

如果在初始打开数据库连接时识别了 URI 文件名，则在 ATTACH 语句中也将识别 URI 文件名。同样，如果在初始打开数据库连接时未识别 URI 文件名，则 ATTACH 也将不会识别它们。

由于 SQLite 始终将任何不以"`file:`"开头的文件名解释为普通文件名，而不考虑 URI 设置，因为实际上以"`file:`"开头的文件名非常不寻常，大多数应用即使不使用 URI 文件名，也可以安全地启用 URI 处理。

# 3\. URI 格式

根据[RFC 3986](http://tools.ietf.org/html/rfc3986)，URI 包括方案、权限、路径、查询字符串和片段。方案始终是必需的。权限或路径中的一个也是必需的。查询字符串和片段是可选的。

SQLite 使用 "`file:`" URI 语法来标识数据库文件。SQLite 力求以与流行的网络浏览器（如[Firefox](http://www.mozilla.com/en-US/firefox/new/)、[Chrome](http://www.google.com/chrome/)、[Safari](http://www.apple.com/safari/)、[Internet Explorer](http://windows.microsoft.com/en-US/internet-explorer/products/ie/home)和[Opera](http://www.opera.com/)）以及命令行程序（如 Windows 的 ["cmd start"](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/start) 或 ["powershell start"](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process?view=powershell-7.3)、macOS 的 "open" 或 Linux 的 "xdg-open" 命令）相同的方式解释 file: URIs。以下是 URI 解析规则的简要摘要：

+   URI 的 scheme 必须是 "`file:`"。任何其他 scheme 将导致输入被视为普通文件名。

+   权威部分可以省略，可以为空，或者可以是 "`localhost`"。任何其他的权威部分将导致错误。例外：如果 SQLite 编译时启用了 SQLITE_ALLOW_URI_AUTHORITY，则除了 "localhost" 之外的任何权威值都将作为 UNC 文件名传递到底层操作系统。

+   如果存在权威部分，则路径是可选的。如果省略了权威部分，则路径是必需的。

+   查询字符串是可选的。如果存在查询字符串，则所有查询参数都将传递到底层 VFS 的 xOpen 方法中。

+   片段是可选的。如果存在，则将其忽略。

路径、查询字符串或片段中可以出现零个或多个形式为 "**%*HH***" 的转义序列（其中 ***H*** 表示任何十六进制数字）。

不是格式良好的 URI 的文件名被解释为普通文件名。

URI 被处理为 UTF8 文本。文件名参数 sqlite3_open16() 在处理之前从 UTF16 本机字节顺序转换为 UTF8。

## 3.1\. URI 路径

URI 的路径组件指定了要打开的 SQLite 数据库的磁盘文件。如果省略了路径组件，则数据库存储在临时文件中，在数据库连接关闭时将自动删除。如果存在权威部分，则路径始终是绝对路径名。如果省略了权威部分，则如果路径以 "/" 字符（ASCII 代码 0x2f）开头，则路径是绝对路径名，否则是相对路径名。在 Windows 上，如果绝对路径以 "**/*X*:/**" 开头，其中 ***X*** 是任何单个 ASCII 字母字符（"a" 到 "z" 或 "A" 到 "Z"），则 "***X:***" 被理解为包含文件的卷的驱动器号码，而不是顶级目录。

普通文件名通常可以通过下面显示的步骤转换为等效的 URI。唯一的例外是，带有驱动器字母的相对 Windows 路径名不能直接转换为 URI；必须首先更改为绝对路径名。

1.  将所有 "`?`" 字符转换为 "`%3f`"。

1.  将所有 "`#`" 字符转换为 "`%23`"。

1.  仅在 Windows 上，将所有 "`\`" 字符转换为 "`/`"。

1.  将所有两个或更多 "`/`" 字符的序列转换为单个 "`/`" 字符。

1.  仅在 Windows 上，如果文件名以驱动器字母开头，则在前面添加单个 "`/`" 字符。

1.  在 "`file:`" 方案前添加。

## 3.2\. 查询字符串

URI 文件名可以选择后跟查询字符串。查询字符串由第一个 "`?`" 字符后的文本组成，但不包括以 "`#`" 开头的可选片段。查询字符串分为键值对。我们通常将这些键值对称为"查询参数"。键值对由单个 "`&`" 字符分隔。键位于首位，并由单个 "`=`" 字符与值分隔。键和值都可以包含 **%HH** 转义序列。

查询参数的文本附加到 VFS 的 xOpen 方法的文件名参数上。在附加到 xOpen 文件名之前，解析查询参数中的所有 %HH 转义序列。单个零字节将 xOpen 文件名参数与第一个查询参数的键、每个键值对及每个前一个值的后续键分隔开。附加到 xOpen 文件名的查询参数列表以单个零长度键终止。请注意，查询参数的值可以是空字符串。

## 3.3\. 已识别的查询参数

SQLite 核心解释一些查询参数，并用于修改新连接的特性。即使 SQLite 核心已经读取和解释了它们，所有查询参数也始终通过 VFS 的 xOpen 方法传递进去。

以下查询参数被 SQLite 自 版本 3.15.0（2016-10-14）起识别。未来可能会添加新的查询参数。

**cache=shared**

**cache=private**

缓存查询参数确定是否使用 共享缓存模式 或私有缓存打开新数据库。

**immutable=1**

不可变查询参数是一个布尔值，用于向 SQLite 表示底层数据库文件保存在只读媒体上，即使是通过具有提升权限的其他进程也无法修改。SQLite 总是以只读方式打开不可变的数据库文件，并且在不可变的数据库文件上跳过所有文件锁定和更改检测。如果这个查询参数（或者在 xDeviceCharacteristics 中的 SQLITE_IOCAP_IMMUTABLE 位）断言数据库文件是不可变的，而文件还是发生了变化，那么 SQLite 可能会返回不正确的查询结果和/或 SQLITE_CORRUPT 错误。

**mode=ro**

**mode=rw**

mode=rwc

**mode=memory**

mode 查询参数确定新数据库是以只读、读写、读写并在不存在时创建，还是作为永不与磁盘交互的纯内存数据库打开。

**modeof=***filename*

在 Unix 系统上使用 sqlite3_open_v2() 创建新数据库文件时，SQLite 尝试设置新数据库文件的权限以匹配现有文件 "*filename*"。

**nolock=1**

nolock 查询参数是一个布尔值，当设置为 true 时，会禁用 VFS 的 xLock、xUnlock 和 xCheckReservedLock 方法的所有调用。例如，在尝试访问不支持文件锁定的文件系统上的文件时，可以使用 nolock 查询参数。注意：如果两个或更多的 数据库连接 尝试与同一 SQLite 数据库交互，并且其中一个或多个连接已启用了 "nolock"，则可能导致数据库损坏。只有在应用程序可以保证对数据库的写入是串行化的情况下，才应该使用 "nolock" 查询参数。

**psow=0**

**psow=1**

psow 查询参数会覆盖正在打开的数据库文件的 powersafe overwrite 属性。psow 查询参数适用于默认的 Windows 和 Unix VFSes，但对于其他专有或非标准的 VFSes 可能不起作用。

**vfs=***NAME*

vfs 查询参数使得使用 VFS 名称为 *NAME* 的数据库连接打开。如果 *NAME* 不是 SQLite 内置的 VFS 或者之前使用 sqlite3_vfs_register() 注册过的 VFS 的名称，则打开尝试会失败。

# 4\. 参见

+   sqlite3_open() 中的 URI 文件名

+   URI 文件名示例
