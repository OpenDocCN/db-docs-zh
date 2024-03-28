# 6.7.1 mysql_config — 显示编译客户端的选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-config.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-config.html)

**mysql_config** 为您提供了有关编译 MySQL 客户端并连接到 MySQL 所需的有用信息。它是一个 shell 脚本，因此仅在 Unix 和类 Unix 系统上可用。

注意

**pkg-config** 可以用作获取编译 MySQL 应用程序所需的编译器标志或链接库等信息的替代方法，而不是 **mysql_config**。有关更多信息，请参阅使用 pkg-config 构建 C API 客户端程序。

**mysql_config** 支持以下选项。

+   `--cflags`

    用于查找包含文件和编译 `libmysqlclient` 库时使用的关键编译器标志和定义的 C 编译器标志。返回的选项与创建库时使用的特定编译器相关联，可能与您自己的编译器设置冲突。使用 `--include` 获取更具可移植性的选项，其中仅包含包含路径。

+   `--cxxflags`

    类似于 `--cflags`，但用于 C++ 编译器标志。

+   `--include`

    编译选项以查找 MySQL 包含文件。

+   `--libs`

    链接到 MySQL 客户端库所需的库和选项。

+   `--libs_r`

    链接到线程安全 MySQL 客户端库所需的库和选项。在 MySQL 8.0 中，所有客户端库都是线程安全的，因此不需要使用此选项。在所有情况下都可以使用 `--libs` 选项。

+   `--plugindir`

    配置 MySQL 时定义的默认插件目录路径名。

+   `--port`

    配置 MySQL 时定义的默认 TCP/IP 端口号。

+   `--socket`

    配置 MySQL 时定义的默认 Unix 套接字文件。

+   `--variable=*`var_name`*`

    显示命名配置变量的值。允许的 *`var_name`* 值为 `pkgincludedir`（头文件目录）、`pkglibdir`（库目录）和 `plugindir`（插件目录）。

+   `--version`

    MySQL 发行版的版本号。

如果你在没有任何选项的情况下调用**mysql_config**，它会显示它支持的所有选项及其值的列表：

```sql
$> mysql_config
Usage: /usr/local/mysql/bin/mysql_config [options]
Options:
  --cflags         [-I/usr/local/mysql/include/mysql -mcpu=pentiumpro]
  --cxxflags       [-I/usr/local/mysql/include/mysql -mcpu=pentiumpro]
  --include        [-I/usr/local/mysql/include/mysql]
  --libs           [-L/usr/local/mysql/lib/mysql -lmysqlclient
                    -lpthread -lm -lrt -lssl -lcrypto -ldl]
  --libs_r         [-L/usr/local/mysql/lib/mysql -lmysqlclient_r
                    -lpthread -lm -lrt -lssl -lcrypto -ldl]
  --plugindir      [/usr/local/mysql/lib/plugin]
  --socket         [/tmp/mysql.sock]
  --port           [3306]
  --version        [5.8.0-m17]
  --variable=VAR   VAR is one of:
          pkgincludedir [/usr/local/mysql/include]
          pkglibdir     [/usr/local/mysql/lib]
          plugindir     [/usr/local/mysql/lib/plugin]
```

你可以在命令行中使用反引号使用**mysql_config**来包含它为特定选项生成的输出。例如，要编译和链接一个 MySQL 客户端程序，可以按照以下方式使用**mysql_config**：

```sql
gcc -c `mysql_config --cflags` progname.c
gcc -o progname progname.o `mysql_config --libs`
```
