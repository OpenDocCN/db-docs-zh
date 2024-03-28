> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-file-formats.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-file-formats.html)

#### 8.4.5.4 审计日志文件格式

当发生可审计事件时，MySQL 服务器调用审计日志插件将审计记录写入其日志文件。通常，在插件启动后写入的第一条审计记录包含服务器描述和启动选项。随后的元素代表诸如客户端连接和断开事件、执行的 SQL 语句等事件。只记录顶层语句，不记录触发器或存储过程等存储程序中的语句。例如，`LOAD DATA` 中引用的文件内容不会被记录。

要选择审计日志插件用于编写其日志文件的日志格式，请在服务器启动时设置 `audit_log_format` 系统变量。这些格式可用：

+   新式 XML 格式（`audit_log_format=NEW`）：一种与 Oracle Audit Vault 更兼容的 XML 格式，MySQL 8.0 默认使用新式 XML 格式。

+   旧式 XML 格式（`audit_log_format=OLD`）：默认在旧版 MySQL 系列中使用的原始审计日志格式。

+   JSON 格式（`audit_log_format=JSON`）：将审计日志写入 JSON 数组。只有此格式支持从 MySQL 8.0.30 开始提供的可选查询时间和大小统计信息。

默认情况下，审计日志文件内容以新式 XML 格式编写，不进行压缩或加密。

如果更改 `audit_log_format`，建议同时更改 `audit_log_file`。例如，如果将 `audit_log_format` 设置为 `JSON`，则将 `audit_log_file` 设置为 `audit.json`。否则，新的日志文件将具有不同的格式，但它们都将具有相同的基本名称，没有任何指示格式何时更改的内容。

+   新式 XML 审计日志文件格式

+   旧式 XML 审计日志文件格式

+   JSON 审计日志文件格式

##### 新式 XML 审计日志文件格式

这是一个以新式 XML 格式（`audit_log_format=NEW`）编写的示例日志文件，为了可读性稍作重新格式化：

```sql
<?xml version="1.0" encoding="utf-8"?>
<AUDIT>
 <AUDIT_RECORD>
  <TIMESTAMP>2019-10-03T14:06:33 UTC</TIMESTAMP>
  <RECORD_ID>1_2019-10-03T14:06:33</RECORD_ID>
  <NAME>Audit</NAME>
  <SERVER_ID>1</SERVER_ID>
  <VERSION>1</VERSION>
  <STARTUP_OPTIONS>/usr/local/mysql/bin/mysqld
    --socket=/usr/local/mysql/mysql.sock
    --port=3306</STARTUP_OPTIONS>
  <OS_VERSION>i686-Linux</OS_VERSION>
  <MYSQL_VERSION>5.7.21-log</MYSQL_VERSION>
 </AUDIT_RECORD>
 <AUDIT_RECORD>
  <TIMESTAMP>2019-10-03T14:09:38 UTC</TIMESTAMP>
  <RECORD_ID>2_2019-10-03T14:06:33</RECORD_ID>
  <NAME>Connect</NAME>
  <CONNECTION_ID>5</CONNECTION_ID>
  <STATUS>0</STATUS>
  <STATUS_CODE>0</STATUS_CODE>
  <USER>root</USER>
  <OS_LOGIN/>
  <HOST>localhost</HOST>
  <IP>127.0.0.1</IP>
  <COMMAND_CLASS>connect</COMMAND_CLASS>
  <CONNECTION_TYPE>SSL/TLS</CONNECTION_TYPE>
  <CONNECTION_ATTRIBUTES>
   <ATTRIBUTE>
    <NAME>_pid</NAME>
    <VALUE>42794</VALUE>
   </ATTRIBUTE>
   ...
   <ATTRIBUTE>
    <NAME>program_name</NAME>
    <VALUE>mysqladmin</VALUE>
   </ATTRIBUTE>
  </CONNECTION_ATTRIBUTES>
  <PRIV_USER>root</PRIV_USER>
  <PROXY_USER/>
  <DB>test</DB>
 </AUDIT_RECORD>

...

 <AUDIT_RECORD>
  <TIMESTAMP>2019-10-03T14:09:38 UTC</TIMESTAMP>
  <RECORD_ID>6_2019-10-03T14:06:33</RECORD_ID>
  <NAME>Query</NAME>
  <CONNECTION_ID>5</CONNECTION_ID>
  <STATUS>0</STATUS>
  <STATUS_CODE>0</STATUS_CODE>
  <USER>root[root] @ localhost [127.0.0.1]</USER>
  <OS_LOGIN/>
  <HOST>localhost</HOST>
  <IP>127.0.0.1</IP>
  <COMMAND_CLASS>drop_table</COMMAND_CLASS>
  <SQLTEXT>DROP TABLE IF EXISTS t</SQLTEXT>
 </AUDIT_RECORD>

...

 <AUDIT_RECORD>
  <TIMESTAMP>2019-10-03T14:09:39 UTC</TIMESTAMP>
  <RECORD_ID>8_2019-10-03T14:06:33</RECORD_ID>
  <NAME>Quit</NAME>
  <CONNECTION_ID>5</CONNECTION_ID>
  <STATUS>0</STATUS>
  <STATUS_CODE>0</STATUS_CODE>
  <USER>root</USER>
  <OS_LOGIN/>
  <HOST>localhost</HOST>
  <IP>127.0.0.1</IP>
  <COMMAND_CLASS>connect</COMMAND_CLASS>
  <CONNECTION_TYPE>SSL/TLS</CONNECTION_TYPE>
 </AUDIT_RECORD>

...

 <AUDIT_RECORD>
  <TIMESTAMP>2019-10-03T14:09:43 UTC</TIMESTAMP>
  <RECORD_ID>11_2019-10-03T14:06:33</RECORD_ID>
  <NAME>Quit</NAME>
  <CONNECTION_ID>6</CONNECTION_ID>
  <STATUS>0</STATUS>
  <STATUS_CODE>0</STATUS_CODE>
  <USER>root</USER>
  <OS_LOGIN/>
  <HOST>localhost</HOST>
  <IP>127.0.0.1</IP>
  <COMMAND_CLASS>connect</COMMAND_CLASS>
  <CONNECTION_TYPE>SSL/TLS</CONNECTION_TYPE>
 </AUDIT_RECORD>
 <AUDIT_RECORD>
  <TIMESTAMP>2019-10-03T14:09:45 UTC</TIMESTAMP>
  <RECORD_ID>12_2019-10-03T14:06:33</RECORD_ID>
  <NAME>NoAudit</NAME>
  <SERVER_ID>1</SERVER_ID>
 </AUDIT_RECORD>
</AUDIT>
```

