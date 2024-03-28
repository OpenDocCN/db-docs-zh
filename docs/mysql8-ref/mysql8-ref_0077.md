# 2.6 使用 Unbreakable Linux Network (ULN) 安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/uln-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/uln-installation.html)

Linux 支持多种不同的解决方案来安装 MySQL，详见 第 2.5 节，“在 Linux 上安装 MySQL”。其中一种方法在本节中介绍，即从 Oracle 的 Unbreakable Linux Network (ULN) 安装。您可以在 [`linux.oracle.com/`](http://linux.oracle.com/) 下找到有关 Oracle Linux 和 ULN 的信息。

要使用 ULN，您需要获取 ULN 登录并将用于安装的机器注册到 ULN。这在 [ULN FAQ](https://linux.oracle.com/uln_faq.html) 中有详细描述。该页面还描述了如何安装和更新软件包。

社区和商业软件包都得到支持，每个都提供三个 MySQL 频道：

+   `服务器`：MySQL 服务器

+   `连接器`：MySQL Connector/C++、MySQL Connector/J、MySQL Connector/ODBC 和 MySQL Connector/Python。

+   `工具`：MySQL Router、MySQL Shell 和 MySQL Workbench

社区频道对所有 ULN 用户都是可用的。

访问 oracle.linux.com 上的商业 MySQL ULN 软件包需要您提供具有有效商业许可证的 MySQL CSI（企业版或标准版）。截至目前，有效购买订单号为 60944、60945、64911 和 64912。适当的 CSI 会在 ULN GUI 界面中提供商业 MySQL 订阅频道。

一旦使用 ULN 安装了 MySQL，您可以在 第 2.5.7 节，“从本机软件库在 Linux 上安装 MySQL” 中找到有关启动和停止服务器等信息，特别是在 第 2.5.4 节，“使用来自 Oracle 的 RPM 软件包在 Linux 上安装 MySQL” 下。

如果您要将软件包源更改为使用 ULN，但不更改使用的 MySQL 构建版本，则请备份数据，删除现有的二进制文件，并用 ULN 中的文件替换它们。如果涉及到更改构建版本，我们建议备份为转储文件（**mysqldump** 或 **mysqlpump** 或来自 MySQL Shell 的备份实用程序），以防您需要在新的二进制文件放置后重���数据。如果这次转向 ULN 跨越了版本边界，请在继续之前参考本节：第三章，*升级 MySQL*。

注意

Oracle Linux 8 从 MySQL 8.0.17 开始得到支持，社区工具和连接器频道是在 MySQL 8.0.24 发布时添加的。
