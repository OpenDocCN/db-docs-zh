# 17.13 InnoDB 数据静态加密

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-data-encryption.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-data-encryption.html)

`InnoDB`支持对 file-per-table 表空间、通用表空间、`mysql`系统表空间、重做日志和撤销日志进行数据静态加密。

截至 MySQL 8.0.16 版本，还支持为模式和通用表空间设置加密默认值，这使得 DBA 可以控制在这些模式和表空间中创建的表是否加密。

`InnoDB`数据静态加密的特性和功能在本节的以下主题中描述。

+   关于数据静态加密

+   加密先决条件

+   为模式和通用表空间定义加密默认值

+   文件-每表表空间加密

+   通用表空间加密

+   双写文件加密

+   mysql 系统表空间加密

+   重做日志加密

+   撤销日志加密

+   主密钥轮换

+   加密和恢复

+   导出加密表空间

+   加密和复制

+   识别加密表空间和模式

+   监控加密进度

+   加密使用注意事项

+   加密限制

### 关于数据静态加密

`InnoDB` 使用两层加密密钥架构，包括主加密密钥和表空间密钥。当一个表空间被加密时，表空间密钥会被加密并存储在表空间头部。当应用程序或经过身份验证的用户想要访问加密的表空间数据时，`InnoDB` 使用主加密密钥来解密表空间密钥。解密后的表空间密钥版本永远不会改变，但主加密密钥可以根据需要更改。这个操作被称为*主密钥轮换*。

数据静态加密功能依赖于密钥环组件或插件进行主加密密钥管理。

所有 MySQL 版本都提供 `component_keyring_file` 组件和 `keyring_file` 插件，每个都将密钥环数据存储在服务器主机本地的文件中。

MySQL 企业版提供额外的密钥环组件和插件：

+   `component_keyring_encrypted_file`：将密钥环数据存储在服务器主机本地的加密、受密码保护的文件中。

+   `keyring_encrypted_file`：将密钥环数据存储在服务器主机本地的加密、受密码保护的文件中。

+   `keyring_okv`：用于与支持 KMIP 的后端密钥环存储产品一起使用的 KMIP 1.1 插件。支持的 KMIP 兼容产品包括集中式密钥管理解决方案，如 Oracle Key Vault、Gemalto KeySecure、Thales Vormetric 密钥管理服务器和 Fornetix Key Orchestration。

+   `keyring_aws`：与亚马逊网络服务密钥管理服务（AWS KMS）通信，作为密钥生成的后端，并使用本地文件进行密钥存储。

+   `keyring_hashicorp`：与 HashiCorp Vault 通信，作为后端存储。

警告

对于加密密钥管理，`component_keyring_file` 和 `component_keyring_encrypted_file` 组件，以及 `keyring_file` 和 `keyring_encrypted_file` 插件并不作为符合监管合规性的解决方案。诸如 PCI、FIPS 等安全标准要求使用密钥管理系统来在密钥保险库或硬件安全模块（HSM）中安全、管理和保护加密密钥。

安全而强大的加密密钥管理解决方案对于安全性和符合各种安全标准至关重要。当数据静态加密功能使用集中式密钥管理解决方案时，该功能被称为“MySQL 企业透明数据加密（TDE）”。

数据静态加密功能支持高级加密标准（AES）块加密算法。它使用电子密码本（ECB）块加密模式进行表空间密钥加密，使用密码块链接（CBC）块加密模式进行数据加密。

有关数据静态加密功能的常见问题，请参见 Section A.17，“MySQL 8.0 FAQ：InnoDB 数据静态加密”。

### 加密先决条件

+   密钥环组件或插件必须在启动时安装和配置。早期加载确保在初始化`InnoDB`存储引擎之前可用该组件或插件。有关密钥环安装和配置说明，请参见 Section 8.4.4，“MySQL 密钥环”。说明显示如何确保所选组件或插件处于活动状态。

    一次只能启用一个密钥环组件或插件。启用多个密钥环组件或插件是不受支持的，结果可能不如预期。

    重要

    一旦在 MySQL 实例中创建了加密表空间，创建加密表空间时加载的密钥环组件或插件必须继续在启动时加载。如果未能这样做，将在启动服务器和`InnoDB`恢复期间出现错误。

