# 15.1.2 ALTER DATABASE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-database.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-database.html)

```sql
ALTER {DATABASE | SCHEMA} [*db_name*]
    *alter_option* ...

*alter_option*: {
    [DEFAULT] CHARACTER SET [=] *charset_name*
  | [DEFAULT] COLLATE [=] *collation_name*
  | [DEFAULT] ENCRYPTION [=] {'Y' | 'N'}
  | READ ONLY [=] {DEFAULT | 0 | 1}
}
```

`ALTER DATABASE`允许您更改数据库的整体特性。这些特性存储在数据字典中。此语句需要对数据库具有`ALTER`权限。`ALTER SCHEMA`是`ALTER DATABASE`的同义词。

如果省略了数据库名称，则该语句适用于默认数据库。在这种情况下，如果没有默认数据库，则会发生错误。

对于语句中省略的任何*`alter_option`*，数据库将保留其当前选项值，但更改字符集可能会更改校对规则，反之亦然。

+   字符集和校对选项

+   加密选项

+   只读选项

#### 字符集和校对选项

`CHARACTER SET`选项更改默认数据库字符集。`COLLATE`选项更改默认数据库校对规则。有关字符集和校对规则名称的信息，请参见第十二章，*字符集，校对规则，Unicode*。

要查看可用的字符集和校对规则，请分别使用`SHOW CHARACTER SET`和`SHOW COLLATION`语句。请参见第 15.7.7.3 节，“SHOW CHARACTER SET 语句”，以及第 15.7.7.4 节，“SHOW COLLATION 语句”。

在创建存储过程时使用数据库默认值的存储过程将这些默认值作为其定义的一部分。 （在存储过程中，如果未明确指定字符集或校对规则，则具有字符数据类型的变量将使用数据库默认值。请参见第 15.1.17 节，“CREATE PROCEDURE 和 CREATE FUNCTION 语句”。）如果更改数据库的默认字符集或校对规则，则必须删除并重新创建任何要使用新默认值的存储过程。

#### 加密选项

`ENCRYPTION`选项是在 MySQL 8.0.16 中引入的，定义了默认数据库加密，该加密会被创建在数据库中的表继承。允许的值为`'Y'`（启用加密）和`'N'`（禁用加密）。

`mysql`系统模式无法设置为默认加密。其中现有的表属于通用`mysql`表空间，可能已加密。`information_schema`仅包含视图。不可能在其中创建任何表。磁盘上没有任何内容可供加密。`performance_schema`中的所有表都使用`PERFORMANCE_SCHEMA`引擎，纯粹是内存中的。不可能在其中创建任何其他表。磁盘上没有任何内容可供加密。

仅新创建的表继承默认数据库加密。对于与数据库关联的现有表，它们的加密保持不变。如果启用了`table_encryption_privilege_check`系统变量，则需要`TABLE_ENCRYPTION_ADMIN`权限来指定与`default_table_encryption`系统变量的值不同的默认加密设置。有关更多信息，请参阅为模式和通用表空间定义加密默认值。

#### 只读选项

`READ ONLY`选项在 MySQL 8.0.22 中引入，控制是否允许修改数据库及其中的对象。允许的值为`DEFAULT`或`0`（非只读）和`1`（只读）。此选项对数据库迁移很有用，因为启用`READ ONLY`的数据库可以在迁移至另一个 MySQL 实例时，无需担心数据库在操作期间会被更改。

对于 NDB Cluster，在一个**mysqld**服务器上将数据库设置为只读会同步到同一集群中的其他**mysqld**服务器，使得数据库在所有**mysqld**服务器上都变为只读。

如果启用了`READ ONLY`选项，则会在`INFORMATION_SCHEMA`的`SCHEMATA_EXTENSIONS`表中显示。请参阅第 28.3.32 节，“INFORMATION_SCHEMA SCHEMATA_EXTENSIONS 表”。

不能为这些系统模式启用`READ ONLY`选项：`mysql`、`information_schema`、`performance_schema`。

在`ALTER DATABASE`语句中，`READ ONLY`选项与其他实例及其他选项的交互如下：

