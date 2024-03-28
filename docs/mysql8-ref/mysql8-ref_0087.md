# 2.8.7 MySQL 源配置选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html`](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html)

**CMake**程序提供了对如何配置 MySQL 源分发的大量控制。通常，您可以使用**CMake**命令行上的选项来执行此操作。有关**CMake**支持的选项的信息，请在顶层源目录中运行以下命令之一：

```sql
$> cmake . -LH

$> ccmake .
```

您还可以使用某些环境变量影响**CMake**。请参阅第 6.9 节“环境变量”。

对于布尔选项，值可以指定为`1`或`ON`以启用该选项，或者指定为`0`或`OFF`以禁用该选项。

许多选项配置编译时的默认值，可以在服务器启动时进行覆盖。例如，`CMAKE_INSTALL_PREFIX`、`MYSQL_TCP_PORT`和`MYSQL_UNIX_ADDR`选项配置默认安装基目录位置、TCP/IP 端口号和 Unix 套接字文件，可以通过`--basedir`、`--port`和`--socket`选项在服务器启动时进行更改，用于**mysqld**。在适用的情况下，配置选项描述指示相应的**mysqld**启动选项。

以下部分提供有关**CMake**选项的更多信息。

+   CMake 选项参考

+   常规选项

+   安装布局选项

+   存储引擎选项

+   功能选项

+   编译器标志

+   编译 NDB 集群的 CMake 选项

#### CMake 选项参考

以下表格显示了可用的**CMake**选项。在`默认`列中，`PREFIX`代表`CMAKE_INSTALL_PREFIX`选项的值，该选项指定安装基目录。该值用作多个安装子目录的父位置。

**表 2.14 MySQL 源配置选项参考（CMake）**