+   在加密生产数据时，请确保采取措施防止主加密密钥丢失。*如果主加密密钥丢失，则存储在加密表空间文件中的数据将无法恢复。* 如果您使用`component_keyring_file`或`component_keyring_encrypted_file`组件，或者`keyring_file`或`keyring_encrypted_file`插件，在创建第一个加密表空间后立即创建密钥环数据文件的备份，在主密钥轮换之前和之后。对于每个组件，其配置文件指示数据文件位置。`keyring_file_data`配置选项定义了`keyring_file`插件的密钥环数据文件位置。`keyring_encrypted_file_data`配置选项定义了`keyring_encrypted_file`插件的密钥环数据文件位置。如果您使用`keyring_okv`或`keyring_aws`插件，请确保已执行必要的配置。有关说明，请参见 Section 8.4.4，“MySQL 密钥环”。

### 为模式和常规表空间定义加密默认值

从 MySQL 8.0.16 开始，`default_table_encryption`系统变量定义了模式和常规表空间的默认加密设置。在未明确指定`ENCRYPTION`子句时，`CREATE TABLESPACE`和`CREATE SCHEMA`操作将应用`default_table_encryption`设置。

`ALTER SCHEMA`和`ALTER TABLESPACE`操作不适用于`default_table_encryption`设置。必须明确指定`ENCRYPTION`子句才能更改现有模式或通用表空间的加密。

可以使用`SET`语法为单个客户端连接或全局设置`default_table_encryption`变量。例如，以下语句在全局范围内启用默认模式和表空间加密：

```sql
mysql> SET GLOBAL default_table_encryption=ON;
```

还可以在创建或更改模式时使用`DEFAULT ENCRYPTION`子句来定义模式的默认加密设置，如下例所示：

```sql
mysql> CREATE SCHEMA test DEFAULT ENCRYPTION = 'Y';
```

如果在创建模式时未指定`DEFAULT ENCRYPTION`子句，则将应用`default_table_encryption`设置。必须指定`DEFAULT ENCRYPTION`子句才能更改现有模式的默认加密。否则，模式将保留其当前的加密设置。

默认情况下，表继承所在模式或通用表空间的加密设置。例如，在启用加密的模式中创建的表默认是加密的。此行为使得数据库管理员可以通过定义和强制模式和通用表空间加密默认值来控制表加密的使用。

加密默认值是通过启用`table_encryption_privilege_check`系统变量来强制执行的。当启用`table_encryption_privilege_check`时，在创建或更改具有与`default_table_encryption`设置不同的加密设置的模式或通用表空间，或者在创建或更改具有与默认模式加密不同的加密设置的表时，将进行权限检查。当禁用`table_encryption_privilege_check`（默认情况下）时，权限检查不会发生，前述操作将被允许继续并显示警告。

当启用`table_encryption_privilege_check`时，需要`TABLE_ENCRYPTION_ADMIN`权限来覆盖默认加密设置。数据库管理员可以授予此权限，以使用户能够在创建或更改模式或通用表空间时偏离`default_table_encryption`设置，或在创建或更改表时偏离默认模式加密。此权限不允许在创建或更改表时偏离通用表空间的加密。表必须与其所在的通用表空间具有相同的加密设置。

### 每表表空间加密

从 MySQL 8.0.16 开始，每表表空间将继承创建表所在模式的默认加密，除非在`CREATE TABLE`语句中明确指定了一个`ENCRYPTION`子句。在 MySQL 8.0.16 之前，必须指定`ENCRYPTION`子句才能启用加密。

```sql
mysql> CREATE TABLE t1 (c1 INT) ENCRYPTION = 'Y';
```

要更改现有每表表空间的加密方式，必须指定一个`ENCRYPTION`子句。

```sql
mysql> ALTER TABLE t1 ENCRYPTION = 'Y';
```

从 MySQL 8.0.16 开始，如果启用了`table_encryption_privilege_check`变量，则指定一个与默认模式加密不同的设置的`ENCRYPTION`子句需要`TABLE_ENCRYPTION_ADMIN`权限。请参阅为模式和通用表空间定义加密默认值。

### 通用表空间加密

