# 第三章 升级 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrading.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrading.html)

**目录**

3.1 开始之前

3.2 升级路径

3.3 升级最佳实践

3.4 MySQL 升级过程升级了什么

3.5 MySQL 8.0 中的更改

3.6 准备升级安装

3.7 在 Unix/Linux 上升级 MySQL 二进制或基于包的安装

3.8 使用 MySQL Yum 存储库升级 MySQL

3.9 使用 MySQL APT 存储库升级 MySQL

3.10 使用 MySQL SLES 存储库升级 MySQL

3.11 在 Windows 上升级 MySQL

3.12 升级 MySQL 的 Docker 安装

3.13 升级故障排除

3.14 重建或修复表或索引

3.15 将 MySQL 数据库复制到另一台机器

本章描述了升级 MySQL 安装的步骤。

升级是一个常见的过程，因为您可以在相同的 MySQL 发行系列中获取错误修复或在主要 MySQL 发行版之间获取重要功能。您首先在一些测试系统上执行此过程，以确保一切顺利运行，然后在生产系统上执行。

注意

在以下讨论中，必须使用具有管理权限的 MySQL 帐户运行的 MySQL 命令包括在命令行中使用 `-u `root`` 指定 MySQL `root` 用户。需要 `root` 密码的命令还包括 `-p` 选项。因为 `-p` 后面没有选项值，这些命令会提示输入密码。在提示时输入密码并按 Enter 键。

SQL 语句可以使用**mysql**命令行客户端执行（以 `root` 身份连接以确保具有必要的权限）。
