# SQLite Android 绑定

> 原文：[`sqlite.org/android/`](https://sqlite.org/android/)

文档登录☰首页 搜索 文件 # SQLite Android 绑定

SQLite 库是 Android 环境的核心部分。Java 应用程序和内容提供者使用 [android.database.sqlite](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html) 命名空间中的接口访问 SQLite。

对于大多数应用程序来说，这很方便且运行良好。然而，这意味着应用程序必须接受作为操作系统一部分安装在目标设备上的 SQLite 版本和构建。如果您的应用程序恰好需要更新版本的 SQLite，或者安装了自定义扩展或[VFS](http://www.sqlite.org/vfs.html)，那么您就没那么幸运了。

一个解决方案是直接将 SQLite 库捆绑到应用程序中，绕过 Android 内置的版本。这个项目，*SQLite Android 绑定*，提供了一种简单的方法来做到这一点。这允许应用程序使用自定义版本或 SQLite 构建，而不受部署到的 Android 版本的限制，同时继续使用标准的 Android 接口。

可用的用户文档：

+   安装指南 - 此页面描述了将 SQLite Android 绑定与应用程序集成的各种方法。

+   应用程序编程 - 此页面描述了使用 SQLite Android 绑定而不是 Android 内置 SQLite 支持所需的非常小的代码修改。

+   使用简单加密扩展 (SEE) - 关于使用专有加密扩展 SEE 的额外说明。本页面由 Fossil 2.25 [fc8d476aca] 2024-06-19 18:26:12 生成，耗时约 0.004 秒。
