# 22.3.1 MySQL Shell

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-shell.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-shell.html)

本快速入门指南假定您对 MySQL Shell 有一定的了解。以下部分是一个高级概述，请参阅 MySQL Shell 文档以获取更多信息。MySQL Shell 是到 MySQL 服务器的统一脚本接口。它支持 JavaScript 和 Python 脚本。JavaScript 是默认处理模式。

#### 启动 MySQL Shell

安装并启动 MySQL 服务器后，将 MySQL Shell 连接到服务器实例。您需要知道要连接的 MySQL 服务器实例的地址。要将实例用作文档存储，服务器实例必须安装 X 插件，并且您应该使用 X 协议连接到服务器。例如，要连接到默认 X 协议端口 33060 上的实例 `ds1.example.com`，请使用网络字符串 `*`user`*@ds1.example.com:33060`。

提示

如果您使用经典的 MySQL 协议连接到实例，例如使用默认的`port` 3306，而不是`mysqlx_port`，则*无法*使用本教程中展示的文档存储功能。例如，`db` 全局对象未被填充。要使用文档存储功能，请始终使用 X 协议连接。

如果 MySQL Shell 尚未运行，请打开一个终端窗口并输入：

```sql
mysqlsh *user*@ds1.example.com:33060/world_x
```

或者，如果 MySQL Shell 已在运行，请使用 `\connect` 命令输入：

```sql
\connect *user*@ds1.example.com:33060/world_x
```

您需要指定要将 MySQL Shell 连接到的 MySQL 服务器实例的地址。例如，在上一个示例中：

+   *`user`* 代表您的 MySQL 帐户的用户名。

+   `ds1.example.com` 是运行 MySQL 的服务器实例的主机名。请将其替换为您用作文档存储的 MySQL 服务器实例的主机名。

+   本会话的默认模式为 `world_x`。有关设置 `world_x` 模式的说明，请参见 Section 22.3.2, “下载和导入 world_x 数据库”。

有关更多信息，请参见 Section 6.2.5, “使用类似 URI 字符串或键值对连接到服务器”。

一旦 MySQL Shell 打开，`mysql-js>` 提示表示此会话的活动语言为 JavaScript。

```sql
mysql-js>
```

MySQL Shell 支持以下输入行编辑：

+   **左箭头** 和 **右箭头** 键在当前输入行内水平移动。

+   **上箭头** 和 **下箭头** 键在先前输入的行集中上下移动。

+   **Backspace** 删除光标前的字符，输入新字符会在光标位置输入。

+   **Enter**发送当前输入行到服务器。

#### 获取 MySQL Shell 帮助

在命令解释器提示符处输入**mysqlsh --help**以获取命令行选项列表。

```sql
mysqlsh --help
```

在 MySQL Shell 提示符处输入`\help`以获取可用命令及其描述列表。

```sql
mysql-js> \help
```

输入`\help`后跟命令名称以获取有关单个 MySQL Shell 命令的详细帮助。例如，要查看`\connect`命令的帮助，请输入：

```sql
mysql-js> \help \connect
```

#### 退出 MySQL Shell

要退出 MySQL Shell，请输入以下命令：

```sql
mysql-js> \quit
```

#### 相关信息

+   查看交互式代码执行以了解 MySQL Shell 中交互式代码执行的工作原理。

+   查看使用 MySQL Shell 入门以了解会话和连接选项。
