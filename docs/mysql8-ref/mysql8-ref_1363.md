> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-stop-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-stop-replica.html)

#### 19.1.5.6 停止多源复制

`STOP REPLICA`语句可用于停止多源复制。默认情况下，如果在多源复制上使用`STOP REPLICA`语句，则所有通道都会停止。可选择使用`FOR CHANNEL *`channel`*`子句仅停止特定通道。

+   停止所有当前配置的复制通道：

    ```sql
    mysql> STOP SLAVE;
    Or from MySQL 8.0.22:
    mysql> STOP REPLICA;
    ```

+   若要仅停止命名通道，请使用`FOR CHANNEL *`channel`*`子句：

    ```sql
    mysql> STOP SLAVE FOR CHANNEL "source_1";
    Or from MySQL 8.0.22:
    mysql> STOP REPLICA FOR CHANNEL "source_1";
    ```

有关`STOP REPLICA`命令的完整语法和其他可用选项，请参见 Section 15.4.2.8, “STOP REPLICA Statement”。
