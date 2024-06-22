# SQLite Android 绑定

> 原文：[`sqlite.org/android/`](http://sqlite.org/android/)

文档登录☰首页 搜索 文件 # SQLite Android 绑定

SQLite 库是 Android 环境的核心组成部分。Java 应用程序和内容提供者使用[android.database.sqlite](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html)命名空间中的接口访问 SQLite。

对于大多数应用程序来说，这很方便且效果良好。然而，这意味着应用程序必须接受作为操作系统一部分安装在目标设备上的 SQLite 版本和构建。如果您的应用程序恰好需要更新的 SQLite 版本或安装了自定义扩展或[VFS](http://www.sqlite.org/vfs.html)，那就倒霉了。

一个解决方案是直接将 SQLite 库捆绑到应用程序中，绕过内置到 Android 中的版本。这个项目，*SQLite Android 绑定*，提供了一个简单的方法来实现这一点。这允许应用程序使用自定义的 SQLite 构建或版本，而不受其部署到的 Android 版本的限制，同时继续使用标准的 Android 接口。

可用的用户文档：

+   安装指南 - 本页面描述了将 SQLite Android 绑定集成到应用程序中的各种方法。

+   应用程序编程 - 本页面描述了使用 SQLite Android 绑定而不是 Android 内置 SQLite 支持所需的极少代码修改。

+   使用简单加密扩展 (SEE) - 关于使用专有加密扩展 SEE 的额外注意事项。此页面由 Fossil 2.25 [fc8d476aca] 2024-06-19 18:26:12 生成，用时约 0.004 秒。
