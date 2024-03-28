# 27.7 存储程序二进制日志记录

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html)

二进制日志包含有关修改数据库内容的 SQL 语句的信息。这些信息以描述修改的“事件”形式存储。（二进制日志事件与计划事件存储对象不同。）二进制日志有两个重要目的：

+   对于复制，二进制日志在源复制服务器上用作要发送到副本服务器的语句记录。源服务器将其二进制日志中包含的事件发送到其副本，副本执行这些事件以进行与源上进行的相同数据更改。请参阅 Section 19.2, “Replication Implementation”。

+   某些数据恢复操作需要使用二进制日志。在恢复备份文件后，将重新执行在备份文件生成后记录的二进制日志中的事件。这些事件将数据库从备份点更新到最新。请参阅 Section 9.3.2, “Using Backups for Recovery”。

然而，如果日志记录发生在语句级别，那么关于存储程序（存储过程和函数、触发器和事件）的二进制日志记录存在某些问题：

+   在某些情况下，一条语句可能会影响源和副本上的不同行集。

+   在副本上执行的复制语句由副本的应用程序线程处理。除非您实现了复制权限检查，这些权限从 MySQL 8.0.18 开始提供（请参阅 Section 19.3.3, “Replication Privilege Checks”），否则应用程序线程具有完全权限。在这种情况下，一个过程可能会在源服务器和副本服务器上遵循不同的执行路径，因此用户可以编写一个包含仅在副本上执行的危险语句的例程。

+   如果修改数据的存储程序是不确定性的，那么它就不可重复。这可能导致源和副本上的数据不同，或导致恢复的数据与原始数据不同。

本节描述了 MySQL 如何处理存储程序的二进制日志记录。它说明了实现对存储程序使用的当前条件，以及您可以采取什么措施避免日志记录问题。它还提供了关于这些条件原因的额外信息。

除非另有说明，这里的备注假定服务器上已启用二进制日志记录（参见第 7.4.4 节，“二进制日志”）。如果未启用二进制日志，则无法进行复制，也无法使用二进制日志进行数据恢复。从 MySQL 8.0 开始，默认启用二进制日志记录，只有在启动时指定 `--skip-log-bin` 或 `--disable-log-bin` 选项时才会禁用。

通常，描述的问题是在 SQL 语句级别发生二进制日志记录时（基于语句的二进制日志记录）产生的。如果使用基于行的二进制日志记录，日志将包含执行 SQL 语句导致的单个行的更改。当例程或触发器执行时，将记录行更改，而不是进行更改的语句。对于存储过程，这意味着 `CALL` 语句不会被记录。对于存储函数，将记录函数内部进行的行更改，而不是函数调用。对于触发器，将记录触发器进行的行更改。在副本端，只能看到行更改，而看不到存储程序调用。

混合格式二进制日志记录（`binlog_format=MIXED`）使用基于语句的二进制日志记录，除非只有基于行的二进制日志记录才能确保产生正确结果的情况。使用混合格式时，当存储函数、存储过程、触发器、事件或准备语句包含任何不适合基于语句的二进制日志记录的内容时，整个语句将被标记为不安全，并以行格式记录。用于创建和删除过程、函数、触发器和事件的语句始终是安全的，并以语句格式记录。有关基于行、混合和基于语句的日志记录以及如何确定安全和不安全语句的更多信息，请参见第 19.2.1 节，“复制格式”。

MySQL 中对存储函数使用的条件可以总结如下。这些条件不适用于存储过程或事件调度器事件，也不适用于未启用二进制日志记录的情况。

+   要创建或更改存储函数，您必须具有`SET_USER_ID`权限（或已弃用的`SUPER`权限），除了通常所需的`CREATE ROUTINE`或`ALTER ROUTINE`权限。 （根据函数定义中的`DEFINER`值，无论是否启用了二进制日志记录，可能需要`SET_USER_ID`或`SUPER`。请参阅第 15.1.17 节，“CREATE PROCEDURE and CREATE FUNCTION Statements”.）

