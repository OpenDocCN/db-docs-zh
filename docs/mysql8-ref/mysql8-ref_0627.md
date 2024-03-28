> 原文：[`dev.mysql.com/doc/refman/8.0/en/symbolic-links-to-databases.html`](https://dev.mysql.com/doc/refman/8.0/en/symbolic-links-to-databases.html)

#### 10.12.2.1 在 Unix 上使用符号链接进行数据库操作

在 Unix 上，可以按照以下步骤使用符号链接来创建数据库：

1.  使用`CREATE DATABASE`创建数据库：

    ```sql
    mysql> CREATE DATABASE mydb1;
    ```

    使用`CREATE DATABASE`在 MySQL 数据目录中创建数据库，并允许服务器更新数据字典中有关数据库目录的信息。

1.  停止服务器以确保在移动数据库时不会发生任何活动。

1.  将数据库目录移动到某个具有可用空间的磁盘上。例如，使用**tar**或**mv**。如果使用复制而不是移动数据库目录的方法，请在复制后删除原始数据库目录。

1.  在数据目录中创建一个软链接，指向已移动的数据库目录。

    ```sql
    $> ln -s */path/to/mydb1* */path/to/datadir*
    ```

    该命令在数据目录中创建一个名为`mydb1`的符号链接。

1.  重新启动服务器。
