> 原文：[`dev.mysql.com/doc/refman/8.0/en/option-modifiers.html`](https://dev.mysql.com/doc/refman/8.0/en/option-modifiers.html)

#### 6.2.2.4 程序选项修饰符

一些选项是“布尔”类型的，控制可以打开或关闭的行为。例如，**mysql** 客户端支持一个 `--column-names` 选项，用于确定是否在查询结果开头显示一行列名。默认情况下，此选项已启用。但是，在某些情况下，您可能希望禁用它，例如当将 **mysql** 的输出发送到另一个只期望看到数据而不是初始标题行的程序时。

要禁用列名，可以使用以下任何形式指定该选项：

```sql
--disable-column-names
--skip-column-names
--column-names=0
```

`--disable` 和 `--skip` 前缀以及 `=0` 后缀都具有相同的效果：它们关闭选项。

选项的“启用”形式可以通过以下任何方式指定：

```sql
--column-names
--enable-column-names
--column-names=1
```

对于布尔选项，值 `ON`、`TRUE`、`OFF` 和 `FALSE` 也被识别（不区分大小写）。

如果一个选项以 `--loose` 为前缀，程序在不识别该选项时不会退出，而是只发出警告：

```sql
$> mysql --loose-no-such-option
mysql: WARNING: unknown option '--loose-no-such-option'
```

当您在同一台机器上从多个 MySQL 安装运行程序并在选项文件中列出选项时，`--loose` 前缀可能很有用。可以使用 `--loose` 前缀（或选项文件中的 `loose`）提供可能不被所有程序版本识别的选项。识别该选项的程序版本会正常处理它，而不识别它的版本会发出警告并忽略它。

`--maximum` 前缀仅适用于 **mysqld**，允许限制客户端程序设置会话系统变量的大小。要实现这一点，使用带有变量名的 `--maximum` 前缀。例如，`--maximum-max_heap_table_size=32M` 防止任何客户端将堆表大小限制设置为大于 32M。

`--maximum` 前缀用于具有会话值的系统变量。如果应用于仅具有全局值的系统变量，将会出现错误。例如，使用 `--maximum-back_log=200`，服务器会产生此错误：

```sql
Maximum value of 'back_log' cannot be set
```
