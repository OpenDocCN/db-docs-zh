# 6.2.2 指定程序选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/program-options.html`](https://dev.mysql.com/doc/refman/8.0/en/program-options.html)

6.2.2.1 在命令行上使用选项

6.2.2.2 使用选项文件

6.2.2.3 影响选项文件处理的命令行选项

6.2.2.4 程序选项修饰符

6.2.2.5 使用选项设置程序变量

6.2.2.6 选项默认值，期望值的选项和=符号

有几种方法可以为 MySQL 程序指定选项：

+   在程序名称后面列出命令行上的选项。这对于适用于程序特定调用的选项很常见。

+   在程序启动时读取的选项文件中列出选项。这对于您希望程序每次运行时使用的选项很常见。

+   在环境变量中列出选项（参见第 6.2.9 节，“设置环境变量”）。这种方法适用于您希望每次程序运行时应用的选项。在实践中，选项文件更常用于此目的，但第 7.8.3 节，“在 Unix 上运行多个 MySQL 实例”讨论了一种情况，其中环境变量可以非常有用。它描述了一种使用这些变量指定服务器和客户端程序的 TCP/IP 端口号和 Unix 套接字文件的方便技术。

选项按顺序处理，因此如果一个选项被多次指定，最后一次出现的选项优先。以下命令使**mysql**连接到运行在`localhost`上的服务器：

```sql
mysql -h example.com -h localhost
```

有一个例外情况：对于**mysqld**，第一个`--user`选项实例被用作安全预防措施，以防止在选项文件中指定的用户被在命令行上覆盖。

如果给出冲突或相关选项，后续选项优先于先前选项。以下命令以“无列名”模式运行**mysql**：

```sql
mysql --column-names --skip-column-names
```

MySQL 程序通过检查环境变量确定首先给出哪些选项，然后通过处理选项文件，最后通过检查命令行。因为后续选项优先于先前选项，处理顺序意味着环境变量具有最低优先级，命令行选项具有最高优先级。

对于服务器，有一个例外情况：数据目录中的**mysqld-auto.cnf**选项文件最后处理，因此甚至优先于命令行选项。

你可以利用 MySQL 程序处理选项的方式，通过在选项文件中指定程序的默认选项值。这样一来，你就可以避免每次运行程序时都输入它们，同时又可以通过使用命令行选项在必要时覆盖默认值。
