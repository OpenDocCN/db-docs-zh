# 8.1.6 `LOAD DATA LOCAL`的安全考虑

> 原文：[`dev.mysql.com/doc/refman/8.0/en/load-data-local-security.html`](https://dev.mysql.com/doc/refman/8.0/en/load-data-local-security.html)

`LOAD DATA`语句将数据文件加载到表中。该语句可以加载位于服务器主机上的文件，或者如果指定了`LOCAL`关键字，则可以加载位于客户端主机上的文件。

`LOAD DATA`的`LOCAL`版本存在两个潜在的安全问题：

+   因为`LOAD DATA LOCAL`是一个 SQL 语句，解析发生在服务器端，并且文件从客户端主机传输到服务器主机是由 MySQL 服务器发起的，MySQL 服务器告诉客户端语句中命名的文件。理论上，一个经过修补的服务器可以告诉客户端程序传输服务器选择的文件，而不是语句中命名的文件。这样的服务器可以访问客户端用户具有读取访问权限的任何文件。（事实上，一个经过修补的服务器实际上可以对任何语句回复文件传输请求，而不仅仅是`LOAD DATA LOCAL`，因此更根本的问题是客户端不应连接到不受信任的服务器。）

+   在 Web 环境中，客户端是从 Web 服务器连接的，用户可以使用`LOAD DATA LOCAL`读取 Web 服务器进程具有读取访问权限的任何文件（假设用户可以对 SQL 服务器运行任何语句）。在这种环境中，客户端实际上是 Web 服务器，而不是由连接到 Web 服务器的用户运行的远程程序。

为了避免连接到不受信任的服务器，客户端可以通过使用`--ssl-mode=VERIFY_IDENTITY`选项和适当的 CA 证书建立安全连接并验证服务器身份。要实现这种级别的验证，您必须首先确保服务器的 CA 证书可靠地可供副本使用，否则将导致可用性问题。有关更多信息，请参见加密连接的命令选项。

为了避免`LOAD DATA`问题，客户端应避免使用`LOCAL`，除非已采取适当的客户端端预防措施。

为了控制本地数据加载，MySQL 允许启用或禁用该功能。此外，从 MySQL 8.0.21 开始，MySQL 使客户端能够将本地数据加载操作限制为位于指定目录中的文件。

+   启用或禁用本地数据加载功能

+   限制本地数据加载的文件

+   MySQL Shell 和本地数据加载

#### 启用或禁用本地数据加载功能

管理员和应用程序可以配置是否允许本地数据加载如下：

+   在服务器端：

    +   `local_infile`系统变量控制服务器端的`LOCAL`功能。根据`local_infile`设置，服务器拒绝或允许请求本地数据加载的客户端的本地数据加载。

    +   默认情况下，`local_infile`被禁用。（这是与 MySQL 先前版本的更改。）为了使服务器明确拒绝或允许`LOAD DATA LOCAL`语句（无论客户端程序和库在构建时或运行时如何配置），请使用禁用或启用`local_infile`启动**mysqld**。`local_infile`也可以在运行时设置。

+   在客户端：

    +   `ENABLED_LOCAL_INFILE` **CMake**选项控制 MySQL 客户端库的编译默认`LOCAL`功能（参见 Section 2.8.7，“MySQL 源配置选项”）。因此，未做明确安排的客户端根据 MySQL 构建时指定的`ENABLED_LOCAL_INFILE`设置，`LOCAL`功能被禁用或启用。

    +   默认情况下，MySQL 二进制发行版中的客户端库编译时禁用了`ENABLED_LOCAL_INFILE`。如果从源代码编译 MySQL，请根据未做明确安排的客户端是否应该禁用或启用`LOCAL`功能，配置`ENABLED_LOCAL_INFILE`为禁用或启用。

    +   对于使用 C API 的客户端程序，本地数据加载功能取决于编译到 MySQL 客户端库中的默认值。要显式启用或禁用它，请调用`mysql_options()` C API 函数来禁用或启用`MYSQL_OPT_LOCAL_INFILE`选项。参见 mysql_options()。

    +   对于**mysql**客户端，本地数据加载功能由默认编译到 MySQL 客户端库中。要显式禁用或启用它，请使用`--local-infile=0`或[`--local-infile[=1]`](mysql-command-options.html#option_mysql_local-infile)选项。

    +   对于**mysqlimport**客户端，默认情况下不使用本地数据加载。要显式禁用或启用它，请使用`--local=0`或[`--local[=1]`](mysqlimport.html#option_mysqlimport_local)选项。

    +   如果您在读取选项文件中的`[client]`组的 Perl 脚本或其他程序中使用`LOAD DATA LOCAL`，您可以向该组添加一个`local-infile`选项设置。为了防止不理解此选项的程序出现问题，请使用`loose-`前缀指定它：

        ```sql
        [client]
        loose-local-infile=0
        ```

        或：

        ```sql
        [client]
        loose-local-infile=1
        ```

    +   在所有情况下，客户端成功使用`LOCAL`加载操作还需要服务器允许本地加载。

如果`LOCAL`功能被禁用，无论是在服务器端还是客户端端，尝试发出`LOAD DATA LOCAL`语句的客户端将收到以下错误消息：

```sql
ERROR 3950 (42000): Loading local data is disabled; this must be
enabled on both the client and server side
```

#### 限制允许用于本地数据加载的文件

截至 MySQL 8.0.21，MySQL 客户端库使客户端应用程序能够将本地数据加载操作限制为位于指定目录中的文件。某些 MySQL 客户端程序利用了这一功能。

使用 C API 的客户端程序可以通过`mysql_options()` C API 函数的`MYSQL_OPT_LOCAL_INFILE`和`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`选项来控制允许加载数据的文件（参见 mysql_options()）。

`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`的效果取决于`LOCAL`数据加载是启用还是禁用的：

+   如果通过默认在 MySQL 客户端库中或显式启用`MYSQL_OPT_LOCAL_INFILE`来启用`LOCAL`数据加载，则`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`选项不起作用。

+   如果通过默认在 MySQL 客户端库中或显式禁用`MYSQL_OPT_LOCAL_INFILE`来禁用`LOCAL`数据加载，则可以使用`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`选项指定允许本地加载文件的目录。在这种情况下，`LOCAL`数据加载被允许但仅限于位于指定目录中的文件。`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`值的解释如下：

    +   如果该值为空指针（默认值），则不指定任何目录，结果是不允许任何文件进行`LOCAL`数据加载。

    +   如果值是目录路径名，则允许`LOCAL`数据加载，但仅限于位于指定目录中的文件。无论基础文件系统的大小写敏感性如何，目录路径名和要加载的文件的路径名的比较都是区分大小写的。

MySQL 客户端程序如下使用前述的`mysql_options()`选项：

+   **mysql**客户端有一个`--load-data-local-dir`选项，接受一个目录路径或空字符串。**mysql**使用该选项值来设置`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`选项（将空字符串设置为 null 指针）。`--load-data-local-dir`的效果取决于是否启用了`LOCAL`数据加载：

    +   如果启用了`LOCAL`数据加载，无论是在 MySQL 客户端库中默认设置，还是通过指定[`--local-infile[=1]`](mysql-command-options.html#option_mysql_local-infile)，都会忽略`--load-data-local-dir`选项。

    +   如果`LOCAL`数据加载被禁用，无论是在 MySQL 客户端库中默认设置，还是通过指定`--local-infile=0`，都会应用`--load-data-local-dir`选项。

    当应用`--load-data-local-dir`时，选项值指定了本地数据文件必须位于的目录。无论基础文件系统的大小写敏感性如何，目录路径名和要加载的文件的路径名的比较都是区分大小写的。如果选项值为空字符串，则不指定任何目录，结果是不允许进行本地数据加载。

+   **mysqlimport**为其处理的每个文件设置`MYSQL_OPT_LOAD_DATA_LOCAL_DIR`，以便包含文件的目录是允许的本地加载目录。

+   对应于`LOAD DATA`语句的数据加载操作，**mysqlbinlog**从二进制日志事件中提取文件，将它们写入本地文件系统作为临时文件，并写入`LOAD DATA LOCAL`语句以使文件被加载。默认情况下，**mysqlbinlog**将这些临时文件写入操作系统特定的目录。可以使用`--local-load`选项来明确指定**mysqlbinlog**应准备本地临时文件的目录。

    因为其他进程可以将文件写入默认的系统特定目录，建议指定`--local-load`选项给**mysqlbinlog**，以指定不同的目录用于数据文件，然后通过在处理来自**mysqlbinlog**的输出时指定`--load-data-local-dir`选项给**mysql**来指定相同的目录。

#### MySQL Shell 和本地数据加载

MySQL Shell 提供了许多实用程序来转储表、模式或服务器实例，并将它们加载到其他实例中。当您使用这些实用程序处理数据时，MySQL Shell 提供额外的功能，如输入预处理、多线程并行加载、文件压缩和解压缩，以及处理访问 Oracle Cloud Infrastructure 对象存储桶的功能。为了获得最佳功能，请始终使用 MySQL Shell 的最新版本的转储和加载实用程序。

MySQL Shell 的数据上传实用程序使用`LOAD DATA LOCAL INFILE`语句来上传数据，因此目标服务器实例上必须将`local_infile`系统变量设置为`ON`。您可以在上传数据之前执行此操作，然后再次将其删除。这些实用程序安全地处理文件传输请求，以处理本主题中讨论的安全考虑。

MySQL Shell 包括以下转储和加载实用程序：

表导出工具 `util.exportTable()`

将 MySQL 关系表导出为数据文件，可以使用 MySQL Shell 的并行表导入实用程序上传到 MySQL 服务器实例，导入到不同的应用程序，或用作逻辑备份。该实用程序具有预设选项和自定义选项，可生成不同的输出格式。

并行表导入实用程序 `util.importTable()`

将数据文件导入到 MySQL 关系表中。数据文件可以是 MySQL Shell 的表导出实用程序的输出，也可以是实用程序的预设和自定义选项支持的其他格式。该实用程序可以在将数据添加到表之前进行输入预处理。它可以接受多个数据文件合并到单个关系表中，并自动解压缩压缩文件。

实例转储实用程序 `util.dumpInstance()`，模式转储实用程序 `util.dumpSchemas()` 和表转储实用程序 `util.dumpTables()`

将实例、模式或表导出为一组转储文件，然后可以使用 MySQL Shell 的转储加载实用程序将其上传到 MySQL 实例。这些实用程序提供 Oracle Cloud 基础设施对象存储流、MySQL HeatWave 服务兼容性检查和修改，并能够进行干跑以在继续进行转储之前识别问题。

转储加载实用程序 `util.loadDump()`

使用 MySQL Shell 的实例、模式或表转储实用程序创建的转储文件导入到 MySQL HeatWave 服务 DB 系统或 MySQL 服务器实例中。该实用程序管理上传过程，并提供从远程存储的数据流、表或表块的并行加载、进度状态跟踪、恢复和重置功能，以及在转储仍在进行时进行并发加载的选项。MySQL Shell 的并行表导入实用程序可以与转储加载实用程序结合使用，在将数据上传到目标 MySQL 实例之前修改数据。

有关实用程序的详细信息，请参见 MySQL Shell 实用程序。