审计日志文件以 XML 格式编写，使用 UTF-8（每个字符最多 4 个字节）。根元素是`<AUDIT>`。根元素包含`<AUDIT_RECORD>`元素，每个元素提供有关审计事件的信息。当审计日志插件开始写入新的日志文件时，它会写入 XML 声明和打开的`<AUDIT>`根元素标记。当插件关闭日志文件时，它会写入关闭的`</AUDIT>`根元素标记。在文件打开时不存在关闭标记。

`<AUDIT_RECORD>`元素内的元素具有以下特征：

+   一些元素出现在每个`<AUDIT_RECORD>`元素中。其他是可选的，可能会根据审计记录类型出现。

+   `<AUDIT_RECORD>`元素内元素的顺序不保证。

+   元素值不是固定长度。长值可能会被截断，如后面给出的元素描述中所示。

+   `<`、`>`、`"`和`&`字符分别编码为`&lt;`、`&gt;`、`&quot;`和`&amp;`。NUL 字节（U+00）编码为`?`字符。

+   不是 XML 字符的字符使用数字字符引用进行编码。有效的 XML 字符包括：

    ```sql
    #x9 | #xA | #xD | [#x20-#xD7FF] | [#xE000-#xFFFD] | [#x10000-#x10FFFF]
    ```

每个`<AUDIT_RECORD>`元素中以下元素是必需的：

+   `<NAME>`

    代表生成审计事件的指令类型的字符串，例如服务器从客户端接收的命令。

    示例:

    ```sql
    <NAME>Query</NAME>
    ```

    一些常见的`<NAME>`值：

    ```sql
    Audit    When auditing starts, which may be server startup time
    Connect  When a client connects, also known as logging in
    Query    An SQL statement (executed directly)
    Prepare  Preparation of an SQL statement; usually followed by Execute
    Execute  Execution of an SQL statement; usually follows Prepare
    Shutdown Server shutdown
    Quit     When a client disconnects
    NoAudit  Auditing has been turned off
    ```

    可能的取值为`Audit`、`Binlog Dump`、`Change user`、`Close stmt`、`Connect Out`、`Connect`、`Create DB`、`Daemon`、`Debug`、`Delayed insert`、`Drop DB`、`Execute`、`Fetch`、`Field List`、`Init DB`、`Kill`、`Long Data`、`NoAudit`、`Ping`、`Prepare`、`Processlist`、`Query`、`Quit`、`Refresh`、`Register Slave`、`Reset stmt`、`Set option`、`Shutdown`、`Sleep`、`Statistics`、`Table Dump`、`TableDelete`、`TableInsert`、`TableRead`、`TableUpdate`、`Time`。

    这些值中的许多对应于`my_command.h`头文件中列出的`COM_*`xxx`*`命令值。例如，`Create DB`和`Change user`分别对应于`COM_CREATE_DB`和`COM_CHANGE_USER`。

    具有`Table*`XXX`*`的`<NAME>`值的事件伴随着`Query`事件。例如，以下语句生成一个`Query`事件，两个`TableRead`事件和一个`TableInsert`事件：

    ```sql
    INSERT INTO t3 SELECT t1.* FROM t1 JOIN t2;
    ```

    每个`Table*`XXX`*`事件包含`<TABLE>`和`<DB>`元素，用于标识事件所涉及的表和包含表的数据库。

+   `<RECORD_ID>`

    审计记录的唯一标识符。该值由序列号和时间戳组成，格式为`*`SEQ_TIMESTAMP`*`。当审计日志插件打开审计日志文件时，它将序列号初始化为审计日志文件的大小，然后为记录的每个记录递增序列号。时间戳是`*`YYYY-MM-DD`*T*`hh:mm:ss`*`格式的 UTC 值，指示审计日志插件打开文件的日期和时间。

    示例:

    ```sql
    <RECORD_ID>12_2019-10-03T14:06:33</RECORD_ID>
    ```

+   `<TIMESTAMP>`

    代表以`*`YYYY-MM-DD`*T*`hh:mm:ss`* UTC`格式表示的 UTC 值的字符串，指示生成审计事件的日期和时间。例如，对于从客户端接收的 SQL 语句执行对应的事件，其`<时间戳>`值发生在语句完成后，而不是接收时。

    示例：

    ```sql
    <TIMESTAMP>2019-10-03T14:09:45 UTC</TIMESTAMP>
    ```

下列元素在`<审计记录>`元素中是可选的。其中许多元素仅在特定`<名称>`元素值出现时才会出现。

+   `<COMMAND_CLASS>`

    指示执行的操作类型的字符串。

    示例：

    ```sql
    <COMMAND_CLASS>drop_table</COMMAND_CLASS>
    ```

    这些值对应于`statement/sql/*`xxx`*`命令计数器。例如，*`xxx`*分别是`drop_table`和`select`，用于`DROP TABLE`和`SELECT`语句。以下语句显示可能的名称：

    ```sql
    SELECT REPLACE(EVENT_NAME, 'statement/sql/', '') AS name
    FROM performance_schema.events_statements_summary_global_by_event_name
    WHERE EVENT_NAME LIKE 'statement/sql/%'
    ORDER BY name;
    ```