| 格式 | 描述 | 默认 | 引入 | 移除 |
| --- | --- | --- | --- | --- |
| `ADD_GDB_INDEX` | 是否启用二进制文件中.gdb_index 部分的生成 |  | 8.0.18 |  |
| `BUILD_CONFIG` | 使用与官方发布相同的构建选项 |  |  |  |
| `BUNDLE_RUNTIME_LIBRARIES` | 将运行时库与 Windows 服务器 MSI 和 Zip 软件包捆绑在一起 | `OFF` |  |  |
| `CMAKE_BUILD_TYPE` | 生成的构建类型 | `RelWithDebInfo` |  |  |
| `CMAKE_CXX_FLAGS` | C++编译器标志 |  |  |  |
| `CMAKE_C_FLAGS` | C 编译器标志 |  |  |  |
| `CMAKE_INSTALL_PREFIX` | 安装基本目录 | `/usr/local/mysql` |  |  |
| `COMPILATION_COMMENT` | 编译环境注释 |  |  |  |
| `COMPILATION_COMMENT_SERVER` | 用于 mysqld 使用的编译环境注释 |  | 8.0.14 |  |
| `COMPRESS_DEBUG_SECTIONS` | 压缩二进制可执行文件的调试部分 | `OFF` | 8.0.22 |  |
| `CPACK_MONOLITHIC_INSTALL` | 包构建是否生成单个文件 | `OFF` |  |  |
| `DEFAULT_CHARSET` | 默认服务器字符集 | `utf8mb4` |  |  |
| `DEFAULT_COLLATION` | 默认服务器排序规则 | `utf8mb4_0900_ai_ci` |  |  |
| `DISABLE_PSI_COND` | 排除性能模式条件仪表化 | `OFF` |  |  |
| `DISABLE_PSI_DATA_LOCK` | 排除性能模式数据锁定仪表化 | `OFF` |  |  |
| `DISABLE_PSI_ERROR` | 排除性能模式服务器错误仪表化 | `OFF` |  |  |
| `DISABLE_PSI_FILE` | 排除性能模式文件仪表化 | `OFF` |  |  |
| `DISABLE_PSI_IDLE` | 排除性能模式空闲仪表化 | `OFF` |  |  |
| `DISABLE_PSI_MEMORY` | 排除性能模式内存仪器 | `OFF` |  |  |
| `DISABLE_PSI_METADATA` | 排除性能模式 metadata 仪器 | `OFF` |  |  |
| `DISABLE_PSI_MUTEX` | 排除性能模式互斥仪器 | `OFF` |  |  |
| `DISABLE_PSI_PS` | 排除性能模式预处理语句 | `OFF` |  |  |
| `DISABLE_PSI_RWLOCK` | 排除性能模式 rwlock 仪器 | `OFF` |  |  |
| `DISABLE_PSI_SOCKET` | 排除性能模式套接字仪器 | `OFF` |  |  |
| `DISABLE_PSI_SP` | 排除性能模式存储过程仪器 | `OFF` |  |  |
| `DISABLE_PSI_STAGE` | 排除性能模式 stage 仪器 | `OFF` |  |  |
| `DISABLE_PSI_STATEMENT` | 排除性能模式语句仪器 | `OFF` |  |  |
| `DISABLE_PSI_STATEMENT_DIGEST` | 排除性能模式 statements_digest 仪器 | `OFF` |  |  |
| `DISABLE_PSI_TABLE` | 排除性能模式表仪器 | `OFF` |  |  |
| `DISABLE_PSI_THREAD` | 排除性能模式线程仪器 | `OFF` |  |  |
| `DISABLE_PSI_TRANSACTION` | 排除性能模式事务仪器 | `OFF` |  |  |
| `DISABLE_SHARED` | 不构建共享库，编译位置相关代码 | `OFF` |  | 8.0.18 |
| `DOWNLOAD_BOOST` | 是否下载 Boost 库 | `OFF` |  |  |
| `DOWNLOAD_BOOST_TIMEOUT` | 下载 Boost 库的超时时间（秒） | `600` |  |  |
| `ENABLED_LOCAL_INFILE` | 是否启用 LOAD DATA 的 LOCAL 功能 | `OFF` |  |  |
| `ENABLED_PROFILING` | 是否启用查询分析代码 | `ON` |  |  |
| `ENABLE_DOWNLOADS` | 是否下载可选文件 | `OFF` |  | 8.0.26 |
| `ENABLE_EXPERIMENTAL_SYSVARS` | 是否启用实验性 InnoDB 系统变量 | `OFF` |  |  |
| `ENABLE_GCOV` | 是否包含 gcov 支持 |  |  |  |
| `ENABLE_GPROF` | 启用 gprof（仅优化的 Linux 构建） | `OFF` |  |  |
| `FORCE_COLORED_OUTPUT` | 是否给编译输出着色 | `OFF` | 8.0.33 |  |
| `FORCE_INSOURCE_BUILD` | 是否强制在源代码构建 | `OFF` | 8.0.14 |  |
| `FORCE_UNSUPPORTED_COMPILER` | 是否允许不支持的编译器 | `OFF` |  |  |
| `FPROFILE_GENERATE` | 是否生成配置引导优化数据 | `OFF` | 8.0.19 |  |
| `FPROFILE_USE` | 是否使用配置引导优化数据 | `OFF` | 8.0.19 |  |
| `HAVE_PSI_MEMORY_INTERFACE` | 启用性能模式内存跟踪模块，用于动态存储超对齐类型的内存分配函数 | `OFF` | 8.0.26 |  |
| `IGNORE_AIO_CHECK` | 使用 -DBUILD_CONFIG=mysql_release 时，忽略 libaio 检查 | `OFF` |  |  |
| `INSTALL_BINDIR` | 用户可执行文件目录 | `PREFIX/bin` |  |  |
| `INSTALL_DOCDIR` | 文档目录 | `PREFIX/docs` |  |  |
| `INSTALL_DOCREADMEDIR` | README 文件目录 | `PREFIX` |  |  |
| `INSTALL_INCLUDEDIR` | 头文件目录 | `PREFIX/include` |  |  |
| `INSTALL_INFODIR` | 信息文件目录 | `PREFIX/docs` |  |  |
| `INSTALL_LAYOUT` | 选择预定义的安装布局 | `STANDALONE` |  |  |
| `INSTALL_LIBDIR` | 库文件目录 | `PREFIX/lib` |  |  |
| `INSTALL_MANDIR` | 手册页目录 | `PREFIX/man` |  |  |
| `INSTALL_MYSQLKEYRINGDIR` | keyring_file 插件数据文件目录 | `特定于平台` |  |  |
| `INSTALL_MYSQLSHAREDIR` | 共享数据目录 | `PREFIX/share` |  |  |
| `INSTALL_MYSQLTESTDIR` | mysql-test 目录 | `PREFIX/mysql-test` |  |  |
| `INSTALL_PKGCONFIGDIR` | mysqlclient.pc pkg-config 文件目录 | `INSTALL_LIBDIR/pkgconfig` |  |  |
| `INSTALL_PLUGINDIR` | 插件目录 | `PREFIX/lib/plugin` |  |  |
| `INSTALL_PRIV_LIBDIR` | 安装私有库目录 |  | 8.0.18 |  |
| `INSTALL_SBINDIR` | 服务器可执行文件目录 | `PREFIX/bin` |  |  |
| `INSTALL_SECURE_FILE_PRIVDIR` | secure_file_priv 默认值 | `特定于平台` |  |  |
| `INSTALL_SHAREDIR` | aclocal/mysql.m4 安装目录 | `PREFIX/share` |  |  |
| `INSTALL_STATIC_LIBRARIES` | 是否安装静态库 | `ON` |  |  |
| `INSTALL_SUPPORTFILESDIR` | 额外支持文件目录 | `PREFIX/support-files` |  |  |
| `LINK_RANDOMIZE` | 是否随机化 mysqld 二进制文件中符号的顺序 | `OFF` |  |  |
| `LINK_RANDOMIZE_SEED` | LINK_RANDOMIZE 选项的种子值 | `mysql` |  |  |
| `MAX_INDEXES` | 每个表的最大索引数 | `64` |  |  |
| `MEMCACHED_HOME` | memcached 路径；已过时 | `[none]` |  | 8.0.23 |
| `MSVC_CPPCHECK` | 启用 MSVC 代码分析。 | `OFF` | 8.0.33 |  |
| `MUTEX_TYPE` | InnoDB 互斥类型 | `event` |  |  |
| `MYSQLX_TCP_PORT` | X 插件使用的 TCP/IP 端口号 | `33060` |  |  |
| `MYSQLX_UNIX_ADDR` | X 插件使用的 Unix 套接字文件 | `/tmp/mysqlx.sock` |  |  |
| `MYSQL_DATADIR` | 数据目录 |  |  |  |
| `MYSQL_MAINTAINER_MODE` | 是否启用 MySQL 维护者特定的开发环境 | `OFF` |  |  |
| `MYSQL_PROJECT_NAME` | Windows/macOS 项目名称 | `MySQL` |  |  |
| `MYSQL_TCP_PORT` | TCP/IP 端口号 | `3306` |  |  |
| `MYSQL_UNIX_ADDR` | Unix 套接字文件 | `/tmp/mysql.sock` |  |  |
| `NDB_UTILS_LINK_DYNAMIC` | 使 NDB 工具动态链接到 ndbclient |  | 8.0.22 |  |
| `ODBC_INCLUDES` | ODBC 包含目录 |  |  |  |
| `ODBC_LIB_DIR` | ODBC 库目录 |  |  |  |
| `OPTIMIZER_TRACE` | 是否支持优化器跟踪 |  |  |  |
| `OPTIMIZE_SANITIZER_BUILDS` | 是否优化 sanitizer 构建 | `ON` | 8.0.34 |  |
| `REPRODUCIBLE_BUILD` | 特别注意创建与构建位置和时间无关的构建结果 |  |  |  |
| `SHOW_SUPPRESSED_COMPILER_WARNING` | 是否显示被抑制的编译器警告并且不使用-Werror 失败。 | `OFF` | 8.0.30 |  |
| `SYSCONFDIR` | 选项文件目录 |  |  |  |
| `SYSTEMD_PID_DIR` | systemd 下 PID 文件目录 | `/var/run/mysqld` |  |  |
| `SYSTEMD_SERVICE_NAME` | systemd 下 MySQL 服务的名称 | `mysqld` |  |  |
| `TMPDIR` | tmpdir 默认值 |  |  |  |
| `USE_LD_GOLD` | 是否使用 GNU gold 链接器 | `ON` |  | 8.0.31 |
| `USE_LD_LLD` | 是否使用 LLVM lld 链接器 | `ON` | 8.0.16 |  |
| `WIN_DEBUG_NO_INLINE` | 是否禁用函数内联 | `OFF` |  |  |
| `WITHOUT_SERVER` | 不构建服务器 | `OFF` |  |  |
| `WITHOUT_xxx_STORAGE_ENGINE` | 从构建中排除存储引擎 xxx |  |  |  |
| `WITH_ANT` | 用于构建 GCS Java 包装器的 Ant 路径 |  |  |  |
| `WITH_ASAN` | 启用 AddressSanitizer | `OFF` |  |  |
| `WITH_ASAN_SCOPE` | 启用 AddressSanitizer 的 -fsanitize-address-use-after-scope Clang 标志 | `OFF` |  |  |
| `WITH_AUTHENTICATION_CLIENT_PLUGINS` | 如果构建了相应的服务器认证插件，则自动启用 |  | 8.0.26 |  |
| `WITH_AUTHENTICATION_LDAP` | 是否在无法构建 LDAP 认证插件时报告错误 | `OFF` |  |  |
| `WITH_AUTHENTICATION_PAM` | 构建 PAM 认证插件 | `OFF` |  |  |
| `WITH_AWS_SDK` | Amazon Web Services 软件开发工具包的位置 |  |  |  |
| `WITH_BOOST` | Boost 库源代码的位置 |  |  |  |
| `WITH_BUILD_ID` | 在 Linux 系统上生成唯一的构建 ID | `ON` | 8.0.31 |  |
| `WITH_BUNDLED_LIBEVENT` | 在构建 ndbmemcache 时使用捆绑的 libevent；已过时 | `ON` |  | 8.0.23 |
| `WITH_BUNDLED_MEMCACHED` | 在构建 ndbmemcache 时使用捆绑的 memcached；已过时 | `ON` |  | 8.0.23 |
| `WITH_CLASSPATH` | 构建 MySQL Cluster Connector for Java 时要使用的类路径。默认为空字符串。 |  |  |  |
| `WITH_CLIENT_PROTOCOL_TRACING` | 构建客户端协议跟踪框架 | `ON` |  |  |
| `WITH_CURL` | curl 库的位置 |  |  |  |
| `WITH_DEBUG` | 是否包含调试支持 | `OFF` |  |  |
| `WITH_DEFAULT_COMPILER_OPTIONS` | 是否使用默认编译器选项 | `ON` |  |  |
| `WITH_DEFAULT_FEATURE_SET` | 是否使用默认功能集 | `ON` |  | 8.0.22 |
| `WITH_DEVELOPER_ENTITLEMENTS` | 是否在 macOS 上为所有可执行文件添加 'get-task-allow' 权限，以便在服务器意外停止时生成核心转储 | `OFF` | 8.0.30 |  |
| `WITH_EDITLINE` | 使用哪个 libedit/editline 库 | `bundled` |  |  |
| `WITH_ERROR_INSERT` | 启用 NDB 存储引擎中的错误注入。不应用于构建用于生产的二进制文件。 | `OFF` |  |  |
| `WITH_FIDO` | FIDO 库支持类型 | `bundled` | 8.0.27 |  |
| `WITH_GMOCK` | googlemock 分发路径 |  |  | 8.0.26 |
| `WITH_ICU` | ICU 支持类型 | `bundled` |  |  |
| `WITH_INNODB_EXTRA_DEBUG` | 是否包含 InnoDB 的额外调试支持。 | `OFF` |  |  |
| `WITH_INNODB_MEMCACHED` | 是否生成 memcached 共享库。 | `OFF` |  |  |
| `WITH_JEMALLOC` | 是否链接 -ljemalloc | `OFF` | 8.0.16 |  |
| `WITH_KEYRING_TEST` | 构建密钥环测试程序 | `OFF` |  |  |
| `WITH_LIBEVENT` | 使用哪个 libevent 库 | `bundled` |  |  |
| `WITH_LIBWRAP` | 是否包含 libwrap（TCP wrappers）支持 | `OFF` |  |  |
| `WITH_LOCK_ORDER` | 是否启用 LOCK_ORDER 工具 | `OFF` | 8.0.17 |  |
| `WITH_LSAN` | 是否运行 LeakSanitizer，不包括 AddressSanitizer | `OFF` | 8.0.16 |  |
| `WITH_LTO` | 启用链接时优化器 | `OFF` | 8.0.13 |  |
| `WITH_LZ4` | LZ4 库支持类型 | `bundled` |  |  |
| `WITH_LZMA` | LZMA 库支持类型 | `bundled` |  | 8.0.16 |
| `WITH_MECAB` | 编译 MeCab |  |  |  |
| `WITH_MSAN` | 启用 MemorySanitizer | `OFF` |  |  |
| `WITH_MSCRT_DEBUG` | 启用 Visual Studio CRT 内存泄漏跟踪 | `OFF` |  |  |
| `WITH_MYSQLX` | 是否禁用 X 协议 | `ON` |  |  |
| `WITH_NDB` | 构建 MySQL NDB 集群 | `OFF` | 8.0.31 |  |
| `WITH_NDBAPI_EXAMPLES` | 构建 API 示例程序 | `OFF` |  |  |
| `WITH_NDBCLUSTER` | 构建 NDB 存储引擎 | `OFF` |  |  |
| `WITH_NDBCLUSTER_STORAGE_ENGINE` | 供内部使用；在所有情况下可能不起作用；用户应该使用 WITH_NDBCLUSTER 代替 | `ON` |  |  |
| `WITH_NDBMTD` | 构建多线程数据节点 | `ON` |  |  |
| `WITH_NDB_DEBUG` | 生成用于测试或故障排除的调试构建 | `OFF` |  |  |
| `WITH_NDB_JAVA` | 启用构建 Java 和 ClusterJ 支持。默认启用。仅在 MySQL Cluster 中受支持。 | `ON` |  |  |
| `WITH_NDB_PORT` | 使用此选项构建的管理服务器使用的默认端口。如果未使用此选项构建它，则管理服务器的默认端口为 1186。 | `[none]` |  |  |
| `WITH_NDB_TEST` | 包括 NDB API 测试程序 | `OFF` |  |  |
| `WITH_NUMA` | 设置 NUMA 内存分配策略 |  |  |  |
| `WITH_PACKAGE_FLAGS` | 用于 RPM/DEB 包通常使用的标志，是否将它们添加到这些平台上的独立构建中 |  | 8.0.26 |  |
| `WITH_PLUGIN_NDBCLUSTER` | 供内部使用；在所有情况下可能不起作用。用户应该使用 WITH_NDBCLUSTER 或 WITH_NDB 代替 |  | 8.0.13 | 8.0.31 |
| `WITH_PROTOBUF` | 使用哪个 Protocol Buffers 包 | `bundled` |  |  |
| `WITH_RAPID` | 是否构建快速开发周期插件 | `ON` |  |  |
| `WITH_RAPIDJSON` | RapidJSON 支持类型 | `bundled` | 8.0.13 |  |
| `WITH_RE2` | RE2 库支持类型 | `bundled` |  | 8.0.18 |
| `WITH_ROUTER` | 是否构建 MySQL Router | `ON` | 8.0.16 |  |
| `WITH_SSL` | SSL 支持类型 | `system` |  |  |
| `WITH_SYSTEMD` | 启用 systemd 支持文件的安装 | `OFF` |  |  |
| `WITH_SYSTEMD_DEBUG` | 启用额外的 systemd 调试信息 | `OFF` | 8.0.22 |  |
| `WITH_SYSTEM_LIBS` | 设置未显式设置的库选项的系统值 | `OFF` |  |  |
| `WITH_TCMALLOC` | 是否链接 -ltcmalloc | `OFF` | 8.0.22 |  |
| `WITH_TEST_TRACE_PLUGIN` | 构建测试协议跟踪插件 | `OFF` |  |  |
| `WITH_TSAN` | 启用线程检测器 | `OFF` |  |  |
| `WITH_UBSAN` | 启用未定义行为检测器 | `OFF` |  |  |
| `WITH_UNIT_TESTS` | 使用单元测试编译 MySQL | `ON` |  |  |
| `WITH_UNIXODBC` | 启用 unixODBC 支持 | `OFF` |  |  |
| `WITH_VALGRIND` | 是否编译 Valgrind 头文件 | `OFF` |  |  |
| `WITH_WIN_JEMALLOC` | 包含 jemalloc.dll 的目录路径 |  | 8.0.29 |  |
| `WITH_ZLIB` | zlib 支持类型 | `bundled` |  |  |
| `WITH_ZSTD` | zstd 支持类型 | `bundled` | 8.0.18 |  |
| `WITH_xxx_STORAGE_ENGINE` | 将存储引擎 xxx 静态编译到服务器中 |  |  |  |
| 格式 | 描述 | 默认值 | 引入版本 | 移除版本 |

