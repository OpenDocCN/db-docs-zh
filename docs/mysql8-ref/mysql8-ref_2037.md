> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-prepared-statements-instances-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-prepared-statements-instances-table.html)

#### 29.12.6.4 prepared_statements_instances 表

Performance Schema 为准备语句提供了仪表化，有两种协议：

+   二进制协议。通过 MySQL C API 访问，映射到下表中显示的底层服务器命令。

    | C API 函数 | 对应的服务器命令 |
    | --- | --- |
    | `mysql_stmt_prepare()` | `COM_STMT_PREPARE` |
    | `mysql_stmt_execute()` | `COM_STMT_EXECUTE` |
    | `mysql_stmt_close()` | `COM_STMT_CLOSE` |

+   文本协议。通过 SQL 语句访问，映射到下表中显示的底层服务器命令。

    | SQL 语句 | 对应的服务器命令 |
    | --- | --- |
    | `PREPARE` | `SQLCOM_PREPARE` |
    | `EXECUTE` | `SQLCOM_EXECUTE` |
    | `DEALLOCATE PREPARE`, `DROP PREPARE` | `SQLCOM_DEALLOCATE PREPARE` |

Performance Schema 准备语句仪表化涵盖了两种协议。以下讨论涉及服务器命令，而不是 C API 函数或 SQL 语句。

有关准备语句的信息可在 `prepared_statements_instances` 表中找到。此表允许检查服务器中使用的准备语句，并提供有关它们的聚合统计信息。要控制此表的大小，请在服务器启动时设置 `performance_schema_max_prepared_statements_instances` 系统变量。

收集准备语句信息取决于下表中显示的语句工具。这些工具默认启用。要修改它们，请更新 `setup_instruments` 表。

| 工具 | 服务器命令 |
| --- | --- |
| `statement/com/Prepare` | `COM_STMT_PREPARE` |
| `statement/com/Execute` | `COM_STMT_EXECUTE` |
| `statement/sql/prepare_sql` | `SQLCOM_PREPARE` |
| `statement/sql/execute_sql` | `SQLCOM_EXECUTE` |

Performance Schema 管理 `prepared_statements_instances` 表的内容如下：

+   语句准备

    `COM_STMT_PREPARE`或`SQLCOM_PREPARE`命令在服务器中创建一个预处理语句。如果成功地对语句进行了仪表化，将向`prepared_statements_instances`表中添加一行新记录。如果无法对语句进行仪表化，则会增加`Performance_schema_prepared_statements_lost`状态变量。

+   预处理语句执行

    对于仪表化预处理语句实例的`COM_STMT_EXECUTE`或`SQLCOM_PREPARE`命令的执行将更新相应的`prepared_statements_instances`表行。

+   预处理语句释放

    对于仪表化预处理语句实例的`COM_STMT_CLOSE`或`SQLCOM_DEALLOCATE_PREPARE`命令的执行将删除相应的`prepared_statements_instances`表行。为避免资源泄漏，即使禁用了先前描述的预处理语句仪表化，也会发生删除。

`prepared_statements_instances`表具有这些列：

+   `OBJECT_INSTANCE_BEGIN`

    仪表化预处理语句的内存地址。

+   `STATEMENT_ID`

    服务器分配的内部语句 ID。文本和二进制协议都使用语句 ID。

+   `STATEMENT_NAME`

    对于二进制协议，此列为`NULL`。对于文本协议，此列为用户分配的外部语句名称。例如，对于以下 SQL 语句，预处理语句的名称为`stmt`：

    ```sql
    PREPARE stmt FROM 'SELECT 1';
    ```

+   `SQL_TEXT`

    带有`?`占位符标记的预处理语句文本。

+   `OWNER_THREAD_ID`，`OWNER_EVENT_ID`

    这些列指示创建预处理语句的事件。

+   `OWNER_OBJECT_TYPE`，`OWNER_OBJECT_SCHEMA`，`OWNER_OBJECT_NAME`

    对于由客户端会话创建的预处理语句，这些列为`NULL`。对于由存储过程创建的预处理语句，这些列指向存储过程。一个典型的用户错误是忘记释放预处理语句。这些列可用于查找泄漏预处理语句的存储过程：

    ```sql
    SELECT
      OWNER_OBJECT_TYPE, OWNER_OBJECT_SCHEMA, OWNER_OBJECT_NAME,
      STATEMENT_NAME, SQL_TEXT
    FROM performance_schema.prepared_statements_instances
    WHERE OWNER_OBJECT_TYPE IS NOT NULL;
    ```

+   查询执行引擎。其值为`PRIMARY`或`SECONDARY`。用于 MySQL HeatWave 服务和 HeatWave，其中`PRIMARY`引擎为`InnoDB`，`SECONDARY`引擎为 HeatWave（`RAPID`）。对于 MySQL 社区版服务器、MySQL 企业版服务器（本地）和没有 HeatWave 的 MySQL HeatWave 服务，该值始终为`PRIMARY`。此列在 MySQL 8.0.29 中添加。

+   `TIMER_PREPARE`

    执行语句准备本身所花费的时间。

+   `COUNT_REPREPARE`

    语句在内部重新准备的次数（参见第 10.10.3 节，“准备语句和存储程序的缓存”）。重新准备的时间统计数据不可用，因为它被计算为语句执行的一部分，而不是作为单独的操作。

+   `COUNT_EXECUTE`, `SUM_TIMER_EXECUTE`, `MIN_TIMER_EXECUTE`, `AVG_TIMER_EXECUTE`, `MAX_TIMER_EXECUTE`

    准备语句执行的聚合统计数据。

+   `SUM_*`xxx`*`

    剩余的`SUM_*`xxx`*`列与语句摘要表相同（参见第 29.12.20.3 节，“语句摘要表”）。

+   `MAX_CONTROLLED_MEMORY`

    报告了执行期间准备语句使用的受控内存的最大量。

    此列在 MySQL 8.0.31 中添加。

+   `MAX_TOTAL_MEMORY`

    报告了执行期间准备语句使用的最大内存量。

    此列在 MySQL 8.0.31 中添加。

`prepared_statements_instances`表具有以下索引：

+   在(`OBJECT_INSTANCE_BEGIN`)上的主键

+   在(`STATEMENT_ID`)上的索引

+   在(`STATEMENT_NAME`)上的索引

+   在(`OWNER_THREAD_ID`, `OWNER_EVENT_ID`)上的索引

+   在(`OWNER_OBJECT_TYPE`, `OWNER_OBJECT_SCHEMA`, `OWNER_OBJECT_NAME`)上的索引

`TRUNCATE TABLE` 重置`prepared_statements_instances`表的统计列。
