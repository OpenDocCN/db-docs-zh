# 12.15 字符集配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-configuration.html)

MySQL 服务器具有编译默认字符集和排序规则。要更改这些默认值，请在启动服务器时使用`--character-set-server`和`--collation-server`选项。请参阅第 7.1.7 节，“服务器命令选项”。排序规则必须是默认字符集的合法排序规则。要确定每个字符集可用的排序规则，请使用`SHOW COLLATION`语句或查询`INFORMATION_SCHEMA` `COLLATIONS`表。

如果您尝试使用未编译到二进制文件中的字符集，可能会遇到以下问题：

+   如果您的程序使用不正确的路径来确定字符集存储的位置（通常是 MySQL 安装目录下的`share/mysql/charsets`或`share/charsets`目录），可以通过在运行程序时使用`--character-sets-dir`选项来解决。例如，要指定一个目录供 MySQL 客户端程序使用，请在您的选项文件的`[client]`组中列出它。这里给出的示例显示了在 Unix 或 Windows 中分别设置可能是什么样子的：

    ```sql
    [client]
    character-sets-dir=/usr/local/mysql/share/mysql/charsets

    [client]
    character-sets-dir="C:/Program Files/MySQL/MySQL Server 8.0/share/charsets"
    ```

+   如果字符集是一个无法动态加载的复杂字符集，您必须重新编译程序以支持该字符集。

    对于 Unicode 字符集，您可以使用 LDML 表示法定义排序规则，而无需重新编译。请参阅第 12.14.4 节，“向 Unicode 字符集添加 UCA 排序规则”。

+   如果字符集是动态字符集，但您没有配置文件，您应该从新的 MySQL 发行版中安装字符集的配置文件。

+   如果您的字符集索引文件（`Index.xml`）不包含字符集的名称，您的程序将显示错误消息：

    ```sql
    Character set '*charset_name*' is not a compiled character set and is not
    specified in the '/usr/share/mysql/charsets/Index.xml' file
    ```

    要解决这个问题，您应该获取一个新的索引文件或手动将任何缺失的字符集名称添加到当前文件中。

您可以强制客户端程序使用特定的字符集，如下所示：

```sql
[client]
default-character-set=*charset_name*
```

这通常是不必要的。然而，当`character_set_system`与`character_set_server`或`character_set_client`不同时，并且您手动输入字符（作为数据库对象标识符、列值或两者），这些字符可能会在客户端输出中显示不正确，或者输出本身可能格式不正确。在这种情况下，使用`--default-character-set=*`system_character_set`*`启动 mysql 客户端——即，将客户端字符集设置为与系统字符集匹配——应该可以解决问题。