+   `<连接属性>`

    从 MySQL 8.0.19 开始，具有`connect`值的`<COMMAND_CLASS>`事件可能包含一个`<连接属性>`元素，以显示客户端在连接时传递的连接属性。 （有关这些属性的信息，这些属性也在性能模式表中公开，请参见第 29.12.9 节，“性能模式连接属性表”.）

    `<连接属性>`元素包含一个`<属性>`元素，每个元素包含`<名称>`和`<值>`元素，分别指示属性名称和值。

    示例：

    ```sql
    <CONNECTION_ATTRIBUTES>
     <ATTRIBUTE>
      <NAME>_pid</NAME>
      <VALUE>42794</VALUE>
     </ATTRIBUTE>
     <ATTRIBUTE>
      <NAME>_os</NAME>
      <VALUE>macos0.14</VALUE>
     </ATTRIBUTE>
     <ATTRIBUTE>
      <NAME>_platform</NAME>
      <VALUE>x86_64</VALUE>
     </ATTRIBUTE>
     <ATTRIBUTE>
      <NAME>_client_version</NAME>
      <VALUE>8.0.19</VALUE>
     </ATTRIBUTE>
     <ATTRIBUTE>
      <NAME>_client_name</NAME>
      <VALUE>libmysql</VALUE>
     </ATTRIBUTE>
     <ATTRIBUTE>
      <NAME>program_name</NAME>
      <VALUE>mysqladmin</VALUE>
     </ATTRIBUTE>
    </CONNECTION_ATTRIBUTES>
    ```

    如果事件中没有连接属性，则不记录任何属性，也不会出现`<连接属性>`元素。这可能发生在连接尝试失败，客户端未传递任何属性，或者连接在内部发生，例如在服务器启动期间或由插件发起时。

+   `<连接标识符>`

    代表客户端连接标识符的无符号整数。这与会话中由`CONNECTION_ID()`函数返回的值相同。

    示例：

    ```sql
    <CONNECTION_ID>127</CONNECTION_ID>
    ```

+   `<连接类型>`

    与服务器的连接的安全状态。允许的值为`TCP/IP`（未加密建立的 TCP/IP 连接），`SSL/TLS`（加密建立的 TCP/IP 连接），`Socket`（Unix 套接字文件连接），`Named Pipe`（Windows 命名管道连接）和`Shared Memory`（Windows 共享内存连接）。

    示例：

    ```sql
    <CONNECTION_TYPE>SSL/TLS</CONNECTION_TYPE>
    ```

+   `<数据库>`

    代表数据库名称的字符串。

    示例：

    ```sql
    <DB>test</DB>
    ```

    对于连接事件，此元素指示默认数据库；如果没有默认数据库，则该元素为空。对于表访问事件，该元素指示所访问表所属的数据库。

+   `<主机>`

    代表客户端主机名的字符串。

    示例：

    ```sql
    <HOST>localhost</HOST>
    ```

+   `<IP>`

    代表客户端 IP 地址的字符串。

    示例：

    ```sql
    <IP>127.0.0.1</IP>
    ```

+   `<MYSQL 版本>`

    一个表示 MySQL 服务器版本的字符串。这与`VERSION()`函数或`version`系统变量的值相同。

    示例:

    ```sql
    <MYSQL_VERSION>5.7.21-log</MYSQL_VERSION>
    ```

+   `<操作系统登录>`

    一个表示在认证过程中使用的外部用户名称的字符串，由用于认证客户端的插件设置。对于本机（内置）MySQL 认证，或者如果插件未设置该值，则此元素为空。该值与`external_user`系统变量的值相同（参见第 8.2.19 节，“代理用户”）。

    示例:

    ```sql
    <OS_LOGIN>jeffrey</OS_LOGIN>
    ```

+   `<操作系统版本>`

    一个表示构建或运行服务器的操作系统的字符串。

    示例:

    ```sql
    <OS_VERSION>x86_64-Linux</OS_VERSION>
    ```

+   `<特权用户>`

    一个表示服务器对客户端进行身份验证的用户的字符串。这是服务器用于权限检查的用户名，可能与 `<用户>` 值不同。

    示例:

    ```sql
    <PRIV_USER>jeffrey</PRIV_USER>
    ```

+   `<代理用户>`

    一个表示代理用户的字符串（参见第 8.2.19 节，“代理用户”）。如果未启用用户代理，则值为空。

    示例:

    ```sql
    <PROXY_USER>developer</PROXY_USER>
    ```

+   `<服务器 ID>`

    一个表示服务器 ID 的无符号整数。这与`server_id`系统变量的值相同。

    示例:

    ```sql
    <SERVER_ID>1</SERVER_ID>
    ```

+   `<SQL 文本>`

    一个表示 SQL 语句文本的字符串。该值可以为空。长值可能会被截断。该字符串与审计日志文件本身一样，使用 UTF-8 编写（每个字符最多 4 个字节），因此该值可能是转换的结果。例如，原始语句可能已从客户端作为 SJIS 字符串接收。

    示例:

    ```sql
    <SQLTEXT>DELETE FROM t1</SQLTEXT>
    ```

+   `<启动选项>`

    一个表示在启动 MySQL 服务器时在命令行或选项文件中给定的选项的字符串。第一个选项是服务器可执行文件的路径。

    示例:

    ```sql
    <STARTUP_OPTIONS>/usr/local/mysql/bin/mysqld
      --port=3306 --log_output=FILE</STARTUP_OPTIONS>
    ```

+   `<状态>`

    一个表示命令状态的无符号整数：成功为 0，发生错误为非零值。这与`mysql_errno()` C API 函数的值相同。有关 `<状态码>`与 `<状态>` 之间的区别的信息，请参阅描述。

    审计日志不包含 SQLSTATE 值或错误消息。要查看错误代码、SQLSTATE 值和消息之间的关联，请参阅服务器错误消息参考。

    警告未记录。

    示例:

    ```sql
    <STATUS>1051</STATUS>
    ```

+   `<状态码>`

    一个表示命令状态的无符号整数：成功为 0，发生错误为 1。

    `STATUS_CODE` 值与 `STATUS` 值不同：`STATUS_CODE` 成功时为 0，错误时为 1，这与 Audit Vault 的 EZ_collector 消费者兼容。`STATUS` 是 `mysql_errno()` C API 函数的值。成功时为 0，错误时为非零，因此错误不一定为 1。

    例子：

    ```sql
    <STATUS_CODE>0</STATUS_CODE>
    ```

