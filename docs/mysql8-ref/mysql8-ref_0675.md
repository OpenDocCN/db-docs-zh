# 12.3.2 服务器字符集和校对

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-server.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-server.html)

MySQL 服务器有一个服务器字符集和一个服务器校对。默认情况下，它们分别为`utf8mb4`和`utf8mb4_0900_ai_ci`，但可以在服务器启动时通过命令行或选项文件显式设置，并在运行时更改。

最初，服务器字符集和校对取决于启动**mysqld**时使用的选项。您可以使用`--character-set-server`设置字符集。此外，您还可以添加`--collation-server`设置校对。如果您没有指定字符集，那就相当于说`--character-set-server=utf8mb4`。如果您只指定了字符集（例如，`utf8mb4`）但没有指定校对，那就相当于说`--character-set-server=utf8mb4` `--collation-server=utf8mb4_0900_ai_ci`，因为`utf8mb4_0900_ai_ci`是`utf8mb4`的默认校对。因此，以下三个命令都具有相同的效果：

```sql
mysqld
mysqld --character-set-server=utf8mb4
mysqld --character-set-server=utf8mb4 \
  --collation-server=utf8mb4_0900_ai_ci
```

更改设置的一种方法是重新编译。在从源代码构建时更改默认服务器字符集和校对，可以使用**CMake**的`DEFAULT_CHARSET`和`DEFAULT_COLLATION`选项。例如：

```sql
cmake . -DDEFAULT_CHARSET=latin1
```

或：

```sql
cmake . -DDEFAULT_CHARSET=latin1 \
  -DDEFAULT_COLLATION=latin1_german1_ci
```

**mysqld**和**CMake**都会验证字符集/校对组合是否有效。如果无效，每个程序都会显示错误消息并终止。

如果在`CREATE DATABASE`语句中未指定数据库字符集和校对，服务器字符集和校对将用作默认值。它们没有其他用途。

可以通过`character_set_server`和`collation_server`系统变量的值来确定当前服务器字符集和校对。这些变量可以在运行时更改。
