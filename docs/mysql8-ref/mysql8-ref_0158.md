> 原文：[`dev.mysql.com/doc/refman/8.0/en/program-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/program-variables.html)

#### 6.2.2.5 使用选项设置程序变量

许多 MySQL 程序具有内部变量，可以使用`SET`语句在运行时设置。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”，以及 Section 7.1.9, “Using System Variables”。

这些程序变量中的大多数也可以通过使用适用于指定程序选项的相同语法在服务器启动时设置。例如，**mysql**有一个`max_allowed_packet`变量，控制其通信缓冲区的最大大小。要将**mysql**的`max_allowed_packet`变量设置为 16MB 的值，请使用以下任一命令：

```sql
mysql --max_allowed_packet=16777216
mysql --max_allowed_packet=16M
```

第一个命令以字节为单位指定值。第二个以兆字节为单位指定值。对于需要数字值的变量，值可以附加`K`、`M`或`G`后缀，表示 1024、1024²或 1024³的倍增器。（例如，用于设置`max_allowed_packet`时，后缀表示千字节、兆字节或千兆字节的单位。）从 MySQL 8.0.14 开始，后缀也可以是`T`、`P`和`E`，表示 1024⁴、1024⁵或 1024⁶的倍增器。后缀字母可以是大写或小写。

在选项文件中，变量设置不带前导破折号：

```sql
[mysql]
max_allowed_packet=16777216
```

或：

```sql
[mysql]
max_allowed_packet=16M
```

如果愿意，选项名称中的下划线可以指定为破折号。以下选项组是等效的。两者都将服务器的键缓冲区大小设置为 512MB：

```sql
[mysqld]
key_buffer_size=512M

[mysqld]
key-buffer-size=512M
```

使用后缀来指定值的倍增器可以在程序调用时设置变量，但不能在运行时使用`SET`来设置值。另一方面，使用`SET`，你可以使用表达式来赋值变量的值，在服务器启动时设置变量时不适用这一点。例如，以下行中的第一行在程序调用时是合法的，但第二行不是：

```sql
$> mysql --max_allowed_packet=16M
$> mysql --max_allowed_packet=16*1024*1024
```

相反，以下行中的第二行在运行时是合法的，但第一行不是：

```sql
mysql> SET GLOBAL max_allowed_packet=16M;
mysql> SET GLOBAL max_allowed_packet=16*1024*1024;
```
