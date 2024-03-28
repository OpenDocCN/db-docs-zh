> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-engines.html`](https://dev.mysql.com/doc/refman/8.0/en/show-engines.html)

#### 15.7.7.16 显示引擎语句

```sql
SHOW [STORAGE] ENGINES
```

`显示引擎` 显示关于服务器存储引擎的状态信息。这对于检查存储引擎是否受支持或查看默认引擎特别有用。

有关 MySQL 存储引擎的信息，请参阅 第十七章，“InnoDB 存储引擎” 和 第十八章，“替代存储引擎”。

```sql
mysql> SHOW ENGINES\G
*************************** 1\. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 2\. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3\. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 4\. row ***************************
      Engine: FEDERATED
     Support: NO
     Comment: Federated MySQL storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
*************************** 5\. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6\. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7\. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 8\. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 9\. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
```

`显示引擎` 的输出可能根据使用的 MySQL 版本和其他因素而有所不同。

`显示引擎` 输出包含以下列：

+   `引擎`

    存储引擎的名称。

+   `支持`

    服务器对存储引擎的支持级别，如下表所示。

    | 值 | 含义 |
    | --- | --- |
    | `YES` | 引擎受支持且处于活动状态 |
    | `DEFAULT` | 类似于 `YES`，并且这是默认引擎 |
    | `NO` | 引擎不受支持 |
    | `DISABLED` | 引擎受支持但已被禁用 |

    `NO` 的值表示服务器在编译时没有对该引擎的支持，因此无法在运行时启用。

    `DISABLED` 的值可能是因为服务器启动时使用了禁用引擎的选项，或者因为未提供启用引擎所需的所有选项。在后一种情况下，错误日志应包含指示为何选项被禁用的原因。参见 第 7.4.2 节，“错误日志”。

    如果服务器在编译时支持某个存储引擎，但启动时使用了 `--skip-*`engine_name`*` 选项，则可能会看到存储引擎的 `DISABLED`。对于 `NDB` 存储引擎，`DISABLED` 表示服务器已编译支持 NDB Cluster，但未使用 `--ndbcluster` 选项启动。

    所有 MySQL 服务器都支持 `MyISAM` 表。无法禁用 `MyISAM`。

+   `注释`

    存储引擎的简要描述。

+   `事务`

    存储引擎是否支持事务。

+   `XA`

    存储引擎是否支持 XA 事务。

+   `保存点`

    存储引擎是否支持保存点。

存储引擎信息也可以从 `INFORMATION_SCHEMA` `ENGINES` 表中获取。参见 第 28.3.13 节，“INFORMATION_SCHEMA ENGINES 表”。
