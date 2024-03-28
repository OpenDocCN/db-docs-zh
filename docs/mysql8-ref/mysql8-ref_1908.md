# 28.3.13 `INFORMATION_SCHEMA ENGINES` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-engines-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-engines-table.html)

[`ENGINES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-engines-table.html) 表提供有关存储引擎的信息。这对于检查存储引擎是否受支持或查看默认引擎非常有用。

[`ENGINES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-engines-table.html) 表具有以下列：

+   `ENGINE`

    存储引擎的名称。

+   `SUPPORT`

    服务器对存储引擎的支持级别，如下表所示。

    | 值 | 含义 |
    | --- | --- |
    | `YES` | 该引擎受支持且处于活动状态 |
    | `DEFAULT` | 类似于 `YES`，并且这是默认引擎 |
    | `NO` | 该引擎不受支持 |
    | `DISABLED` | 该引擎受支持但已被禁用 |

    一个值为`NO`表示服务器在编译时没有对该引擎提供支持，因此无法在运行时启用。

    值为 `DISABLED` 可能是因为服务器启动时使用了禁用引擎的选项，或者因为没有提供启用它所需的所有选项。在后一种情况下，错误日志应包含指示为什么选项被禁用的原因。请参阅 Section 7.4.2, “The Error Log”。

    如果服务器编译时支持某个存储引擎，但是启动时使用了 `--skip-*`engine_name`*` 选项，则可能会看到存储引擎的 `DISABLED`。对于 `NDB` 存储引擎，`DISABLED` 表示服务器编译时支持 NDB Cluster，但未使用 `--ndbcluster` 选项启动。

    所有 MySQL 服务器都支持 `MyISAM` 表。无法禁用 `MyISAM`。

+   `COMMENT`

    存储引擎的简要描述。

+   `TRANSACTIONS`

    存储引擎是否支持事务。

+   `XA`

    存储引擎是否支持 XA 事务。

+   `SAVEPOINTS`

    存储引擎是否支持保存点。

#### 注意

+   [`ENGINES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-engines-table.html) 是一个非标准的 `INFORMATION_SCHEMA` 表。

存储引擎信息也可以通过 `SHOW ENGINES` 语句获取。请参阅 Section 15.7.7.16, “SHOW ENGINES Statement”。以下语句是等效的：

```sql
SELECT * FROM INFORMATION_SCHEMA.ENGINES

SHOW ENGINES
```
