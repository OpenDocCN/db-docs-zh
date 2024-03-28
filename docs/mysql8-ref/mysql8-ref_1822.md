> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-security-mysql-privileges.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-security-mysql-privileges.html)

#### 25.6.20.2 NDB Cluster 和 MySQL 特权

在本节中，我们讨论了 MySQL 特权系统在与 NDB Cluster 的关系中的工作原理，以及这对保持 NDB Cluster 安全性的影响。

标准的 MySQL 特权适用于 NDB Cluster 表。这包括在数据库、表和列级别授予的所有 MySQL 特权类型（`SELECT`特权、`UPDATE`特权、`DELETE`特权等），与任何其他 MySQL 服务器一样，用户和特权信息存储在`mysql`系统数据库中。用于在`NDB`表、包含这些表的数据库以及这些表中的列上授予和撤销特权的 SQL 语句在所有方面与用于涉及任何（其他）MySQL 存储引擎的数据库对象的`GRANT`和`REVOKE`语句相同。对于`CREATE USER`和`DROP USER`语句也是如此。

重要的是要记住，默认情况下，MySQL 授权表使用`InnoDB`存储引擎。因此，这些表通常不会在充当 NDB Cluster 中 SQL 节点的 MySQL 服务器之间复制或共享。换句话说，默认情况下，用户及其特权的更改不会在 SQL 节点之间自动传播。如果需要，您可以启用 NDB Cluster SQL 节点之间的 MySQL 用户和特权同步；有关详细信息，请参见第 25.6.13 节“特权同步和 NDB_STORED_USER”。

相反，因为 MySQL 中没有拒绝特权的方法（特权可以被撤销或者一开始就不授予，但不能被明确拒绝），因此没有办法在一个 SQL 节点上对`NDB`表进行特殊保护，使其免受在另一个 SQL 节点上具有特权的用户的影响；即使您没有使用用户特权的自动分发，这也是如此。这一点的明显例子是 MySQL 的`root`账户，它可以对任何数据库对象执行任何操作。结合`config.ini`文件中空的`[mysqld]`或`[api]`部分，这个账户可能特别危险。要理解原因，请考虑以下情景：

+   `config.ini` 文件至少包含一个空的 `[mysqld]` 或 `[api]` 部分。这意味着 NDB 集群管理服务器不会检查 MySQL 服务器（或其他 API 节点）访问 NDB 集群的主机。

+   没有防火墙，或者防火墙无法阻止外部主机访问 NDB 集群。

+   NDB 集群管理服务器的主机名或 IP 地址已知或可以从网络外部确定。

如果这些条件成立，那么任何人在任何地方都可以启动一个带有 `--ndbcluster` `--ndb-connectstring=*`management_host`*` 的 MySQL 服务器，并访问这个 NDB 集群。使用 MySQL 的 `root` 账户，这个人可以执行以下操作：

+   执行元数据语句，比如`SHOW DATABASES`语句（获取服务器上所有`NDB`数据库的列表）或`SHOW TABLES FROM *`some_ndb_database`*`语句（获取给定数据库中所有`NDB`表的列表）

+   在任何发现的表上运行任何合法的 MySQL 语句，比如：

    +   `SELECT * FROM *`some_table`*` 或 `TABLE *`some_table`*` 从任意表中读取所有数据

    +   `DELETE FROM *`some_table`*` 或 TRUNCATE TABLE 删除表中的所有数据

    +   `DESCRIBE *`some_table`*` 或 `SHOW CREATE TABLE *`some_table`*` 确定表结构

    +   `UPDATE *`some_table`* SET *`column1`* = *`some_value`*` 用“垃圾”数据填充表列；这实际上可能造成比简单删除所有数据更大的损害

        更加隐匿的变种可能包括以下语句：

        ```sql
        UPDATE *some_table* SET *an_int_column* = *an_int_column* + 1
        ```

        或者

        ```sql
        UPDATE *some_table* SET *a_varchar_column* = REVERSE(*a_varchar_column*)
        ```

        这样的恶意语句仅受攻击者想象力的限制。

    唯一安全的表将是使用非 `NDB` 存储引擎创建的表，因此对于“流氓” SQL 节点不可见。

    一个能够以 `root` 身份登录的用户也可以访问 `INFORMATION_SCHEMA` 数据库及其表，从而获取关于存储在 `INFORMATION_SCHEMA` 中的数据库、表、存储过程、定时事件以及其他数据库对象的信息。

    对于不使用共享权限的情况，最好为不同的 NDB 集群 SQL 节点上的 `root` 账户使用不同的密码。

总的来说，如果直接从本地网络之外可以访问 NDB 集群，那么你无法拥有一个安全的 NDB 集群。

重要提示

*永远不要将 MySQL root 账户密码留空*。无论是在作为 NDB 集群 SQL 节点运行 MySQL 还是作为独立（非集群）MySQL 服务器运行时，都是正确的，应该在将 MySQL 服务器配置为 NDB 集群中的 SQL 节点之前作为 MySQL 安装过程的一部分来完成。

如果需要在 SQL 节点之间同步`mysql`系统表，可以使用标准的 MySQL 复制来实现，或者使用脚本在 MySQL 服务器之间复制表条目。用户及其权限可以使用`NDB_STORED_USER`特权进行共享和保持同步。

**摘要。** 关于 MySQL 特权系统与 NDB 集群相关的最重要的要点如下：

1.  在一个 SQL 节点上建立的用户和权限不会自动存在或生效于集群中的其他 SQL 节点。反之，在集群中的一个 SQL 节点上删除用户或权限并不会从任何其他 SQL 节点中删除用户或权限。

1.  你可以使用`NDB_STORED_USER`在 SQL 节点之间共享 MySQL 用户和权限。

1.  一旦在 NDB 集群中的一个 SQL 节点上为一个 MySQL 用户授予了对`NDB`表的权限，该用户可以“看到”该表中的任何数据，无论数据来自哪个 SQL 节点，即使该用户没有共享。