从 MySQL 8.0.16 开始，`default_table_encryption`变量确定新创建的通用表空间的加密，除非在`CREATE TABLESPACE`语句中明确指定了一个`ENCRYPTION`子句。在 MySQL 8.0.16 之前，必须指定`ENCRYPTION`子句才能启用加密。

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' ENCRYPTION = 'Y' Engine=InnoDB;
```

要更改现有通用表空间的加密方式，必须指定一个`ENCRYPTION`子句。

```sql
mysql> ALTER TABLESPACE ts1 ENCRYPTION = 'Y';
```

截至 MySQL 8.0.16，如果启用了`table_encryption_privilege_check`变量，则指定与`default_table_encryption`设置不同的设置的`ENCRYPTION`子句需要`TABLE_ENCRYPTION_ADMIN`权限。参见为模式和通用表空间定义加密默认值。

### 双写文件加密

从 MySQL 8.0.23 开始，双写文件的加密支持可用。`InnoDB`会自动加密属于加密表空间的双写文件页。不需要任何操作。双写文件页使用相关表空间的加密密钥进行加密。写入表空间数据文件的相同加密页也会写入双写文件。属于未加密表空间的双写文件页保持未加密状态。

在恢复过程中，加密的双写文件页会被解密并检查是否损坏。

### mysql 系统表空间加密

从 MySQL 8.0.16 开始，`mysql`系统表空间的加密支持可用。

`mysql`系统表空间包含`mysql`系统数据库和 MySQL 数据字典表。默认情况下，它是未加密的。要为`mysql`系统表空间启用加密，需在`ALTER TABLESPACE`语句中指定表空间名称和`ENCRYPTION`选项。

```sql
mysql> ALTER TABLESPACE mysql ENCRYPTION = 'Y';
```

要禁用`mysql`系统表空间的加密，使用`ALTER TABLESPACE`语句设置`ENCRYPTION = 'N'`。

```sql
mysql> ALTER TABLESPACE mysql ENCRYPTION = 'N';
```

启用或禁用`mysql`系统表空间的加密需要在实例中所有表上具有`CREATE TABLESPACE`权限（`CREATE TABLESPACE on *.*`）。

### 重做日志加密

重做日志数据加密是通过`innodb_redo_log_encrypt`配置选项启用的。默认情况下，重做日志加密是禁用的。

与表空间数据一样，重做日志数据加密发生在重做日志数据写入磁盘时，解密发生在重做日志数据从磁盘读取时。一旦重做日志数据被读入内存，它就是未加密的形式。重做日志数据使用表空间加密密钥进行加密和解密。

当启用`innodb_redo_log_encrypt`时，磁盘上存在的未加密重做日志页面保持未加密，新的重做日志页面以加密形式写入磁盘。同样，当禁用`innodb_redo_log_encrypt`时，磁盘上存在的加密重做日志页面保持加密，新的重做日志页面以未加密形式写入磁盘。

警告

MySQL 8.0.30 中引入的一个回归阻止一旦启用重做日志加密就禁用它。（Bug #108052，Bug #34456802）。

从 MySQL 8.0.30 开始，包括表空间加密密钥在内的重做日志加密元数据存储在具有最新检查点 LSN 的重做日志文件的头部。在 MySQL 8.0.30 之前，包括表空间加密密钥在内的重做日志加密元数据存储在第一个重做日志文件（`ib_logfile0`）的头部。如果带有加密元数据的重做日志文件被移除，则重做日志加密被禁用。

一旦启用重做日志加密，没有密钥环组件或插件或没有加密密钥的情况下，无法进行正常重启，因为`InnoDB`必须能够在启动时扫描重做页面，如果重做日志页面被加密，则无法实现。没有密钥环组件或插件或加密密钥，只能进行强制启动而不使用重做日志（`SRV_FORCE_NO_LOG_REDO`）。参见第 17.21.3 节，“强制 InnoDB 恢复”。

### 撤销日志加密

使用`innodb_undo_log_encrypt`配置选项启用撤销日志数据加密。撤销日志加密适用于驻留在撤销表空间中的撤销日志。参见第 17.6.3.4 节，“撤销表空间”。默认情况下，撤销日志数据加密是禁用的。

与表空间数据一样，撤销日志数据加密发生在将撤销日志数据写入磁盘时，解密发生在从磁盘读取撤销日志数据时。一旦撤销日志数据被读入内存，它就是未加密的形式。撤销日志数据使用表空间加密密钥进行加密和解密。

当启用`innodb_undo_log_encrypt`时，磁盘上存在的未加密撤销日志页面保持未加密，新的撤销日志页面以加密形式写入磁盘。同样，当禁用`innodb_undo_log_encrypt`时，磁盘上存在的加密撤销日志页面保持加密，新的撤销日志页面以未加密形式写入磁盘。

撤销日志加密元数据，包括表空间加密密钥，存储在撤销日志文件的头部。

注意

当撤销日志加密被禁用时，服务器将继续要求用于加密撤销日志数据的密钥环组件或插件，直到包含加密撤销日志数据的撤销表空间被截断。（仅当撤销表空间被截断时，加密头部才会从撤销表空间中移除。）有关截断撤销表空间的信息，请参阅截断撤销表空间。

### 主密钥旋转

主加密密钥应定期旋转，以及在怀疑密钥已被泄露时。

主密钥旋转是一个原子级别的实例操作。每次旋转主加密密钥时，MySQL 实例中的所有表空间密钥都会重新加密并保存回各自的表空间头部。作为原子操作，一旦启动旋转操作，重新加密必须成功完成所有表空间密钥。如果主密钥旋转在服务器故障时中断，`InnoDB` 在服务器重新启动时将操作向前推进。有关更多信息，请参阅加密和恢复。

旋转主加密密钥仅会更改主加密密钥并重新加密表空间密钥。它不会解密或重新加密相关的表空间数据。

旋转主加密密钥需要 `ENCRYPTION_KEY_ADMIN` 权限（或已弃用的 `SUPER` 权限）。

要旋转主加密密钥，请运行：

```sql
mysql> ALTER INSTANCE ROTATE INNODB MASTER KEY;
```

`ALTER INSTANCE ROTATE INNODB MASTER KEY` 支持并发 DML。但是，它不能与表空间加密操作同时运行，并且会采取锁定以防止由并发执行引起的冲突。如果正在运行一个 `ALTER INSTANCE ROTATE INNODB MASTER KEY` 操作，则必须等到该操作完成后，才能继续进行表空间加密操作，反之亦然。

### 加密和恢复

如果在加密操作期间发生服务器故障，则在服务器重新启动时将向前推进操作。对于一般表空间，加密操作将在后台线程中从上次处理的页面继续。

如果在主密钥旋转期间发生服务器故障，`InnoDB` 在服务器重新启动时继续操作。

必须在存储引擎初始化之前加载密钥环组件或插件，以便在`InnoDB` 初始化和恢复活动访问表空间数据之前从表空间头部检索解密表空间数据页所需的信息（请参阅加密先决条件）。

当`InnoDB`初始化和恢复开始时，主密钥旋转操作会继续。由于服务器故障，一些表空间密钥可能已经使用新的主加密密钥加密。`InnoDB`从每个表空间头读取加密数据，如果数据表明表空间密钥是使用旧的主加密密钥加密的，`InnoDB`会从 keyring 中检索旧密钥并用它解密表空间密钥。然后，`InnoDB`使用新的主加密密钥重新加密表空间密钥，并将重新加密的表空间密钥保存回表空间头。

### 导出加密表空间

仅支持文件-每表表空间的表空间导出。

当导出加密表空间时，`InnoDB`会生成一个*传输密钥*，用于加密表空间密钥。加密的表空间密钥和传输密钥存储在一个`*`tablespace_name`*.cfp`文件中。这个文件和加密的表空间文件一起需要执行导入操作。在导入时，`InnoDB`使用传输密钥解密`*`tablespace_name`*.cfp`文件中的表空间密钥。有关更多信息，请参见第 17.6.1.3 节，“导入 InnoDB 表”。

### 加密和复制

+   只有在源和副本运行支持表空间加密的 MySQL 版本的复制环境中才支持`ALTER INSTANCE ROTATE INNODB MASTER KEY`语句。

+   成功的`ALTER INSTANCE ROTATE INNODB MASTER KEY`语句会被写入二进制日志，用于副本的复制。

+   如果`ALTER INSTANCE ROTATE INNODB MASTER KEY`语句失败，则不会记录到二进制日志中，也不会在副本上复制。

+   如果源上安装了 keyring 组件或插件，但副本上没有安装，则`ALTER INSTANCE ROTATE INNODB MASTER KEY`操作的复制会失败。

+   如果源和副本上都安装了`keyring_file`或`keyring_encrypted_file`插件，但副本没有 keyring 数据文件，则复制的`ALTER INSTANCE ROTATE INNODB MASTER KEY`语句会在副本上创建 keyring 数据文件，假设 keyring 文件数据没有缓存在内存中。如果可用，`ALTER INSTANCE ROTATE INNODB MASTER KEY`会使用缓存在内存中的 keyring 文件数据。

### 识别加密表空间和模式

MySQL 8.0.13 中引入的信息模式`INNODB_TABLESPACES`表包括一个`ENCRYPTION`列，可用于识别加密的表空间。

```sql
mysql> SELECT SPACE, NAME, SPACE_TYPE, ENCRYPTION FROM INFORMATION_SCHEMA.INNODB_TABLESPACES
       WHERE ENCRYPTION='Y'\G
