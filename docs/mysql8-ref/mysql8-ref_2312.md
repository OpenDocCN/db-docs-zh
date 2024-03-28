> 原文：[`dev.mysql.com/doc/refman/8.0/en/cannot-initialize-character-set.html`](https://dev.mysql.com/doc/refman/8.0/en/cannot-initialize-character-set.html)

#### B.3.2.15 无法初始化字符集

如果您遇到字符集问题，可能会看到类似于这样的错误：

```sql
MySQL Connection Failed: Can't initialize character set *charset_name*
```

此错误可能有以下任一原因：

+   字符集是多字节字符集，而客户端不支持该字符集。在这种情况下，您需要通过使用**CMake**运行`-DDEFAULT_CHARSET=*`charset_name`*`选项重新编译客户端。参见第 2.8.7 节，“MySQL 源配置选项”。

    所有标准的 MySQL 二进制文件都是使用对所有多字节字符集的支持进行编译的。

+   字符集是一个简单的字符集，没有编译进**mysqld**，并且字符集定义文件不在客户端期望找到它们的位置。

    在这种情况下，您需要使用以下方法之一来解决问题：

    +   重新编译客户端以支持字符集。参见第 2.8.7 节，“MySQL 源配置选项”。

    +   指定客户端字符集定义文件所在的目录。对于许多客户端，您可以使用`--character-sets-dir`选项来实现这一点。

    +   将字符定义文件复制到客户端期望它们在的路径。