+   `<TABLE>`

    代表表名的字符串。

    例子：

    ```sql
    <TABLE>t3</TABLE>
    ```

+   `<USER>`

    代表客户端发送的用户名的字符串。这可能与 `<PRIV_USER>` 值不同。

    例子：

    ```sql
    <USER>root[root] @ localhost [127.0.0.1]</USER>
    ```

+   `<VERSION>`

    代表审计日志文件格式版本的无符号整数。

    例子：

    ```sql
    <VERSION>1</VERSION>
    ```

##### 旧式 XML 审计日志文件格式

这是一个以旧式 XML 格式（`audit_log_format=OLD`）稍作调整以提高可读性的示例日志文件：

```sql
<?xml version="1.0" encoding="utf-8"?>
<AUDIT>
  <AUDIT_RECORD
    TIMESTAMP="2019-10-03T14:25:00 UTC"
    RECORD_ID="1_2019-10-03T14:25:00"
    NAME="Audit"
    SERVER_ID="1"
    VERSION="1"
    STARTUP_OPTIONS="--port=3306"
    OS_VERSION="i686-Linux"
    MYSQL_VERSION="5.7.21-log"/>
  <AUDIT_RECORD
    TIMESTAMP="2019-10-03T14:25:24 UTC"
    RECORD_ID="2_2019-10-03T14:25:00"
    NAME="Connect"
    CONNECTION_ID="4"
    STATUS="0"
    STATUS_CODE="0"
    USER="root"
    OS_LOGIN=""
    HOST="localhost"
    IP="127.0.0.1"
    COMMAND_CLASS="connect"
    CONNECTION_TYPE="SSL/TLS"
    PRIV_USER="root"
    PROXY_USER=""
    DB="test"/>

...

  <AUDIT_RECORD
    TIMESTAMP="2019-10-03T14:25:24 UTC"
    RECORD_ID="6_2019-10-03T14:25:00"
    NAME="Query"
    CONNECTION_ID="4"
    STATUS="0"
    STATUS_CODE="0"
    USER="root[root] @ localhost [127.0.0.1]"
    OS_LOGIN=""
    HOST="localhost"
    IP="127.0.0.1"
    COMMAND_CLASS="drop_table"
    SQLTEXT="DROP TABLE IF EXISTS t"/>

...

  <AUDIT_RECORD
    TIMESTAMP="2019-10-03T14:25:24 UTC"
    RECORD_ID="8_2019-10-03T14:25:00"
    NAME="Quit"
    CONNECTION_ID="4"
    STATUS="0"
    STATUS_CODE="0"
    USER="root"
    OS_LOGIN=""
    HOST="localhost"
    IP="127.0.0.1"
    COMMAND_CLASS="connect"
    CONNECTION_TYPE="SSL/TLS"/>
  <AUDIT_RECORD
    TIMESTAMP="2019-10-03T14:25:32 UTC"
    RECORD_ID="12_2019-10-03T14:25:00"
    NAME="NoAudit"
    SERVER_ID="1"/>
</AUDIT>
```

审计日志文件以 XML 格式编写，使用 UTF-8（每个字符最多 4 个字节）。根元素是 `<AUDIT>`。根元素包含 `<AUDIT_RECORD>` 元素，每个元素提供有关审计事件的信息。当审计日志插件开始写入新日志文件时，它会写入 XML 声明和打开 `<AUDIT>` 根元素标记。当插件关闭日志文件时，它会写入关闭 `</AUDIT>` 根元素标记。在文件打开时不存在关闭标记。

`<AUDIT_RECORD>` 元素的属性具有以下特征：

+   一些属性��现在每个 `<AUDIT_RECORD>` 元素中。其他属性是可选的，可能会根据审计记录类型而出现。

+   `<AUDIT_RECORD>` 元素内属性的顺序不保证。

+   属性值不是固定长度。长值可能会根据稍后给出的属性描述而被截断。

+   `<`、`>`、`"` 和 `&` 字符分别编码为 `&lt;`、`&gt;`、`&quot;` 和 `&amp;`。NUL 字节（U+00）编码为 `?` 字符。

+   不是有效 XML 字符的字符使用数字字符引用进行编码。有效的 XML 字符包括：

    ```sql
    #x9 | #xA | #xD | [#x20-#xD7FF] | [#xE000-#xFFFD] | [#x10000-#x10FFFF]
    ```

每个 `<AUDIT_RECORD>` 元素中以下属性是必需的：

+   `NAME`

    代表生成审计事件的指令类型的字符串，例如服务器从客户端接收的命令。

    例子：`NAME="Query"`

    一些常见的 `NAME` 值：

    ```sql
    Audit    When auditing starts, which may be server startup time
    Connect  When a client connects, also known as logging in
    Query    An SQL statement (executed directly)
    Prepare  Preparation of an SQL statement; usually followed by Execute
    Execute  Execution of an SQL statement; usually follows Prepare
    Shutdown Server shutdown
    Quit     When a client disconnects
    NoAudit  Auditing has been turned off
    ```

    可能的值为 `Audit`、`Binlog Dump`、`Change user`、`Close stmt`、`Connect Out`、`Connect`、`Create DB`、`Daemon`、`Debug`、`Delayed insert`、`Drop DB`、`Execute`、`Fetch`、`Field List`、`Init DB`、`Kill`、`Long Data`、`NoAudit`、`Ping`、`Prepare`、`Processlist`、`Query`、`Quit`、`Refresh`、`Register Slave`、`Reset stmt`、`Set option`、`Shutdown`、`Sleep`、`Statistics`、`Table Dump`、`TableDelete`、`TableInsert`、`TableRead`、`TableUpdate`、`Time`。

    许多这些值对应于 `my_command.h` 头文件中列出的 `COM_*`xxx`*` 命令值。例如，`"Create DB"` 和 `"Change user"` 分别对应于 `COM_CREATE_DB` 和 `COM_CHANGE_USER`。

    具有`NAME`值为`Table*`XXX`*`的事件伴随着`Query`事件。例如，以下语句生成一个`Query`事件，两个`TableRead`事件和一个`TableInsert`事件：

    ```sql
    INSERT INTO t3 SELECT t1.* FROM t1 JOIN t2;
    ```

    每个`Table*`XXX`*`事件都有`TABLE`和`DB`属性，用于标识事件所涉及的表和包含该表的数据库。

    旧式 XML 审计日志格式的`Connect`事件不包括连接属性。