#### 通用选项

+   `-DBUILD_CONFIG=mysql_release`

    此选项配置源分发，使用 Oracle 用于生成官方 MySQL 发行版的二进制分发的相同构建选项。

+   `-DWITH_BUILD_ID=*`bool`*`

    在 Linux 系统上，生成一个唯一的构建 ID，该 ID 用作`build_id`系统变量的值，并在 MySQL 服务器启动时写入 MySQL 服务器日志。将此选项设置为`OFF`以禁用此功能。

    在 MySQL 8.0.31 中添加，此选项对 Linux 以外的平台没有影响。

+   `-DBUNDLE_RUNTIME_LIBRARIES=*`bool`*`

    是否将运行时库与 Windows 服务器 MSI 和 Zip 包捆绑在一起。

+   `-DCMAKE_BUILD_TYPE=*`type`*`

    要生成的构建类型：

    +   `RelWithDebInfo`：启用优化并生成调试信息。这是默认的 MySQL 构建类型。

    +   `Release`：启用优化但省略调试信息以减小构建大小。此构建类型在 MySQL 8.0.13 中添加。

    +   `Debug`：禁用优化并生成调试信息。如果启用了`WITH_DEBUG`选项，则也使用此构建类型。也就是说，`-DWITH_DEBUG=1`与`-DCMAKE_BUILD_TYPE=Debug`具有相同效果。

    选项值`None`和`MinSizeRel`不受支持。

+   `-DCPACK_MONOLITHIC_INSTALL=*`bool`*`

    此选项影响**make package**操作是生成多个安装包文件还是单个文件。如果禁用，则操作会生成多个安装包文件，这可能对于只想安装完整 MySQL 安装的子集很有用。如果启用，则会生成一个文件用于安装所有内容。

+   `-DFORCE_INSOURCE_BUILD=*`bool`*`

    定义是否强制在源代码构建。推荐使用源外构建，因为它们允许从同一源代码进行多次构建，并且可以通过删除构建目录快速进行清理。要强制在源代码构建，请使用`-DFORCE_INSOURCE_BUILD=ON`调用**CMake**。

+   `-DFORCE_COLORED_OUTPUT=*`bool`*`

    定义是否在命令行编译时为**gcc**和**clang**启用带颜色的编译器输出。默认为`OFF`。

#### 安装布局选项

`CMAKE_INSTALL_PREFIX`选项指示基本安装目录。其他具有`INSTALL_*`xxx`*`形式名称的组件位置指示选项相对于前缀进行解释，它们的值是相对路径名。它们的值不应包括前缀。

+   `-DCMAKE_INSTALL_PREFIX=*`dir_name`*`

    安装基本目录。

    可以使用`--basedir`选项在服务器启动时设置此值。

+   `-DINSTALL_BINDIR=*`dir_name`*`

    安装用户程序的位置。

+   `-DINSTALL_DOCDIR=*`dir_name`*`

    安装文档的位置。

+   `-DINSTALL_DOCREADMEDIR=*`dir_name`*`

    安装`README`文件的位置。

+   `-DINSTALL_INCLUDEDIR=*`dir_name`*`

    头文件安装位置。

+   `-DINSTALL_INFODIR=*`dir_name`*`

    Info 文件安装位置。

+   `-DINSTALL_LAYOUT=*`name`*`

    选择预定义的安装布局：

    +   `STANDALONE`：与`.tar.gz`和`.zip`包使用的布局相同。这是默认设置。

    +   `RPM`：类似于 RPM 软件包的布局。

    +   `SVR4`：Solaris 软件包布局。

    +   `DEB`：DEB 软件包布局（实验性）。

    您可以选择预定义的布局，但通过指定其他选项修改各个组件的安装位置。例如：

    ```sql
    cmake . -DINSTALL_LAYOUT=SVR4 -DMYSQL_DATADIR=/var/mysql/data
    ```

    `INSTALL_LAYOUT`的值确定`secure_file_priv`、`keyring_encrypted_file_data`和`keyring_file_data`系统变量的默认值。请参阅第 7.1.8 节“服务器系统变量”和第 8.4.4.19 节“密钥环系统变量”中这些变量的描述。

+   `-DINSTALL_LIBDIR=*`dir_name`*`

    库文件安装位置。

+   `-DINSTALL_MANDIR=*`dir_name`*`

    手册页安装位置。

+   `-DINSTALL_MYSQLKEYRINGDIR=*`dir_path`*`

    用作`keyring_file`插件数据文件位置的默认目录。默认值是平台特定的，取决于`INSTALL_LAYOUT` **CMake**选项的值；请参阅第 7.1.8 节“服务器系统变量”中`keyring_file_data`系统变量的描述。

+   `-DINSTALL_MYSQLSHAREDIR=*`dir_name`*`

    安装共享数据文件的位置。

+   `-DINSTALL_MYSQLTESTDIR=*`dir_name`*`

    安装`mysql-test`目录的位置。要禁止安装此目录，请将选项明确设置为空值（`-DINSTALL_MYSQLTESTDIR=`）。

+   `-DINSTALL_PKGCONFIGDIR=*`dir_name`*`

    用于安装 `mysqlclient.pc` 文件以供 **pkg-config** 使用的目录。默认值是 `INSTALL_LIBDIR/pkgconfig`，除非 `INSTALL_LIBDIR` 以 `/mysql` 结尾，在这种情况下会首先移除该部分。

+   `-DINSTALL_PLUGINDIR=*`dir_name`*`

    插件目录的位置。

    可以在服务器启动时使用 `--plugin_dir` 选项设置此值。

+   `-DINSTALL_PRIV_LIBDIR=*`dir_name`*`

    动态库目录的位置。

    **默认位置. ** 对于 RPM 构建，这是 `/usr/lib64/mysql/private/`，对于 DEB 是 `/usr/lib/mysql/private/`，对于 TAR 是 `lib/private/`。

    **Protobuf. ** 由于这是一个私有位置，加载器（例如 Linux 上的 `ld-linux.so`）可能无法找到 `libprotobuf.so` 文件，除非得到帮助。为了指导加载器，将 `RPATH=$ORIGIN/../$INSTALL_PRIV_LIBDIR` 添加到 **mysqld** 和 **mysqlxtest**。这对大多数情况都有效，但在使用 Resource Group 功能时，**mysqld** 是 `setsuid`，加载器会忽略包含 `$ORIGIN` 的任何 `RPATH`。为了克服这个问题，在 DEB 和 RPM 版本的 **mysqld** 中设置了目录的明确完整路径，因为目标位置是已知的。对于 tarball 安装，需要使用类似 **patchelf** 的工具对 **mysqld** 进行修补。

    这个选项是在 MySQL 8.0.18 版本中添加的。

+   `-DINSTALL_SBINDIR=*`dir_name`*`

    安装 **mysqld** 服务器的位置。

+   `-DINSTALL_SECURE_FILE_PRIVDIR=*`dir_name`*`

    `secure_file_priv` 系统变量的默认值。默认值是平台特定的，取决于 `INSTALL_LAYOUT` **CMake** 选项的值；请参阅 Section 7.1.8, “Server System Variables” 中的 `secure_file_priv` 系统变量的描述。

+   `-DINSTALL_SHAREDIR=*`dir_name`*`

    安装 `aclocal/mysql.m4` 的位置。

+   `-DINSTALL_STATIC_LIBRARIES=*`bool`*`

    是否安装静态库。默认值是`ON`。如果设置为`OFF`，这些库文件不会被安装：`libmysqlclient.a`，`libmysqlservices.a`。

+   `-DINSTALL_SUPPORTFILESDIR=*`dir_name`*`

    安装额外支持文件的位置。

+   `-DLINK_RANDOMIZE=*`bool`*`

    是否随机化**mysqld**二进制文件中符号的顺序。默认值是`OFF`。此选项仅应用于调试目的。

+   `-DLINK_RANDOMIZE_SEED=*`val`*`

    `LINK_RANDOMIZE`选项的种子值。该值是一个字符串。默认值是`mysql`，一个任意的选择。

