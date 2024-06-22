# 1\. 概述

> 原文：[`sqlite.com/loadext.html`](https://sqlite.com/loadext.html)

SQLite 具备在运行时加载扩展（包括新的 应用程序定义的 SQL 函数，排序序列，虚拟表 和 VFSes）的能力。这个特性允许扩展的代码可以独立于应用程序进行开发和测试，然后在需要时进行加载。

扩展也可以与应用程序静态链接。下面展示的代码模板既可以作为静态链接的扩展使用，也可以作为运行时可加载的扩展使用，只是如果你的应用程序包含两个或更多扩展，应该给入口函数 ("sqlite3_extension_init") 起一个不同的名称，以避免名称冲突。

# 2\. 加载扩展

SQLite 扩展是一个共享库或 DLL。要加载它，需要向 SQLite 提供包含共享库或 DLL 的文件名以及初始化扩展的入口点。在 C 代码中，可以使用 sqlite3_load_extension() API 提供这些信息。有关此例程的详细信息，请参阅文档。

注意，不同的操作系统使用不同的文件名后缀来表示它们的共享库。Windows 使用 ".dll"，Mac 使用 ".dylib"，大多数除了 Mac 之外的 Unix 系统使用 ".so"。如果你希望使你的代码具备可移植性，可以省略共享库文件名的后缀，sqlite3_load_extension() 接口会自动添加适当的后缀。

SQLite 也有一个可以用来加载扩展的 SQL 函数：load_extension(X,Y)。它的工作方式类似于 sqlite3_load_extension() 的 C 接口。

加载扩展的两种方法都允许你指定扩展的入口点名称。你可以将这个参数留空 - 对于 sqlite3_load_extension() 的 C 语言接口传入一个空指针，或者对于 SQL 接口的 load_extension() 省略第二个参数 - 扩展加载器逻辑会尝试自动确定入口点。它首先尝试通用的扩展名称 "sqlite3_extension_init"。如果这样不起作用，它会使用模板 "sqlite3_X_init"，其中 X 是文件名中最后一个 "/" 后和下一个 "." 前的 ASCII 字符的小写等价，如果第一个三个字符碰巧是 "lib"，则省略它们。例如，如果文件名是 "/usr/lib/libmathfunc-4.8.so"，则入口点名称将是 "sqlite3_mathfunc_init"。或者如果文件名是 "./SpellFixExt.dll"，则入口点将被称为 "sqlite3_spellfixext_init"。

出于安全考虑，默认关闭扩展加载。为了使用 C 语言或 SQL 扩展加载函数，必须首先在应用程序中使用 sqlite3_db_config(db,SQLITE_DBCONFIG_ENABLE_LOAD_EXTENSION,1,NULL) C 语言 API 启用扩展加载。

从 命令行 shell 中，可以使用 ".load" 命令加载扩展。例如：

> ```sql
> .load ./YourCode
> 
> ```

注意，命令行 shell 程序已经为你启用了扩展加载（作为其设置的一部分调用了 sqlite3_enable_load_extension() 接口），因此上述命令可以无需任何特殊开关、设置或其他复杂操作即可工作。

".load" 命令带有一个参数，调用 sqlite3_load_extension()，zProc 参数设置为 NULL，导致 SQLite 首先查找名为 "sqlite3_extension_init" 的入口点，然后是名为 "sqlite3_X_init" 的入口点，其中 "X" 源自文件名。如果你的扩展具有不同名称的入口点，只需作为第二个参数提供该名称。例如：

> ```sql
> .load ./YourCode nonstandard_entry_point
> 
> ```

# 3\. 编译可加载扩展

可加载扩展是 C 代码。在大多数类 Unix 操作系统上编译它们的通常命令是这样的：

> ```sql
> gcc -g -fPIC -shared YourCode.c -o YourCode.so
> 
> ```

Mac 是类 Unix 的操作系统，但不遵循通常的共享库约定。要在 Mac 上编译共享库，使用类似以下的命令：

> ```sql
> gcc -g -fPIC -dynamiclib YourCode.c -o YourCode.dylib
> 
> ```

如果尝试加载库时返回 "mach-o, but wrong architecture" 错误消息，则可能需要为 gcc 添加命令行选项 "-arch i386" 或 "arch x86_64"，具体取决于你的应用程序如何构建。

要在 Windows 上使用 MSVC 编译，类似以下的命令通常会起作用：

> ```sql
> cl YourCode.c -link -dll -out:YourCode.dll
> 
> ```

要在 Windows 上使用 MinGW 进行编译，命令行与 Unix 上的一样，只是输出文件后缀改为 ".dll"，并且省略了 -fPIC 参数。

> ```sql
> gcc -g -shared YourCode.c -o YourCode.dll
> 
> ```

# 4\. 编写可加载扩展

一个模板加载扩展包含以下三个元素：

1.  在你的源代码文件顶部使用 "`#include <sqlite3ext.h>`"，而不是 "`#include <sqlite3.h>`"。

1.  在 "`#include <sqlite3ext.h>`" 行后面，单独放置宏 "`SQLITE_EXTENSION_INIT1`"。

1.  添加一个类似以下内容的扩展加载入口点例程：

    ```sql
    #ifdef _WIN32
    __declspec(dllexport)
    #endif
    int sqlite3_extension_init( /* <== Change this name, maybe */
      sqlite3 *db, 
      char **pzErrMsg, 
      const sqlite3_api_routines *pApi
    ){
      int rc = SQLITE_OK;
      SQLITE_EXTENSION_INIT2(pApi);
      /* insert code to initialize your extension here */
      return rc;
    }

    ```

    你最好自定义入口点的名称，以对应生成的共享库的名称，而不是使用通用的 "sqlite3_extension_init" 名称。给扩展命名一个自定义的入口点名称将使你能够在同一程序中静态链接两个或更多扩展，而不会出现链接器冲突，如果你后来决定使用静态链接而不是运行时链接。如果你的共享库最终被命名为 "YourCode.so" 或 "YourCode.dll" 或 "YourCode.dylib"，如编译示例所示，那么正确的入口点名称将是 "sqlite3_yourcode_init"。

这里是一个完整的模板扩展，你可以复制/粘贴以开始：

```sql
/* Add your header comment here */
#include <sqlite3ext.h> /* Do not use <sqlite3.h>! */
SQLITE_EXTENSION_INIT1

/* Insert your extension code here */

#ifdef _WIN32
__declspec(dllexport)
#endif
/* TODO: Change the entry point name so that "extension" is replaced by
** text derived from the shared library filename as follows:  Copy every
** ASCII alphabetic character from the filename after the last "/" through
** the next following ".", converting each character to lowercase, and
** discarding the first three characters if they are "lib".
*/
int sqlite3_extension_init(
  sqlite3 *db,
  char **pzErrMsg,
  const sqlite3_api_routines *pApi
){
  int rc = SQLITE_OK;
  SQLITE_EXTENSION_INIT2(pApi);
  /* Insert here calls to
  **     sqlite3_create_function_v2(),
  **     sqlite3_create_collation_v2(),
  **     sqlite3_create_module_v2(), and/or
  **     sqlite3_vfs_register()
  ** to register the new features that your extension adds.
  */
  return rc;
}

```

## 4.1\. 示例扩展

在 SQLite 源代码树的[ext/misc](https://www.sqlite.org/src/file/ext/misc)子目录中，可以看到许多完整且可用的加载扩展示例。该目录中的每个文件都是一个独立的扩展。每个文件都提供了关于扩展的头部注释文档。以下是[ext/misc](https://www.sqlite.org/src/file/ext/misc)子目录中一些扩展的简要说明：

+   [carray.c](https://www.sqlite.org/src/file/ext/misc/carray.c) — 实现了 carray 表值函数。

+   [compress.c](https://www.sqlite.org/src/file/ext/misc/compress.c) — 实现了应用程序定义的 SQL 函数 compress() 和 uncompress()，用于对文本或 blob 内容进行 zLib 压缩。

+   [json1.c](https://www.sqlite.org/src/file/ext/misc/json1.c) — 实现了 JSON SQL 函数和表值函数。这是一个较大且更复杂的扩展。

+   [memvfs.c](https://www.sqlite.org/src/file/ext/misc/memvfs.c) — 实现了一个新的 VFS，将所有内容存储在内存中。

+   [rot13.c](https://www.sqlite.org/src/file/ext/misc/rot13.c) — 实现了[rot13()](https://en.wikipedia.org/wiki/ROT13) SQL 函数。这是一个非常简单的扩展函数示例，并且作为创建新扩展的模板非常有用。

+   [series.c](https://www.sqlite.org/src/file/ext/misc/series.c) — 实现了 generate_series 虚拟表 和 表值函数。这是一个相对简单的虚拟表实现示例，可以作为编写新虚拟表的模板。

其他更复杂的扩展可以在[ext/](https://www.sqlite.org/src/file/ext)目录下的子文件夹中找到。

# 5\. 持久化加载扩展

可加载扩展的默认行为是，在调用 sqlite3_load_extension()的数据库连接关闭时，扩展会从进程内存中卸载（换句话说，当数据库连接关闭时，sqlite3_vfs 对象的 xDlClose 方法将会为所有扩展调用）。但是，如果初始化过程返回 SQLITE_OK_LOAD_PERMANENTLY 而不是 SQLITE_OK，那么扩展将不会被卸载（xDlClose 将不会被调用），扩展将无限期地保留在进程内存中。SQLITE_OK_LOAD_PERMANENTLY 返回值对于想要注册新 VFS 的扩展非常有用。

为了澄清：初始化函数返回 SQLITE_OK_LOAD_PERMANENTLY 的扩展会在数据库连接关闭后继续存在于内存中。但是，该扩展*不会*自动注册到后续的数据库连接中。这使得可以加载实现新的 VFSes 的扩展。为了持续加载并注册实现新 SQL 函数、排序序列和/或虚拟表的扩展，使得这些新增功能对所有后续数据库连接都可用，初始化例程还应调用 sqlite3_auto_extension()来注册这些服务的子函数。

[vfsstat.c](https://sqlite.org/src/file/ext/misc/vfsstat.c)扩展展示了一个示例，它持久地注册了一个新的 VFS 和一个新的虚拟表。该扩展中的[sqlite3_vfsstat_init()](https://sqlite.org/src/info/77b5b4235c9f7f11?ln=801-819)初始化例程只在扩展首次加载时调用。它只在那个时间注册新的"vfslog" VFS，并且返回 SQLITE_OK_LOAD_PERMANENTLY，使得用于实现"vfslog" VFS 的代码保持在内存中。初始化例程还调用 sqlite3_auto_extension()来指向"vstatRegister()"函数的指针，以便所有后续的数据库连接在启动时调用"vstatRegister()"函数，从而注册"vfsstat"虚拟表。

# 6\. 静态链接运行时可加载扩展

相同的源代码可以用于运行时可加载共享库或 DLL，也可以作为静态链接到应用程序的模块。这提供了灵活性，并允许您以不同的方式重用相同的代码。

要静态链接您的扩展，只需添加-DSQLITE_CORE 编译时选项。SQLITE_CORE 宏使 SQLITE_EXTENSION_INIT1 和 SQLITE_EXTENSION_INIT2 宏成为无操作。然后修改您的应用程序直接调用入口点，将 NULL 指针作为第三个“pApi”参数传递。

如果您将静态链接两个或多个扩展模块，则使用基于扩展文件名而不是通用的"sqlite3_extension_init"入口点名称尤其重要。如果使用通用名称，将会有多个相同符号的定义，导致链接失败。

如果您的应用程序将打开多个数据库连接，而不是为每个数据库连接单独调用扩展入口点，则可能希望考虑使用 sqlite3_auto_extension() 接口注册扩展，并在打开每个数据库连接时自动启动它们。您只需注册每个扩展一次，并且可以在主() 函数的开始附近执行此操作。使用 sqlite3_auto_extension() 接口注册扩展使得您的扩展工作起来就像内置到核心 SQLite 中一样 - 每当打开新的数据库连接时它们就自动存在，无需初始化。只需确保在注册扩展之前完成任何需要使用 sqlite3_config() 完成的配置，因为 sqlite3_auto_extension() 接口会隐式调用 sqlite3_initialize()。

# 7\. 实现细节

SQLite 使用 `xDlOpen()`、`xDlError()`、`xDlSym()` 和 `xDlClose()` 方法实现运行时扩展加载，这些方法是通过 sqlite3_vfs 对象实现的。在 Unix 上，这些方法使用 dlopen() 库实现（这解释了为什么 SQLite 在 Unix 系统上通常需要链接 "-ldl" 库），而在 Windows 上则使用 LoadLibrary() API。在自定义的 VFS 中，对于不寻常的系统，可以省略这些方法，这样运行时扩展加载机制将不起作用（尽管您仍然可以静态链接扩展代码，假设入口点名称是唯一的）。可以使用 SQLITE_OMIT_LOAD_EXTENSION 来编译 SQLite，从而在构建中省略扩展加载代码。
