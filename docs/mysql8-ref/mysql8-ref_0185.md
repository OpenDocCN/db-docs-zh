> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-tips.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-tips.html)

#### 6.5.1.6 mysql 客户端技巧

本节提供有关更有效使用 **mysql** 和 **mysql** 操作行为的技术信息。

+   输入行编辑

+   禁用交互式历史记录

+   Windows 上的 Unicode 支持

+   垂直显示查询结果

+   使用安全更新模式（--safe-updates）

+   禁用 mysql 自动重新连接

+   mysql 客户端解析器与服务器解析器

##### 输入行编辑

**mysql** 支持输入行编辑，使您能够就地修改当前输入行或回忆先前输入的行。例如，**左箭头** 和 **右箭头** 键在当前输入行内水平移动，**上箭头** 和 **下箭头** 键在先前输入的行集合中上下移动。**退格键** 删除光标前的字符，输入新字符会在光标位置输入。要输入行，按 **Enter** 键。

在 Windows 上，编辑键序列与控制台窗口中支持的命令编辑相同。在 Unix 上，键序列取决于用于构建 **mysql** 的输入库（例如，`libedit` 或 `readline` 库）。

`libedit` 和 `readline` 库的文档可在线获取。要更改给定输入库允许的键序列集合，请在库启动文件中定义键绑定。这是位于您的主目录中的文件：对于 `libedit` 是 `.editrc`，对于 `readline` 是 `.inputrc`。

例如，在 `libedit` 中，**Control+W** 删除当前光标位置之前的所有内容，**Control+U** 删除整行。在 `readline` 中，**Control+W** 删除光标前的单词，**Control+U** 删除当前光标位置之前的所有内容。如果 **mysql** 是使用 `libedit` 构建的，喜欢这两个键的 `readline` 行为的用户可以将以下行放入 `.editrc` 文件中（必要时创建文件）：

```sql
bind "^W" ed-delete-prev-word
bind "^U" vi-kill-line-prev
```

要查看当前的键绑定集，请在`.editrc`的末尾暂时放置一行只写`bind`。**mysql** 在启动时显示绑定。

##### 禁用交互式历史记录

**上箭头**键可让您回忆当前和以前会话中的输入行。在共享控制台的情况下，此行为可能不合适。**mysql**支持根据主机平台部分或完全禁用交互式历史记录。

在 Windows 上，历史记录存储在内存中。**Alt+F7**删除当前历史缓冲区中存储的所有输入行。它还删除了显示在输入行前面的顺序号列表，该列表可以通过**F7**显示，并通过**F9**按编号召回。按下**Alt+F7**后输入的新输入行将重新填充当前历史缓冲区。清除缓冲区不会阻止记录到 Windows 事件查看器，如果使用`--syslog`选项启动**mysql**。关闭控制台窗口也会清除当前历史缓冲区。

要在 Unix 上禁用交互式历史记录，首先删除`.mysql_history`文件（如果存在的话，否则将回忆以前的条目）。然后使用`--histignore="*"`选项启动**mysql**以忽略所有新的输入行。要重新启用回忆（和记录）行为，请重新启动**mysql**而不使用该选项。

如果阻止创建`.mysql_history`文件（请参见控制历史文件），并使用`--histignore="*"`启动**mysql**客户端，则完全禁用交互式历史记录设施。或者，如果省略`--histignore`选项，则可以回忆在当前会话期间输入的输入行。

##### Windows 上的 Unicode 支持

Windows 提供基于 UTF-16LE 的 API，用于从控制台读取和写入；Windows 上的**mysql**客户端可以使用这些 API。Windows 安装程序在 MySQL 菜单中创建一个名为`MySQL command line client - Unicode`的项目。此项目调用**mysql**客户端，并设置属性以通过控制台使用 Unicode 与 MySQL 服务器通信。

要手动利用此支持，请在使用兼容 Unicode 字体的控制台中运行**mysql**，并将默认字符集设置为与服务器通信支持的 Unicode 字符集：

1.  打开一个控制台窗口。