+   `RECORD_ID`

    审计记录的唯一标识符。该值由序列号和时间戳组成，格式为`*`SEQ_TIMESTAMP`*`。当审计日志插件打开审计日志文件时，它将序列号初始化为审计日志文件的大小，然后为每个记录递增 1。时间戳是以`*`YYYY-MM-DD`*T*`hh:mm:ss`*`格式的 UTC 值，指示审计日志插件打开文件的日期和时间��

    示例: `RECORD_ID="12_2019-10-03T14:25:00"`

+   `TIMESTAMP`

    代表以`*`YYYY-MM-DD`*T*`hh:mm:ss`* UTC`格式的 UTC 值的字符串，指示生成审计事件的日期和时间。例如，对于从客户端接收的 SQL 语句执行对应的事件，`TIMESTAMP`值发生在语句完成之后，而不是接收时。

    示例: `TIMESTAMP="2019-10-03T14:25:32 UTC"`

`<AUDIT_RECORD>`元素中以下属性是可选的。其中许多属性仅在具有特定`NAME`属性值的元素中出现。

+   `COMMAND_CLASS`

    表示执行的操作类型的字符串。

    示例: `COMMAND_CLASS="drop_table"`

    这些值对应于`statement/sql/*`xxx`*`命令计数器。例如，*`xxx`*分别为`drop_table`和`select`，用于`DROP TABLE`和`SELECT`语句。以下语句显示可能的名称：

    ```sql
    SELECT REPLACE(EVENT_NAME, 'statement/sql/', '') AS name
    FROM performance_schema.events_statements_summary_global_by_event_name
    WHERE EVENT_NAME LIKE 'statement/sql/%'
    ORDER BY name;
    ```

+   `CONNECTION_ID`

    代表客户端连接标识符的无符号整数。这与会话中`CONNECTION_ID()`函数返回的值相同。

    示例: `CONNECTION_ID="127"`

+   `CONNECTION_TYPE`

    与服务器的连接的安全状态。允许的值为`TCP/IP`（未加密的 TCP/IP 连接建立），`SSL/TLS`（已加密的 TCP/IP 连接建立），`Socket`（Unix 套接字文件连接），`Named Pipe`（Windows 命名管道连接）和`Shared Memory`（Windows 共享内存连接）。

    示例: `CONNECTION_TYPE="SSL/TLS"`

+   `DB`

    代表数据库名称的字符串。

    示例: `DB="test"`

    对于连接事件，此属性指示默认数据库；如果没有默认数据库，则该属性为空。对于表访问事件，该属性指示访问表所属的数据库。

+   `HOST`

    代表客户端主机名的字符串。

    示例: `HOST="localhost"`

+   `IP`

    代表客户端 IP 地址的字符串。

    示例: `IP="127.0.0.1"`

+   `MYSQL_VERSION`

    代表 MySQL 服务器版本的字符串。这与`VERSION()`函数或`version`系统变量的值相同。

    示例：`MYSQL_VERSION="5.7.21-log"`

+   `OS_LOGIN`

    代表在认证过程中使用的外部用户名称的字符串，由用于认证客户端的插件设置。对于本地（内置）MySQL 认证，或者如果插件未设置该值，则此属性为空。该值与`external_user`系统变量的值相同（参见第 8.2.19 节，“代理用户”）。

    示例：`OS_LOGIN="jeffrey"`

+   `OS_VERSION`

    代表构建或运行服务器的操作系统的字符串。

    示例：`OS_VERSION="x86_64-Linux"`

+   `PRIV_USER`

    代表服务器对客户端进行身份验证的用户的字符串。这是服务器用于特权检查的用户名，可能与`USER`值不同。

    示例：`PRIV_USER="jeffrey"`

+   `PROXY_USER`

    代表代理用户的字符串（参见第 8.2.19 节，“代理用户”）。如果未启用用户代理，则值为空。

    示例：`PROXY_USER="developer"`

+   `SERVER_ID`

    代表服务器 ID 的无符号整数。这与`server_id`系统变量的值相同。

    示例：`SERVER_ID="1"`

+   `SQLTEXT`

    代表 SQL 语句的文本的字符串。该值可以为空。长值可能会被截断。该字符串与审计日志文件本身一样，使用 UTF-8 编写（每个字符最多 4 个字节），因此该值可能是转换的结果。例如，原始语句可能已从客户端接收为 SJIS 字符串。

    示例：`SQLTEXT="DELETE FROM t1"`

+   `STARTUP_OPTIONS`

    代表 MySQL 服务器启动时在命令行或选项文件中给定的选项的字符串。

    示例：`STARTUP_OPTIONS="--port=3306 --log_output=FILE"`

+   `STATUS`

    代表命令状态的无符号整数：成功为 0，发生错误为非零。这与`mysql_errno()` C API 函数的值相同。有关其与`STATUS`的区别的信息，请参阅`STATUS_CODE`的描述。

    审计日志不包含 SQLSTATE 值或错误消息。要查看错误代码、SQLSTATE 值和消息之间的关联，请参阅服务器错误消息参考。

    警告不会被记录。

    示例：`STATUS="1051"`

+   `STATUS_CODE`

    代表命令状态的无符号整数：成功为 0，发生错误为 1。

    `STATUS_CODE` 值与 `STATUS` 值不同：`STATUS_CODE` 对于成功为 0，对于错误为 1，这与 Audit Vault 的 EZ_collector 消费者兼容。`STATUS` 是 `mysql_errno()` C API 函数的值。这对于成功为 0，对于错误为非零，因此并不一定为 1 表示错误。

    例如：`STATUS_CODE="0"`

+   `TABLE`

    代表表名的字符串。

    例如：`TABLE="t3"`