+   `-DMYSQL_DATADIR=*`dir_name`*`

    MySQL 数据目录的位置。

    可以在服务器启动时使用`--datadir`选项设置此值。

+   `-DODBC_INCLUDES=*`dir_name`*`

    ODBC 包含目录的位置，在配置 Connector/ODBC 时可能会用到。

+   `-DODBC_LIB_DIR=*`dir_name`*`

    ODBC 库目录的位置，在配置 Connector/ODBC 时可能会用到。

+   `-DSYSCONFDIR=*`dir_name`*`

    默认的`my.cnf`选项文件目录。

    无法在服务器启动时设置此位置，但可以使用`--defaults-file=*`file_name`*`选项启动具有给定选项文件的服务器，其中*`file_name`*是文件的完整路径名。

+   `-DSYSTEMD_PID_DIR=*`dir_name`*`

    当 MySQL 由 systemd 管理时创建 PID 文件的目录名称。默认值是`/var/run/mysqld`；根据`INSTALL_LAYOUT`的值可能会隐式更改。

    除非启用了`WITH_SYSTEMD`，否则此选项将被忽略。

+   `-DSYSTEMD_SERVICE_NAME=*`name`*`

    当 MySQL 由**systemd**管理时要使用的 MySQL 服务名称。默认值是`mysqld`；根据`INSTALL_LAYOUT`的值可能会隐式更改。

    除非启用了`WITH_SYSTEMD`，否则此选项将被忽略。

+   `-DTMPDIR=*`dir_name`*`

    用于`tmpdir`系统变量的默认位置。如果未指定，则默认值为`<stdio.h>`中的`P_tmpdir`。

#### 存储引擎选项

存储引擎被构建为插件。您可以将插件构建为静态模块（编译到服务器中）或动态模块（构建为必须使用`INSTALL PLUGIN`语句或`--plugin-load`选项安装到服务器中才能使用的动态库）。一些插件可能不支持静态或动态构建。

`InnoDB`、`MyISAM`、`MERGE`、`MEMORY`和`CSV`引擎是强制性的（始终编译到服务器中）且无需显式安装。

要将存储引擎静态编译到服务器中，请使用`-DWITH_*`engine`*_STORAGE_ENGINE=1`。一些允许的*`engine`*值包括`ARCHIVE`、`BLACKHOLE`、`EXAMPLE`和`FEDERATED`。示例：

```sql
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1
```

要构建支持 NDB 集群的 MySQL，请使用`WITH_NDB`选项。(*NDB 8.0.30 及更早版本*：使用`WITH_NDBCLUSTER`。)

注意

无法在没有性能模式支持的情况下进行编译。如果希望在没有特定类型的仪器的情况下进行编译，可以使用以下**CMake**选项：

```sql
DISABLE_PSI_COND
DISABLE_PSI_DATA_LOCK
DISABLE_PSI_ERROR
DISABLE_PSI_FILE
DISABLE_PSI_IDLE
DISABLE_PSI_MEMORY
DISABLE_PSI_METADATA
DISABLE_PSI_MUTEX
DISABLE_PSI_PS
DISABLE_PSI_RWLOCK
DISABLE_PSI_SOCKET
DISABLE_PSI_SP
DISABLE_PSI_STAGE
DISABLE_PSI_STATEMENT
DISABLE_PSI_STATEMENT_DIGEST
DISABLE_PSI_TABLE
DISABLE_PSI_THREAD
DISABLE_PSI_TRANSACTION
```

例如，要在没有互斥仪器的情况下进行编译，请使用`-DDISABLE_PSI_MUTEX=1`配置 MySQL。

要排除构建中的存储引擎，请使用`-DWITH_*`engine`*_STORAGE_ENGINE=0`。示例：

```sql
-DWITH_ARCHIVE_STORAGE_ENGINE=0
-DWITH_EXAMPLE_STORAGE_ENGINE=0
-DWITH_FEDERATED_STORAGE_ENGINE=0
```

也可以使用`-DWITHOUT_*`engine`*_STORAGE_ENGINE=1`（但更倾向于`-DWITH_*`engine`*_STORAGE_ENGINE=0`）来排除构建中的存储引擎。示例：

```sql
-DWITHOUT_ARCHIVE_STORAGE_ENGINE=1
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1
```

如果对于给定的存储引擎既未指定`-DWITH_*`engine`*_STORAGE_ENGINE`也未指定`-DWITHOUT_*`engine`*_STORAGE_ENGINE`，则该引擎将作为共享模块构建，或者如果无法作为共享模块构建，则将被排除。

#### 功能选项

+   `-DADD_GDB_INDEX=*`bool`*`

    此选项确定是否启用在二进制文件中生成`.gdb_index`部分，从而使在调试器中加载它���更快。默认情况下禁用该选项。使用**lld**链接器，并且如果使用除**lld**或 GNU **gold**之外的链接器，则禁用它不起作用。

    此选项在 MySQL 8.0.18 中添加。

+   `-DCOMPILATION_COMMENT=*`string`*`

    有关编译环境的描述性注释。从 MySQL 8.0.14 开始，**mysqld**使用`COMPILATION_COMMENT_SERVER`。其他程序继续使用`COMPILATION_COMMENT`。

+   `-DCOMPRESS_DEBUG_SECTIONS=*`bool`*`

    是否压缩二进制可执行文件的调试部分（仅限 Linux）。压缩可执行文件的调试部分可以节省空间，但在构建过程中会增加额外的 CPU 时间。

    默认为`OFF`。如果未显式设置此选项，但设置了`COMPRESS_DEBUG_SECTIONS`环境变量，则该选项将从该变量中获取其值。

    此选项在 MySQL 8.0.22 中添加。

+   `-DCOMPILATION_COMMENT_SERVER=*`string`*`

    用于**mysqld**的编译环境的描述性注释（例如，设置`version_comment`系统变量）。此选项在 MySQL 8.0.14 中添加。在 8.0.14 之前，服务器使用`COMPILATION_COMMENT`。

+   `-DDEFAULT_CHARSET=*`charset_name`*`

    服务器字符集。默认情况下，MySQL 使用`utf8mb4`字符集。

    *`charset_name`*可以是`binary`、`armscii8`、`ascii`、`big5`、`cp1250`、`cp1251`、`cp1256`、`cp1257`、`cp850`、`cp852`、`cp866`、`cp932`、`dec8`、`eucjpms`、`euckr`、`gb2312`、`gbk`、`geostd8`、`greek`、`hebrew`、`hp8`、`keybcs2`、`koi8r`、`koi8u`、`latin1`、`latin2`、`latin5`、`latin7`、`macce`、`macroman`、`sjis`、`swe7`、`tis620`、`ucs2`、`ujis`、`utf8mb3`、`utf8mb4`、`utf16`、`utf16le`、`utf32`。

    可以在服务器启动时使用`--character-set-server`选项设置此值。

+   `-DDEFAULT_COLLATION=*`collation_name`*`

    服务器排序规则。默认情况下，MySQL 使用`utf8mb4_0900_ai_ci`。使用`SHOW COLLATION`语句确定每个字符集可用的排序规则。

    可以在服务器启动时使用`--collation_server`选项设置此值。

+   `-DDISABLE_PSI_COND=*`bool`*`

    是否排除性能模式条件仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_FILE=*`bool`*`

    是否排除性能模式文件仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_IDLE=*`bool`*`

    是否排除性能模式空闲仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_MEMORY=*`bool`*`

    是否排除性能模式内存仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_METADATA=*`bool`*`

    是否排除性能模式元数据仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_MUTEX=*`bool`*`

    是否排除性能模式互斥仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_RWLOCK=*`bool`*`

    是否排除性能模式读写锁仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_SOCKET=*`bool`*`

    是否排除性能模式套接字仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_SP=*`bool`*`

    是否排除性能模式存储程序仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_STAGE=*`bool`*`

    是否排除性能模式阶段仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_STATEMENT=*`bool`*`

    是否排除性能模式语句仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_STATEMENT_DIGEST=*`bool`*`

    是否排除性能模式语句摘要仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_TABLE=*`bool`*`

    是否排除性能模式表仪器。默认为`OFF`（包括）。

+   `-DDISABLE_SHARED=*`bool`*`

    是否禁用构建共享库和编译位置相关代码。默认为`OFF`（编译位置无关代码）。

    此选项未使用，并在 MySQL 8.0.18 中已删除。

+   `-DDISABLE_PSI_PS=*`bool`*`

    排除性能模式预备语句实例仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_THREAD=*`bool`*`

    排除性能模式线程仪器。默认为`OFF`（包括）。

    仅在没有任何仪器的情况下构建时禁用线程，因为其他仪器对线程有依赖。

+   `-DDISABLE_PSI_TRANSACTION=*`bool`*`

    排除性能模式事务仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_DATA_LOCK=*`bool`*`

    排除性能模式数据锁仪器。默认为`OFF`（包括）。

+   `-DDISABLE_PSI_ERROR=*`bool`*`

    排除性能模式服务器错误仪器。默认为`OFF`（包括）。

+   `-DDOWNLOAD_BOOST=*`bool`*`

    是否下载 Boost 库。默认为`OFF`。

    有关使用 Boost 的更多讨论，请参见`WITH_BOOST`选项。

+   `-DDOWNLOAD_BOOST_TIMEOUT=*`seconds`*`

    下载 Boost 库的超时时间（秒）。默认为 600 秒。

    有关使用 Boost 的更多讨论，请参见`WITH_BOOST`选项。

+   `-DENABLE_DOWNLOADS=*`bool`*`

    是否下载可选文件。例如，启用此选项后，**CMake**会下载用于运行单元测试的 Google Test 分发，或者构建 GCS Java 包装器所需的 Ant 和 JUnit。

    截至 MySQL 8.0.26，MySQL 源代码分发捆绑了用于运行单元测试的 Google Test 源代码。因此，从该版本开始，`WITH_GMOCK`和`ENABLE_DOWNLOADS` **CMake**选项已被移除，并且如果指定了这些选项，则会被忽略。

