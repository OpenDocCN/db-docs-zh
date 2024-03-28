> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-row-events.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-row-events.html)

#### 6.6.9.2 mysqlbinlog 行事件显示

以下示例说明了**mysqlbinlog**如何显示指定数据修改的行事件。这些对应于具有`WRITE_ROWS_EVENT`、`UPDATE_ROWS_EVENT`和`DELETE_ROWS_EVENT`类型代码的事件。可以使用`--base64-output=DECODE-ROWS`和`--verbose`选项来影响行事件输出。

假设服务器正在使用基于行的二进制日志记录，并且您执行以下语句序列：

```sql
CREATE TABLE t
(
  id   INT NOT NULL,
  name VARCHAR(20) NOT NULL,
  date DATE NULL
) ENGINE = InnoDB;

START TRANSACTION;
INSERT INTO t VALUES(1, 'apple', NULL);
UPDATE t SET name = 'pear', date = '2009-01-01' WHERE id = 1;
DELETE FROM t WHERE id = 1;
COMMIT;
```

默认情况下，**mysqlbinlog**将行事件显示为使用`BINLOG`语句编码的 base-64 字符串。省略多余的行，由前述语句序列产生的行事件的输出如下所示：

```sql
$> mysqlbinlog *log_file*
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAANoAAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBcBAAAAKAAAAAIBAAAQABEAAAAAAAEAA//8AQAAAAVhcHBsZQ==
'/*!*/;
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAC4BAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBgBAAAANgAAAGQBAAAQABEAAAAAAAEAA////AEAAAAFYXBwbGX4AQAAAARwZWFyIbIP
'/*!*/;
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAJABAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBkBAAAAKgAAALoBAAAQABEAAAAAAAEAA//4AQAAAARwZWFyIbIP
'/*!*/;
```

要将行事件显示为“伪 SQL”语句形式的注释，请使用`--verbose`或`-v`选项运行**mysqlbinlog**。此输出级别还在适用时显示表分区信息。输出包含以`###`开头的行：

```sql
$> mysqlbinlog -v *log_file*
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAANoAAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBcBAAAAKAAAAAIBAAAQABEAAAAAAAEAA//8AQAAAAVhcHBsZQ==
'/*!*/;
### INSERT INTO test.t
### SET
###   @1=1
###   @2='apple'
###   @3=NULL
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAC4BAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBgBAAAANgAAAGQBAAAQABEAAAAAAAEAA////AEAAAAFYXBwbGX4AQAAAARwZWFyIbIP
'/*!*/;
### UPDATE test.t
### WHERE
###   @1=1
###   @2='apple'
###   @3=NULL
### SET
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAJABAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBkBAAAAKgAAALoBAAAQABEAAAAAAAEAA//4AQAAAARwZWFyIbIP
'/*!*/;
### DELETE FROM test.t
### WHERE
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
```

指定`--verbose`或`-v`两次，还会显示每列的数据类型和一些元数据，以及信息日志事件，如`binlog_rows_query_log_events`系统变量设置为`TRUE`时的行查询日志事件。输出包含每个列更改后的额外注释：

```sql
$> mysqlbinlog -vv *log_file*
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAANoAAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBcBAAAAKAAAAAIBAAAQABEAAAAAAAEAA//8AQAAAAVhcHBsZQ==
'/*!*/;
### INSERT INTO test.t
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='apple' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3=NULL /* VARSTRING(20) meta=0 nullable=1 is_null=1 */
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAC4BAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBgBAAAANgAAAGQBAAAQABEAAAAAAAEAA////AEAAAAFYXBwbGX4AQAAAARwZWFyIbIP
'/*!*/;
### UPDATE test.t
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='apple' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3=NULL /* VARSTRING(20) meta=0 nullable=1 is_null=1 */
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='pear' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3='2009:01:01' /* DATE meta=0 nullable=1 is_null=0 */
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAJABAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBkBAAAAKgAAALoBAAAQABEAAAAAAAEAA//4AQAAAARwZWFyIbIP
'/*!*/;
### DELETE FROM test.t
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='pear' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3='2009:01:01' /* DATE meta=0 nullable=1 is_null=0 */
```

您可以告诉**mysqlbinlog**通过使用`--base64-output=DECODE-ROWS`选项来抑制行事件的`BINLOG`语句。这类似于`--base64-output=NEVER`，但如果找到行事件，则不会退出错误。`--base64-output=DECODE-ROWS`和`--verbose`的组合提供了一个方便的方式，只将行事件显示为 SQL 语句：

```sql
$> mysqlbinlog -v --base64-output=DECODE-ROWS *log_file*
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F
### INSERT INTO test.t
### SET
###   @1=1
###   @2='apple'
###   @3=NULL
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F
### UPDATE test.t
### WHERE
###   @1=1
###   @2='apple'
###   @3=NULL
### SET
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F
### DELETE FROM test.t
### WHERE
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
```

注意

如果您打算重新执行**mysqlbinlog**输出，则不应该抑制`BINLOG`语句。

通过`--verbose`生成的行事件的 SQL 语句比相应的`BINLOG`语句更易读。然而，它们并不完全对应生成事件的原始 SQL 语句。以下限制适用：

+   原始列名丢失，被`@*`N`*`替换，其中*`N`*是列号。

+   二进制日志中不包含字符集信息，这会影响字符串列的显示：

    +   不区分对应的二进制和非二进制字符串类型（`BINARY`和`CHAR`，`VARBINARY`和`VARCHAR`，`BLOB`和`TEXT`）。输出使用`STRING`作为定长字符串的数据类型，使用`VARSTRING`作为可变长度字符串的数据类型。

    +   对于多字节字符集，二进制日志中不包含每个字符的最大字节数，因此字符串类型的长度以字节而不是字符显示。例如，`STRING(4)`用作以下任一列类型的值的数据类型：

        ```sql
        CHAR(4) CHARACTER SET latin1
        CHAR(2) CHARACTER SET ucs2
        ```

    +   由于`UPDATE_ROWS_EVENT`类型事件的存储格式，`UPDATE`语句显示`WHERE`子句在`SET`子句之前。

正确解释行事件需要来自二进制日志开头的格式描述事件的信息。因为**mysqlbinlog**事先不知道日志的其余部分是否包含行事件，所以默认情况下，它在输出的初始部分使用`BINLOG`语句显示格式描述事件。

如果已知二进制日志不包含任何需要`BINLOG`语句（即没有行事件）的事件，则可以使用`--base64-output=NEVER`选项防止写入此标题。