1.  转到控制台窗口属性，选择字体选项卡，并选择 Lucida Console 或其他兼容的 Unicode 字体。这是必要的，因为控制台窗口默认使用不适合 Unicode 的 DOS 点阵字体。

1.  执行**mysql.exe**时，使用`--default-character-set=utf8mb4`（或`utf8mb3`）选项。这个选项是必需的，因为`utf16le`是不能用作客户端字符集的字符集之一。参见不允许的客户端字符集。

通过这些更改，**mysql**使用 Windows API 与控制台使用 UTF-16LE 进行通信，并使用 UTF-8 与服务器进行通信。（前面提到的菜单项设置了如上述的字体和字符集。）

为了避免每次运行**mysql**都要执行这些步骤，你可以创建一个快捷方式来调用**mysql.exe**。这个快捷方式应该将控制台字体设置为 Lucida Console 或其他兼容的 Unicode 字体，并传递`--default-character-set=utf8mb4`（或`utf8mb3`）选项给**mysql.exe**。

或者，创建一个仅设置控制台字体的快捷方式，并在你的`my.ini`文件的`[mysql]`组中设置字符集：

```sql
[mysql]
default-character-set=utf8mb4 # or utf8mb3
```

##### 垂直显示查询结果

一些查询结果以垂直显示比通常的水平表格格式更易读。可以通过在查询末尾使用\G 而不是分号来垂直显示查询。例如，包含换行符的较长文本值通常使用垂直输出更容易阅读：

```sql
mysql> SELECT * FROM mails WHERE LENGTH(txt) < 300 LIMIT 300,1\G
*************************** 1\. row ***************************
  msg_nro: 3068
     date: 2000-03-01 23:29:50
time_zone: +0200
mail_from: Jones
    reply: jones@example.com
  mail_to: "John Smith" <smith@example.com>
      sbj: UTF-8
      txt: >>>>> "John" == John Smith writes: 
John> Hi.  I think this is a good idea.  Is anyone familiar
John> with UTF-8 or Unicode? Otherwise, I'll put this on my
John> TODO list and see what happens.

Yes, please do that.

Regards,
Jones
     file: inbox-jani-1
     hash: 190402944
1 row in set (0.09 sec)
```

##### 使用安全更新模式（--safe-updates）

对于初学者，一个有用的启动选项是`--safe-updates`（或`--i-am-a-dummy`，效果相同）。安全更新模式对于可能已经发出`UPDATE`或`DELETE`语句但忘记在`WHERE`子句中指定要修改的行时很有帮助。通常，这些语句会更新或删除表中的所有行。使用`--safe-updates`，您只能通过指定标识它们的键值、`LIMIT`子句或两者来修改行。这有助于防止意外发生。安全更新模式还限制产生（或估计产生）非常大结果集的`SELECT`语句。

`--safe-updates`选项在连接到 MySQL 服务器时，会让**mysql**执行以下语句，以设置`sql_safe_updates`、`sql_select_limit`和`max_join_size`系统变量的会话值：

```sql
SET sql_safe_updates=1, sql_select_limit=1000, max_join_size=1000000;
```

`SET`语句会影响语句处理如下：

+   启用`sql_safe_updates`会导致`UPDATE`和`DELETE`语句如果在`WHERE`子句中未指定关键约束，或提供`LIMIT`子句，或两者都没有，则产生错误。例如：

    ```sql
    UPDATE *tbl_name* SET *not_key_column*=*val* WHERE *key_column*=*val*;

    UPDATE *tbl_name* SET *not_key_column*=*val* LIMIT 1;
    ```

+   将`sql_select_limit`设置为 1,000 会导致服务器将所有`SELECT`结果集限制为 1,000 行，除非语句包含`LIMIT`子句。

+   将`max_join_size`设置为 1,000,000 会导致多表`SELECT`语句在服务器估计必须检查超过 1,000,000 行组合时产生错误。

若要指定不同于 1,000 和 1,000,000 的结果集限制，可以在调用**mysql**时使用`--select-limit`和`--max-join-size`选项覆盖默认值：