+   `-DENABLE_EXPERIMENTAL_SYSVARS=*`bool`*`

    是否启用实验性`InnoDB`系统变量。实验性系统变量适用于从事 MySQL 开发的人员，应仅在开发或测试环境中使用，并可能在未来的 MySQL 版本中被删除而不另行通知。有关实验性系统变量的信息，请参考 MySQL 源代码树中的`/storage/innobase/handler/ha_innodb.cc`。实验性系统变量可以通过搜索“PLUGIN_VAR_EXPERIMENTAL”来识别。

+   `-DWITHOUT_SERVER=*`bool`*`

    是否在没有 MySQL 服务器的情况下构建。默认为 OFF，即构建服务器。

    这被视为一个实验选项；最好与服务器一起构建。

+   `-DENABLE_GCOV=*`bool`*`

    是否包含**gcov**支持（仅限 Linux）。

+   `-DENABLE_GPROF=*`bool`*`

    是否启用**gprof**（仅优化的 Linux 构建）。

+   `-DENABLED_LOCAL_INFILE=*`bool`*`

    此选项控制 MySQL 客户端库的默认`LOCAL`功能。因此，未做明确安排的客户端将根据 MySQL 构建时指定的`ENABLED_LOCAL_INFILE`设置来禁用或启用`LOCAL`功能。

    在 MySQL 二进制发行版中，默认情况下，客户端库是使用禁用的`ENABLED_LOCAL_INFILE`编译的。如果你从源代码编译 MySQL，请根据客户端是否需要禁用或启用`LOCAL`功能，相应地使用禁用或启用`ENABLED_LOCAL_INFILE`进行配置。

    `ENABLED_LOCAL_INFILE`控制客户端端的`LOCAL`功能的默认设置。对于服务器端，`local_infile`系统变量控制服务器端的`LOCAL`功能。要明确导致服务器拒绝或允许`LOAD DATA LOCAL`语句（无论客户端程序和库在构建时或运行时如何配置），请分别使用启用或禁用`--local-infile`启动**mysqld**。`local_infile`也可以在运行时设置。请参阅第 8.1.6 节，“LOAD DATA LOCAL 的安全注意事项”。

+   `-DENABLED_PROFILING=*`bool`*`

    是否启用查询分析代码（用于`SHOW PROFILE`和`SHOW PROFILES`语句）。

+   `-DFORCE_UNSUPPORTED_COMPILER=*`bool`*`

    默认情况下，**CMake**会检查支持的编译器的最低版本；要禁用此检查，请使用`-DFORCE_UNSUPPORTED_COMPILER=ON`。

+   `-DSHOW_SUPPRESSED_COMPILER_WARNINGS=*`bool`*`

    显示被抑制的编译器警告，并且不会因为`-Werror`而失败。默认为`OFF`。

    这个选项是在 MySQL 8.0.30 版本中添加的。

+   `-DFPROFILE_GENERATE=*`bool`*`

    是否生成基于配置文件的优化（PGO）数据。此选项可用于在 GCC 中尝试 PGO。有关使用 `FPROFILE_GENERATE` 和 `FPROFILE_USE` 的信息，请参阅 MySQL 源代码分发中的 `cmake/fprofile.cmake`。这些选项已在 GCC 8 和 9 中进行了测试。

    这个选项是在 MySQL 8.0.19 版本中添加的。

+   `-DFPROFILE_USE=*`bool`*`

    是否使用基于配置文件的优化（PGO）数据。此选项可用于在 GCC 中尝试 PGO。有关使用 `FPROFILE_GENERATE` 和 `FPROFILE_USE` 的信息，请参阅 MySQL 源代码分发中的 `cmake/fprofile.cmake` 文件。这些选项已在 GCC 8 和 9 中进行了测试。

    启用 `FPROFILE_USE` 也会启用 `WITH_LTO`。

    这个选项是在 MySQL 8.0.19 版本中添加的。

+   `-DHAVE_PSI_MEMORY_INTERFACE=*`bool`*`

    是否启用性能模式内存跟踪模块，用于动态存储超对齐类型的内存分配函数（`ut::aligned_*`name`*` 库函数）。

+   `-DIGNORE_AIO_CHECK=*`bool`*`

    如果在 Linux 上给出了 `-DBUILD_CONFIG=mysql_release` 选项，则默认情况下必须链接 `libaio` 库。如果您没有 `libaio` 或不想安装它，可以通过指定 `-DIGNORE_AIO_CHECK=1` 来抑制对其的检查。

+   `-DMAX_INDEXES=*`num`*`

    每个表的最大索引数。默认值为 64。最大值为 255。小于 64 的值将被忽略，使用默认值 64。

+   `-DMYSQL_MAINTAINER_MODE=*`bool`*`

    是否启用 MySQL 维护者特定的开发环境。如果启用，此选项会导致编译器警告变为错误。

+   `-DWITH_DEVELOPER_ENTITLEMENTS=*`bool`*`

    是否向所有可执行文件添加 `get-task-allow` 权限，以便在服务器意外停止时生成核心转储。

    在 macOS 11+ 上，核心转储仅限于具有 `com.apple.security.get-task-allow` 权限的进程，此 CMake 选项启用了该权限。该权限允许其他进程附加并读取/修改进程内存，并允许 `--core-file` 正常工作。

    此选项是在 MySQL 8.0.30 中添加的。

+   `-DMUTEX_TYPE=*`type`*`

    `InnoDB` 使用的互斥类型。选项包括：

    +   `event`：使用事件互斥量。这是默认值和原始的 `InnoDB` 互斥量实现。

    +   `sys`：在 UNIX 系统上使用 POSIX 互斥量。如果可用，Windows 上使用 `CRITICAL_SECTION` 对象。

    +   `futex`：使用 Linux futexes 而不是条件变量来调度等待线程。

+   `-DMYSQLX_TCP_PORT=*`port_num`*`

    X Plugin 监听 TCP/IP 连接的端口号。默认值为 33060。

    可以使用 `mysqlx_port` 系统变量在服务器启动时设置此值。

+   `-DMYSQLX_UNIX_ADDR=*`file_name`*`

    服务器监听 X Plugin 套接字连接的 Unix 套接字文件路径。这必须是绝对路径名。默认值为 `/tmp/mysqlx.sock`。

    可以使用 `mysqlx_port` 系统变量在服务器启动时设置此值。

+   `-DMYSQL_PROJECT_NAME=*`name`*`

    对于 Windows 或 macOS，要合并到项目文件名中的项目名称。

+   `-DMYSQL_TCP_PORT=*`port_num`*`

    服务器监听 TCP/IP 连接的端口号。默认值为 3306。

    可以使用 `--port` 选项在服务器启动时设置此值。

+   `-DMYSQL_UNIX_ADDR=*`file_name`*`

    服务器监听套接字连接的 Unix 套接字文件路径。这必须是绝对路径名。默认值为 `/tmp/mysql.sock`。

    可以使用 `--socket` 选项在服务器启动时设置此值。

+   `-DOPTIMIZER_TRACE=*`bool`*`

    是否支持优化器跟踪。请参阅 MySQL 内部：优化器跟踪。

+   `-DREPRODUCIBLE_BUILD=*`bool`*`

    对于在 Linux 系统上构建的版本，此选项控制是否要特别注意创建与构建位置和时间无关的构建结果。

    此选项是在 MySQL 8.0.11 中添加的。从 MySQL 8.0.12 开始，默认为 `RelWithDebInfo` 构建的 `ON`。

+   `-DUSE_LD_GOLD=*`bool`*`

    MySQL 8.0.31 中删除了 GNU **gold**链接器支持；此 CMake 选项也已删除。

    **CMake**在可用且未明确禁用时会导致构建过程链接到 GNU **gold**链接器。要禁用此链接器的使用，请指定`-DUSE_LD_GOLD=OFF`选项。

+   `-DUSE_LD_LLD=*`bool`*`

    **CMake**在可用且未明确禁用时会导致使用 LLVM **lld**链接器为 Clang 进行链接。要禁用此链接器的使用，请指定`-DUSE_LD_LLD=OFF`选项。

    此选项是在 MySQL 8.0.16 中添加的。

+   `-DWIN_DEBUG_NO_INLINE=*`bool`*`

    是否在 Windows 上禁用函数内联。默认值为`OFF`（启用内联）。

+   `-DWITH_ANT=*`path_name`*`

    设置 Ant 的路径，在构建 GCS Java 包装器时需要。将`WITH_ANT`设置为保存 Ant 压缩包或解压缩存档的目录路径。当未设置`WITH_ANT`，或设置为特殊值`system`时，构建过程假定二进制`ant`存在于`$PATH`中。

+   `-DWITH_ASAN=*`bool`*`

    是否启用 AddressSanitizer，适用于支持它的编译器。默认值为`OFF`。

+   `-DWITH_ASAN_SCOPE=*`bool`*`

    是否启用 AddressSanitizer `-fsanitize-address-use-after-scope` Clang 标志以进行使用后范围检测。默认值为关闭。要使用此选项，必须同时启用`-DWITH_ASAN`。

+   `-DWITH_AUTHENTICATION_CLIENT_PLUGINS=*`bool`*`

    如果构建了任何相应的服务器身份验证插件，则此选项将自动启用。因此，其值取决于其他**CMake**选项，不应明确设置。

    此选项是在 MySQL 8.0.26 中添加的。