+   `USER`

    代表客户端发送的用户名的字符串。这可能与 `PRIV_USER` 值不同。

+   `VERSION`

    代表审计日志文件格式版本的无符号整数。

    例如：`VERSION="1"`

##### JSON 审计日志文件格式

对于 JSON 格式的审计日志记录（`audit_log_format=JSON`），日志文件内容形成一个 `JSON` 数组，每个数组元素表示一个审计事件，作为一个 `JSON` 键值对的哈希。完整事件记录的示例稍后在本节中出现。以下是部分事件的摘录：

```sql
[
  {
    "timestamp": "2019-10-03 13:50:01",
    "id": 0,
    "class": "audit",
    "event": "startup",
    ...
  },
  {
    "timestamp": "2019-10-03 15:02:32",
    "id": 0,
    "class": "connection",
    "event": "connect",
    ...
  },
  ...
  {
    "timestamp": "2019-10-03 17:37:26",
    "id": 0,
    "class": "table_access",
    "event": "insert",
      ...
  }
  ...
]
```

审计日志文件使用 UTF-8 编写（每个字符最多 4 个字节）。当审计日志插件开始写入新的日志文件时，它会写入开头的 `[` 数组标记。当插件关闭日志文件时，它会写入结尾的 `]` 数组标记。在文件打开时，结尾标记不存在。

审计记录中的项目具有以下特征：

+   一些项目出现在每个审计记录中。其他项目是可选的，可能会根据审计记录类型而出现。

+   审计记录中项目的顺序不保证。

+   项目值不是固定长度。长值可能会根据稍后给出的项目描述而被截断。

+   `"` 和 `\` 字符分别编码为 `\"` 和 `\\`。

JSON 格式是唯一支持可选查询时间和大小统计信息的审计日志文件格式，这些信息从 MySQL 8.0.30 开始可用。对于符合条件的查询，此数据在慢查询日志中可用，并且在审计日志的上下文中，类似地有助于检测活动分析的异常值。

要将查询统计信息添加到日志文件中，必须将其设置为使用 `audit_log_filter_set_filter()` 审计日志函数作为 JSON 过滤语法的服务元素的过滤器。有关如何执行此操作的说明，请参见为异常检测添加查询统计信息。要填充 `bytes_sent` 和 `bytes_received` 字段，必须将系统变量 `log_slow_extra` 设置为 ON。

以下示例展示了不同事件类型的 JSON 对象格式（由 `class` 和 `event` 项目指示），稍作调整以提高可读性：

审计启动事件：

```sql
{ "timestamp": "2019-10-03 14:21:56",
  "id": 0,
  "class": "audit",
  "event": "startup",
  "connection_id": 0,
  "startup_data": { "server_id": 1,
                    "os_version": "i686-Linux",
                    "mysql_version": "5.7.21-log",
                    "args": ["/usr/local/mysql/bin/mysqld",
                             "--loose-audit-log-format=JSON",
                             "--log-error=log.err",
                             "--pid-file=mysqld.pid",
                             "--port=3306" ] } }
```

当由于服务器启动而启动审计日志插件（而不是在运行时启用）时，`connection_id`设置为 0，`account`和`login`不存在。

审计关闭事件:

```sql
{ "timestamp": "2019-10-03 14:28:20",
  "id": 3,
  "class": "audit",
  "event": "shutdown",
  "connection_id": 0,
  "shutdown_data": { "server_id": 1 } }
```

当由于服务器关闭而卸载审计日志插件（而不是在运行时禁用）时，`connection_id`设置为 0，`account`和`login`不存在。

连接或更改用户事件:

```sql
{ "timestamp": "2019-10-03 14:23:18",
  "id": 1,
  "class": "connection",
  "event": "connect",
  "connection_id": 5,
  "account": { "user": "root", "host": "localhost" },
  "login": { "user": "root", "os": "", "ip": "::1", "proxy": "" },
  "connection_data": { "connection_type": "ssl",
                       "status": 0,
                       "db": "test",
                       "connection_attributes": {
                         "_pid": "43236",
                         ...
                         "program_name": "mysqladmin"
                       } }
}
```

断开连接事件:

```sql
{ "timestamp": "2019-10-03 14:24:45",
  "id": 3,
  "class": "connection",
  "event": "disconnect",
  "connection_id": 5,
  "account": { "user": "root", "host": "localhost" },
  "login": { "user": "root", "os": "", "ip": "::1", "proxy": "" },
  "connection_data": { "connection_type": "ssl" } }
```

查询事件:

```sql
{ "timestamp": "2019-10-03 14:23:35",
  "id": 2,
  "class": "general",
  "event": "status",
  "connection_id": 5,
  "account": { "user": "root", "host": "localhost" },
  "login": { "user": "root", "os": "", "ip": "::1", "proxy": "" },
  "general_data": { "command": "Query",
                    "sql_command": "show_variables",
                    "query": "SHOW VARIABLES",
                    "status": 0 } }
```

具有用于异常检测的可选查询统计信息的查询事件:

```sql
{ "timestamp": "2022-01-28 13:09:30", 
  "id": 0, 
  "class": "general", 
  "event": "status", 
  "connection_id": 46, 
  "account": { "user": "user", "host": "localhost" }, 
  "login": { "user": "user", “os": "", “ip": "127.0.0.1", “proxy": "" }, 
  "general_data": { "command": "Query", 
                    "sql_command": "insert",
	            "query": "INSERT INTO audit_table VALUES(4)",
	            "status": 1146 }
  "query_statistics": { "query_time": 0.116250,
                        "bytes_sent": 18384,
                        "bytes_received": 78858,
                        "rows_sent": 3,
                        "rows_examined": 20878 } }
```

表访问事件（读取、删除、插入、更新）:

```sql
{ "timestamp": "2019-10-03 14:23:41",
  "id": 0,
  "class": "table_access",
  "event": "insert",
  "connection_id": 5,
  "account": { "user": "root", "host": "localhost" },
  "login": { "user": "root", "os": "", "ip": "127.0.0.1", "proxy": "" },
  "table_access_data": { "db": "test",
                         "table": "t1",
                         "query": "INSERT INTO t1 (i) VALUES(1),(2),(3)",
                         "sql_command": "insert" } }
```

