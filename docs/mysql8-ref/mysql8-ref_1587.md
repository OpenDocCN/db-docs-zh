# 22.4.2 下载并导入 world_x 数据库

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-download.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-download.html)

作为快速入门指南的一部分，提供了一个示例模式，称为`world_x`模式。许多示例演示了使用此模式的文档存储功能。启动您的 MySQL 服务器以加载`world_x`模式，然后按照以下步骤操作：

1.  下载[world_x-db.zip](http://downloads.mysql.com/docs/world_x-db.zip)。

1.  将安装归档文件提取到临时位置，如`/tmp/`。解压缩归档文件会生成一个名为`world_x.sql`的单个文件。

1.  将`world_x.sql`文件导入到您的服务器。您可以选择：

    +   在 SQL 模式下启动 MySQL Shell，并通过以下方式导入文件：

        ```sql
        mysqlsh -u root --sql --file /tmp/world_x-db/world_x.sql
        Enter password: ****
        ```

    +   在运行时将 MySQL Shell 设置为 SQL 模式，并通过以下方式源化模式文件：

        ```sql
        \sql
        Switching to SQL mode... Commands end with ;
        \source /tmp/world_x-db/world_x.sql
        ```

    将`/tmp/`替换为您系统上`world_x.sql`文件的路径。如果提示，请输入密码。只要账户有权限创建新模式，非 root 账户也可以使用。

#### world_x 模式

`world_x`示例模式包含以下 JSON 集合和关系表：

+   集合

    +   `countryinfo`：世界各国的信息。

+   表

    +   `country`：世界各国的最少信息。

    +   `city`：这些国家中一些城市的信息。

    +   `countrylanguage`：每个国家使用的语言。

#### 相关信息

+   MySQL Shell 会话解释了会话类型。