*************************** 1\. row ***************************
     SPACE: 4294967294
      NAME: mysql
SPACE_TYPE: General
ENCRYPTION: Y
*************************** 2\. row ***************************
     SPACE: 2
      NAME: test/t1
SPACE_TYPE: Single
ENCRYPTION: Y
*************************** 3\. row ***************************
     SPACE: 3
      NAME: ts1
SPACE_TYPE: General
ENCRYPTION: Y
```

当在`CREATE TABLE`或`ALTER TABLE`语句中指定`ENCRYPTION`选项时，它将记录在`INFORMATION_SCHEMA.TABLES`的`CREATE_OPTIONS`列中。可以查询此列以识别位于加密文件-每表表空间中的表。

```sql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME, CREATE_OPTIONS FROM INFORMATION_SCHEMA.TABLES
       WHERE CREATE_OPTIONS LIKE '%ENCRYPTION%';
+--------------+------------+----------------+
| TABLE_SCHEMA | TABLE_NAME | CREATE_OPTIONS |
+--------------+------------+----------------+
| test         | t1         | ENCRYPTION="Y" |
+--------------+------------+----------------+
```

查询信息模式`INNODB_TABLESPACES`表，以检索与特定模式和表关联的表空间的信息。

```sql
mysql> SELECT SPACE, NAME, SPACE_TYPE FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE NAME='test/t1';
+-------+---------+------------+
| SPACE | NAME    | SPACE_TYPE |
+-------+---------+------------+
|     3 | test/t1 | Single     |
+-------+---------+------------+
```

您可以通过查询信息模式`SCHEMATA`表来识别启用加密的模式。

```sql
mysql> SELECT SCHEMA_NAME, DEFAULT_ENCRYPTION FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE DEFAULT_ENCRYPTION='YES';
+-------------+--------------------+
| SCHEMA_NAME | DEFAULT_ENCRYPTION |
+-------------+--------------------+
| test        | YES                |
+-------------+--------------------+
```

`SHOW CREATE SCHEMA`还显示`DEFAULT ENCRYPTION`子句。

### 监控加密进度

您可以使用性能模式监视通用表空间和`mysql`系统表空间的加密进度。

`stage/innodb/alter tablespace (encryption)`阶段事件工具报告了通用表空间加密操作的`WORK_ESTIMATED`和`WORK_COMPLETED`信息。

以下示例演示了如何启用`stage/innodb/alter tablespace (encryption)`阶段事件工具和相关消费者表，以监视通用表空间或`mysql`系统表空间的加密进度。有关性能模式阶段事件工具和相关消费者的信息，请参阅第 29.12.5 节，“性能模式阶段事件表”。

1.  启用`stage/innodb/alter tablespace (encryption)`工具：

    ```sql
    mysql> USE performance_schema;
    mysql> UPDATE setup_instruments SET ENABLED = 'YES'
           WHERE NAME LIKE 'stage/innodb/alter tablespace (encryption)';
    ```

1.  启用包括`events_stages_current`、`events_stages_history`和`events_stages_history_long`的阶段事件消费者表。

    ```sql
    mysql> UPDATE setup_consumers SET ENABLED = 'YES' WHERE NAME LIKE '%stages%';
    ```

1.  运行表空间加密操作。在此示例中，一个名为`ts1`的通用表空间被加密。

    ```sql
    mysql> ALTER TABLESPACE ts1 ENCRYPTION = 'Y';
    ```

1.  通过查询性能模式`events_stages_current`表来检查加密操作的进度。`WORK_ESTIMATED`报告表空间中的总页数。`WORK_COMPLETED`报告已处理的页数。

    ```sql
    mysql> SELECT EVENT_NAME, WORK_ESTIMATED, WORK_COMPLETED FROM events_stages_current;
    +--------------------------------------------+----------------+----------------+
    | EVENT_NAME                                 | WORK_COMPLETED | WORK_ESTIMATED |
    +--------------------------------------------+----------------+----------------+
    | stage/innodb/alter tablespace (encryption) |           1056 |           1407 |
    +--------------------------------------------+----------------+----------------+
    ```

    如果加密操作已完成，`events_stages_current`表将返回一个空集。在这种情况下，您可以查询`events_stages_history`表查看已完成操作的事件数据。例如：

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM events_stages_history;
    +--------------------------------------------+----------------+----------------+
    | EVENT_NAME                                 | WORK_COMPLETED | WORK_ESTIMATED |
    +--------------------------------------------+----------------+----------------+
    | stage/innodb/alter tablespace (encryption) |           1407 |           1407 |
    +--------------------------------------------+----------------+----------------+
    ```