+   创建存储函数时，必须声明其是确定性的或不修改数据。否则，可能对数据恢复或复制不安全。

    默认情况下，要接受`CREATE FUNCTION`语句，必须明确指定`DETERMINISTIC`、`NO SQL`或`READS SQL DATA`中的至少一个。否则会出现错误：

    ```sql
    ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL,
    or READS SQL DATA in its declaration and binary logging is enabled
    (you *might* want to use the less safe log_bin_trust_function_creators
    variable)
    ```

    此函数是确定性的（且不修改数据），因此是安全的：

    ```sql
    CREATE FUNCTION f1(i INT)
    RETURNS INT
    DETERMINISTIC
    READS SQL DATA
    BEGIN
      RETURN i;
    END;
    ```

    此函数使用`UUID()`，这是不确定性的，因此该函数也是不确定性的且不安全的：

    ```sql
    CREATE FUNCTION f2()
    RETURNS CHAR(36) CHARACTER SET utf8mb4
    BEGIN
      RETURN UUID();
    END;
    ```

    此函数修改数据，因此可能不安全：

    ```sql
    CREATE FUNCTION f3(p_id INT)
    RETURNS INT
    BEGIN
      UPDATE t SET modtime = NOW() WHERE id = p_id;
      RETURN ROW_COUNT();
    END;
    ```

    对函数性质的评估基于创建者的“诚实”。MySQL 不会检查声明为`DETERMINISTIC`的函数是否不包含产生非确定性结果的语句。

+   当您尝试执行存储函数时，如果设置了`binlog_format=STATEMENT`，则必须在函数定义中指定`DETERMINISTIC`关键字。如果没有这样做，将生成错误并且函数不会运行，除非指定`log_bin_trust_function_creators=1`以覆盖此检查（见下文）。对于递归函数调用，只有在最外层调用上才需要`DETERMINISTIC`关键字。如果使用基于行或混合二进制日志记录，则即使函数在没有`DETERMINISTIC`关键字的情况下定义，该语句也会被接受和复制。

+   因为 MySQL 在创建时不会检查函数是否真的是确定性的，所以带有`DETERMINISTIC`关键字的存储函数的调用可能执行对基于语句的日志记录不安全的操作，或调用包含不安全语句的函数或过程。如果在设置了`binlog_format=STATEMENT`时发生这种情况，则会发出警告消息。如果使用基于行或混合二进制日志记录，则不会发出警告，并且该语句以基于行的格式被复制。

+   放宽函数创建的先决条件（必须具有`SUPER`权限，并且函数必须声明为确定性或不修改数据），将全局`log_bin_trust_function_creators`系统变量设置为 1。默认情况下，此变量的值为 0，但您可以像这样更改它：

    ```sql
    mysql> SET GLOBAL log_bin_trust_function_creators = 1;
    ```

    您也可以在服务器启动时设置此变量。

    如果未启用二进制日志记录，则`log_bin_trust_function_creators`不适用。除非如前所述，函数定义中的`DEFINER`值要求，否则不需要`SUPER`权限来创建函数。

+   有关内置函数可能不安全用于复制（因此导致使用它们的存储函数也不安全）的信息，请参阅 Section 19.5.1, “Replication Features and Issues”。

触发器类似于存储函数，因此关于函数的先前备注也适用于触发器，唯一的例外是：`CREATE TRIGGER`没有可选的`DETERMINISTIC`特征，因此假定触发器始终是确定性的。但是，在某些情况下，这种假设可能是无效的。例如，`UUID()`函数是不确定性的（且不会复制）。在触发器中使用此类函数时要小心。

触发器可以更新表，因此如果没有所需权限，与存储函数类似的错误消息将在`CREATE TRIGGER`时出现。在副本端，副本使用触发器的`DEFINER`属性来确定哪个用户被视为触发器的创建者。

本节的其余部分提供了有关日志记录实现及其影响的额外细节。除非您对当前与存储例程使用相关的日志记录条件的背景感兴趣，否则无需阅读。此讨论仅适用于基于语句的日志记录，不适用于基于行的日志记录，除了第一项：`CREATE`和`DROP`语句无论日志记录模式如何都将作为语句记录。

+   服务器将`CREATE EVENT`、`CREATE PROCEDURE`、`CREATE FUNCTION`、`ALTER EVENT`、`ALTER PROCEDURE`、`ALTER FUNCTION`、`DROP EVENT`、`DROP PROCEDURE`和`DROP FUNCTION`语句写入二进制日志。

