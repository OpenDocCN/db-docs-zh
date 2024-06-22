# Fossil

> 原文：[`www.fossil-scm.org/`](http://www.fossil-scm.org/)

![Fossil](https://www.fossil-scm.org/) 一个完整的软件配置管理系统登录☰主页 时间轴 文档 代码 [论坛](https://fossil-scm.org/forum/forum) 下载

# 一个完整的软件配置管理系统

### 它是什么？

+   下载

+   快速入门

+   安装

+   [支持/论坛](https://fossil-scm.org/forum)

+   提示与技巧

+   更改日志

+   许可证

+   用户链接

+   黑客如何

+   Fossil vs. Git

+   文档索引

![Fossil logo](img/dce8566cc8fcbc280c360091c2d0496b.png)

Fossil 是一个简单、高可靠性的分布式[SCM](https://en.wikipedia.org/wiki/Software_configuration_management)系统，具备这些高级功能：

1.  **项目管理** — 除了执行像 Git 和 Mercurial 一样的分布式版本控制，Fossil 还支持 bug 跟踪，wiki，论坛，电子邮件警报，聊天和技术笔记。

1.  **内置 Web 界面** — Fossil 具有内置、可主题化、可扩展且直观的 Web 界面，具有丰富的信息页面（示例），有助于提升情境意识。

    整个网站就是 Fossil 的一个运行实例。你在这里看到的页面都是 wiki 或嵌入文档，或者（在下载页面的情况下）未版本化文件。

1.  **一体化** — Fossil 是一个单独的自包含可执行文件。要安装它，只需下载适用于 Linux、Mac 或 Windows 的预编译二进制文件，并将其放置在 $PATH 中。易于编译的源代码也是可用的。

1.  **自托管友好** — 使用各种技术在几分钟内建立一个项目网站。Fossil CPU 和内存效率高。大多数项目可以舒适地托管在每月 $5 的 VPS 或者 Raspberry Pi 上。您还可以设置一个自动的 GitHub 镜像。

1.  **简单网络** — Fossil 使用普通的 HTTPS（或者如果您喜欢的话，SSH）进行网络通信，因此可以很好地在防火墙和代理后使用。该协议的带宽效率高到足以使 Fossil 在拨号、弱 3G 或者飞机上的 Wifi 上舒适地使用。

1.  **自动同步** — Fossil 支持"自动同步"模式，通过减少分布式项目中常见的分叉和合并来帮助项目持续前进。

1.  **强大可靠** — Fossil 使用一个持久文件格式存储内容在 SQLite 数据库中，因此即使在断电或系统崩溃时也能保证事务的原子性。自动的自检在每次提交之前验证仓库的所有方面是否一致。

1.  **自由开源** — 2-clause BSD 许可证。

* * *

### 最新版本：2.24（2024-04-23）

+   下载

+   变更摘要

+   2.24 版本的提交记录

+   从 2.24 版本衍生的提交记录

+   所有过往版本的时间线

* * *

### 快速开始

1.  下载 或通过包管理器安装，或者从源代码编译。

1.  `fossil init` *REPOSITORY-DIR/new-repository*

1.  `fossil open` *REPOSITORY-DIR/new-repository*

1.  `fossil add` *文件或目录*

1.  `fossil commit -m` "*提交信息*"

1.  `fossil ui`

1.  根据需要以任何顺序重复步骤 4、5 和 6。参见快速开始指南获取更多详细信息。此页面由 Fossil 2.25 [fc8d476aca] 2024-06-19 18:26:12 生成，耗时约 0.008 秒。