```sql
mysql --safe-updates --select-limit=500 --max-join-size=10000
```

即使在`WHERE`子句中指定了关键列，如果优化器决定不使用关键列上的索引，则`UPDATE`和`DELETE`语句在安全更新模式下也可能产生错误：

+   如果内存使用超过`range_optimizer_max_mem_size`系统变量允许的范围，则无法使用索引上的范围访问。优化器然后退回到表扫描。请参阅限制范围优化的内存使用。

+   如果键比较需要类型转换，则可能不使用索引（参见第 10.3.1 节，“MySQL 如何使用索引”）。假设一个索引的字符串列`c1`与数值`2222`进行比较，使用`WHERE c1 = 2222`。对于这样的比较，字符串值会转换为数字，然后进行数值比较（参见第 14.3 节，“表达式求值中的类型转换”），从而阻止使用索引。如果启用了安全更新模式，将会产生错误。

截至 MySQL 8.0.13，安全更新模式还包括以下行为：

+   `UPDATE`和`DELETE`语句的`EXPLAIN`不会产生安全更新错误。这使得可以使用`EXPLAIN`加上`SHOW WARNINGS`来查看为什么不使用索引，这在出现`range_optimizer_max_mem_size`违规或类型转换并且优化器没有使用索引的情况下非常有帮助，即使在`WHERE`子句中指定了关键列。

+   当发生安全更新错误时，错误消息包含首次生成的诊断信息，以提供关于失败原因的信息。例如，消息可能指出`range_optimizer_max_mem_size`值超过限制或发生类型转换，这两种情况都可能导致无法使用索引。

+   对于多表删除和更新，仅当任何目标表使用表扫描时，启用安全更新时才会产生错误。

##### 禁用 mysql 自动重新连接

如果**mysql**客户端在发送语句时失去与服务器的连接，它会立即自动尝试重新连接到服务器并重新发送语句。然而，即使**mysql**成功重新连接，你的第一个连接已经结束，所有之前的会话对象和设置都会丢失：临时表、自动提交模式以及用户定义和会话变量。此外，任何当前事务都会回滚。这种行为可能对你很危险，就像以下示例中，在第一条和第二条语句之间服务器被关闭并重新启动而你并不知情的情况下：

```sql
mysql> SET @a=1;
Query OK, 0 rows affected (0.05 sec)

mysql> INSERT INTO t VALUES(@a);
ERROR 2006: MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    1
Current database: test Query OK, 1 row affected (1.30 sec)

mysql> SELECT * FROM t;
+------+
| a    |
+------+
| NULL |
+------+
1 row in set (0.05 sec)
```

`@a`用户变量在连接断开后已丢失，并在重新连接后未定义。如果重要的是在连接丢失时让**mysql**以错误终止，可以使用`--skip-reconnect`选项启动**mysql**客户端。

有关自动重新连接及其在重新连接发生时对状态信息的影响的更多信息，请参阅 Automatic Reconnection Control。

##### mysql 客户端解析器与服务器解析器

**mysql**客户端在客户端端使用一个解析器，这个解析器不是**mysqld**服务器在服务器端使用的完整解析器的副本。这可能导致对某些结构的处理方式不同。例如：

+   如果启用了`ANSI_QUOTES` SQL 模式，服务器解析器会将由`"`字符界定的字符串视为标识符而不是普通字符串。

    **mysql**客户端解析器不考虑`ANSI_QUOTES` SQL 模式。它会将由`"`, `'`, 和```字符界定的字符串视为相同，无论是否启用了`ANSI_QUOTES`。

+   在`/*! ... */`和`/*+ ... */`注释中，**mysql**客户端解析器会解释简短形式的**mysql**命令。服务器解析器不会解释它们，因为这些命令在服务器端没有意义。

    如果希望**mysql**不解释注释中的简短命令，部分解决方法是使用`--binary-mode`选项，该选项会导致所有**mysql**命令在非交互模式下被禁用，除了`\C`和`\d`（对于通过管道输入到**mysql**或使用`source`命令加载的输入）。
