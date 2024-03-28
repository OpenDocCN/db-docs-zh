# 3.15 将 MySQL 数据库复制到另一台机器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/copying-databases.html`](https://dev.mysql.com/doc/refman/8.0/en/copying-databases.html)

在需要在不同架构之间传输数据库的情况下，您可以使用**mysqldump**创建包含 SQL 语句的文件。然后，您可以将文件传输到另一台机器，并将其作为输入提供给**mysql**客户端。

使用**mysqldump --help**查看可用的选项。

注意

如果在创建转储的服务器上使用了 GTIDs（`gtid_mode=ON`），默认情况下，**mysqldump**会在转储中包含`gtid_executed`集的内容，以将其传输到新机器。这样做的结果可能会因涉及的 MySQL 服务器版本而有所不同。查看**mysqldump**的`--set-gtid-purged`选项的描述，以了解您正在使用的版本的情况，以及如何更改行为，如果默认行为的结果不适合您的情况。

将数据库在两台机器之间移动的最简单（尽管不是最快）方法是在存储数据库的机器上运行以下命令：

```sql
mysqladmin -h '*other_hostname*' create *db_name*
mysqldump *db_name* | mysql -h '*other_hostname*' *db_name*
```

如果您想要通过缓慢的网络从远程机器复制数据库，可以使用以下命令：

```sql
mysqladmin create *db_name*
mysqldump -h '*other_hostname*' --compress *db_name* | mysql *db_name*
```

您还可以将转储存储在文件中，将文件传输到目标机器，然后在那里将文件加载到数据库中。例如，您可以在源机器上将数据库转储到压缩文件中，如下所示：

```sql
mysqldump --quick *db_name* | gzip > *db_name*.gz
```

将包含数据库内容的文件传输到目标机器，并在那里运行这些命令：

```sql
mysqladmin create *db_name*
gunzip < *db_name*.gz | mysql *db_name*
```

您还可以使用**mysqldump**和**mysqlimport**来传输数据库。对于大表，这比简单使用**mysqldump**要快得多。在以下命令中，*`DUMPDIR`*代表您用于存储**mysqldump**输出的完整路径名。

首先，创建用于输出文件的目录并转储数据库：

```sql
mkdir *DUMPDIR*
mysqldump --tab=*DUMPDIR*
   *db_name*
```

然后将*`DUMPDIR`*目录中的文件传输到目标机器上的相应目录，并在那里将文件加载到 MySQL 中：

```sql
mysqladmin create *db_name*           # create database
cat *DUMPDIR*/*.sql | mysql *db_name*   # create tables in database
mysqlimport *db_name*
   *DUMPDIR*/*.txt   # load data into tables
```

不要忘记复制`mysql`数据库，因为那里存储着授权表。在新机器上，您可能需要以 MySQL `root`用户身份运行命令，直到`mysql`数据库就位。

在新机器上导入`mysql`数据库后，执行**mysqladmin flush-privileges**，以便服务器重新加载授权表信息。
