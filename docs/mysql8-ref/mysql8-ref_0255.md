# 7.1.18 服务器跟踪客户端会话状态

> 译文：[`dev.mysql.com/doc/refman/8.0/en/session-state-tracking.html`](https://dev.mysql.com/doc/refman/8.0/en/session-state-tracking.html)

MySQL 服务器实现了几个会话状态跟踪器。客户端可以启用这些跟踪器以接收有关其会话状态更改的通知。

+   会话状态跟踪器的用途

+   可用的会话状态跟踪器

+   C API 会话状态跟踪器支持

+   测试套件会话状态跟踪支持

#### 会话状态跟踪器的用途

会话状态跟踪器有以下用途：

+   为了促进会话迁移。

+   为了促进事务切换。

跟踪机制提供了一个方法，使得 MySQL 连接器和客户端应用程序能够确定是否有任何会话上下文可用，以允许会话从一个服务器迁移到另一个服务器。（在负载平衡环境中更改会话时，需要检测是否有会话状态需要考虑，以便在决定是否可以进行切换时考虑。）

跟踪机制允许应用程序知道何时可以将事务从一个会话移动到另一个会话。事务状态跟踪使这成为可能，对于可能希望将事务从繁忙服务器移动到负载较轻的服务器的应用程序很有用。例如，管理客户端连接池的负载平衡连接器可以在池中的可用会话之间移动事务。

然而，会话切换不能在任意时间进行。如果一个会话正在进行读取或写入的事务，切换到另一个会话意味着在原始会话上进行事务回滚。只有在事务尚未在其中执行任何读取或写入时才能进行会话切换。

何时可以合理地切换事务的示例：

+   立即在`START TRANSACTION`之后

+   在`COMMIT AND CHAIN`之后

除了了解事务状态外，了解事务特性也很有用，以便在将事务移动到不同会话时使用相同的特性。以下特性对此很重要：

```sql
READ ONLY
READ WRITE
ISOLATION LEVEL
WITH CONSISTENT SNAPSHOT
```

#### 可用的会话状态跟踪器

为了支持会话跟踪活动，可用于这些类型的客户端会话状态信息的通知：

+   客户端会话状态的这些属性的更改：

    +   默认模式（数据库）。

    +   系统变量的会话特定值。

    +   用户定义变量。

    +   临时表。

    +   预处理语句。

    `session_track_state_change` 系统变量控制此跟踪。

+   默认模式名称更改。`session_track_schema` 系统变量控制此跟踪。

+   系统变量的会话值更改。`session_track_system_variables` 系统变量控制此跟踪。需要 `SENSITIVE_VARIABLES_OBSERVER` 权限来跟踪对敏感系统变量值的更改。

+   可用的 GTID。`session_track_gtids` 系统变量控制此跟踪。

+   有关事务状态和特性的信息。`session_track_transaction_info` 系统变量控制此跟踪。

有关与跟踪相关的系统变量的描述，请参阅 第 7.1.8 节，“服务器系统变量”。这些系统变量允许控制哪些更改通知发生，但不提供访问通知信息的方法。通知发生在 MySQL 客户端/服务器协议中，该协议在 OK 数据包中包含跟踪信息，以便检测会话状态更改。

#### C API 会话状态跟踪支持

为了使客户端应用程序能够从服务器返回的 OK 数据包中提取状态更改信息，MySQL C API 提供了一对函数：

+   `mysql_session_track_get_first()` 获取从服务器接收的状态更改信息的第一部分。请参阅 mysql_session_track_get_first()。

+   `mysql_session_track_get_next()` 从服务器接收的任何剩余状态更改信息。在成功调用 `mysql_session_track_get_first()` 后，只要它返回成功，就重复调用此函数。请参阅 mysql_session_track_get_next()。

#### 测试套件会话状态跟踪支持

**mysqltest** 程序具有 `disable_session_track_info` 和 `enable_session_track_info` 命令，用于控制会话跟踪通知的发生。您可以使用这些命令从命令行查看 SQL 语句产生的通知。假设一个文件 `testscript` 包含以下 **mysqltest** 脚本：

```sql
DROP TABLE IF EXISTS test.t1;
CREATE TABLE test.t1 (i INT, f FLOAT);
--enable_session_track_info
SET @@SESSION.session_track_schema=ON;
SET @@SESSION.session_track_system_variables='*';
SET @@SESSION.session_track_state_change=ON;
USE information_schema;
SET NAMES 'utf8mb4';
SET @@SESSION.session_track_transaction_info='CHARACTERISTICS';
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET TRANSACTION READ WRITE;
START TRANSACTION;
SELECT 1;
INSERT INTO test.t1 () VALUES();
INSERT INTO test.t1 () VALUES(1, RAND());
COMMIT;
```

运行以下脚本以查看已启用跟踪器提供的信息。有关**mysqltest**为各种跟踪器显示的`Tracker:`信息的描述，请参阅 mysql_session_track_get_first()。

```sql
$> mysqltest < testscript
DROP TABLE IF EXISTS test.t1;
CREATE TABLE test.t1 (i INT, f FLOAT);
SET @@SESSION.session_track_schema=ON;
SET @@SESSION.session_track_system_variables='*';
-- Tracker : SESSION_TRACK_SYSTEM_VARIABLES
-- session_track_system_variables
-- *

SET @@SESSION.session_track_state_change=ON;
-- Tracker : SESSION_TRACK_SYSTEM_VARIABLES
-- session_track_state_change
-- ON

USE information_schema;
-- Tracker : SESSION_TRACK_SCHEMA
-- information_schema

-- Tracker : SESSION_TRACK_STATE_CHANGE
-- 1

SET NAMES 'utf8mb4';
-- Tracker : SESSION_TRACK_SYSTEM_VARIABLES
-- character_set_client
-- utf8mb4
-- character_set_connection
-- utf8mb4
-- character_set_results
-- utf8mb4

-- Tracker : SESSION_TRACK_STATE_CHANGE
-- 1

SET @@SESSION.session_track_transaction_info='CHARACTERISTICS';
-- Tracker : SESSION_TRACK_SYSTEM_VARIABLES
-- session_track_transaction_info
-- CHARACTERISTICS

-- Tracker : SESSION_TRACK_STATE_CHANGE
-- 1

-- Tracker : SESSION_TRACK_TRANSACTION_CHARACTERISTICS
--

-- Tracker : SESSION_TRACK_TRANSACTION_STATE
-- ________

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Tracker : SESSION_TRACK_TRANSACTION_CHARACTERISTICS
-- SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SET TRANSACTION READ WRITE;
-- Tracker : SESSION_TRACK_TRANSACTION_CHARACTERISTICS
-- SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; SET TRANSACTION READ WRITE;

START TRANSACTION;
-- Tracker : SESSION_TRACK_TRANSACTION_CHARACTERISTICS
-- SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; START TRANSACTION READ WRITE;

-- Tracker : SESSION_TRACK_TRANSACTION_STATE
-- T_______

SELECT 1;
1
1
-- Tracker : SESSION_TRACK_TRANSACTION_STATE
-- T_____S_

INSERT INTO test.t1 () VALUES();
-- Tracker : SESSION_TRACK_TRANSACTION_STATE
-- T___W_S_

INSERT INTO test.t1 () VALUES(1, RAND());
-- Tracker : SESSION_TRACK_TRANSACTION_STATE
-- T___WsS_

COMMIT;
-- Tracker : SESSION_TRACK_TRANSACTION_CHARACTERISTICS
--

-- Tracker : SESSION_TRACK_TRANSACTION_STATE
-- ________

ok
```

在`START TRANSACTION`语句之前，执行两个`SET TRANSACTION`语句，设置下一个事务的隔离级别和访问模式特性。`SESSION_TRACK_TRANSACTION_CHARACTERISTICS`值指示已设置的下一个事务值。

在结束事务的`COMMIT`语句之后，`SESSION_TRACK_TRANSACTION_CHARACTERISTICS`值报告为空。这表明在事务开始之前设置的下一个事务特性已被重置，并且会应用会话默认值。要跟踪这些会话默认值的更改，请跟踪`transaction_isolation`和`transaction_read_only`系统变量的会话值。

要查看有关 GTID 的信息，请使用`session_track_gtids`系统变量启用`SESSION_TRACK_GTIDS`跟踪器。
