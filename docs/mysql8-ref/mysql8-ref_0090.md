# 2.8.10 生成 MySQL Doxygen 文档内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-installation-doxygen.html`](https://dev.mysql.com/doc/refman/8.0/en/source-installation-doxygen.html)

MySQL 源代码包含使用 Doxygen 编写的内部文档。生成的 Doxygen 内容可在 `dev.mysql.com/doc/index-other.html` 找到。还可以使用以下过程从 MySQL 源分发本地生成此内容：

1.  安装 **doxygen** 1.9.2 或更高版本。可在 [`www.doxygen.nl/`](http://www.doxygen.nl/) 找到发行版。

    安装 **doxygen** 后，请验证版本号：

    ```sql
    $> doxygen --version
    1.9.2
    ```

1.  安装 [PlantUML](http://plantuml.com/download.html)。

    在 Windows 上安装 PlantUML（在 Windows 10 上测试过）时，必须至少以管理员身份运行一次，以便创建注册表键。打开管理员控制台并运行以下命令：

    ```sql
    $> java -jar *path-to-plantuml.jar*
    ```

    该命令应该打开一个 GUI 窗口，并在控制台上不返回任何错误。

1.  将 `PLANTUML_JAR_PATH` 环境设置为安装 PlantUML 的位置。例如：

    ```sql
    $> export PLANTUML_JAR_PATH=*path-to-plantuml.jar*
    ```

1.  安装 [Graphviz](http://www.graphviz.org/) 的 **dot** 命令。

    安装 Graphviz 后，请验证 **dot** 的可用性。例如：

    ```sql
    $> which dot
    /usr/bin/dot

    $> dot -V
    dot - graphviz version 2.40.1 (20161225.0304)
    ```

1.  将位置更改到 MySQL 源分发的顶层目录，并执行以下操作：

    首先，执行 **cmake**：

    ```sql
    $> cd *mysql-source-directory*
    $> mkdir build
    $> cd build
    $> cmake ..
    ```

    接下来，生成 **doxygen** 文档：

    ```sql
    $> make doxygen
    ```

    检查错误日志，该日志位于顶层目录中的 `doxyerror.log` 文件中。假设构建成功执行，请使用浏览器查看生成的输出。例如：

    ```sql
    $> firefox doxygen/html/index.html
    ```