+   `-DWITH_AUTHENTICATION_LDAP=*`bool`*`

    是否在无法构建 LDAP 身份验证插件时报告错误：

    +   如果禁用此选项（默认情况下），则仅当找到所需的头文件和库时才构建 LDAP 插件。如果未找到，**CMake**会显示相关提示。

    +   如果启用此选项，则找不到所需的头文件和库将导致 CMake 生成错误，阻止服务器的构建。

    有关 LDAP 认证的信息，请参见第 8.4.1.7 节，“LDAP 可插拔认证”。

+   `-DWITH_AUTHENTICATION_PAM=*`bool`*`

    是否构建 PAM 认证插件，对于包含此插件的源树。 （请参见第 8.4.1.5 节，“PAM 可插拔认证”。）如果指定了此选项且无法编译插件，则构建将失败。

+   `-DWITH_AWS_SDK=*`path_name`*`

    亚马逊 Web 服务软件开发工具包的位置。

+   `-DWITH_BOOST=*`path_name`*`

    构建 MySQL 需要 Boost 库。这些**CMake**选项可以控制库源位置，并决定是否自动下载：

    +   `-DWITH_BOOST=*`path_name`*` 指定 Boost 库目录位置。还可以通过设置 `BOOST_ROOT` 或 `WITH_BOOST` 环境变量来指定 Boost 位置。

        `-DWITH_BOOST=system` 也是允许的，表示在标准位置上编译主机上安装了正确版本的 Boost。在这种情况下，将使用已安装的 Boost 版本，而不是任何与 MySQL 源分发包含的版本。

    +   `-DDOWNLOAD_BOOST=*`bool`*` 指定是否在指定位置不存在 Boost 源时下载 Boost 源。默认值为 `OFF`。

    +   `-DDOWNLOAD_BOOST_TIMEOUT=*`seconds`*` 下载 Boost 库的超时时间（秒）。默认值为 600 秒。

    例如，如果通常将 MySQL 构建时将对象输出放在 MySQL 源树的 `bld` 子目录中，您可以这样构建 Boost：

    ```sql
    mkdir bld
    cd bld
    cmake .. -DDOWNLOAD_BOOST=ON -DWITH_BOOST=$HOME/my_boost
    ```

    这将导致 Boost 被下载到您的主目录下的 `my_boost` 目录中。如果所需的 Boost 版本已经存在，将不会进行下载。如果所需的 Boost 版本发生变化，则会下载新版本。

    如果 Boost 已经在本地安装，并且您的编译器自行找到 Boost 头文件，则可能不需要指定前述的**CMake**选项。但是，如果 MySQL 需要的 Boost 版本发生变化，而本地安装的版本没有升级，可能会出现构建问题。使用**CMake**选项应该可以成功构建。

    使用上述设置允许将 Boost 下载到指定位置时，当所需的 Boost 版本发生变化时，您需要删除 `bld` 文件夹，重新创建它，并再次执行 **cmake** 步骤。否则，新的 Boost 版本可能不会被下载，编译可能会失败。

+   `-DWITH_CLIENT_PROTOCOL_TRACING=*`bool`*`

    是否将客户端协议跟踪框架构建到客户端库中。默认情况下，此选项已启用。

    有关编写协议跟踪客户端插件的信息，请参阅 编写协议跟踪插件。

    另请参阅 `WITH_TEST_TRACE_PLUGIN` 选项。

+   `-DWITH_CURL=*`curl_type`*`

    `curl` 库的位置。*`curl_type`* 可以是 `system`（使用系统 `curl` 库）或 `curl` 库的路径名。

+   `-DWITH_DEBUG=*`bool`*`

    是否包含调试支持。

    配置 MySQL 以启用调试支持，使您可以在启动服务器时使用 `--debug="d,parser_debug"` 选项。这会导致用于处理 SQL 语句的 Bison 解析器将解析跟踪转储到服务器的标准错误输出。通常，此输出会写入错误日志。

    `InnoDB` 存储引擎的同步调试检查在 `UNIV_DEBUG` 下定义，并且在使用 `WITH_DEBUG` 选项编译时可用。当编译时启用调试支持时，`innodb_sync_debug` 配置选项可用于启用或禁用 `InnoDB` 同步调试检查。

    启用 `WITH_DEBUG` 也会启用调试同步。此功能用于测试和调试。编译时，调试同步在运行时默认情况下是禁用的。要启用它，请使用 `--debug-sync-timeout=*`N`*` 选项启动 **mysqld**，其中 *`N`* 是大于 0 的超时值。（默认值为 0，表示禁用调试同步。）*`N`* 成为单个同步点的默认超时时间。

    当使用 `WITH_DEBUG` 选项编译时，`InnoDB` 存储引擎的同步调试检查可用。

    有关调试同步功能及如何使用同步点的描述，请参阅 MySQL 内部：测试同步。

+   `-DWITH_DEFAULT_FEATURE_SET=*`bool`*`

    是否使用`cmake/build_configurations/feature_set.cmake`中的标志。此选项在 MySQL 8.0.22 中已移除。

+   `-DWITH_EDITLINE=*`value`*`

    要使用的`libedit`/`editline`库。允许的值为`bundled`（默认）和`system`。

+   `-DWITH_FIDO=*`fido_type`*`

    `authentication_fido`身份验证插件是使用 FIDO 库实现的（请参阅第 8.4.1.11 节，“FIDO 可插拔认证”）。`WITH_FIDO`选项指示 FIDO 支持的来源：

    +   `bundled`: 使用与分发包捆绑的 FIDO 库。这是默认设置。

        从 MySQL 8.0.30 开始，MySQL 包含`fido2`版本 1.8.0\.（之前的版本使用了`fido2` 1.5.0）。

    +   `system`: 使用系统 FIDO 库。

    如果所有身份验证插件都已禁用，则`WITH_FIDO`将被禁用（设置为`none`）。

    此选项在 MySQL 8.0.27 中添加。

+   `-DWITH_GMOCK=*`path_name`*`

    用于 Google Test 基于单元测试的 googlemock 分发路径。选项值是分发 zip 文件的路径。或者，将`WITH_GMOCK`环境变量设置为路径名。还可以使用`-DENABLE_DOWNLOADS=1`，这样 CMake 会从 GitHub 下载分发包。

    如果您在构建 MySQL 时没有包含 Google Test 单元测试（通过配置不包含`WITH_GMOCK`），CMake 会显示一条消息，指示如何下载它。

    从 MySQL 8.0.26 开始，MySQL 源代码分发包含 Google Test 源代码。因此，从那个版本开始，`WITH_GMOCK`和`ENABLE_DOWNLOADS` CMake 选项被移除，并且如果指定了这些选项，则会被忽略。

+   `-DWITH_ICU={*`icu_type`*|*`path_name`*}`

    MySQL 使用国际 Unicode 组件（ICU）来支持正则表达式操作。`WITH_ICU`选项指示要包含的 ICU 支持类型或要使用的 ICU 安装路径。

    +   *`icu_type`*可以是以下值之一：

        +   `bundled`: 使用与分发包捆绑的 ICU 库。这是默认设置，也是 Windows 唯一支持的选项。

        +   `system`: 使用系统 ICU 库。

    +   *`path_name`* 是要使用的 ICU 安装路径名。这可能比使用 `system` 的 *`icu_type`* 值更可取，因为它可以防止 CMake 检测和使用系统上安装的旧版或不正确的 ICU 版本。（执行相同操作的另一种允许方式是将 `WITH_ICU` 设置为 `system` 并将 `CMAKE_PREFIX_PATH` 选项设置为 *`path_name`*。）

+   `-DWITH_INNODB_EXTRA_DEBUG=*`bool`*`

    是否包含额外的 InnoDB 调试支持。

    启用 `WITH_INNODB_EXTRA_DEBUG` 将打开额外的 InnoDB 调试检查。只有在启用 `WITH_DEBUG` 时才能启用此选项。

+   `-DWITH_INNODB_MEMCACHED=*`bool`*`

    是否生成 memcached 共享库（`libmemcached.so` 和 `innodb_engine.so`）。

+   `-DWITH_JEMALLOC=*`bool`*`

    是否链接 `-ljemalloc`。如果启用，内置的 `malloc()`、`calloc()`、`realloc()` 和 `free()` 函数将被禁用。默认值为 `OFF`。

    `WITH_JEMALLOC` 和 `WITH_TCMALLOC` 是互斥的。

    这个选项是在 MySQL 8.0.16 版本中添加的。

+   `-DWITH_WIN_JEMALLOC=*`string`*`

    在 Windows 上，传入一个包含 `jemalloc.dll` 的目录路径以启用 jemalloc 功能。构建系统将 `jemalloc.dll` 复制到与 `mysqld.exe` 和/或 `mysqld-debug.exe` 相同的目录中，并将其用于内存管理操作。如果找不到 `jemalloc.dll` 或者没有导出所需函数，则使用标准内存函数。一个 INFORMATION 级别的日志消息记录了是否找到并使用了 jemalloc。

    这个选项对于官方 MySQL Windows 二进制文件是启用的。

    这个选项是在 MySQL 8.0.29 版本中添加的。

+   `-DWITH_KEYRING_TEST=*`bool`*`

    是否构建伴随 `keyring_file` 插件的测试程序。默认值为 `OFF`。测试文件源代码位于 `plugin/keyring/keyring-test` 目录中。

+   `-DWITH_LIBEVENT=*`string`*`

    要使用哪个 `libevent` 库。允许的值为 `bundled`（默认）和 `system`。在 MySQL 8.0.21 之前，如果指定 `system`，则如果存在系统 `libevent` 库，则使用该库，否则会出现错误。在 MySQL 8.0.21 及更高版本中，如果指定 `system` 并且找不到系统 `libevent` 库，则无论如何都会出现错误，并且不会使用捆绑的 `libevent`。

    `libevent` 库是 `InnoDB` memcached、X Plugin 和 MySQL Router 所必需的。