+   如果函数更改数据并且出现在否则不会被记录的语句中，则存储函数调用将被记录为`SELECT`语句。这可以防止由于在非记录语句中使用存储函数而导致的数据更改不被复制。例如，`SELECT`语句不会被写入二进制日志，但是`SELECT`可能调用一个进行更改的存储函数。为了处理这种情况，当给定函数进行更改时，将`SELECT *`func_name`*()`语句写入二进制日志。假设以下语句在源服务器上执行：

    ```sql
    CREATE FUNCTION f1(a INT) RETURNS INT
    BEGIN
      IF (a < 3) THEN
        INSERT INTO t2 VALUES (a);
      END IF;
      RETURN 0;
    END;

    CREATE TABLE t1 (a INT);
    INSERT INTO t1 VALUES (1),(2),(3);

    SELECT f1(a) FROM t1;
    ```

    当`SELECT`语句执行时，函数`f1()`被调用三次。其中两次调用插入一行，并且 MySQL 为每次插入都记录了一个`SELECT`语句。也就是说，MySQL 将以下语句写入二进制日志：

    ```sql
    SELECT f1(1);
    SELECT f1(2);
    ```

    当函数调用存储过程导致错误时，服务器还会记录一个`SELECT`语句。在这种情况下，服务器将`SELECT`语句与预期的错误代码一起写入日志。在副本中，如果发生相同的错误，那就是预期的结果，复制将继续。否则，复制将停止。

+   记录存储函数调用而不是函数执行的语句对复制有安全影响，这是由两个因素引起的：

    +   函数可能在源服务器和副本服务器上遵循不同的执行路径。

    +   在副本上执行的语句由副本的应用程序线程处理。除非您实现了复制权限检查，这在 MySQL 8.0.18 中可用（参见 Section 19.3.3, “Replication Privilege Checks”），否则应用程序线程具有完全权限。

    这意味着，尽管用户必须拥有 `CREATE ROUTINE` 权限才能创建函数，但用户可以编写一个包含危险语句的函数，只在复制品上执行，由具有完全权限的线程处理。例如，如果源服务器和复制品服务器的服务器 ID 值分别为 1 和 2，则源服务器上的用户可以创建并调用一个不安全的函数 `unsafe_func()` 如下：

    ```sql
    mysql> delimiter //
    mysql> CREATE FUNCTION unsafe_func () RETURNS INT
     -> BEGIN
     ->   IF @@server_id=2 THEN *dangerous_statement*; END IF;
     ->   RETURN 1;
     -> END;
     -> //
    mysql> delimiter ;
    mysql> INSERT INTO t VALUES(unsafe_func());
    ```

    `CREATE FUNCTION` 和 `INSERT` 语句被写入二进制日志，因此复制品会执行它们。由于复制品的应用程序线程拥有完全权限，它会执行危险的语句。因此，函数调用对源和复制品有不同的影响，不是复制安全的。

    为了防范启用了二进制日志记录的服务器的这种危险，存储函数创建者必须拥有 `SUPER` 权限，除了通常需要的 `CREATE ROUTINE` 权限。同样，要使用 `ALTER FUNCTION`，除了 `ALTER ROUTINE` 权限外，还必须拥有 `SUPER` 权限。如果没有 `SUPER` 权限，会出现错误：

    ```sql
    ERROR 1419 (HY000): You do not have the SUPER privilege and
    binary logging is enabled (you *might* want to use the less safe
    log_bin_trust_function_creators variable)
    ```

    如果不希望要求函数创建者拥有 `SUPER` 权限（例如，如果系统上所有具有 `CREATE ROUTINE` 权限的用户都是经验丰富的应用程序开发人员），请将全局 `log_bin_trust_function_creators` 系统变量设置为 1\. 您也可以在服务器启动时设置此变量。如果未启用二进制日志记录，则 `log_bin_trust_function_creators` 不适用。除非如前所述，函数定义中的 `DEFINER` 值要求，否则不需要 `SUPER` 权限来创建函数。

+   建议无论您对函数创建者的权限做出何种选择，都使用可用的复制权限检查（从 MySQL 8.0.18 开始）。可以设置复制权限检查以确保仅授权复制通道的预期和相关操作。有关如何执行此操作的说明，请参见 Section 19.3.3, “Replication Privilege Checks”。

+   如果执行更新的函数是非确定性的，则不可重复。这可能会产生两个不良影响：

    +   它导致复制品与源不同。

    +   恢复的数据与原始数据不匹配。

    为了解决这些问题，MySQL 强制执行以下要求：在源服务器上，除非声明函数是确定性的或不修改数据，否则拒绝创建和修改函数。这里有两组函数特性：

    +   `DETERMINISTIC` 和 `NOT DETERMINISTIC` 特性指示函数是否对给定输入始终产生相同的结果。如果没有给出任何特性，则默认为 `NOT DETERMINISTIC`。要声明函数是确定性的，必须明确指定 `DETERMINISTIC`。

    +   `CONTAINS SQL`、`NO SQL`、`READS SQL DATA` 和 `MODIFIES SQL DATA` 特性提供了关于函数是否读取或写入数据的信息。`NO SQL` 或 `READS SQL DATA` 表明函数不会改变数据，但如果没有明确指定特性，则默认为 `CONTAINS SQL`。

    默认情况下，要接受`CREATE FUNCTION`语句，必须明确指定至少一个 `DETERMINISTIC`、`NO SQL` 或 `READS SQL DATA`。否则会出现错误：

    ```sql
    ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL,
    or READS SQL DATA in its declaration and binary logging is enabled
    (you *might* want to use the less safe log_bin_trust_function_creators
    variable)
    ```

    如果将`log_bin_trust_function_creators`设置为 1，则不再需要函数是确定性的或不修改数据的要求。