### 加密使用注意事项

+   当修改具有`ENCRYPTION`选项的现有每个文件表表空间时，请做好计划。驻留在每个文件表表空间中的表将使用`COPY`算法重建。当修改普通表空间或`mysql`系统表空间的`ENCRYPTION`属性时，将使用`INPLACE`算法。`INPLACE`算法允许在普通表空间中的表上进行并发 DML。并发 DDL 被阻止。

+   当一个普通表空间或`mysql`系统表空间被加密时，驻留在表空间中的所有表都会被加密。同样，创建在加密表空间中的表也会被加密。

+   如果服务器在正常运行期间退出或停止，建议使用之前配置的相同加密设置重新启动服务器。

+   第一个主加密密钥是在第一个新的或现有的表空间加密时生成的。

+   主密钥轮换会重新加密表空间密钥，但不会更改表空间密钥本身。要更改表空间密钥，必须禁用并重新启用加密。对于每个表的文件表空间，重新加密表空间是一个`ALGORITHM=COPY`操作，重新构建表。对于普通表空间和`mysql`系统表空间，这是一个`ALGORITHM=INPLACE`操作，不需要重新构建驻留在表空间中的表。

+   如果一个表同时使用`COMPRESSION`和`ENCRYPTION`选项创建，压缩会在表空间数据加密之前执行。

