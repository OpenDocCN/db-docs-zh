> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-howto-masterstatus.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-howto-masterstatus.html)

#### 19.1.2.4 获取复制源二进制日志坐标

要配置副本以在正确位置开始复制过程，您需要记录源在其二进制日志中的当前坐标。

警告

此过程使用`FLUSH TABLES WITH READ LOCK`，会阻塞`InnoDB`表的`COMMIT`操作。

如果您计划关闭源以创建数据快照，则可以选择跳过此过程，而是将二进制日志索引文件的副本与数据快照一起存储。在那种情况下，源在重新启动时会创建一个新的二进制日志文件。因此，副本必须开始复制过程的源二进制日志坐标是该新文件的开头，即在复制的二进制日志索引文件中列出的文件之后的源上的下一个二进制日志文件。

要获取源二进制日志坐标，请按照以下步骤操作：

1.  在源上启动一个会话，通过命令行客户端连接到它，并通过执行`FLUSH TABLES WITH READ LOCK`语句刷新所有表并阻止写入语句：

    ```sql
    mysql> FLUSH TABLES WITH READ LOCK;
    ```

    警告

    保持发出`FLUSH TABLES`语句的客户端运行，以使读锁保持生效。如果退出客户端，则锁将被释放。

1.  在源的另一个会话中，使用`SHOW MASTER STATUS`语句确定当前二进制日志文件名和位置：

    ```sql
    mysql> SHOW MASTER STATUS\G
    *************************** 1\. row ***************************
                 File: mysql-bin.000003
             Position: 73
         Binlog_Do_DB: test
     Binlog_Ignore_DB: manual, mysql
    Executed_Gtid_Set: 3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5 1 row in set (0.00 sec)
    ```

    `File`列显示日志文件的名称，`Position`列显示文件内的位置。在此示例中，二进制日志文件为`mysql-bin.000003`，位置为 73。记录这些值。在设置副本时，您将需要它们。它们代表副本应开始处理源的新更新的复制坐标。

    如果源先前已禁用二进制日志记录运行，则`SHOW MASTER STATUS`或**mysqldump --master-data**显示的日志文件名和位置值为空。在这种情况下，您稍后在指定源的二进制日志文件和位置时需要使用的值是空字符串（`''`）和`4`。

您现在拥有启用副本从源的二进制日志中正确位置开始读取所需的信息。

下一步取决于您在源端是否有现有数据。请选择以下选项之一：

+   如果您有现有数据需要在开始复制之前与副本同步，请保持客户端运行，以便锁定保持在原位。这样可以防止进行进一步的更改，从而使复制到副本的数据与源数据同步。继续查看 Section 19.1.2.5, “选择数据快照方法”。

+   如果您正在设置新的源和副本组合，您可以退出第一个会话以释放读锁。查看 Section 19.1.2.6.1, “使用新源和副本设置复制”以了解如何继续。