+   如果多个`READ ONLY`实例发生冲突（例如，`READ ONLY = 1 READ ONLY = 0`），则会发生错误。

+   即使是只读数据库，也允许包含（不冲突的）`READ ONLY`选项的`ALTER DATABASE`语句。

+   如果数据库在语句执行前或执行后的只读状态允许修改，则允许将（不冲突的）`READ ONLY`选项与其他选项混合使用。如果数据库在执行前后的只读状态都禁止更改，则会发生错误。

    无论数据库是否只读，此语句都会成功：

    ```sql
    ALTER DATABASE mydb READ ONLY = 0 DEFAULT COLLATE utf8mb4_bin;
    ```

    如果数据库不是只读，则此语句成功，但如果数据库已经是只读，则失败：

    ```sql
    ALTER DATABASE mydb READ ONLY = 1 DEFAULT COLLATE utf8mb4_bin;
    ```

启用`READ ONLY`会影响数据库的所有用户，以下情况不受只读检查限制：

+   作为服务器初始化、重启、升级或复制的一部分由服务器执行的语句。

+   由`init_file`系统变量在服务器启动时命名的文件中的语句。

+   `TEMPORARY`表；在只读数据库中可以创建、修改、删除和写入`TEMPORARY`表。

+   NDB Cluster 非 SQL 插入和更新。

除了刚刚列出的例外操作外，启用`READ ONLY`会禁止对数据库及其对象（包括定义、数据和元数据）进行写操作。以下列表详细说明受影响的 SQL 语句和操作：

+   数据库本身：

    +   `CREATE DATABASE`

    +   `ALTER DATABASE`（除了更改`READ ONLY`选项）

    +   `DROP DATABASE`

+   视图：

    +   `CREATE VIEW`

    +   `ALTER VIEW`

    +   `DROP VIEW`

    +   从调用具有副作用的函数的视图中进行选择。

    +   更新可更新的视图。

    +   如果影响只读数据库中视图的元数据（例如，使视图有效或无效）的对象在可写数据库中创建或删除，则会拒绝这些语句。

+   存储过程：

    +   `CREATE PROCEDURE`

    +   `DROP PROCEDURE`

    +   `CALL`（具有副作用的过程调用）

    +   `CREATE FUNCTION`

    +   `DROP FUNCTION`

    +   `SELECT`（具有副作用的函数选择）

    +   对于存储过程和函数，只读检查遵循预锁定行为。对于`CALL`语句，只读检查是基于每个语句进行的，因此如果某个有条件执行的写入只读数据库的语句实际上没有执行，调用仍然成功。另一方面，在`SELECT`中调用的函数，函数体的执行是在预锁定模式下进行的。只要函数中的某个语句写入只读数据库，无论该语句是否实际执行，函数的执行都会因错误而失败。

+   触发器：

    +   `CREATE TRIGGER`

    +   `DROP TRIGGER`

    +   触发器调用。

+   事件：

    +   `CREATE EVENT`

    +   `ALTER EVENT`

    +   `DROP EVENT`

    +   事件执行：

        +   在数据库中执行事件会失败，因为这会更改最后执行时间戳，这是存储在数据字典中的事件元数据。事件执行失败还会导致事件调度程序停止。

        +   如果事件写入只读数据库中的对象，则事件执行将因错误而失败，但事件调度程序不会停止。

+   表：

    +   `CREATE TABLE`

    +   `ALTER TABLE`

    +   `CREATE INDEX`

    +   `DROP INDEX`

    +   `RENAME TABLE`

    +   `TRUNCATE TABLE`

    +   `DROP TABLE`

    +   `DELETE`

    +   `INSERT`

    +   `IMPORT TABLE`

    +   `LOAD DATA`

    +   `LOAD XML`

    +   `REPLACE`

    +   `UPDATE`

    +   对于级联外键，如果子表位于只读数据库中，则父表的更新和删除将被拒绝，即使子表并未直接受到影响。

    +   对于`MERGE`表，例如`CREATE TABLE s1.t(i int) ENGINE MERGE UNION (s2.t, s3.t), INSERT_METHOD=...`，以下行为适用：

        +   如果插入`MERGE`表（`INSERT into s1.t`）时，`s1`、`s2`、`s3`中至少有一个是只读的，则无论插入方法如何，插入都会失败。即使实际上插入的数据最终会进入可写表中，也会被拒绝。

        +   删除`MERGE`表（`DROP TABLE s1.t`）成功，只要`s1`不是只读的。允许删除引用只读数据库的`MERGE`表。

