> 原文：[`dev.mysql.com/doc/refman/8.0/en/making-windows-dumps.html`](https://dev.mysql.com/doc/refman/8.0/en/making-windows-dumps.html)

#### 7.9.1.3 使用 WER 与 PDB 创建 Windows 崩溃转储

程序数据库文件（带有后缀`pdb`）包含在 MySQL 的**ZIP Archive Debug Binaries & Test Suite**分发中。这些文件提供了在出现问题时调试 MySQL 安装的信息。这是与标准 MSI 或 Zip 文件分开下载的。

注意

PDB 文件包含在一个名为"ZIP Archive Debug Binaries & Test Suite"的单独文件中。

PDB 文件包含有关`mysqld`和其他工具的更详细信息，可以创建更详细的跟踪和转储文件。您可以使用这些文件与**WinDbg**或 Visual Studio 一起调试**mysqld**。

有关 PDB 文件的更多信息，请参阅[Microsoft Knowledge Base Article 121366](http://support.microsoft.com/kb/121366/)。有关可用的调试选项的更多信息，请参阅[Windows 调试工具](http://www.microsoft.com/whdc/devtools/debugging/default.mspx)。

要使用 WinDbg，要么安装完整的 Windows 驱动程序套件（WDK），要么安装独立版本。

重要

`.exe`和`.pdb`文件必须完全匹配（版本号和 MySQL 服务器版本）；否则，WinDBG 在尝试加载符号时会报错。

1.  要生成一个 minidump `mysqld.dmp`，请在`my.ini`中的[mysqld]部分下启用`core-file`选项。在进行这些更改后重新启动 MySQL 服务器。

1.  创建一个目录来存储生成的文件，例如`c:\symbols`

1.  使用查找 GUI 或命令行确定您的**windbg.exe**可执行文件的路径，例如：`dir /s /b windbg.exe` -- 一个常见的默认路径是*C:\Program Files\Debugging Tools for Windows (x64)\windbg.exe*

1.  启动`windbg.exe`，并将`mysqld.exe`、`mysqld.pdb`、`mysqld.dmp`和源代码的路径传递给它。或者，从 WinDbg GUI 中传递每个路径。例如：

    ```sql
    windbg.exe -i "C:\mysql-8.0.36-winx64\bin\"^
     -z "C:\mysql-8.0.36-winx64\data\mysqld.dmp"^
     -srcpath "E:\ade\mysql_archives\8.0\8.0.36\mysql-8.0.36"^
     -y "C:\mysql-8.0.36-winx64\bin;SRV*c:\symbols*http://msdl.microsoft.com/download/symbols"^
     -v -n -c "!analyze -vvvvv"
    ```

    注意

    Windows 命令行处理器会删除`^`字符和换行符，因此请确保空格保持完整。
