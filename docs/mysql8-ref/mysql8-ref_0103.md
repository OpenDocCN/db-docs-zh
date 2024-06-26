# 3.1 开始之前

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrade-before-you-begin.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrade-before-you-begin.html)

在升级之前，请查看本节中的信息。执行任何建议的操作。

+   了解升级过程中可能发生的情况。请参见第 3.4 节，“MySQL 升级过程升级了什么”。

+   通过创建备份来保护您的数据。备份应包括包含 MySQL 数据字典表和系统表的`mysql`系统数据库。参见第 9.2 节，“数据库备份方法”。

    重要

    从 MySQL 8.0 降级到 MySQL 5.7，或从一个 MySQL 8.0 版本降级到以前的 MySQL 8.0 版本，不受支持。唯一支持的替代方法是恢复在升级*之前*进行的备份。因此，在开始升级过程之前，备份数据至关重要。

+   查看第 3.2 节，“升级路径”以确保您打算的升级路径得到支持。

+   查看第 3.5 节，“MySQL 8.0 中的更改”以了解升级前应注意的更改。某些更改可能需要采取行动。

+   查看第 1.3 节，“MySQL 8.0 中的新功能”以了解已弃用和移除的功能。如果您使用其中任何功能，则升级可能需要对这些功能进行更改。

+   查看第 1.4 节，“MySQL 8.0 中添加、弃用或移除的服务器和状态变量和选项”。如果您使用已弃用或移除的变量，则升级可能需要配置更改。

+   查看发布说明以获取有关修复、更改和新功能的信息。

+   如果您使用复制，请查看第 19.5.3 节，“升级复制拓扑”。

+   查看第 3.3 节，“升级最佳实践”并相应地进行计划。

+   升级过程因平台和初始安装方式而异。使用适用于当前 MySQL 安装的过程：

    +   对于非 Windows 平台上的二进制和基于软件包的安装，请参考第 3.7 节，“在 Unix/Linux 上升级 MySQL 二进制或基于软件包的安装”。

        注意

        对于支持的 Linux 发行版，升级基于软件包的安装的首选方法是使用 MySQL 软件仓库（MySQL Yum 仓库，MySQL APT 仓库和 MySQL SLES 仓库）。

    +   对于在企业 Linux 平台或 Fedora 上使用 MySQL Yum 仓库进行安装，请参考 第 3.8 节，“使用 MySQL Yum 仓库升级 MySQL”。

    +   对于在 Ubuntu 上使用 MySQL APT 仓库进行安装，请参考 第 3.9 节，“使用 MySQL APT 仓库升级 MySQL”。

    +   对于在 SLES 上使用 MySQL SLES 仓库进行安装，请参考 第 3.10 节，“使用 MySQL SLES 仓库升级 MySQL”。

    +   对于使用 Docker 进行的安装，请参考 第 3.12 节，“升级 Docker 安装的 MySQL”。

    +   对于在 Windows 上安装，请参考 第 3.11 节，“在 Windows 上升级 MySQL”。

+   如果您的 MySQL 安装包含大量数据，在原地升级后可能需要很长时间才能转换，那么创建一个用于评估所需转换和执行工作量的测试实例可能会很有用。要创建一个测试实例，请复制包含 `mysql` 数据库和其他数据库但不包含数据的 MySQL 实例。在测试实例上运行升级过程，以评估执行实际数据转换所需的工作量。

+   重新构建和重新安装 MySQL 语言接口是在安装或升级到新版本的 MySQL 时建议的。这适用于 MySQL 接口，如 PHP `mysql` 扩展和 Perl `DBD::mysql` 模块。
