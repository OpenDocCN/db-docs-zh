# 18.8.3 FEDERATED 存储引擎注意事项和提示

> 原文：[`dev.mysql.com/doc/refman/8.0/en/federated-usagenotes.html`](https://dev.mysql.com/doc/refman/8.0/en/federated-usagenotes.html)

在使用`FEDERATED`存储引擎时，应注意以下几点：

+   `FEDERATED`表可以被复制到其他副本，但必须确保副本服务器能够使用在`CONNECTION`字符串（或`mysql.servers`表中的行）中定义的用户/密码组合连接到远程服务器。

以下项目指示了`FEDERATED`存储引擎支持和不支持的功能：

+   远程服务器必须是一个 MySQL 服务器。

+   `FEDERATED`表指向的远程表在通过`FEDERATED`表访问之前*必须*存在。

+   一个`FEDERATED`表可以指向另一个表，但必须小心不要创建循环。

+   `FEDERATED`表不支持通常意义上的索引；因为对表数据的访问是远程处理的，实际上是远程表利用索引。这意味着，对于无法使用任何索引并因此需要完整表扫描的查询，服务器会从远程表中获取所有行并在本地进行过滤。这将发生无论此`SELECT`语句中使用的任何`WHERE`或`LIMIT`；这些子句在本地应用于返回的行。

    未能使用索引的查询可能导致性能不佳和网络负载过重。此外，由于返回的行必须存储在内存中，这样的查询也可能导致本地服务器交换，甚至挂起。

+   在创建`FEDERATED`表时应当小心，因为来自等效`MyISAM`或其他表的索引定义可能不被支持。例如，如果表在任何`VARCHAR`、`TEXT`或`BLOB`列上使用索引前缀，则创建`FEDERATED`表将失败。以下使用`MyISAM`的定义是有效的：

    ```sql
    CREATE TABLE `T1`(`A` VARCHAR(100),UNIQUE KEY(`A`(30))) ENGINE=MYISAM;
    ```

    此示例中的键前缀与`FEDERATED`引擎不兼容，等效语句将失败：

    ```sql
    CREATE TABLE `T1`(`A` VARCHAR(100),UNIQUE KEY(`A`(30))) ENGINE=FEDERATED
      CONNECTION='MYSQL://127.0.0.1:3306/TEST/T1';
    ```

    如果可能的话，在远程服务器和本地服务器创建表时，应尽量分开列和索引定义，以避免这些索引问题。

+   在内部，实现使用`SELECT`、`INSERT`、`UPDATE`和`DELETE`，但不使用`HANDLER`。

+   `FEDERATED`存储引擎支持`SELECT`、`INSERT`、`UPDATE`、`DELETE`、`TRUNCATE TABLE`和索引。它不支持`ALTER TABLE`或任何直接影响表结构的数据定义语言语句，除了`DROP TABLE`。当前的实现不使用预处理语句。

+   `FEDERATED`接受`INSERT ... ON DUPLICATE KEY UPDATE`语句，但如果发生重复键违规，该语句将失败并显示错误。

+   不支持事务。

+   `FEDERATED`执行批量插入处理，使多行以批量方式发送到远程表，从而提高性能。此外，如果远程表是事务性的，它可以使远程存储引擎在发生错误时正确执行语句回滚。此功能具有以下限制：

    +   插入的大小不能超过服务器之间的最大数据包大小。如果插入超过此大小，它将被分成多个数据包，可能会出现回滚问题。

    +   不会对`INSERT ... ON DUPLICATE KEY UPDATE`进行批量插入处理。

+   `FEDERATED`引擎无法知道远程表是否发生了更改。原因是该表必须像数据文件一样工作，除了数据库系统之外不会被任何其他东西写入。如果远程数据库发生任何更改，本地表中的数据完整性可能会受到破坏。

+   在使用`CONNECTION`字符串时，密码中不能使用'@'字符。您可以通过使用`CREATE SERVER`语句创建服务器连接来绕过此限制。

+   `insert_id`和`timestamp`选项不会传播到数据提供程序。

+   对`FEDERATED`表执行的任何`DROP TABLE`语句仅删除本地表，而不是远程表。

+   不支持对`FEDERATED`表进行用户定义的分区。