以下列表中的项目出现在 JSON 格式审计记录的顶层：每个项目的值可以是标量或`JSON`哈希。对于具有哈希值的项目，描述仅列出该哈希内的项目名称。有关第二级哈希项目的更完整描述，请参见本节后面的内容。

+   `account`

    与事件关联的 MySQL 账户。该值是一个包含这些项目的哈希，等同于`CURRENT_USER()`函数中的值：`user`, `host`.

    示例:

    ```sql
    "account": { "user": "root", "host": "localhost" }
    ```

+   `class`

    表示事件类的字符串。类定义了事件的类型，与指定事件子类的`event`项目一起。

    示例:

    ```sql
    "class": "connection"
    ```

    以下表格显示了`class`和`event`值的允许组合。

    **表 8.34 审计日志类和事件组合**

    | 类值 | 允许的事件值 |
    | --- | --- |
    | `audit` | `startup`, `shutdown` |
    | `connection` | `connect`, `change_user`, `disconnect` |
    | `general` | `status` |
    | `table_access_data` | `read`, `delete`, `insert`, `update` |

+   `connection_data`

    有关客户端连接的信息。该值是一个包含这些项目的哈希：`connection_type`, `status`, `db`，可能还有`connection_attributes`。此项目仅出现在`class`值为`connection`的审计记录中。

    示例:

    ```sql
    "connection_data": { "connection_type": "ssl",
                         "status": 0,
                         "db": "test" }
    ```

    截至 MySQL 8.0.19，具有`class`值为`connection`和`event`值为`connect`的事件可能包括一个`connection_attributes`项目，以显示客户端在连接时传递的连接属性。（有关这些属性的信息，也在性能模式表中公开，参见第 29.12.9 节，“性能模式连接属性表”.）

    `connection_attributes`值是一个哈希，通过其名称和值表示每个属性。

    示例:

    ```sql
    "connection_attributes": {
      "_pid": "43236",
      "_os": "macos0.14",
      "_platform": "x86_64",
      "_client_version": "8.0.19",
      "_client_name": "libmysql",
      "program_name": "mysqladmin"
    }
    ```

    如果事件中没有连接属性，则不记录任何属性，也不会出现`connection_attributes`项目。这可能发生在连接尝试失败、客户端未传递任何属性或连接在服务器启动期间内部发生时。

+   `connection_id`

    代表客户端连接标识符的无符号整数。这与会话中`CONNECTION_ID()`函数返回的值相同。

    示例:

    ```sql
    "connection_id": 5
    ```

+   `event`

    代表事件类别的子类的字符串。子类定义了事件的类型，当与指定事件类别的`class`项目一起考虑时。有关更多信息，请参阅`class`项目描述。

    示例:

    ```sql
    "event": "connect"
    ```

+   `general_data`

    执行语句或命令的信息。该值是一个包含以下项目的哈希：`command`、`sql_command`、`query`、`status`。此项目仅出现在`class`值为`general`的审计记录中。

    示例:

    ```sql
    "general_data": { "command": "Query",
                      "sql_command": "show_variables",
                      "query": "SHOW VARIABLES",
                      "status": 0 }
    ```

+   `id`

    代表事件 ID 的无符号整数。

    示例:

    ```sql
    "id": 2
    ```

    对于具有相同`timestamp`值的审计记录，它们的`id`值可以区分它们并形成一个序列。在审计日志中，`timestamp`/`id`对是唯一的。这些对是标识日志中事件位置的书签。

+   `login`

    指示客户端如何连接到服务器的信息。该值是一个包含以下项目的哈希：`user`、`os`、`ip`、`proxy`。

    示例:

    ```sql
    "login": { "user": "root", "os": "", "ip": "::1", "proxy": "" }
    ```

+   `query_statistics`

    用于异常检测的可选查询统计信息。该值是一个包含以下项目的哈希：`query_time`、`rows_sent`、`rows_examined`、`bytes_received`、`bytes_sent`。有关设置查询统计信息的说明，请参见为异常检测添加查询统计信息。

    示例:

    ```sql
    "query_statistics": { "query_time": 0.116250,
                          "bytes_sent": 18384,
                          "bytes_received": 78858,
                          "rows_sent": 3,
                          "rows_examined": 20878 }
    ```

+   `shutdown_data`

    与审计日志插件终止相关的信息。该值是一个包含以下项目的哈希：`server_id`。此项目仅出现在`class`和`event`值分别为`audit`和`shutdown`的审计记录中。

    示例:

    ```sql
    "shutdown_data": { "server_id": 1 }
    ```

+   `startup_data`

    与审计日志插件初始化相关的信息。该值是一个包含以下项目的哈希：`server_id`、`os_version`、`mysql_version`、`args`。此项目仅出现在`class`和`event`值分别为`audit`和`startup`的审计记录中。

    示例:

    ```sql
    "startup_data": { "server_id": 1,
                      "os_version": "i686-Linux",
                      "mysql_version": "5.7.21-log",
                      "args": ["/usr/local/mysql/bin/mysqld",
                               "--loose-audit-log-format=JSON",
                               "--log-error=log.err",
                               "--pid-file=mysqld.pid",
                               "--port=3306" ] }
    ```

+   `table_access_data`

    访问表格的信息。该值是一个包含以下项目的哈希：`db`、`table`、`query`、`sql_command`。此项目仅出现在`class`值为`table_access`的审计记录中。

    示例:

    ```sql
    "table_access_data": { "db": "test",
                           "table": "t1",
                           "query": "INSERT INTO t1 (i) VALUES(1),(2),(3)",
                           "sql_command": "insert" }
    ```

+   `time`

    此字段类似于`timestamp`字段中的字段，但其值是一个整数，表示生成审计事件的日期和时间的 UNIX 时间戳值。

    示例:

    ```sql
    "time" : 1618498687
    ```

    只有在启用`audit_log_format_unix_timestamp`系统变量时，`time`字段才会出现在 JSON 格式的日志文件中。

