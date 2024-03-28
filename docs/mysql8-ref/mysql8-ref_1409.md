> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-solutions-backups-mysqldump.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-backups-mysqldump.html)

#### 19.4.1.1 使用 **mysqldump** 备份副本

使用**mysqldump**创建数据库副本可以捕获数据库中的所有数据，并以一种格式导入到另一个 MySQL Server 实例中（参见第 6.5.4 节，“mysqldump — A Database Backup Program”）。由于信息的格式是 SQL 语句，因此文件可以轻松分发并应用于正在运行的服务器，以便在紧急情况下访问数据。但是，如果您的数据集很大，使用**mysqldump**可能不切实际。

提示

考虑使用 MySQL Shell 转储工具，提供多线程并行转储、文件压缩和进度信息显示，以及云功能，如 Oracle Cloud Infrastructure Object Storage 流式传输和 MySQL HeatWave Service 兼容性检查和修改。转储可以使用 MySQL Shell 加载转储工具轻松导入到 MySQL Server 实例或 MySQL HeatWave Service DB System 中。MySQL Shell 的安装说明可在此处找到。

使用**mysqldump**在开始转储过程之前应停止副本上的复制，以确保转储包含一致的数据集：

1.  停止副本处理请求。您可以使用**mysqladmin**完全停止副本上的复制：

    ```sql
    $> mysqladmin stop-slave
    ```

    或者，您可以仅停止复制 SQL 线程以暂停事件执行：

    ```sql
    $> mysql -e 'STOP SLAVE SQL_THREAD;'
    Or from MySQL 8.0.22:
    $> mysql -e 'STOP REPLICA SQL_THREAD;'
    ```

    这使得副本可以继续接收来自源二进制日志的数据更改事件，并使用复制接收器线程将其存储在中继日志中，但阻止副本执行这些事件并更改其数据。在繁忙的复制环境中，允许复制接收器线程在备份期间运行可能会加快重新启动复制应用程序线程时的赶上过程。

1.  运行 **mysqldump** 来转储您的数据库。您可以转储所有数据库或选择要转储的数据库。例如，要转储所有数据库：

    ```sql
    $> mysqldump --all-databases > fulldb.dump
    ```

1.  转储完成后，重新启动复制：

    ```sql
    $> mysqladmin start-slave
    ```

在上面的示例中，您可能希望将登录凭据（用户名、密码）添加到命令中，并将整个过程打包成一个脚本，以便每天自动运行。

如果您采用这种方法，请确保监控复制过程，以确保运行备份所需的时间不会影响副本跟上源事件的能力。请参见第 19.1.7.1 节，“检查复制状态”。如果副本无法跟上，您可能需要添加另一个副本并分发备份过程。有关如何配置此场景的示例，请参见第 19.4.6 节，“将不同数据库复制到不同副本”。