+   如果一个密钥环数据文件（由`keyring_file_data`或`keyring_encrypted_file_data`命名的文件）为空或丢失，第一次执行`ALTER INSTANCE ROTATE INNODB MASTER KEY`将创建一个主加密密钥。

+   卸载`component_keyring_file`或`component_keyring_encrypted_file`组件不会删除现有的密钥环数据文件。卸载`keyring_file`或`keyring_encrypted_file`插件不会删除现有的密钥环数据文件。

+   建议不要将密钥环数据文件放在与表空间数据文件相同的目录下。

+   在运行时或重新启动服务器时修改`keyring_file_data`或`keyring_encrypted_file_data`设置可能导致先前加密的表空间无法访问，导致数据丢失。

+   支持对通过添加`FULLTEXT`索引隐式创建的`InnoDB` `FULLTEXT`索引表进行加密。有关相关信息，请参阅 InnoDB 全文索引表。

### 加密限制

+   高级加密标准（AES）是唯一支持的加密算法。`InnoDB`表空间加密使用电子密码本（ECB）块加密模式进行表空间密钥加密，使用密码块链接（CBC）块加密模式进行数据加密。在 CBC 块加密模式下不使用填充。相反，`InnoDB`确保要加密的文本是块大小的倍数。

+   加密仅支持 file-per-table 表空间、general 表空间和`mysql`系统表空间。MySQL 8.0.13 引入了对 general 表空间的加密支持。MySQL 8.0.16 引入了对`mysql`系统表空间的加密支持。不支持对其他表空间类型（包括`InnoDB`系统表空间）进行加密。

+   无法将加密的 file-per-table 表空间、general 表空间或`mysql`系统表空间中的表移动或复制到不支持加密的表空间类型。

+   无法将表从加密的表空间移动或复制到未加密的表空间。但是，允许将表从未加密的表空间移动到加密的表空间。例如，可以将表从未加密的 file-per-table 或 general 表空间移动或复制到加密的 general 表空间。

+   默认情况下，表空间加密仅适用于表空间中的数据。可以通过启用`innodb_redo_log_encrypt`和`innodb_undo_log_encrypt`来加密重做日志和撤销日志数据。参见重做日志加密和撤销日志加密。有关二进制日志文件和中继日志文件加密的信息，请参见第 19.3.2 节，“加密二进制日志文件和中继日志文件”。

+   不允许更改位于加密表空间中或先前位于加密表空间中的表的存储引擎。
