> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-source.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-source.html)

#### 25.3.2.2 在 Windows 上从源代码编译和安装 NDB Cluster

Oracle 为 Windows 提供了预编译的 NDB Cluster 二进制文件，对大多数用户来说应该足够了。但是，如果你愿意，也可以从源代码为 Windows 编译 NDB Cluster。这样做的过程几乎与用于为 Windows 编译标准 MySQL Server 二进制文件的过程相同，并且使用相同的工具。但是，有两个主要区别：

+   构建 MySQL NDB Cluster 8.0 需要使用 MySQL Server 8.0 源代码。这些源代码可以从 MySQL 下载页面获取，链接为[`dev.mysql.com/downloads/`](https://dev.mysql.com/downloads/)。存档的源文件应该类似于 `mysql-8.0.34.tar.gz`。你也可以从 GitHub 获取这些源代码，链接为[`github.com/mysql/mysql-server`](https://github.com/mysql/mysql-server)。

+   你必须在**CMake**中使用`WITH_NDB`选项来配置构建，以及任何其他你希望与之一起使用的构建选项。`WITH_NDBCLUSTER`也支持用于向后兼容，但在 NDB 8.0.31 中已被弃用。

重要提示

`WITH_NDB_JAVA`选项默认启用。这意味着，默认情况下，如果**CMake**在你的系统上找不到 Java 的位置，配置过程将失败；如果你不希望启用 Java 和 ClusterJ 支持，你必须通过使用 `-DWITH_NDB_JAVA=OFF` 明确指示。 (Bug #12379735) 如果需要，使用`WITH_CLASSPATH`来提供 Java 类路径。

关于构建 NDB Cluster 的**CMake**选项的更多信息，请参见用于编译 NDB Cluster 的 CMake 选项。

构建过程完成后，你可以创建一个包含编译后二进制文件的 Zip 归档文件；第 2.8.4 节，“使用标准源分发安装 MySQL” 提供了在 Windows 系统上执行此任务所需的命令。NDB Cluster 的二进制文件可以在生成的归档文件的 `bin` 目录中找到，该目录等同于 `no-install` 归档文件，并且可以以相同的方式安装和配置。更多信息，请参阅第 25.3.2.1 节，“从二进制发布版在 Windows 上安装 NDB Cluster”。
