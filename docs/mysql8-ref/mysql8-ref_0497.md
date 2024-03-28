> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-copying-to-other-server.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-copying-to-other-server.html)

#### 9.4.5.2 从一个服务器复制数据库到另一个服务器

在服务器 1 上：

```sql
$> mysqldump --databases db1 > dump.sql
```

将转储文件从服务器 1 复制到服务器 2。

在服务器 2 上：

```sql
$> mysql < dump.sql
```

使用`--databases`选项与**mysqldump**命令行一起使用，导致转储文件包含`CREATE DATABASE`和`USE`语句，如果数据库存在则创建数据库，并将其设置为重新加载数据的默认数据库。

或者，您可以从**mysqldump**命令中省略`--databases`。然后，您需要在服务器 2 上创建数据库（如果需要），并在重新加载转储文件时将其指定为默认数据库。

在服务器 1 上：

```sql
$> mysqldump db1 > dump.sql
```

在服务器 2 上：

```sql
$> mysqladmin create db1
$> mysql db1 < dump.sql
```

在这种情况下，您可以指定不同的数据库名称，因此从**mysqldump**命令中省略`--databases`使您能够从一个数据库转储数据并加载到另一个数据库中。