+   存储过程调用在语句级别而不是`CALL`级别记录。也就是说，服务器不会记录`CALL`语句，而是记录实际执行的过程中的语句。因此，在源服务器上发生的更改也会在副本上发生。这可以防止由于过程在不同机器上具有不同的执行路径而导致的问题。

    一般来说，在存储过程中执行的语句会按照在独立方式执行时应用的相同规则写入二进制日志。在记录过程语句时需要特别注意，因为过程内的语句执行与非过程上下文中的执行不完全相同：

    +   要记录的语句可能包含对本地过程变量的引用。这些变量在存储过程上下文之外不存在，因此引用这样一个变量的语句不能直接记录。而是为了记录目的，将每个对本地变量的引用替换为以下构造：

        ```sql
        NAME_CONST(*var_name*, *var_value*)
        ```

        *`var_name`* 是本地变量名称，*`var_value`* 是指示变量在记录语句时的值的常量。`NAME_CONST()` 的值为 *`var_value`*，“名称”为 *`var_name`*。因此，如果直接调用此函数，将获得如下结果：

        ```sql
        mysql> SELECT NAME_CONST('myname', 14);
        +--------+
        | myname |
        +--------+
        |     14 |
        +--------+
        ```

        `NAME_CONST()`使得可以在副本上执行一个已记录的独立语句，其效果与在源上执行的原始语句相同，而原始语句是在存储过程中执行的。

        使用`NAME_CONST()`可能会导致在`CREATE TABLE ... SELECT`语句中出现问题，当源列表达式引用本地变量时。将这些引用转换为`NAME_CONST()`表达式可能导致源服务器和副本服务器上的列名不同，或者列名过长而无法成为合法的列标识符。一个解决方法是为引用本地变量的列提供别名。考虑当`myvar`的值为 1 时的语句：

        ```sql
        CREATE TABLE t1 SELECT myvar;
        ```

        重写如下：

        ```sql
        CREATE TABLE t1 SELECT NAME_CONST(myvar, 1);
        ```

        为确保源和副本表具有相同的列名，应该这样写语句：

        ```sql
        CREATE TABLE t1 SELECT myvar AS myvar;
        ```

        重写后的语句如下：

        ```sql
        CREATE TABLE t1 SELECT NAME_CONST(myvar, 1) AS myvar;
        ```

    +   要记录的语句可能包含对用户定义变量的引用。为了处理这个问题，MySQL 在二进制日志中写入一个`SET`语句，以确保变量在副本上存在，并且具有与源相同的值。例如，如果一个语句引用变量`@my_var`，那么该语句在二进制日志中的前面会有以下语句，其中*`value`*是源上`@my_var`的值：

        ```sql
        SET @my_var = *value*;
        ```

    +   过程调用可以发生在已提交或已回滚的事务中。事务上下文被考虑，以便正确复制过程执行的事务方面。也就是说，服务器记录那些在过程中实际执行和修改数据的语句，并根据需要记录`BEGIN`、`COMMIT`和`ROLLBACK`语句。例如，如果一个过程只更新事务表，并在回滚的事务中执行，那些更新不会被记录。如果过程发生在已提交的事务中，`BEGIN`和`COMMIT`语句将与更新一起记录。对于在已回滚事务中执行的过程，其语句将使用与在独立方式下执行时相同的规则进行记录：

        +   对事务表的更新不会被记录。

        +   对非事务表的更新是被记录的，因为回滚不会取消它们。

        +   更新混合事务和非事务表的操作被记录在`BEGIN`和`ROLLBACK`之间，以便副本执行与源相同的更改和回滚。

+   如果存储过程是在存储函数内部调用的，则存储过程调用*不会*以语句级别写入二进制日志。在这种情况下，记录的仅是调用函数的语句（如果它出现在被记录的语句内部）或一个`DO`语句（如果它出现在未记录的语句内部）。因此，即使存储过程本身是安全的，也应谨慎使用调用存储过程的存储函数。
