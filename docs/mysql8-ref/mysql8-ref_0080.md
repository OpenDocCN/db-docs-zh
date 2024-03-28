# 2.8 从源代码安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/source-installation.html)

2.8.1 源代码安装方法

2.8.2 源代码安装先决条件

2.8.3 源代码安装的 MySQL 布局

2.8.4 使用标准源代码发行版安装 MySQL

2.8.5 使用开发源代码树安装 MySQL

2.8.6 配置 SSL 库支持

2.8.7 MySQL 源代码配置选项

2.8.8 处理编译 MySQL 时的问题

2.8.9 MySQL 配置和第三方工具

2.8.10 生成 MySQL Doxygen 文档内容

从源代码构建 MySQL 可以让您自定义构建参数、编译器优化和安装位置。有关已知可运行 MySQL 的系统列表，请参阅 [`www.mysql.com/support/supportedplatforms/database.html`](https://www.mysql.com/support/supportedplatforms/database.html)。

在从源代码进行安装之前，请检查 Oracle 是否为您的平台提供了预编译的二进制发行版，并且它是否适合您使用。我们非常努力确保我们的二进制文件是使用最佳选项构建的，以获得最佳性能。有关安装二进制发行版的说明，请参阅第 2.2 节，“在 Unix/Linux 上使用通用二进制文件安装 MySQL”。

如果您有兴趣使用与 Oracle 在您的平台上生成二进制发行版时使用的构建选项相同或类似的源代码构建 MySQL，请获取一个二进制发行版，解压缩它，并查看 `docs/INFO_BIN` 文件，其中包含有关该 MySQL 发行版的配置和编译信息。

警告

使用非标准选项构建 MySQL 可能会导致功能、性能或安全性降低。

MySQL 源代码包含使用 Doxygen 编写的内部文档。生成的 Doxygen 内容可在 `dev.mysql.com/doc/index-other.html` 上找到。还可以使用 第 2.8.10 节，“生成 MySQL Doxygen 文档内容” 中的说明从 MySQL 源代码发行版本地生成此内容。
