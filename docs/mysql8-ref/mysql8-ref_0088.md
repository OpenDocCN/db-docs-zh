# 2.8.8 处理编译 MySQL 时出现的问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/compilation-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/compilation-problems.html)

许多问题的解决方案涉及重新配置。如果您重新配置，请注意以下事项：

+   如果 **CMake** 在之前已经运行过后再次运行，它可能会使用在先前调用期间收集的信息。这些信息存储在 `CMakeCache.txt` 中。当 **CMake** 启动时，它会查找该文件，并在假定信息仍然正确的情况下读取其内容。当您重新配置时，这种假设是无效的。

+   每次运行 **CMake**，您必须再次运行 **make** 进行重新编译。但是，您可能希望在重新编译之前删除先前构建的旧目标文件，因为它们是使用不同的配置选项编译的。

为了防止使用旧的目标文件或配置信息，请在重新运行 **CMake** 之前运行以下命令：

在 Unix 系统上：

```sql
$> make clean
$> rm CMakeCache.txt
```

在 Windows 系统上：

```sql
$> devenv MySQL.sln /clean
$> del CMakeCache.txt
```

如果在源代码树之外构建，请在重新运行 **CMake** 之前删除并重新创建您的构建目录。有关在源代码树之外构建的说明，请参阅 如何使用 CMake 构建 MySQL 服务器。

在某些系统上，由于系统包含文件的差异，可能会出现警告。以下列表描述了���编译 MySQL 时最常出现的其他问题：

+   要定义要使用的 C 和 C++ 编译器，您可以定义 `CC` 和 `CXX` 环境变量。例如：

    ```sql
    $> CC=gcc
    $> CXX=g++
    $> export CC CXX
    ```

    虽然可以在命令行上执行此操作，就像刚才展示的那样，但您可能更喜欢在构建脚本中定义这些值，这种情况下不需要 **export** 命令。

    要指定自己的 C 和 C++ 编译器标志，请使用 `CMAKE_C_FLAGS` 和 `CMAKE_CXX_FLAGS` CMake 选项。参见 编译器标志。

    要查看可能需要指定的标志，请使用 **mysql_config** 呼叫 `--cflags` 和 `--cxxflags` 选项。

+   在使用 **CMake** 配置 MySQL 后，要查看编译阶段执行的命令，请运行 **make VERBOSE=1** 而不是仅运行 **make**。

+   如果编译失败，请检查是否启用了 `MYSQL_MAINTAINER_MODE` 选项。此模式会导致编译器警告变为错误，因此禁用它可能会使编译继续进行。

+   如果您的编译失败并出现以下任何错误，请升级您的 **make** 版本为 GNU **make**：

    ```sql
    make: Fatal error in reader: Makefile, line 18:
    Badly formed macro assignment
    ```

    或者：

    ```sql
    make: file `Makefile' line 18: Must be a separator (:
    ```

    或者：

    ```sql
    pthread.h: No such file or directory
    ```

    Solaris 和 FreeBSD 已知具有问题的 **make** 程序。

    GNU **make** 3.75 已知可用。

+   `sql_yacc.cc` 文件是从 `sql_yacc.yy` 生成的。通常，构建过程不需要创建 `sql_yacc.cc`，因为 MySQL 自带一个预生成的副本。但是，如果你确实需要重新创建它，可能会遇到这个错误：

    ```sql
    "sql_yacc.yy", line *xxx* fatal: default action causes potential...
    ```

    这表明你的 **yacc** 版本不足。你可能需要安装一个较新版本的 **bison**（**yacc** 的 GNU 版本）并使用它代替。

    **bison** 版本旧于 1.75 可能会报告此错误：

    ```sql
    sql_yacc.yy:#####: fatal error: maximum table size (32767) exceeded
    ```

    实际上并未超出最大表大小；错误是由旧版本的 **bison** 中的错误引起的。

有关获取或更新工具的信息，请参阅第 2.8 节，“从源代码安装 MySQL”中的系统要求。