+   `-DWITH_LIBWRAP=*`bool`*`

    是否包括`libwrap`（TCP 包装程序）支持。

+   `-DWITH_LOCK_ORDER=*`bool`*`

    是否启用 LOCK_ORDER 工具。默认情况下，此选项已禁用，服务器构建不包含任何工��。如果启用工具，则 LOCK_ORDER 工具可用，并且可以按照第 7.9.3 节，“LOCK_ORDER 工具”中描述的方式使用。

    注意

    启用`WITH_LOCK_ORDER`选项后，MySQL 构建需要**flex**程序。

    此选项在 MySQL 8.0.17 中添加。

+   `-DWITH_LSAN=*`bool`*`

    是否运行 LeakSanitizer，不包括 AddressSanitizer。默认值为`OFF`。

    此选项在 MySQL 8.0.16 中添加。

+   `-DWITH_LTO=*`bool`*`

    是否启用链接时优化器，如果编译器支持。默认值为`OFF`，除非启用`FPROFILE_USE`。

    此选项在 MySQL 8.0.13 中添加。

+   `-DWITH_LZ4=*`lz4_type`*`

    `WITH_LZ4`选项指示`zlib`支持的来源：

    +   `bundled`: 使用分发的`lz4`库。这是默认值。

    +   `system`: 使用系统的`lz4`库。如果`WITH_LZ4`设置为此值，则不会构建**lz4_decompress**实用程序。在这种情况下，可以使用系统的**lz4**命令。

+   `-DWITH_LZMA=*`lzma_type`*`

    要包含的 LZMA 库支持类型。*`lzma_type`*可以是以下值之一：

    +   `bundled`: 使用分发的 LZMA 库。这是默认值。

    +   `system`: 使用系统的 LZMA 库。

    此选项在 MySQL 8.0.16 中移除。

+   `-DWITH_MECAB={disabled|system|*`path_name`*}`

    使用此选项编译 MeCab 解析器。如果已将 MeCab 安装到其默认安装目录，请设置`-DWITH_MECAB=system`。`system`选项适用于从源代码或使用本机软件包管理工具从二进制文件安装的 MeCab。如果将 MeCab 安装到自定义安装目录，请指定 MeCab 安装路径，例如`-DWITH_MECAB=/opt/mecab`。如果`system`选项不起作用，则在所有情况下指定 MeCab 安装路径应该起作用。

    有关相关信息，请参见第 14.9.9 节，“MeCab 全文本解析器插件”。

+   `-DWITH_MSAN=*`bool`*`

    是否启用 MemorySanitizer，适用于支持它的编译器。默认为关闭。

    如果启用此选项，要使其生效，所有链接到 MySQL 的库也必须使用启用该选项进行编译。

+   `-DWITH_MSCRT_DEBUG=*`bool`*`

    是否启用 Visual Studio CRT 内存泄漏跟踪。默认为`OFF`。

+   `-DMSVC_CPPCHECK=*`bool`*`

    是否启用 MSVC 代码分析。默认为`OFF`。

+   `-DWITH_MYSQLX=*`bool`*`

    是否构建支持 X 插件。默认为`ON`。参见第二十二章，*将 MySQL 用作文档存储*。

+   `-DWITH_NUMA=*`bool`*`

    明确设置 NUMA 内存分配策略。**CMake**根据当前平台是否支持`NUMA`来设置默认值`WITH_NUMA`。对于不支持 NUMA 的平台，**CMake**的行为如下：

    +   没有 NUMA 选项（正常情况下），**CMake**会继续正常运行，只会产生以下警告：NUMA 库缺失或所需版本不可用。

    +   使用`-DWITH_NUMA=ON`，**CMake**会中止并显示以下错误：NUMA 库缺失或所需版本不可用。

+   `-DWITH_PACKAGE_FLAGS=*`bool`*`

    对于通常用于 RPM 和 Debian 软件包的标志，是否将它们添加到这些平台上的独立构建中。对于非调试构建，默认值为`ON`。

    此选项在 MySQL 8.0.26 中添加。

+   `-DWITH_PROTOBUF=*`protobuf_type`*`

    要使用的 Protocol Buffers 包。*`protobuf_type`*可以是以下值之一：

    +   `bundled`：使用与发行版捆绑的包。这是默认值。可选择使用`INSTALL_PRIV_LIBDIR`来修改动态 Protobuf 库目录。

    +   `system`：使用系统上安装的包。

    其他值将被忽略，并回退到`bundled`。

+   `-DWITH_RAPID=*`bool`*`

    是否构建快速开发周期插件。启用时，在构建树中创建一个`rapid`目录，其中包含这些插件。禁用时，在构建树中不会创建`rapid`目录。默认为`ON`，除非从源树中删除了`rapid`目录，此时默认值变为`OFF`。

+   `-DWITH_RAPIDJSON=*`rapidjson_type`*`

    要包含的 RapidJSON 库支持类型。*`rapidjson_type`*可以是以下值之一：

    +   `bundled`: 使用与发行版捆绑的 RapidJSON 库。这是默认设置。

    +   `system`: 使用系统 RapidJSON 库。需要版本 1.1.0 或更高版本。

    此选项在 MySQL 8.0.13 中添加。

+   `-DWITH_RE2=*`re2_type`*`

    要包含的 RE2 库支持类型。*`re2_type`*可以是以下值之一：

    +   `bundled`: 使用与发行版捆绑的 RE2 库。这是默认设置。

    +   `system`: 使用系统 RE2 库。

    从 MySQL 8.0.18 开始，MySQL 不再使用 RE2 库，此选项已被移除。

+   `-DWITH_ROUTER=*`bool`*`

    是否构建 MySQL Router。默认为`ON`。

    此选项在 MySQL 8.0.16 中添加。

+   `-DWITH_SSL={*`ssl_type`*`|*`path_name`*}

    为了支持加密连接、用于随机数生成的熵以及其他与加密相关的操作，MySQL 必须使用 SSL 库构建。此选项指定要使用的 SSL 库。

    +   *`ssl_type`*可以是以下值之一：

        +   `system`: 使用系统 OpenSSL 库。这是默认设置。

            在 macOS 和 Windows 上，使用`system`配置 MySQL 构建，就好像使用*`path_name`*指向手动安装的 OpenSSL 库调用了 CMake 一样。这是因为它们没有系统 SSL 库。在 macOS 上，*brew install openssl*安装到`/usr/local/opt/openssl`，以便`system`可以找到它。在 Windows 上，它检查`%ProgramFiles%/OpenSSL`，`%ProgramFiles%/OpenSSL-Win32`，`%ProgramFiles%/OpenSSL-Win64`，`C:/OpenSSL`，`C:/OpenSSL-Win32`和`C:/OpenSSL-Win64`。

        +   `yes`: 这是`system`的同义词。

        +   `openssl*`version`*`:（*MySQL 8.0.30 及更高版本：*）使用替代的 OpenSSL 系统包，如 EL7 上的`openssl11`，或 EL8 上的`openssl3`。

            身份验证插件，如 LDAP 和 Kerberos，已禁用，因为它们不支持这些替代版本的 OpenSSL。

    +   *`path_name`*是要使用的 OpenSSL 安装路径名。这可能比使用`system`的*`ssl_type`*值更可取，因为它可以防止 CMake 检测和使用系统上安装的旧版或不正确的 OpenSSL 版本。（执行相同操作的另一种允许的方法是将`WITH_SSL`设置为`system`并将`CMAKE_PREFIX_PATH`选项设置为*`path_name`*。）

    有关配置 SSL 库的其他信息，请参见第 2.8.6 节“配置 SSL 库支持”。

+   `-DWITH_SYSTEMD=*`bool`*`

    是否启用 **systemd** 支持文件的安装。默认情况下，此选项已禁用。启用时，将安装 **systemd** 支持文件，并且诸如 **mysqld_safe** 和 System V 初始化脚本等脚本将不会安装。在 **systemd** 不可用的平台上，启用 `WITH_SYSTEMD` 将导致 **CMake** 报错。

    有关使用 **systemd** 的更多信息，请参见 第 2.5.9 节，“使用 systemd 管理 MySQL 服务器”。该部分还包括有关指定在 `[mysqld_safe]` 选项组中另行指定的选项的信息。因为在使用 **systemd** 时未安装 **mysqld_safe**，这些选项必须以另一种方式指定。

+   `-DWITH_SYSTEM_LIBS=*`bool`*`

    这个选项作为一个“总称”选项，用于设置任何未明确设置的以下 **CMake** 选项的 `system` 值：`WITH_CURL`、`WITH_EDITLINE`、`WITH_FIDO`、`WITH_ICU`、`WITH_LIBEVENT`、`WITH_LZ4`、`WITH_LZMA`、`WITH_PROTOBUF`、`WITH_RE2`、`WITH_SSL`、`WITH_ZSTD`。

    `WITH_ZLIB` 在 MySQL 8.0.30 之前包含在此处。

+   `-DWITH_SYSTEMD_DEBUG=*`bool`*`

    是否生成额外的 **systemd** 调试信息，用于在 **systemd** 上运行 MySQL 的平台。默���值为 `OFF`。

    这个选项是在 MySQL 8.0.22 版本中添加的。

+   `-DWITH_TCMALLOC=*`bool`*`

    是否链接 `-ltcmalloc`。如果启用，内置的 `malloc()`、`calloc()`、`realloc()` 和 `free()` 例程将被禁用。默认值为 `OFF`。

    `WITH_TCMALLOC` 和 `WITH_JEMALLOC` 是互斥的。

    这个选项是在 MySQL 8.0.22 版本中添加的。

