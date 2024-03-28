# 2.8.2 源码安装先决条件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-installation-prerequisites.html`](https://dev.mysql.com/doc/refman/8.0/en/source-installation-prerequisites.html)

从源码安装 MySQL 需要几个开发工具。一些工具无论您使用标准源码分发还是开发源码树都是必需的。其他工具要求取决于您使用的安装方法。

要从源码安装 MySQL，必须满足以下系统要求，无论使用哪种安装方法：

+   **CMake**，用作所有平台上的构建框架。**CMake** 可从[`www.cmake.org`](http://www.cmake.org)下载。

+   一个良好的 **make** 程序。尽管一些平台自带其自己的 **make** 实现，但强烈建议使用 GNU **make** 3.75 或更高版本。它可能已经在您的系统上作为 **gmake** 可用。GNU **make** 可从[`www.gnu.org/software/make/`](http://www.gnu.org/software/make/)获取。

    在类 Unix 系统上，包括 Linux，在终端中可以通过以下方式检查系统的 **make** 版本：

    ```sql
    $> make --version
    GNU Make 4.2.1
    ```

+   从 MySQL 8.0.26 开始，MySQL 8.0 源码允许使用 C++17 特性。为了在所有支持的平台上启用必要的 C++17 支持级别，以下最低编译器版本适用：

    +   Linux：GCC 10 或 Clang 5

    +   macOS：XCode 10

    +   Solaris：GCC 10

    +   Windows：Visual Studio 2019

+   MySQL C API 需要 C++ 或 C99 编译器进行编译。

+   支持加密连接、随机数生成的熵以及其他与加密相关的操作需要 SSL 库。默认情况下，构建使用主机系统上安装的 OpenSSL 库。要显式指定库，请在调用 **CMake** 时使用 `WITH_SSL` 选项。有关更多信息，请参见第 2.8.6 节，“配置 SSL 库支持”。

+   构建 MySQL 需要 Boost C++ 库（但不需要使用它）。MySQL 编译需要特定的 Boost 版本。通常情况下，这是当前的 Boost 版本，但如果特定的 MySQL 源码分发需要不同的版本，则配置过程将停止，并显示需要的 Boost 版本。要获取 Boost 及其安装说明，请访问[官方 Boost 网站](https://www.boost.org)。安装 Boost 后，根据在调用 CMake 时设置的 `WITH_BOOST` 选项的值，告诉构建系统 Boost 文件的放置位置。例如：

    ```sql
    cmake . -DWITH_BOOST=/usr/local/boost_*version_number*
    ```

    根据需要调整路径以匹配您的安装。

+   [ncurses](https://www.gnu.org/software/ncurses/ncurses.html) 库。

+   充足的空闲内存。如果在编译大型源文件时遇到内部编译器错误等构建错误，可能是内存太少了。如果在虚拟机上编译，请尝试增加内存分配。

+   如果你打算运行测试脚本，则需要 Perl。大多数类 Unix 系统都包含 Perl。对于 Windows，你可以使用[ActiveState Perl](https://www.activestate.com/products/perl/)或[Strawberry Perl](https://strawberryperl.com/)。

要从标准源分发安装 MySQL，需要以下工具之一来解压缩分发文件：

+   对于`.tar.gz`压缩的**tar**文件：使用 GNU `gunzip`解压分发文件，然后使用一个合理的**tar**来解压缩。如果你的**tar**程序支持`z`选项，它可以同时解压缩和解包文件。

    GNU **tar**已知可用。某些操作系统提供的标准**tar**无法解压缩 MySQL 分发中的长文件名。你应该下载并安装 GNU **tar**，或者如果可用，使用预安装的 GNU tar 版本。通常这可作为**gnutar**、**gtar**或在 GNU 或自由软件目录中的**tar**，如`/usr/sfw/bin`或`/usr/local/bin`中找到。GNU **tar**可从[`www.gnu.org/software/tar/`](https://www.gnu.org/software/tar/)获取。

+   对于`.zip` Zip 归档文件：**WinZip**或其他能够读取`.zip`文件的工具。

+   对于`.rpm` RPM 软件包：用于构建分发的**rpmbuild**程序会解压缩它。

要从开发源代码树安装 MySQL，需要以下额外的工具：

+   Git 修订控制系统是获取开发源代码所必需的。[GitHub 帮助](https://help.github.com/)提供了在不同平台上下载和安装 Git 的说明。

+   **bison** 2.1 或更高版本，可从[`www.gnu.org/software/bison/`](http://www.gnu.org/software/bison/)获取。（不再支持版本 1。）尽可能使用最新版本的**bison**；如果遇到问题，请升级到更高版本，而不是回退到早期版本。

    **bison**可从[`www.gnu.org/software/bison/`](http://www.gnu.org/software/bison/)获取。Windows 上的`bison`可从[`gnuwin32.sourceforge.net/packages/bison.htm`](http://gnuwin32.sourceforge.net/packages/bison.htm)下载。下载标记为“Complete package, excluding sources”的软件包。在 Windows 上，**bison**的默认位置是`C:\Program Files\GnuWin32`目录。由于目录名中有空格，一些实用程序可能无法找到**bison**。此外，如果路径中有空格，Visual Studio 可能会简单地挂起。你可以通过安装到不包含空格的目录（例如`C:\GnuWin32`）来解决这些问题。

+   在 Solaris Express 上，除了**bison**外，还必须安装**m4**。**m4**可从[`www.gnu.org/software/m4/`](http://www.gnu.org/software/m4/)获取。

注意

如果你需要安装任何程序，请修改你的`PATH`环境变量，包括程序所在的任何目录。参见第 6.2.9 节，“设置环境变量”。

如果遇到问题需要提交错误报告，请按照第 1.5 节，“如何报告错误或问题”中的说明。
