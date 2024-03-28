# 7.4.1 选择一般查询日志和慢查询日志输出目的地

> 原文：[`dev.mysql.com/doc/refman/8.0/en/log-destinations.html`](https://dev.mysql.com/doc/refman/8.0/en/log-destinations.html)

MySQL 服务器可以灵活控制写入一般查询日志和慢查询日志的输出目的地，如果这些日志已启用。日志条目的可能目的地是日志文件或`mysql`系统数据库中的`general_log`和`slow_log`表。可以选择文件输出、表输出或两者。

+   服务器启动时的日志控制

+   运行时的日志控制

+   日志表的优势和特点

#### 服务器启动时的日志控制

`log_output`系统变量指定日志输出的目的地。设置此变量本身不会启用日志；它们必须单独启用。

+   如果在启动时未指定`log_output`，则默认的日志记录目的地是`FILE`。

+   如果在启动时指定了`log_output`，其值是从`TABLE`（记录到表）、`FILE`（记录到文件）或`NONE`（不记录到表或文件）中选择的一个或多个逗号分隔的单词列表。如果存在`NONE`，则优先于任何其他指定项。

`general_log`系统变量控制所选日志目的地的一般查询日志的记录。如果在服务器启动时指定，`general_log`接受一个可选参数 1 或 0 来启用或禁用日志。要为文件记录指定除默认文件名以外的文件名，请设置`general_log_file`变量。类似地，`slow_query_log`变量控制所选目的地的慢查询日志的记录，并设置`slow_query_log_file`指定文件记录的文件名。如果启用了任一日志，则服务器会打开相应的日志文件并将启动消息写入其中。但是，除非选择了`FILE`日志目的地，否则不会进一步记录查询到文件中。

示例:

+   要将一般查询日志条目写入日志表和日志文件，请使用`--log_output=TABLE,FILE`选择两个日志目的地，并使用`--general_log`启用一般查询日志。

+   仅将一般和慢查询日志条目写入日志表，使用`--log_output=TABLE`选择表格作为日志目的地，并使用`--general_log`和`--slow_query_log`启用两个日志。

+   仅将慢查询日志条目写入日志文件，使用`--log_output=FILE`选择文件作为日志目的地，并使用`--slow_query_log`启用慢查询日志。在这种情况下，因为默认的日志目的地是`FILE`，您可以省略`log_output`设置。

#### 运行时日志控制

与日志表和文件相关的系统变量使得可以在运行时控制日志记录：

+   `log_output`变量指示当前的日志输出目的地。可以在运行时修改以更改目的地。

+   `general_log`和`slow_query_log`变量指示一般查询日志和慢查询日志是否已启用（`ON`）或已禁用（`OFF`）。您可以在运行时设置这些变量以控制日志是否已启用。

+   `general_log_file`和`slow_query_log_file`变量指示一般查询日志和慢查询日志文件的名称。您可以在服务器启动时或在运行时设置这些变量以更改日志文件的名称。

+   要为当前会话禁用或启用一般查询日志记录，请将会话`sql_log_off`变量设置为`ON`或`OFF`。（假设一般查询日志本身已启用。）

#### 日志表的优点和特点

使用表格进行日志输出具有以下优点：

+   日志条目具有标准格式。要显示日志表的当前结构，请使用以下语句：

    ```sql
    SHOW CREATE TABLE mysql.general_log;
    SHOW CREATE TABLE mysql.slow_log;
    ```

+   日志内容可通过 SQL 语句访问。这使得可以使用仅选择满足特定条件的日志条目的查询。例如，要选择与特定客户关联的日志内容（这对于识别来自该客户的问题查询很有用），使用日志表比使用日志文件更容易。

+   日志可通过任何能连接到服务器并发出查询的客户端远程访问（如果客户端具有适当的日志表权限）。无需登录到服务器主机并直接访问文件系统。

日志表实现具有以下特点：

+   一般来说，日志表的主要目的是为用户提供一个接口来观察服务器的运行时执行情况，而不是干预其运行时执行。

+   `CREATE TABLE`、`ALTER TABLE`和`DROP TABLE` 是对日志表的有效操作。对于`ALTER TABLE`和`DROP TABLE`，日志表不能在使用中，必须被禁用，如后面所述。

+   默认情况下，日志表使用将数据以逗号分隔值格式写入的`CSV`存储引擎。对于可以访问包含日志表数据的`.CSV`文件的用户，这些文件易于导入到其他程序中，如可以处理 CSV 输入的电子表格程序。

    可以修改日志表以使用`MyISAM`存储引擎。不能使用`ALTER TABLE`来修改正在使用的日志表。必须先禁用日志。日志表只能使用`CSV`或`MyISAM`引擎，其他引擎不合法。

    **日志表和“打开文件过多”错误。** 如果将`TABLE`作为日志目的地，并且日志表使用`CSV`存储引擎，您可能会发现在运行时反复禁用和启用常规查询日志或慢查询日志会导致`.CSV`文件的打开文件描述符数量增加，可能导致“打开文件过多”错误。为解决此问题，执行`FLUSH TABLES`或确保`open_files_limit`的值大于`table_open_cache_instances`的值。

+   要禁用日志记录以便修改（或删除）日志表，可以使用以下策略。示例使用常规查询日志；慢查询日志的过程类似，但使用`slow_log`表和`slow_query_log`系统变量。

    ```sql
    SET @old_log_state = @@GLOBAL.general_log;
    SET GLOBAL general_log = 'OFF';
    ALTER TABLE mysql.general_log ENGINE = MyISAM;
    SET GLOBAL general_log = @old_log_state;
    ```

+   `TRUNCATE TABLE` 是对日志表的有效操作。可用于清除日志条目。

+   `RENAME TABLE` 是对日志表的有效操作。可以使用以下策略来原子地重命名日志表（例如执行日志轮换）：

    ```sql
    USE mysql;
    DROP TABLE IF EXISTS general_log2;
    CREATE TABLE general_log2 LIKE general_log;
    RENAME TABLE general_log TO general_log_backup, general_log2 TO general_log;
    ```

+   `CHECK TABLE` 是对日志表的有效操作。

+   `LOCK TABLES` 不能用于日志表。

+   `INSERT`、`DELETE`和`UPDATE`不能用于日志表。这些操作仅在服务器内部允许。

+   `FLUSH TABLES WITH READ LOCK`和`read_only`系统变量的状态对日志表没有影响。服务器始终可以写入日志表。

+   写入日志表的条目不会写入二进制日志，因此不会被复制到副本。

+   要刷新日志表或日志文件，请分别使用`FLUSH TABLES`或`FLUSH LOGS`。

+   不允许对日志表进行分区。

+   **mysqldump**转储包括重新创建这些表的语句，以便在重新加载转储文件后它们不会丢失。日志表内容不会被转储。
