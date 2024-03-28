# 11.3 关键字和保留字

> 原文：[`dev.mysql.com/doc/refman/8.0/en/keywords.html`](https://dev.mysql.com/doc/refman/8.0/en/keywords.html)

关键字是在 SQL 中具有重要意义的单词。 某些关键字，例如`SELECT`，`DELETE`或`BIGINT`，是保留的，需要特殊处理才能用作标识符，例如表和列名。 对于内置函数的名称也可能是如此。

非保留关键字可以作为标识符而无需引用。 如果引用它们如第 11.2 节，“模式对象名称”中所述，保留字可以作为标识符使用：

```sql
mysql> CREATE TABLE interval (begin INT, end INT);
ERROR 1064 (42000): You have an error in your SQL syntax ...
near 'interval (begin INT, end INT)'
```

`BEGIN` 和 `END` 是关键字但不是保留字，因此它们作为标识符的使用不需要引用。 `INTERVAL` 是一个保留关键字，必须引用才能用作标识符：

```sql
mysql> CREATE TABLE `interval` (begin INT, end INT);
Query OK, 0 rows affected (0.01 sec)
```

例外：在限定名称中跟在句点后面的单词必须是标识符，因此即使它是保留字，也不需要引用：

```sql
mysql> CREATE TABLE mydb.interval (begin INT, end INT);
Query OK, 0 rows affected (0.01 sec)
```

内置函数的名称可以作为标识符，但可能需要小心使用。 例如，`COUNT` 可以作为列名。 但是，默认情况下，在函数名称和后续`(`字符之间不允许有空格。 此要求使解析器能够区分名称是在函数调用中使用还是在非函数上下文中使用。 有关函数名称识别的更多详细信息，请参阅第 11.2.5 节，“函数名称解析和解析”。

`INFORMATION_SCHEMA.KEYWORDS` 表列出了 MySQL 认为是关键字的单词，并指示它们是否是保留字。 请参阅第 28.3.17 节，“INFORMATION_SCHEMA KEYWORDS 表”。

+   MySQL 8.0 关键字和保留字

+   MySQL 8.0 新关键字和保留字

+   MySQL 8.0 已移除的关键字和保留字

### MySQL 8.0 关键字和保留字

下面的列表显示了 MySQL 8.0 中的关键字和保留字，以及从版本到版本的单词更改。 保留关键字用（R）标记。此外，`_FILENAME` 是保留的。

在某个时候，您可能会升级到更高版本，因此查看未来保留字也是一个好主意。您可以在涵盖更高版本 MySQL 的手册中找到这些内容。列���中的大多数保留字被标准 SQL 禁止用作列或表名（例如，`GROUP`）。有一些是保留的，因为 MySQL 需要它们并使用**yacc**解析器。

A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W | X | Y | Z

A

+   `ACCESSIBLE` (R)

+   `ACCOUNT`

+   `ACTION`

+   `ACTIVE`; added in 8.0.14 (nonreserved)

+   `ADD` (R)

+   `ADMIN`; became nonreserved in 8.0.12

+   `AFTER`

+   `AGAINST`

+   `AGGREGATE`

+   `ALGORITHM`

+   `ALL` (R)

+   `ALTER` (R)

+   `ALWAYS`

+   `ANALYSE`; removed in 8.0.1

+   `ANALYZE` (R)

+   `AND` (R)

+   `ANY`

+   `ARRAY`; added in 8.0.17 (reserved); became nonreserved in 8.0.19

+   `AS` (R)

+   `ASC` (R)

+   `ASCII`

+   `ASENSITIVE` (R)

+   `AT`

+   `ATTRIBUTE`; added in 8.0.21 (nonreserved)

+   `AUTHENTICATION`; added in 8.0.27 (nonreserved)

+   `AUTOEXTEND_SIZE`

+   `AUTO_INCREMENT`

+   `AVG`

+   `AVG_ROW_LENGTH`

B

+   `BACKUP`

+   `BEFORE` (R)

+   `BEGIN`

+   `BETWEEN` (R)

+   `BIGINT` (R)

+   `BINARY` (R)

+   `BINLOG`

+   `BIT`

+   `BLOB` (R)

+   `BLOCK`

+   `BOOL`

+   `BOOLEAN`

+   `BOTH` (R)

+   `BTREE`

+   `BUCKETS`; added in 8.0.2 (nonreserved)

+   `BULK`; added in 8.0.32 (nonreserved)

+   `BY` (R)

+   `BYTE`

C

+   `CACHE`

+   `CALL` (R)

+   `CASCADE` (R)

+   `CASCADED`

+   `CASE` (R)

+   `CATALOG_NAME`

+   `CHAIN`

+   `CHALLENGE_RESPONSE`; added in 8.0.27 (nonreserved)

+   `CHANGE` (R)

+   `CHANGED`

+   `CHANNEL`

+   `CHAR` (R)

+   `CHARACTER` (R)

+   `CHARSET`

+   `CHECK` (R)

+   `CHECKSUM`

+   `CIPHER`

+   `CLASS_ORIGIN`

+   `CLIENT`

+   `CLONE`; added in 8.0.3 (nonreserved)

+   `CLOSE`

+   `COALESCE`

+   `CODE`

+   `COLLATE` (R)

+   `COLLATION`

+   `COLUMN` (R)

+   `COLUMNS`

+   `COLUMN_FORMAT`

+   `COLUMN_NAME`

+   `COMMENT`

+   `COMMIT`

+   `COMMITTED`

+   `COMPACT`

+   `COMPLETION`

+   `COMPONENT`

+   `COMPRESSED`

+   `COMPRESSION`

+   `CONCURRENT`

+   `CONDITION` (R)

+   `CONNECTION`

+   `CONSISTENT`

+   `CONSTRAINT` (R)

+   `CONSTRAINT_CATALOG`

+   `CONSTRAINT_NAME`

+   `CONSTRAINT_SCHEMA`

+   `CONTAINS`

+   `CONTEXT`

+   `CONTINUE` (R)

+   `CONVERT` (R)

+   `CPU`

+   `CREATE` (R)

+   `CROSS` (R)

+   `CUBE` (R); 在 8.0.1 中变为保留

+   `CUME_DIST` (R); 在 8.0.2 中添加（保留）

+   `CURRENT`

+   `CURRENT_DATE` (R)

+   `CURRENT_TIME` (R)

+   `CURRENT_TIMESTAMP` (R)

+   `CURRENT_USER` (R)

+   `CURSOR` (R)

+   `CURSOR_NAME`

D

+   `DATA`

+   `DATABASE` (R)

+   `DATABASES` (R)

+   `DATAFILE`

+   `DATE`

+   `DATETIME`

+   `DAY`

+   `DAY_HOUR` (R)

+   `DAY_MICROSECOND` (R)

+   `DAY_MINUTE` (R)

+   `DAY_SECOND` (R)

+   `DEALLOCATE`

+   `DEC` (R)

+   `DECIMAL` (R)

+   `DECLARE` (R)

+   `DEFAULT` (R)

+   `DEFAULT_AUTH`

+   `DEFINER`

+   `DEFINITION`; 在 8.0.4 中添加（非保留）

+   `DELAYED` (R)

+   `DELAY_KEY_WRITE`

+   `DELETE` (R)

+   `DENSE_RANK` (R); 在 8.0.2 中添加（保留）

+   `DESC` (R)

+   `DESCRIBE` (R)

+   `DESCRIPTION`; 在 8.0.4 中添加（非保留）

+   `DES_KEY_FILE`; 在 8.0.3 中移除

+   `DETERMINISTIC` (R)

+   `DIAGNOSTICS`

+   `DIRECTORY`

+   `DISABLE`

+   `DISCARD`

+   `DISK`

+   `DISTINCT` (R)

+   `DISTINCTROW` (R)

+   `DIV` (R)

+   `DO`

+   `DOUBLE` (R)

+   `DROP` (R)

+   `DUAL` (R)

+   `DUMPFILE`

+   `DUPLICATE`

+   `DYNAMIC`

E

+   `EACH` (R)

+   `ELSE` (R)

+   `ELSEIF` (R)

+   `EMPTY` (R); 在 8.0.4 中添加（保留）

+   `ENABLE`

+   `ENCLOSED` (R)

+   `ENCRYPTION`

+   `END`

+   `ENDS`

+   `ENFORCED`; 在 8.0.16 中添加（非保留）

+   `ENGINE`

+   `ENGINES`

+   `ENGINE_ATTRIBUTE`; 在 8.0.21 中添加（非保留）

+   `ENUM`

+   `ERROR`

+   `ERRORS`

+   `ESCAPE`

+   `ESCAPED` (R)

+   `EVENT`

+   `EVENTS`

+   `EVERY`

+   `EXCEPT` (R)

+   `EXCHANGE`

+   `EXCLUDE`; 在 8.0.2 中添加（非保留）

+   `EXECUTE`

+   `EXISTS` (R)

+   `EXIT` (R)

+   `EXPANSION`

+   `EXPIRE`

+   `EXPLAIN` (R)

+   `EXPORT`

+   `EXTENDED`

+   `EXTENT_SIZE`

F

+   `FACTOR`; 在 8.0.27 中添加（非保留）

+   `FAILED_LOGIN_ATTEMPTS`; 在 8.0.19 中添加（非保留）

+   `FALSE` (R)

+   `FAST`

+   `FAULTS`

+   `FETCH` (R)

+   `FIELDS`

+   `FILE`

+   `FILE_BLOCK_SIZE`

+   `FILTER`

+   `FINISH`; 在 8.0.27 中添加（非保留）

+   `FIRST`

+   `FIRST_VALUE` (R); 在 8.0.2 中添加（保留）

+   `FIXED`

+   `FLOAT` (R)

+   `FLOAT4` (R)

+   `FLOAT8` (R)

+   `FLUSH`

+   `FOLLOWING`; 在 8.0.2 中添加（非保留）

+   `FOLLOWS`

+   `FOR` (R)

+   `FORCE` (R)

+   `FOREIGN` (R)

+   `FORMAT`

+   `FOUND`

+   `FROM` (R)

+   `FULL`

+   `FULLTEXT` (R)

+   `FUNCTION` (R); 在 8.0.1 中变为保留

G

+   `GENERAL`

+   `GENERATE`; 在 8.0.32 中添加（非保留）

+   `GENERATED` (R)

+   `GEOMCOLLECTION`; 在 8.0.11 中添加（非保留）

+   `GEOMETRY`

+   `GEOMETRYCOLLECTION`

+   `GET` (R)

+   `GET_FORMAT`

+   `GET_MASTER_PUBLIC_KEY`; 在 8.0.4 中添加（保留）；在 8.0.11 中变为非保留

+   `GET_SOURCE_PUBLIC_KEY`; 在 8.0.23 中添加（非保留）

+   `GLOBAL`

+   `GRANT` (R)

+   `GRANTS`

+   `GROUP` (R)

+   `GROUPING` (R); 在 8.0.1 中添加（保留）

+   `GROUPS` (R); 在 8.0.2 中添加（保留）

+   `GROUP_REPLICATION`

+   `GTID_ONLY`; 在 8.0.27 中添加（非保留）

H

+   `HANDLER`

+   `HASH`

+   `HAVING` (R)

+   `HELP`

+   `HIGH_PRIORITY` (R)

+   `HISTOGRAM`; 在 8.0.2 中添加（非保留）

+   `HISTORY`; 在 8.0.3 中添加（非保留）

+   `HOST`

+   `HOSTS`

+   `HOUR`

+   `HOUR_MICROSECOND` (R)

+   `HOUR_MINUTE` (R)

+   `HOUR_SECOND` (R)

I

+   `IDENTIFIED`

+   `IF` (R)

+   `IGNORE` (R)

+   `IGNORE_SERVER_IDS`

+   `IMPORT`

+   `IN` (R)

+   `INACTIVE`; 在 8.0.14 中添加（非保留）

+   `INDEX` (R)

+   `INDEXES`

+   `INFILE` (R)

+   `INITIAL`; 在 8.0.27 中添加（非保留）

+   `INITIAL_SIZE`

+   `INITIATE`; 在 8.0.27 中添加（非保留）

+   `INNER` (R)

+   `INOUT` (R)

+   `INSENSITIVE` (R)

+   `INSERT` (R)

+   `INSERT_METHOD`

+   `INSTALL`

+   `INSTANCE`

+   `INT` (R)

+   `INT1` (R)

+   `INT2` (R)

+   `INT3` (R)

+   `INT4` (R)

+   `INT8` (R)

+   `INTEGER` (R)

+   `INTERSECT` (R); added in 8.0.31 (reserved)

+   `INTERVAL` (R)

+   `INTO` (R)

+   `INVISIBLE`

+   `INVOKER`

+   `IO`

+   `IO_AFTER_GTIDS` (R)

+   `IO_BEFORE_GTIDS` (R)

+   `IO_THREAD`

+   `IPC`

+   `IS` (R)

+   `ISOLATION`

+   `ISSUER`

+   `ITERATE` (R)

J

+   `JOIN` (R)

+   `JSON`

+   `JSON_TABLE` (R); added in 8.0.4 (reserved)

+   `JSON_VALUE`; added in 8.0.21 (nonreserved)

K

+   `KEY` (R)

+   `KEYRING`; added in 8.0.24 (nonreserved)

+   `KEYS` (R)

+   `KEY_BLOCK_SIZE`

+   `KILL` (R)

L

+   `LAG` (R); added in 8.0.2 (reserved)

+   `LANGUAGE`

+   `LAST`

+   `LAST_VALUE` (R); added in 8.0.2 (reserved)

+   `LATERAL` (R); added in 8.0.14 (reserved)

+   `LEAD` (R); added in 8.0.2 (reserved)

+   `LEADING` (R)

+   `LEAVE` (R)

+   `LEAVES`

+   `LEFT` (R)

+   `LESS`

+   `LEVEL`

+   `LIKE` (R)

+   `LIMIT` (R)

+   `LINEAR` (R)

+   `LINES` (R)

+   `LINESTRING`

+   `LIST`

+   `LOAD` (R)

+   `LOCAL`

+   `LOCALTIME` (R)

+   `LOCALTIMESTAMP` (R)

+   `LOCK` (R)

+   `LOCKED`; added in 8.0.1 (nonreserved)

+   `LOCKS`

+   `LOGFILE`

+   `LOGS`

+   `LONG` (R)

+   `LONGBLOB` (R)

+   `LONGTEXT` (R)

+   `LOOP` (R)

+   `LOW_PRIORITY` (R)

M

+   `MASTER`

+   `MASTER_AUTO_POSITION`

+   `MASTER_BIND` (R)

+   `MASTER_COMPRESSION_ALGORITHMS`; added in 8.0.18 (nonreserved)

+   `MASTER_CONNECT_RETRY`

+   `MASTER_DELAY`

+   `MASTER_HEARTBEAT_PERIOD`

+   `MASTER_HOST`

+   `MASTER_LOG_FILE`

+   `MASTER_LOG_POS`

+   `MASTER_PASSWORD`

+   `MASTER_PORT`

+   `MASTER_PUBLIC_KEY_PATH`; added in 8.0.4 (nonreserved)

+   `MASTER_RETRY_COUNT`

+   `MASTER_SERVER_ID`; removed in 8.0.23

+   `MASTER_SSL`

+   `MASTER_SSL_CA`

+   `MASTER_SSL_CAPATH`

+   `MASTER_SSL_CERT`

+   `MASTER_SSL_CIPHER`

+   `MASTER_SSL_CRL`

+   `MASTER_SSL_CRLPATH`

+   `MASTER_SSL_KEY`

+   `MASTER_SSL_VERIFY_SERVER_CERT` (R)

+   `MASTER_TLS_CIPHERSUITES`; added in 8.0.19 (nonreserved)

+   `MASTER_TLS_VERSION`

+   `MASTER_USER`

+   `MASTER_ZSTD_COMPRESSION_LEVEL`; added in 8.0.18 (nonreserved)

+   `MATCH` (R)

+   `MAXVALUE` (R)

+   `MAX_CONNECTIONS_PER_HOUR`

+   `MAX_QUERIES_PER_HOUR`

+   `MAX_ROWS`

+   `MAX_SIZE`

+   `MAX_UPDATES_PER_HOUR`

+   `MAX_USER_CONNECTIONS`

+   `MEDIUM`

+   `MEDIUMBLOB` (R)

+   `MEDIUMINT` (R)

+   `MEDIUMTEXT` (R)

+   `MEMBER`; added in 8.0.17 (reserved); became nonreserved in 8.0.19

+   `MEMORY`

+   `MERGE`

+   `MESSAGE_TEXT`

+   `MICROSECOND`

+   `MIDDLEINT` (R)

+   `MIGRATE`

+   `MINUTE`

+   `MINUTE_MICROSECOND` (R)

+   `MINUTE_SECOND` (R)

+   `MIN_ROWS`

+   `MOD` (R)

+   `MODE`

+   `MODIFIES` (R)

+   `MODIFY`

+   `MONTH`

+   `MULTILINESTRING`

+   `MULTIPOINT`

+   `MULTIPOLYGON`

+   `MUTEX`

+   `MYSQL_ERRNO`

N

+   `NAME`

+   `NAMES`

+   `NATIONAL`

+   `NATURAL` (R)

+   `NCHAR`

+   `NDB`

+   `NDBCLUSTER`

+   `NESTED`; added in 8.0.4 (nonreserved)

+   `NETWORK_NAMESPACE`; added in 8.0.16 (nonreserved)

+   `NEVER`

+   `NEW`

+   `NEXT`

+   `NO`

+   `NODEGROUP`

+   `NONE`

+   `NOT` (R)

+   `NOWAIT`; added in 8.0.1 (nonreserved)

+   `NO_WAIT`

+   `NO_WRITE_TO_BINLOG` (R)

+   `NTH_VALUE` (R); added in 8.0.2 (reserved)

+   `NTILE` (R); added in 8.0.2 (reserved)

+   `NULL` (R)

+   `NULLS`; added in 8.0.2 (nonreserved)

+   `NUMBER`

+   `NUMERIC` (R)

+   `NVARCHAR`

O

+   `OF` (R); added in 8.0.1 (reserved)

+   `OFF`; added in 8.0.20 (nonreserved)

+   `OFFSET`

+   `OJ`; added in 8.0.16 (nonreserved)

+   `OLD`; added in 8.0.14 (nonreserved)

+   `ON` (R)

+   `ONE`

+   `ONLY`

+   `OPEN`

+   `OPTIMIZE` (R)

+   `OPTIMIZER_COSTS` (R)

+   `OPTION` (R)

+   `OPTIONAL`; added in 8.0.13 (nonreserved)

+   `OPTIONALLY` (R)

+   `OPTIONS`

+   `OR` (R)

+   `ORDER` (R)

+   `ORDINALITY`; added in 8.0.4 (nonreserved)

+   `ORGANIZATION`; 在 8.0.4 版本中添加（非保留）

+   `OTHERS`; 在 8.0.2 版本中添加（非保留）

+   `OUT` (R)

+   `OUTER` (R)

+   `OUTFILE` (R)

+   `OVER` (R); 在 8.0.2 版本中添加（保留）

+   `OWNER`

P

+   `PACK_KEYS`

+   `PAGE`

+   `PARSER`

+   `PARTIAL`

+   `PARTITION` (R)

+   `PARTITIONING`

+   `PARTITIONS`

+   `PASSWORD`

+   `PASSWORD_LOCK_TIME`; 在 8.0.19 版本中添加（非保留）

+   `PATH`; 在 8.0.4 版本中添加（非保留）

+   `PERCENT_RANK` (R); 在 8.0.2 版本中添加（保留）

+   `PERSIST`; 在 8.0.16 版本中变为非保留

+   `PERSIST_ONLY`; 在 8.0.2 版本中添加（保留）；在 8.0.16 版本中变为非保留

+   `PHASE`

+   `PLUGIN`

+   `PLUGINS`

+   `PLUGIN_DIR`

+   `POINT`

+   `POLYGON`

+   `PORT`

+   `PRECEDES`

+   `PRECEDING`; 在 8.0.2 版本中添加（非保留）

+   `PRECISION` (R)

+   `PREPARE`

+   `PRESERVE`

+   `PREV`

+   `PRIMARY` (R)

+   `PRIVILEGES`

+   `PRIVILEGE_CHECKS_USER`; 在 8.0.18 版本中添加（非保留）

+   `PROCEDURE` (R)

+   `PROCESS`; 在 8.0.11 版本中添加（非保留）

+   `PROCESSLIST`

+   `PROFILE`

+   `PROFILES`

+   `PROXY`

+   `PURGE` (R)

Q

+   `QUARTER`

+   `QUERY`

+   `QUICK`

R

+   `RANDOM`; 在 8.0.18 版本中添加（非保留）

+   `RANGE` (R)

+   `RANK` (R); 在 8.0.2 版本中添加（保留）

+   `READ` (R)

+   `READS` (R)

+   `READ_ONLY`

+   `READ_WRITE` (R)

+   `REAL` (R)

+   `REBUILD`

+   `RECOVER`

+   `RECURSIVE` (R); 在 8.0.1 版本中添加（保留）

+   `REDOFILE`; 在 8.0.3 版本中移除

+   `REDO_BUFFER_SIZE`

+   `REDUNDANT`

+   `REFERENCE`; 在 8.0.4 版本中添加（非保留）

+   `REFERENCES` (R)

+   `REGEXP` (R)

+   `REGISTRATION`; 在 8.0.27 版本中添加（非保留）

+   `RELAY`

+   `RELAYLOG`

+   `RELAY_LOG_FILE`

+   `RELAY_LOG_POS`

+   `RELAY_THREAD`

+   `RELEASE` (R)

+   `RELOAD`

+   `REMOTE`; 在 8.0.3 版本中添加（非保留）；在 8.0.14 版本中移除

+   `REMOVE`

+   `RENAME` (R)

+   `REORGANIZE`

+   `REPAIR`

+   `REPEAT` (R)

+   `REPEATABLE`

+   `REPLACE` (R)

+   `REPLICA`; 在 8.0.22 版本中添加（非保留）

+   `REPLICAS`; 在 8.0.22 版本中添加（非保留）

+   `REPLICATE_DO_DB`

+   `REPLICATE_DO_TABLE`

+   `REPLICATE_IGNORE_DB`

+   `REPLICATE_IGNORE_TABLE`

+   `REPLICATE_REWRITE_DB`

+   `REPLICATE_WILD_DO_TABLE`

+   `REPLICATE_WILD_IGNORE_TABLE`

+   `REPLICATION`

+   `REQUIRE` (R)

+   `REQUIRE_ROW_FORMAT`; 在 8.0.19 版本中添加（非保留）

+   `RESET`

+   `RESIGNAL` (R)

+   `RESOURCE`; 在 8.0.3 版本中添加（非保留）

+   `RESPECT`; 在 8.0.2 版本中添加（非保留）

+   `RESTART`; 在 8.0.4 版本中添加（非保留）

+   `RESTORE`

+   `RESTRICT` (R)

+   `RESUME`

+   `RETAIN`; 在 8.0.14 版本中添加（非保留）

+   `RETURN` (R)

+   `RETURNED_SQLSTATE`

+   `RETURNING`; 在 8.0.21 版本中添加（非保留）

+   `RETURNS`

+   `REUSE`; 在 8.0.3 版本中添加（非保留）

+   `REVERSE`

+   `REVOKE` (R)

+   `RIGHT` (R)

+   `RLIKE` (R)

+   `ROLE`; 在 8.0.1 版本中变为非保留

+   `ROLLBACK`

+   `ROLLUP`

+   `ROTATE`

+   `ROUTINE`

+   `ROW` (R); 在 8.0.2 版本中变为保留

+   `ROWS` (R); 在 8.0.2 版本中变为保留

+   `ROW_COUNT`

+   `ROW_FORMAT`

+   `ROW_NUMBER` (R); 在 8.0.2 版本中添加（保留）

+   `RTREE`

S

+   `SAVEPOINT`

+   `SCHEDULE`

+   `SCHEMA` (R)

+   `SCHEMAS` (R)

+   `SCHEMA_NAME`

+   `SECOND`

+   `SECONDARY`; 在 8.0.16 版本中添加（非保留）

+   `SECONDARY_ENGINE`; 在 8.0.13 版本中添加（非保留）

+   `SECONDARY_ENGINE_ATTRIBUTE`; 在 8.0.21 版本中添加（非保留）

+   `SECONDARY_LOAD`; 在 8.0.13 版本中添加（非保留）

+   `SECONDARY_UNLOAD`; 在 8.0.13 版本中添加（非保留）

+   `SECOND_MICROSECOND` (R)

+   `SECURITY`

+   `SELECT` (R)

+   `SENSITIVE` (R)

+   `SEPARATOR` (R)

+   `SERIAL`

+   `SERIALIZABLE`

+   `SERVER`

+   `SESSION`

+   `SET` (R)

+   `SHARE`

+   `SHOW` (R)

+   `SHUTDOWN`

+   `SIGNAL` (R)

+   `SIGNED`

+   `SIMPLE`

+   `SKIP`; 在 8.0.1 版本中添加（非保留）

+   `SLAVE`

+   `SLOW`

+   `SMALLINT`（R）

+   `SNAPSHOT`

+   `SOCKET`

+   `SOME`

+   `SONAME`

+   `SOUNDS`

+   `SOURCE`

+   `SOURCE_AUTO_POSITION`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_BIND`; 在 8.0.23 版本中添加（��保留）

+   `SOURCE_COMPRESSION_ALGORITHMS`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_CONNECT_RETRY`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_DELAY`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_HEARTBEAT_PERIOD`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_HOST`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_LOG_FILE`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_LOG_POS`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_PASSWORD`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_PORT`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_PUBLIC_KEY_PATH`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_RETRY_COUNT`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_CA`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_CAPATH`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_CERT`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_CIPHER`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_CRL`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_CRLPATH`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_KEY`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_SSL_VERIFY_SERVER_CERT`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_TLS_CIPHERSUITES`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_TLS_VERSION`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_USER`; 在 8.0.23 版本中添加（非保留）

+   `SOURCE_ZSTD_COMPRESSION_LEVEL`; 在 8.0.23 版本中添加（非保留）

+   `SPATIAL`（R）

+   `SPECIFIC`（R）

+   `SQL`（R）

+   `SQLEXCEPTION`（R）

+   `SQLSTATE`（R）

+   `SQLWARNING`（R）

+   `SQL_AFTER_GTIDS`

+   `SQL_AFTER_MTS_GAPS`

+   `SQL_BEFORE_GTIDS`

+   `SQL_BIG_RESULT`（R）

+   `SQL_BUFFER_RESULT`

+   `SQL_CACHE`; 在 8.0.3 版本中移除

+   `SQL_CALC_FOUND_ROWS`（R）

+   `SQL_NO_CACHE`

+   `SQL_SMALL_RESULT`（R）

+   `SQL_THREAD`

+   `SQL_TSI_DAY`

+   `SQL_TSI_HOUR`

+   `SQL_TSI_MINUTE`

+   `SQL_TSI_MONTH`

+   `SQL_TSI_QUARTER`

+   `SQL_TSI_SECOND`

+   `SQL_TSI_WEEK`

+   `SQL_TSI_YEAR`

+   `SRID`; 在 8.0.3 版本中添加（非保留）

+   `SSL`（R）

+   `STACKED`

+   `START`

+   `STARTING`（R）

+   `STARTS`

+   `STATS_AUTO_RECALC`

+   `STATS_PERSISTENT`

+   `STATS_SAMPLE_PAGES`

+   `STATUS`

+   `STOP`

+   `STORAGE`

+   `STORED`（R）

+   `STRAIGHT_JOIN`（R）

+   `STREAM`; 在 8.0.20 版本中添加（非保留）

+   `STRING`

+   `SUBCLASS_ORIGIN`

+   `SUBJECT`

+   `SUBPARTITION`

+   `SUBPARTITIONS`

+   `SUPER`

+   `SUSPEND`

+   `SWAPS`

+   `SWITCHES`

+   `SYSTEM`（R）; 在 8.0.3 版本中添加（保留）

T

+   `TABLE`（R）

+   `TABLES`

+   `TABLESPACE`

+   `TABLE_CHECKSUM`

+   `TABLE_NAME`

+   `TEMPORARY`

+   `TEMPTABLE`

+   `TERMINATED`（R）

+   `TEXT`

+   `THAN`

+   `THEN`（R）

+   `THREAD_PRIORITY`; 在 8.0.3 版本中添加（非保留）

+   `TIES`; 在 8.0.2 版本中添加（非保留）

+   `TIME`

+   `TIMESTAMP`

+   `TIMESTAMPADD`

+   `TIMESTAMPDIFF`

+   `TINYBLOB`（R）

+   `TINYINT`（R）

+   `TINYTEXT`（R）

+   `TLS`; 在 8.0.21 版本中添加（非保留）

+   `TO`（R）

+   `TRAILING`（R）

+   `TRANSACTION`

+   `TRIGGER`（R）

+   `TRIGGERS`

+   `TRUE`（R）

+   `TRUNCATE`

+   `TYPE`

+   `TYPES`

U

+   `UNBOUNDED`; 在 8.0.2 版本中添加（非保留）

+   `UNCOMMITTED`

+   `UNDEFINED`

+   `UNDO`（R）

+   `UNDOFILE`

+   `UNDO_BUFFER_SIZE`

+   `UNICODE`

+   `UNINSTALL`

+   `UNION`（R）

+   `UNIQUE`（R）

+   `UNKNOWN`

+   `UNLOCK`（R）

+   `UNREGISTER`; 在 8.0.27 版本中添加（非保留）

+   `UNSIGNED`（R）

+   `UNTIL`

+   `UPDATE`（R）

+   `UPGRADE`

+   `URL`; added in 8.0.32 (nonreserved)

+   `USAGE` (R)

+   `USE` (R)

+   `USER`

+   `USER_RESOURCES`

+   `USE_FRM`

+   `USING` (R)

+   `UTC_DATE` (R)

+   `UTC_TIME` (R)

+   `UTC_TIMESTAMP` (R)

V

+   `VALIDATION`

+   `VALUE`

+   `VALUES` (R)

+   `VARBINARY` (R)

+   `VARCHAR` (R)

+   `VARCHARACTER` (R)

+   `VARIABLES`

+   `VARYING` (R)

+   `VCPU`; added in 8.0.3 (nonreserved)

+   `VIEW`

+   `VIRTUAL` (R)

+   `VISIBLE`

W

+   `WAIT`

+   `WARNINGS`

+   `WEEK`

+   `WEIGHT_STRING`

+   `WHEN` (R)

+   `WHERE` (R)

+   `WHILE` (R)

+   `WINDOW` (R); added in 8.0.2 (reserved)

+   `WITH` (R)

+   `WITHOUT`

+   `WORK`

+   `WRAPPER`

+   `WRITE` (R)

X

+   `X509`

+   `XA`

+   `XID`

+   `XML`

+   `XOR` (R)

Y

+   `YEAR`

+   `YEAR_MONTH` (R)

Z

+   `ZEROFILL` (R)

+   `ZONE`; added in 8.0.22 (nonreserved)

### MySQL 8.0 新关键字和保留字

以下列表显示了与 MySQL 5.7 相比，在 MySQL 8.0 中新增的关键字和保留字。保留关键字标记为(R)。

A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | R | S | T | U | V | W | Z

A

+   `ACTIVE`

+   `ADMIN`

+   `ARRAY`

+   `ATTRIBUTE`

+   `AUTHENTICATION`

B

+   `BUCKETS`

+   `BULK`

C

+   `CHALLENGE_RESPONSE`

+   `CLONE`

+   `COMPONENT`

+   `CUME_DIST` (R)

D

+   `DEFINITION`

+   `DENSE_RANK` (R)

+   `DESCRIPTION`

E

+   `EMPTY` (R)

+   `ENFORCED`

+   `ENGINE_ATTRIBUTE`

+   `EXCEPT` (R)

+   `EXCLUDE`

F

+   `FACTOR`

+   `FAILED_LOGIN_ATTEMPTS`

+   `FINISH`

+   `FIRST_VALUE` (R)

+   `FOLLOWING`

G

+   `GENERATE`

+   `GEOMCOLLECTION`

+   `GET_MASTER_PUBLIC_KEY`

+   `GET_SOURCE_PUBLIC_KEY`

+   `GROUPING` (R)

+   `GROUPS` (R)

+   `GTID_ONLY`

H

+   `HISTOGRAM`

+   `HISTORY`

I

+   `INACTIVE`

+   `INITIAL`

+   `INITIATE`

+   `INTERSECT` (R)

+   `INVISIBLE`

J

+   `JSON_TABLE` (R)

+   `JSON_VALUE`

K

+   `KEYRING`

L

+   `LAG` (R)

+   `LAST_VALUE` (R)

+   `LATERAL` (R)

+   `LEAD` (R)

+   `LOCKED`

M

+   `MASTER_COMPRESSION_ALGORITHMS`

+   `MASTER_PUBLIC_KEY_PATH`

+   `MASTER_TLS_CIPHERSUITES`

+   `MASTER_ZSTD_COMPRESSION_LEVEL`

+   `MEMBER`

N

+   `NESTED`

+   `NETWORK_NAMESPACE`

+   `NOWAIT`

+   `NTH_VALUE` (R)

+   `NTILE` (R)

+   `NULLS`

O

+   `OF` (R)

+   `OFF`

+   `OJ`

+   `OLD`

+   `OPTIONAL`

+   `ORDINALITY`

+   `ORGANIZATION`

+   `OTHERS`

+   `OVER` (R)

P

+   `PASSWORD_LOCK_TIME`

+   `PATH`

+   `PERCENT_RANK` (R)

+   `PERSIST`

+   `PERSIST_ONLY`

+   `PRECEDING`

+   `PRIVILEGE_CHECKS_USER`

+   `PROCESS`

R

+   `RANDOM`

+   `RANK` (R)

+   `RECURSIVE` (R)

+   `REFERENCE`

+   `REGISTRATION`

+   `REPLICA`

+   `REPLICAS`

+   `REQUIRE_ROW_FORMAT`

+   `RESOURCE`

+   `RESPECT`

+   `RESTART`

+   `RETAIN`

+   `RETURNING`

+   `REUSE`

+   `ROLE`

+   `ROW_NUMBER` (R)

S

+   `SECONDARY`

+   `SECONDARY_ENGINE`

+   `SECONDARY_ENGINE_ATTRIBUTE`

+   `SECONDARY_LOAD`

+   `SECONDARY_UNLOAD`

+   `SKIP`

+   `SOURCE_AUTO_POSITION`

+   `SOURCE_BIND`

+   `SOURCE_COMPRESSION_ALGORITHMS`

+   `SOURCE_CONNECT_RETRY`

+   `SOURCE_DELAY`

+   `SOURCE_HEARTBEAT_PERIOD`

+   `SOURCE_HOST`

+   `SOURCE_LOG_FILE`

+   `SOURCE_LOG_POS`

+   `SOURCE_PASSWORD`

+   `SOURCE_PORT`

+   `SOURCE_PUBLIC_KEY_PATH`

+   `SOURCE_RETRY_COUNT`

+   `SOURCE_SSL`

+   `SOURCE_SSL_CA`

+   `SOURCE_SSL_CAPATH`

+   `SOURCE_SSL_CERT`

+   `SOURCE_SSL_CIPHER`

+   `SOURCE_SSL_CRL`

+   `SOURCE_SSL_CRLPATH`

+   `SOURCE_SSL_KEY`

+   `SOURCE_SSL_VERIFY_SERVER_CERT`

+   `SOURCE_TLS_CIPHERSUITES`

+   `SOURCE_TLS_VERSION`

+   `SOURCE_USER`

+   `SOURCE_ZSTD_COMPRESSION_LEVEL`

+   `SRID`

+   `STREAM`

+   `SYSTEM` (R)

T

+   `THREAD_PRIORITY`

+   `TIES`

+   `TLS`

U

+   `UNBOUNDED`

+   `UNREGISTER`

+   `URL`

V

+   `VCPU`

+   `VISIBLE`

W

+   `WINDOW` (R)

Z

+   `ZONE`

### MySQL 8.0 Removed Keywords and Reserved Words

以下列表显示了与 MySQL 5.7 相比，在 MySQL 8.0 中被移除的关键字和保留字。保留关键字标记为(R)。

+   `ANALYSE`

+   `DES_KEY_FILE`

+   `MASTER_SERVER_ID`

+   `PARSE_GCOL_EXPR`

+   `REDOFILE`

+   `SQL_CACHE`