+   `-DWITH_TEST_TRACE_PLUGIN=*`bool`*`

    是否构建测试协议跟踪客户端插件（请参阅使用测试协议跟踪插件）。默认情况下，此选项已禁用。启用此选项除非启用了`WITH_CLIENT_PROTOCOL_TRACING`选项，否则不会产生任何效果。如果 MySQL 配置了这两个选项，`libmysqlclient` 客户端库将内置测试协议跟踪插件，并且所有标准 MySQL 客户端都会加载该插件。但是，即使启用了测试插件，默认情况下也不会产生任何效果。可以使用环境变量控制插件；请参阅使用测试协议跟踪插件。

    注意

    如果要使用自己的协议跟踪插件，请*不要*启用`WITH_TEST_TRACE_PLUGIN`选项，因为一次只能加载一个这样的插件，尝试加载第二个插件时会出错。如果您已经使用启用了测试协议跟踪插件的 MySQL 进行构建以查看其工作原理，则必须在使用自己的插件之前重新构建 MySQL 以禁用它。

    有关编写跟踪插件的信息，请参阅编写协议跟踪插件。

+   `-DWITH_TSAN=*`bool`*`

    是否启用 ThreadSanitizer，适用于支持它的编译器。默认情况下为关闭。

+   `-DWITH_UBSAN=*`bool`*`

    是否启用未定义行为检测器，适用于支持它的编译器。默认情况下为关闭。

+   `-DWITH_UNIT_TESTS={ON|OFF}`

    如果启用，使用单元测试编译 MySQL。默认情况下为`ON`，除非服务器未被编译。

+   `-DWITH_UNIXODBC=*`1`*`

    启用 unixODBC 支持，用于 Connector/ODBC。

+   `-DWITH_VALGRIND=*`bool`*`

    是否在 MySQL 代码中暴露 Valgrind API 的 Valgrind 头文件。默认情况下为`OFF`。

    要生成 Valgrind-aware 调试构建，通常将`-DWITH_VALGRIND=1`与`-DWITH_DEBUG=1`结合使用。请参阅构建调试配置。

+   `-DWITH_ZLIB=*`zlib_type`*`

    一些功能要求服务器构建时具有压缩库支持，例如 `COMPRESS()` 和 `UNCOMPRESS()` 函数，以及客户端/服务器协议的压缩。`WITH_ZLIB` 选项指示 `zlib` 支持的来源：

    在 MYSQL 8.0.32 及更高版本中，`zlib` 的最低支持版本为 1.2.13。

    +   `bundled`：使用分发的 `zlib` 库。这是默认值。

    +   `system`：使用系统的 `zlib` 库。如果 `WITH_ZLIB` 设置为此值，则不会构建 **zlib_decompress** 实用程序。在这种情况下，可以使用系统的 **openssl zlib** 命令。

+   `-DWITH_ZSTD=*`zstd_type`*`

    使用 `zstd` 算法进行连接压缩（参见 Section 6.2.8, “Connection Compression Control”) 需要服务器构建时支持 `zstd` 库。`WITH_ZSTD` 选项指示 `zstd` 支持的来源：

    +   `bundled`：使用分发的 `zstd` 库。这是默认值。

    +   `system`：使用系统的 `zstd` 库。

    此选项在 MySQL 8.0.18 中添加。

#### 编译器标志

+   `-DCMAKE_C_FLAGS="*`flags`*`"

    C 编译器的标志。

+   `-DCMAKE_CXX_FLAGS="*`flags`*`"

    C++ 编译器的标志。

+   `-DWITH_DEFAULT_COMPILER_OPTIONS=*`bool`*`

    是否使用 `cmake/build_configurations/compiler_options.cmake` 中的标志。

    注意

    所有优化标志都经过 MySQL 构建团队精心选择和测试。覆盖它们可能导致意外结果，并且由您自行承担风险。

+   `-DOPTIMIZE_SANITIZER_BUILDS=*`bool`*`

    是否将 `-O1 -fno-inline` 添加到 sanitizer 构建中。默认值为 `ON`。

要指定自己的 C 和 C++ 编译器标志，对于不影响优化的标志，请使用 `CMAKE_C_FLAGS` 和 `CMAKE_CXX_FLAGS` CMake 选项。

在提供自己的编译器标志时，您可能希望同时指定`CMAKE_BUILD_TYPE`。

例如，在 64 位 Linux 机器上创建 32 位发布构建，可以执行以下操作：

```sql
$> mkdir build
$> cd build
$> cmake .. -DCMAKE_C_FLAGS=-m32 \
  -DCMAKE_CXX_FLAGS=-m32 \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

如果设置影响优化的标志（`-O*`number`*`），则必须设置 `CMAKE_C_FLAGS_*`build_type`*` 和/或 `CMAKE_CXX_FLAGS_*`build_type`*` 选项，其中 *`build_type`* 对应于 `CMAKE_BUILD_TYPE` 的值。要为默认构建类型（`RelWithDebInfo`）指定不同的优化设置，请设置 `CMAKE_C_FLAGS_RELWITHDEBINFO` 和 `CMAKE_CXX_FLAGS_RELWITHDEBINFO` 选项。例如，在 Linux 上使用 `-O3` 编译并带有调试符号，执行以下操作：

```sql
$> cmake .. -DCMAKE_C_FLAGS_RELWITHDEBINFO="-O3 -g" \
  -DCMAKE_CXX_FLAGS_RELWITHDEBINFO="-O3 -g"
```

#### 编译 NDB 集群的 CMake 选项

在构建带有 NDB 集群支持的 MySQL 源代码时，请使用以下选项。

+   `-DMEMCACHED_HOME=*`dir_name`*`

    `NDB` 在 NDB 8.0.23 中移除了对 memcached 的支持；因此，在此版本或更高版本中不再支持使用此选项构建 `NDB`。

+   `-DNDB_UTILS_LINK_DYNAMIC={ON|OFF}`

    控制 NDB 实用程序（如 **ndb_drop_table**）是否静态链接（`OFF`）或动态链接（`ON`）到 `ndbclient`；默认为静态链接（`OFF`）。通常在构建这些工具时使用静态链接以避免 `LD_LIBRARY_PATH` 的问题，或者当安装了多个版本的 `ndbclient` 时使用。此选项适用于创建 Docker 镜像和可能需要精确控制目标环境并希望减小镜像大小的情况。

    在 NDB 8.0.22 中添加。

+   `-DWITH_BUNDLED_LIBEVENT={ON|OFF}`

    `NDB` 在 NDB 8.0.23 中移除了对 memcached 的支持；因此，在此版本或更高版本中不再支持使用此选项构建 `NDB`。

+   `-DWITH_BUNDLED_MEMCACHED={ON|OFF}`

    `NDB` 在 NDB 8.0.23 中移除了对 memcached 的支持；因此，在此版本或更高版本中不再支持使用此选项构建 `NDB`。

+   `-DWITH_CLASSPATH=*`path`*`

    设置构建 MySQL NDB 集群 Java 连接器的类路径。默认为空。如果使用 `-DWITH_NDB_JAVA=OFF`，则忽略此选项。

+   `-DWITH_ERROR_INSERT={ON|OFF}`

    启用 `NDB` 内核中的错误注入。仅用于测试；不适用于构建生产二进制文件。默认为 `OFF`。

+   `-DWITH_NDB={ON|OFF}`

    构建 MySQL NDB 集群；构建 NDB 插件和所有 NDB 集群程序。

    在 NDB 8.0.31 中添加。

+   `-DWITH_NDBAPI_EXAMPLES={ON|OFF}`

    在`storage/ndb/ndbapi-examples/`中构建 NDB API 示例程序。有关这些示例的信息，请参阅 NDB API 示例。

+   `-DWITH_NDBCLUSTER_STORAGE_ENGINE={ON|OFF}`

    *NDB 8.0.30 及更早版本*：仅供内部使用；可能不总是按预期工作。要构建带有`NDB`支持的内容，请改用`WITH_NDBCLUSTER`。

    *NDB 8.0.31 及更高版本*：仅控制`ndbcluster`插件是否包含在构建中；`WITH_NDB`会自动启用此选项，因此建议您改用`WITH_NDB`。

+   `-DWITH_NDBCLUSTER={ON|OFF}`

    在**mysqld**中构建并链接对`NDB`存储引擎的支持。

    从 NDB 8.0.31 开始，此选项已弃用，并将最终删除；请改用`WITH_NDB`。

+   `-DWITH_NDBMTD={ON|OFF}`

    构建多线程数据节点可执行文件**ndbmtd**。默认值为`ON`。

+   `-DWITH_NDB_DEBUG={ON|OFF}`

    启用构建 NDB 集群二进制文件的调试版本。默认值为`OFF`。

+   `-DWITH_NDB_JAVA={ON|OFF}`

    启用构建带有 Java 支持的 NDB 集群，包括对 ClusterJ 的支持（请参阅 MySQL NDB 集群 Java 连接器）。

    默认情况下，此选项为`ON`。如果不希望使用 Java 支持编译 NDB 集群，则在运行**CMake**时必须显式禁用它，即指定`-DWITH_NDB_JAVA=OFF`。否则，如果找不到 Java，则构建配置将失败。

+   `-DWITH_NDB_PORT=*`port`*`

    导致构建的 NDB 集群管理服务器（**ndb_mgmd**）默认使用此*`port`*。如果未设置此选项，则生成的管理服务器默认尝试使用端口 1186。

+   `-DWITH_NDB_TEST={ON|OFF}`

    如果启用，将包括一组 NDB API 测试程序。默认值为`OFF`。

+   `-DWITH_PLUGIN_NDBCLUSTER={ON|OFF}`

    仅供内部使用；可能不总是按预期工作。此选项在 NDB 8.0.31 中已移除；请改用`WITH_NDB`来构建 MySQL NDB 集群。(*NDB 8.0.30 及更早版本*：请使用`WITH_NDBCLUSTER`。)