+   `timestamp`

    代表以*`YYYY-MM-DD hh:mm:ss`*格式表示的 UTC 值，指示生成审计事件的日期和时间。例如，对应于从客户端接收的 SQL 语句执行的事件具有`timestamp`值，该值在语句完成后发生，而不是在接收时发生。

    示例:

    ```sql
    "timestamp": "2019-10-03 13:50:01"
    ```

    对于具有相同`timestamp`值的审计记录，它们的`id`值可以区分它们并形成一个序列。在审计日志中，`timestamp`/`id`对是唯一的。这些对是标识日志中事件位置的书签。

这些项目出现在与 JSON 格式审计记录顶级项目关联的哈希值中：

+   `args`

    一个数组，其中包含 MySQL 服务器启动时在命令行或选项文件中给定的选项。第一个选项是服务器可执行文件的路径。

    示例:

    ```sql
    "args": ["/usr/local/mysql/bin/mysqld",
             "--loose-audit-log-format=JSON",
             "--log-error=log.err",
             "--pid-file=mysqld.pid",
             "--port=3306" ]
    ```

+   `bytes_received`

    从客户端接收的字节数。此项目是可选查询统计信息的一部分。要填充此字段，必须将系统变量`log_slow_extra`设置为`ON`。

    示例:

    ```sql
    "bytes_received": 78858
    ```

+   `bytes_sent`

    发送到客户端的字节数。此项目是可选查询统计信息的一部分。要填充此字段，必须将系统变量`log_slow_extra`设置为`ON`。

    示例:

    ```sql
    "bytes_sent": 18384
    ```

+   `command`

    代表生成审计事件的指令类型，例如服务器从客户端接收的命令。

    示例:

    ```sql
    "command": "Query"
    ```

+   `connection_type`

    与服务器的连接安全状态相关。允许的值为`tcp/ip`（未加密的 TCP/IP 连接）、`ssl`（已加密的 TCP/IP 连接）、`socket`（Unix 套接字文件连接）、`named_pipe`（Windows 命名管道连接）和`shared_memory`（Windows 共享内存连接）。

    示例:

    ```sql
    "connection_type": "tcp/tcp"
    ```

+   `db`

    代表数据库名称的字符串。对于`connection_data`，它是默认数据库。对于`table_access_data`，它是表数据库。

    示例:

    ```sql
    "db": "test"
    ```

+   `host`

    代表客户端主机名的字符串。

    示例:

    ```sql
    "host": "localhost"
    ```

+   `ip`

    代表客户端 IP 地址的字符串。

    示例:

    ```sql
    "ip": "::1"
    ```

+   `mysql_version`

    代表 MySQL 服务器版本的字符串。这与`VERSION()`函数或`version`系统变量的值相同。

    示例:

    ```sql
    "mysql_version": "5.7.21-log"
    ```

+   `os`

    代表在认证过程中使用的外部用户名，由用于认证客户端的插件设置。对于本地（内置）MySQL 认证，或者如果插件未设置该值，则此属性为空。该值与`external_user`系统变量的值相同。参见 Section 8.2.19, “Proxy Users”。

    示例:

    ```sql
    "os": "jeffrey"
    ```

+   `os_version`

    代表构建或运行服务器的操作系统的字符串。

    示例:

    ```sql
    "os_version": "i686-Linux"
    ```

+   `proxy`

    代理用户的字符串表示（参见第 8.2.19 节，“代理用户”）。如果未启用用户代理，则该值为空。

    示例：

    ```sql
    "proxy": "developer"
    ```

+   `查询`

    一个表示 SQL 语句文本的字符串。该值可以为空。长值可能会被截断。该字符串，就像审计日志文件本身一样，是使用 UTF-8（每个字符最多 4 个字节）编写的，因此该值可能是转换的结果。例如，原始语句可能已从客户端作为 SJIS 字符串接收。

    示例：

    ```sql
    "query": "DELETE FROM t1"
    ```

+   `query_time`

    查询执行时间（如果选择了`longlong`数据类型，则为微秒，如果选择了`double`数据类型，则为秒）。此项是可选查询统计的一部分。

    示例：

    ```sql
    "query_time": 0.116250
    ```

+   `rows_examined`

    查询期间访问的行数。此项是可选查询统计的一部分。

    示例：

    ```sql
    "rows_examined": 20878
    ```

+   `rows_sent`

    作为结果发送给客户端的行数。此项是可选查询统计的一部分。

    示例：

    ```sql
    "rows_sent": 3
    ```

+   `服务器 ID`

    表示服务器 ID 的无符号整数。这与`server_id`系统变量的值相同。

    示例：

    ```sql
    "server_id": 1
    ```

+   `sql_command`

    表示 SQL 语句类型的字符串。

    示例：

    ```sql
    "sql_command": "insert"
    ```

    这些值对应于`statement/sql/*`xxx`*`命令计数器。例如，*`xxx`*是`drop_table`和`select`分别用于`DROP TABLE`和`SELECT`语句。以下语句显示可能的名称：

    ```sql
    SELECT REPLACE(EVENT_NAME, 'statement/sql/', '') AS name
    FROM performance_schema.events_statements_summary_global_by_event_name
    WHERE EVENT_NAME LIKE 'statement/sql/%'
    ORDER BY name;
    ```

+   `状态`

    表示命令状态的无符号整数：成功为 0，发生错误为非零。这与`mysql_errno()` C API 函数的值相同。

    审计日志不包含 SQLSTATE 值或错误消息。要查看错误代码、SQLSTATE 值和消息之间的关联，请参阅服务器错误消息参考。

    警告未记录。

    示例：

    ```sql
    "status": 1051
    ```

+   `表`

    表示表名的字符串。

    示例：

    ```sql
    "table": "t1"
    ```

+   `用户`

    表示用户名的字符串。根据`user`出现的项的不同，含义也不同：

    +   在`account`项中，`user`是一个表示服务器对客户端进行身份验证的用户的字符串。这是服务器用于权限检查的用户名。

    +   在`login`项中，`user`是表示客户端发送的用户名的字符串。

    示例：

    ```sql
    "user": "root"
    ```