一个`ALTER DATABASE`语句会阻塞，直到所有已经访问正在被更改的数据库中的对象的并发事务都已提交。相反，访问正在被并发`ALTER DATABASE`更改的数据库中的对象的写事务会阻塞，直到`ALTER DATABASE`已提交。

如果使用克隆插件克隆本地或远程数据目录，则克隆中的数据库保留其在源数据目录中的只读状态。只读状态不会影响克隆过程本身。如果不希望在克隆中具有相同的数据库只读状态，则必须在克隆过程完成后显式更改选项，使用克隆上的`ALTER DATABASE`操作。

当从捐赠者克隆到接收者时，如果接收者有一个用户数据库是只读的，克隆将因错误消息而失败。在使数据库可写后，可以重试克隆。

`READ ONLY`允许用于`ALTER DATABASE`，但不允许用于`CREATE DATABASE`。然而，对于只读数据库，由`SHOW CREATE DATABASE`生成的语句在注释中包含`READ ONLY=1`，以指示其只读状态：

```sql
mysql> ALTER DATABASE mydb READ ONLY = 1;
mysql> SHOW CREATE DATABASE mydb\G
*************************** 1\. row ***************************
       Database: mydb
Create Database: CREATE DATABASE `mydb`
                 /*!40100 DEFAULT CHARACTER SET utf8mb4
                          COLLATE utf8mb4_0900_ai_ci */
                 /*!80016 DEFAULT ENCRYPTION='N' */
                 /* READ ONLY = 1 */
```

如果服务器执行包含这样一个注释的`CREATE DATABASE`语句，服务器会忽略该注释，`READ ONLY`选项不会被处理。这对于**mysqldump**和**mysqlpump**有影响，它们使用`SHOW CREATE DATABASE`生成转储输出中的`CREATE DATABASE`语句：

+   在转储文件中，只读数据库的`CREATE DATABASE`语句包含了注释的`READ ONLY`选项。

+   转储文件可以像往常一样还原，但由于服务器忽略了注释的`READ ONLY`选项，还原的数据库*不是*只读的。如果在还原后要使数据库变为只读，必须手动执行`ALTER DATABASE`来实现。

假设`mydb`是只读的，并且您将其转储如下：

```sql
$> mysqldump --databases mydb > mydb.sql
```

之后的还原操作必须在`mydb`仍然是只读时跟随`ALTER DATABASE`：

```sql
$> mysql
mysql> SOURCE mydb.sql;
mysql> ALTER DATABASE mydb READ ONLY = 1;
```

MySQL 企业备份不受此问题影响。它备份和恢复只读数据库就像其他数据库一样，但如果备份时启用了`READ ONLY`选项，则在恢复时会启用该选项。

`ALTER DATABASE`会被写入二进制日志，因此在复制源服务器上对`READ ONLY`选项进行更改也会影响副本。为了防止这种情况发生，在执行`ALTER DATABASE`语句之前必须禁用二进制日志。例如，为了准备迁移数据库而不影响副本，执行以下操作：

1.  在单个会话中，禁用二进制日志，并为数据库启用`READ ONLY`：

    ```sql
    mysql> SET sql_log_bin = OFF;
    mysql> ALTER DATABASE mydb READ ONLY = 1;
    ```

1.  使用**mysqldump**或**mysqlpump**等工具对数据库进行转储：

    ```sql
    $> mysqldump --databases mydb > mydb.sql
    ```

1.  在单个会话中，禁用二进制日志并禁用数据库的`READ ONLY`：

    ```sql
    mysql> SET sql_log_bin = OFF;
    mysql> ALTER DATABASE mydb READ ONLY = 0;
    ```
