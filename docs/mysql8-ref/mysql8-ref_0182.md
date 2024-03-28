> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-logging.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-logging.html)

#### 6.5.1.3 mysql 客户端日志记录

**mysql**客户端可以对交互式执行的语句进行以下类型的记录：

+   在 Unix 上，**mysql**将语句写入历史文件。默认情况下，此文件在您的主目录中命名为`.mysql_history`。要指定不同的文件，请设置`MYSQL_HISTFILE`环境变量的值。

+   在所有平台上，如果给定`--syslog`选项，**mysql**会将语句写入系统日志设施。在 Unix 上，这是`syslog`；在 Windows 上，这是 Windows 事件日志。记录消息出现的目的地取决于系统。在 Linux 上，目的地通常是`/var/log/messages`文件。

以下讨论描述适用于所有日志类型的特征，并提供每种日志类型的特定信息。

+   日志记录方式

+   控制历史文件

+   syslog 日志特性

##### 日志记录方式

对于每个启用的日志记录目的地，语句记录如下进行：

+   只有在交互式执行时才记录语句。例如，从文件或管道中读取时，语句是非交互式的。还可以通过使用`--batch`或`--execute`选项来抑制语句记录。

+   如果语句与“忽略”列表中的任何模式匹配，则会被忽略并不记录。此列表稍后描述。

+   **mysql**会单独记录每个非忽略的非空语句行。

+   如果一个非忽略的语句跨越多行（不包括结束分隔符），**mysql**会将这些行连接起来形成完整的语句，将换行符映射为空格，并记录结果，再加上一个分隔符。

因此，跨越多行的输入语句可能会被记录两次。考虑以下输入：

```sql
mysql> SELECT
 -> 'Today is'
 -> ,
 -> CURDATE()
 -> ;
```

在这种情况下，**mysql**记录“SELECT”、“'Today is'”、“,”、“CURDATE()”和“;”行，然后将它们映射到`SELECT 'Today is' , CURDATE()`，再加上一个分隔符。因此，这些行会出现在记录的输出中：

```sql
SELECT
'Today is'
,
CURDATE()
;
SELECT 'Today is' , CURDATE();
```

**mysql** 在记录日志时会忽略与“ignore”列表中任何模式匹配的语句。默认情况下，模式列表是`"*IDENTIFIED*:*PASSWORD*"`，用于忽略涉及密码的语句。模式匹配不区分大小写。在模式中，两个字符是特殊的：

+   `?` 匹配任何单个字符。

+   `*` 匹配零个或多个字符的任意序列。

要指定额外的模式，请使用`--histignore`选项或设置`MYSQL_HISTIGNORE`环境变量。（如果两者都指定，则选项值优先。）该值应该是一个或多个以冒号分隔的模式列表，这些模式将附加到默认模式列表。

在命令行上指定的模式可能需要用引号引起或转义，以防止您的命令解释器将其视为特殊字符。例如，为了除了忽略涉及密码的语句外，还抑制`UPDATE`和`DELETE`语句的记录，请像这样调用**mysql**：

```sql
mysql --histignore="*UPDATE*:*DELETE*"
```

##### 控制历史文件

`.mysql_history`文件应受到限制访问模式的保护，因为可能会写入敏感信息，例如包含密码的 SQL 语句的文本。当使用**上箭头**键召回历史记录时，文件中的语句可从**mysql**客户端访问。请参阅禁用交互式历史记录。

如果您不想维护历史文件，请先删除`.mysql_history`（如果存在）。然后使用以下任一技术防止它再次被创建：

+   将`MYSQL_HISTFILE`环境变量设置为`/dev/null`。要使此设置每次登录时生效，请将其放入您的 shell 启动文件之一。

+   创建`.mysql_history`作为指向`/dev/null`的符号链接；这只需要做一次：

    ```sql
    ln -s /dev/null $HOME/.mysql_history
    ```

##### syslog 日志特性

如果给出`--syslog`选项，**mysql**将交互式语句写入系统日志设施。消息记录具有以下特性。

记录发生在“信息”级别。这对应于 Unix/Linux 上`syslog`功能的`LOG_INFO`优先级和 Windows 事件日志的`EVENTLOG_INFORMATION_TYPE`。请参考您的系统文档以配置您的日志功能。

消息大小限制为 1024 字节。

消息由标识符`MysqlClient`后跟这些值组成：

+   `SYSTEM_USER`

    操作系统用户名（登录名）或如果用户未知则为`--`。

+   `MYSQL_USER`

    MySQL 用户名（使用`--user`选项指定）或者`--`如果用户未知。

+   `CONNECTION_ID`：

    客户端连接标识符。这与会话中的`CONNECTION_ID()`函数值相同。

+   `DB_SERVER`

    服务器主机或者`--`如果主机未知。

+   `DB`

    默认数据库或者`--`如果没有选择数据库。

+   `QUERY`

    记录语句的文本。

这是在 Linux 上使用`--syslog`生成的输出示例。此输出已经格式化以便阅读；每个记录的消息实际上占据一行。

```sql
Mar  7 12:39:25 myhost MysqlClient[20824]:
  SYSTEM_USER:'oscar', MYSQL_USER:'my_oscar', CONNECTION_ID:23,
  DB_SERVER:'127.0.0.1', DB:'--', QUERY:'USE test;'
Mar  7 12:39:28 myhost MysqlClient[20824]:
  SYSTEM_USER:'oscar', MYSQL_USER:'my_oscar', CONNECTION_ID:23,
  DB_SERVER:'127.0.0.1', DB:'test', QUERY:'SHOW TABLES;'
```
