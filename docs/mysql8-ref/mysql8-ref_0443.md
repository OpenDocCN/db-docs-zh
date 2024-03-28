> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-filter-definitions.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-filter-definitions.html)

#### 8.4.5.8 编写审计日志过滤器定义

过滤器定义是`JSON`值。有关在 MySQL 中使用`JSON`数据的信息，请参见第 13.5 节，“JSON 数据类型”。

过滤器定义采用以下形式，其中*`actions`*表示过滤如何进行：

```sql
{ "filter": *actions* }
```

以下讨论描述了过滤器定义中允许的结构。

+   记录所有事件

+   记录特定事件类

+   记录特定事件子类

+   包容性和排他性日志记录

+   测试事件字段值

+   阻止特定事件的执行

+   逻辑运算符

+   引用预定义变量

+   引用预定义函数

+   替换事件字段值

+   替换用户过滤器

##### 记录所有事件

要明确启用或禁用所有事件的记录，请在过滤器中使用`log`项：

```sql
{
  "filter": { "log": true }
}
```

`log`值可以是`true`或`false`。

前面的过滤器启用了所有事件的记录。这等同于：

```sql
{
  "filter": { }
}
```

记录行为取决于`log`值以及是否指定了`class`或`event`项：

+   如果指定了`log`，则使用给定的值。

+   如果未指定`log`，则如果未指定`class`或`event`项，则记录为`true`，否则为`false`（在这种情况下，`class`或`event`可以包括自己的`log`项）。

##### 记录特定事件类

要记录特定类别的事件，请在过滤器中使用`class`项，其��`name`字段表示要记录的类别的名称：

```sql
{
  "filter": {
    "class": { "name": "connection" }
  }
}
```

`name`值可以是`connection`、`general`或`table_access`，分别用于记录连接、一般或表访问事件。

上述过滤器启用了`connection`类中事件的记录。它等同于以下过滤器，其中`log`项目被明确指定：

```sql
{
  "filter": {
    "log": false,
    "class": { "log": true,
               "name": "connection" }
  }
}
```

要启用多个类别的记录，将`class`值定义为`JSON`数组元素，命名这些类别：

```sql
{
  "filter": {
    "class": [
      { "name": "connection" },
      { "name": "general" },
      { "name": "table_access" }
    ]
  }
}
```

注意

当在过滤器定义中同一级别出现给定项目的多个实例时，可以将项目值合并为数组值中的单个项目实例。上述定义可以写成这样：

```sql
{
  "filter": {
    "class": [
      { "name": [ "connection", "general", "table_access" ] }
    ]
  }
}
```

##### 记录特定事件子类

要选择特定的事件子类，使用包含命名子类的`name`项目的`event`项目。由`event`项目选择的事件的默认操作是记录它们。例如，此过滤器启用了命名事件子类的记录：

```sql
{
  "filter": {
    "class": [
      {
        "name": "connection",
        "event": [
          { "name": "connect" },
          { "name": "disconnect" }
        ]
      },
      { "name": "general" },
      {
        "name": "table_access",
        "event": [
          { "name": "insert" },
          { "name": "delete" },
          { "name": "update" }
        ]
      }
    ]
  }
}
```

`event`项目还可以包含显式的`log`项目，以指示是否记录符合条件的事件。此`event`项目选择多个事件，并明确指示它们的记录行为：

```sql
"event": [
  { "name": "read", "log": false },
  { "name": "insert", "log": true },
  { "name": "delete", "log": true },
  { "name": "update", "log": true }
]
```

`event`项目还可以指示是否阻止符合条件的事件，如果包含`abort`项目。详情请参见阻止特定事件的执行。

表 8.35，“事件类和子类组合”描述了每个事件类的允许子类值。

**表 8.35 事件类和子类组合**

| 事件类 | 事件子类 | 描述 |
| --- | --- | --- |
| `connection` | `connect` | 连接初始化（成功或失败） |
| `connection` | `change_user` | 会话期间使用不同用户/密码重新认证 |
| `connection` | `disconnect` | 连接终止 |
| `general` | `status` | 一般操作信息 |
| `message` | `internal` | 内部生成的消息 |
| `message` | `user` | 由`audit_api_message_emit_udf()`生成的消息 |
| `table_access` | `read` | 表读取语句，如`SELECT`或`INSERT INTO ... SELECT` |
| `table_access` | `delete` | 表删除语句，如`DELETE`或`TRUNCATE TABLE` |
| `table_access` | `insert` | 表插入语句，如`INSERT`或`REPLACE` |
| `table_access` | `update` | 表更新语句，如`UPDATE` |
| 事件类 | 事件子类 | 描述 |

表 8.36，“每个事件类和子类组合的记录和中止特性”描述了每个事件子类是否可以被记录或中止。

**表 8.36 每个事件类和子类组合的记录和中止特性**

| 事件类 | 事件子类 | 可以被记录 | 可以被中止 |
| --- | --- | --- | --- |
| `connection` | `connect` | 是 | 否 |
| `connection` | `change_user` | 是 | 否 |
| `connection` | `disconnect` | 是 | 否 |
| `general` | `status` | 是 | 否 |
| `message` | `internal` | 是 | 是 |
| `message` | `user` | 是 | 是 |
| `table_access` | `read` | 是 | 是 |
| `table_access` | `delete` | 是 | 是 |
| `table_access` | `insert` | 是 | 是 |
| `table_access` | `update` | 是 | 是 |
| 事件类 | 事件子类 | 可以被记录 | 可以被中止 |

##### 包容性和独占性记录

可以定义包容性或独占性模式的过滤器：

+   包容性模式仅记录明确指定的项目。

+   独占模式记录除明确指定的项目之外的所有内容。

要执行包容性记录，全局禁用记录并为特定类启用记录。此过滤器记录`connection`类中的`connect`和`disconnect`事件，以及`general`类中的事件：

```sql
{
  "filter": {
    "log": false,
    "class": [
      {
        "name": "connection",
        "event": [
          { "name": "connect", "log": true },
          { "name": "disconnect", "log": true }
        ]
      },
      { "name": "general", "log": true }
    ]
  }
}
```

要执行独占性记录，全局启用记录并为特定类禁用记录。此过滤器记录除`general`类中的事件之外的所有内容：

```sql
{
  "filter": {
    "log": true,
    "class":
      { "name": "general", "log": false }
  }
}
```

此过滤器记录`connection`类中的`change_user`事件，`message`事件和`table_access`事件，因为*不*记录其他所有内容：

```sql
{
  "filter": {
    "log": true,
    "class": [
      {
        "name": "connection",
        "event": [
          { "name": "connect", "log": false },
          { "name": "disconnect", "log": false }
        ]
      },
      { "name": "general", "log": false }
    ]
  }
}
```

##### 测试事件字段值

要基于特定事件字段值启用记录，请在指示字段名称及其预期值的`log`项内指定一个`field`项：

```sql
{
  "filter": {
    "class": {
    "name": "general",
      "event": {
        "name": "status",
        "log": {
          "field": { "name": "general_command.str", "value": "Query" }
        }
      }
    }
  }
}
```

每个事件包含特定于事件类的字段，可以从过滤器内部访问以执行自定义过滤。

`connection`类中的事件指示会话期间发生的与连接相关的活动，例如用户连接到服务器或从服务器断开连接。表 8.37，“连接事件字段”指示了`connection`事件的允许字段。

**表 8.37 连接事件字段**

| 字段名称 | 字段类型 | 描述 |
| --- | --- | --- |
| `status` | 整数 | 事件状态：0：成功否则：失败 |
| `connection_id` | 无符号整数 | 连接 ID |
| `user.str` | 字符串 | 在认证期间指定的用户名 |
| `user.length` | 无符号整数 | 用户名称长度 |
| `priv_user.str` | 字符串 | 已认证的用户名称（帐户用户名） |
| `priv_user.length` | 无符号整数 | 已认证的用户名称长度 |
| `external_user.str` | 字符串 | 外部用户名称（由第三方认证插件提供） |
| `external_user.length` | 无符号整数 | 外部用户名称长度 |
| `proxy_user.str` | string | 代理用户名称 |
| `proxy_user.length` | unsigned integer | 代理用户名称长度 |
| `host.str` | string | 连接用户主机 |
| `host.length` | unsigned integer | 连接用户主机长度 |
| `ip.str` | string | 连接用户 IP 地址 |
| `ip.length` | unsigned integer | 连接用户 IP 地址长度 |
| `database.str` | string | 连接时指定的数据库名称 |
| `database.length` | unsigned integer | 数据库名称长度 |
| `connection_type` | integer | 连接类型：0 或 `"::undefined"`: 未定义 1 或 `"::tcp/ip"`: TCP/IP2 或 `"::socket"`: Socket3 或 `"::named_pipe"`: 命名管道 4 或 `"::ssl"`: 带加密的 TCP/IP5 或 `"::shared_memory"`: 共享内存 |
| 字段名称 | 字段类型 | 描述 |

`"::*`xxx`*"` 值是符号伪常量，可以代替文字数字值。 它们必须被引用为字符串，并且区分大小写。

`general` 类中的事件指示操作的状态代码及其详细信息。 表 8.38，“通用事件字段” 指示 `general` 事件的允许字段。

**表 8.38 通用事件字段**

| 字段名称 | 字段类型 | 描述 |
| --- | --- | --- |
| `general_error_code` | integer | 事件状态：0: OK 否则: 失败 |
| `general_thread_id` | unsigned integer | 连接/线程 ID |
| `general_user.str` | string | 认证期间指定的用户名 |
| `general_user.length` | unsigned integer | 用户��称长度 |
| `general_command.str` | string | 命令名称 |
| `general_command.length` | unsigned integer | 命令名称长度 |
| `general_query.str` | string | SQL 语句文本 |
| `general_query.length` | unsigned integer | SQL 语句文本长度 |
| `general_host.str` | string | 主机名 |
| `general_host.length` | unsigned integer | 主机名长度 |
| `general_sql_command.str` | string | SQL 命令类型名称 |
| `general_sql_command.length` | unsigned integer | SQL 命令类型名称长度 |
| `general_external_user.str` | string | 外部用户名称（由第三方认证插件提供） |
| `general_external_user.length` | unsigned integer | 外部用户名称长度 |
| `general_ip.str` | string | 连接用户 IP 地址 |
| `general_ip.length` | unsigned integer | 连接用户 IP 地址长度 |
| 字段名称 | 字段类型 | 描述 |

`general_command.str` 指示命令名称：`Query`、`Execute`、`Quit` 或 `Change user`。

一个 `general` 事件，其中 `general_command.str` 字段设置为 `Query` 或 `Execute`，包含 `general_sql_command.str` 设置为指定 SQL 命令类型的值：`alter_db`、`alter_db_upgrade`、`admin_commands` 等。 可以在此语句显示的性能模式工具的最后组件中看到可用的 `general_sql_command.str` 值：

```sql
mysql> SELECT NAME FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'statement/sql/%' ORDER BY NAME;
+---------------------------------------+
| NAME                                  |
+---------------------------------------+
| statement/sql/alter_db                |
| statement/sql/alter_db_upgrade        |
| statement/sql/alter_event             |
| statement/sql/alter_function          |
| statement/sql/alter_instance          |
| statement/sql/alter_procedure         |
| statement/sql/alter_server            |
...
```

`table_access` 类中的事件提供有关对表的特定类型访问的信息。Table 8.39, “Table-Access Event Fields” 指示了`table_access`事件的允许字段。

**Table 8.39 Table-Access Event Fields**

| Field Name | Field Type | 描述 |
| --- | --- | --- |
| `connection_id` | 无符号整数 | 事件连接 ID |
| `sql_command_id` | integer | SQL 命令 ID |
| `query.str` | string | SQL 语句文本 |
| `query.length` | 无符号整数 | SQL 语句文本长度 |
| `table_database.str` | string | 与事件相关联的数据库名 |
| `table_database.length` | 无符号整数 | 数据库名长度 |
| `table_name.str` | string | 与事件相关联的表名 |
| `table_name.length` | 无符号整数 | 表名长度 |

以下列表显示了哪些语句产生哪些表访问事件：

+   `read` 事件：

    +   `SELECT`

    +   `INSERT ... SELECT`（用于`SELECT`子句中引用的表）

    +   `REPLACE ... SELECT`（用于`SELECT`子句中引用的表）

    +   `UPDATE ... WHERE`（用于`WHERE`子句中引用的表）

    +   `HANDLER ... READ`

+   `delete` 事件：

    +   `DELETE`

    +   `TRUNCATE TABLE`

+   `insert` 事件：

    +   `INSERT`

    +   `INSERT ... SELECT`（用于`INSERT`子句中引用的表）

    +   `REPLACE`

    +   `REPLACE ... SELECT`（用于`REPLACE`子句中引用的表）

    +   `LOAD DATA`

    +   `LOAD XML`

+   `update` 事件：

    +   `UPDATE`

    +   `UPDATE ... WHERE`（用于`UPDATE`子句中引用的表）

##### 阻止特定事件的执行

`event` 项可以包括一个`abort`项，指示是否阻止符合条件的事件执行。`abort`使得可以编写规则来阻止特定 SQL 语句的执行。

重要

理论上，具有足够权限的用户可能会错误地在审计日志过滤器中创建一个`abort`项，从而阻止自己和其他管理员访问系统。从 MySQL 8.0.28 开始，`AUDIT_ABORT_EXEMPT`权限可用于允许用户帐户的查询始终执行，即使`abort`项会阻止它们。具有此权限的帐户因此可以用于在审计配置错误后恢复对系统的访问。查询仍然记录在审计日志中，但不会被拒绝，而是由于权限而被允许。

在 MySQL 8.0.28 或更高版本中创建的带有`SYSTEM_USER`权限的帐户在创建时会自动分配`AUDIT_ABORT_EXEMPT`权限。在进行 MySQL 8.0.28 或更高版本的升级过程时，如果没有现有帐户被分配该权限，则现有帐户也会被分配`AUDIT_ABORT_EXEMPT`权限。

`abort`项必须出现在`event`项内。例如：

```sql
"event": {
  "name": *qualifying event subclass names*
  "abort": *condition*
}
```

对于由`name`项选择的事件子类，`abort`操作为真或假，取决于*`condition`*的评估。如果条件评估为真，则阻止事件。否则，事件继续执行。

*`condition`*规范可以简单到`true`或`false`，也可以更复杂，以至于评估取决于事件特征。

此过滤器阻止`INSERT`、`UPDATE`和`DELETE`语句：

```sql
{
  "filter": {
    "class": {
      "name": "table_access",
      "event": {
        "name": [ "insert", "update", "delete" ],
        "abort": true
      }
    }
  }
}
```

这个更复杂的过滤器仅针对特定表（`finances.bank_account`）阻止相同的语句：

```sql
{
  "filter": {
    "class": {
      "name": "table_access",
      "event": {
        "name": [ "insert", "update", "delete" ],
        "abort": {
          "and": [
            { "field": { "name": "table_database.str", "value": "finances" } },
            { "field": { "name": "table_name.str", "value": "bank_account" } }
          ]
        }
      }
    }
  }
}
```

由过滤器匹配并阻止的语句会向客户端返回错误：

```sql
ERROR 1045 (28000): Statement was aborted by an audit log filter
```

并非所有事件都可以被阻止（参见表 8.36，“每个事件类和子类组合的日志和中止特性”）。对于无法被阻止的事件，审计日志会向错误日志写入警告，而不是阻止它。

尝试定义一个过滤器，其中`abort`项出现在`event`项之外时，会出现错误。

##### 逻辑运算符

逻辑运算符（`and`、`or`、`not`）允许构建复杂条件，从而编写更高级的过滤配置。以下`log`项仅记录具有特定值和长度的`general_command`字段的`general`事件：

```sql
{
  "filter": {
    "class": {
      "name": "general",
      "event": {
        "name": "status",
        "log": {
          "or": [
            {
              "and": [
                { "field": { "name": "general_command.str",    "value": "Query" } },
                { "field": { "name": "general_command.length", "value": 5 } }
              ]
            },
            {
              "and": [
                { "field": { "name": "general_command.str",    "value": "Execute" } },
                { "field": { "name": "general_command.length", "value": 7 } }
              ]
            }
          ]
        }
      }
    }
  }
}
```

##### 引用预定义变量

要在`log`条件中引用预定义变量，请使用`variable`项，该项接受`name`和`value`项并测试命名变量与给定值的相等性：

```sql
"variable": {
  "name": "*variable_name*",
  "value": *comparison_value*
}
```

如果*`variable_name`*具有值*`comparison_value`*，则为真，否则为假。

示例：

```sql
{
  "filter": {
    "class": {
      "name": "general",
      "event": {
        "name": "status",
        "log": {
          "variable": {
            "name": "audit_log_connection_policy_value",
            "value": "::none"
          }
        }
      }
    }
  }
}
```

每个预定义变量对应一个系统变量。通过编写测试预定义变量的过滤器，您可以通过设置相应的系统变量来修改过滤器操作，而无需重新定义过滤器。例如，通过编写测试`audit_log_connection_policy_value`预定义变量值的过滤器，您可以通过更改`audit_log_connection_policy`系统变量的值来修改过滤器操作。

`audit_log_*`xxx`*_policy`系统变量用于废弃的传统模式审计日志（参见第 8.4.5.10 节，“传统模式审计日志过滤”）。使用基于规则的审计日志过滤时，这些变量仍然可见（例如，使用`SHOW VARIABLES`），但除非编写包含引用它们的结构的过滤器，否则对它们的更改不会产生任何效果。

以下列表描述了`variable`项的允许的预定义变量：

+   `audit_log_connection_policy_value`

    此变量对应于`audit_log_connection_policy`系统变量的值。该值是一个无符号整数。表 8.40，“audit_log_connection_policy_value Values”显示了允许的值以及相应的`audit_log_connection_policy`值。

    **表 8.40 audit_log_connection_policy_value Values**

    | 值 | 对应的 audit_log_connection_policy 值 |
    | --- | --- |
    | `0` 或 `"::none"` | `NONE` |
    | `1` 或 `"::errors"` | `ERRORS` |
    | `2` 或 `"::all"` | `ALL` |

    `"::*`xxx`*"`值是象征性伪常量，可以代替文字数字值。它们必须被引用为字符串，并且区分大小写。

+   `audit_log_policy_value`

    此变量对应于`audit_log_policy`系统变量的值。该值是一个无符号整数。表 8.41，“audit_log_policy_value Values”显示了允许的值以及相应的`audit_log_policy`值。

    **表 8.41 audit_log_policy_value Values**

    | 值 | 对应的 audit_log_policy 值 |
    | --- | --- |
    | `0` 或 `"::none"` | `NONE` |
    | `1` 或 `"::logins"` | `LOGINS` |
    | `2` 或 `"::all"` | `ALL` |
    | `3` 或 `"::queries"` | `QUERIES` |

    `"::*`xxx`*"`值是象征性伪常量，可以代替文字数字值。它们必须被引用为字符串，并且区分大小写。

+   `audit_log_statement_policy_value`

    此变量对应于`audit_log_statement_policy`系统变量的值。该值是一个无符号整数。表 8.42，“audit_log_statement_policy_value Values”显示了允许的值以及相应的`audit_log_statement_policy`值。

    **表 8.42 audit_log_statement_policy_value Values**

    | 值 | 对应的 audit_log_statement_policy 值 |
    | --- | --- |
    | `0` 或 `"::none"` | `NONE` |
    | `1` 或 `"::errors"` | `ERRORS` |
    | `2` 或 `"::all"` | `ALL` |

    `"::*`xxx`*"`值是象征性伪常量，可以代替文字数字值。它们必须被引用为字符串，并且区分大小写。

##### 引用预定义函数

要在 `log` 条件中引用预定义函数，请使用 `function` 项，该项接受 `name` 和 `args` 项分别指定函数名称及其参数：

```sql
"function": {
  "name": "*function_name*",
  "args": *arguments*
}
```

`name` 项应仅指定函数名称，不包括括号或参数列表。

`args` 项必须满足以下条件：

+   如果函数不需要参数，则不应提供 `args` 项。

+   如果函数需要参数，则需要一个 `args` 项，并且参数必须按照函数描述中列出的顺序给出。参数可以是预定义变量、事件字段或字符串或数字常量。

如果参数数量不正确或参数类型与函数所需的正确数据类型不符，将会出现错误。

示例：

```sql
{
  "filter": {
    "class": {
      "name": "general",
      "event": {
        "name": "status",
        "log": {
          "function": {
            "name": "find_in_include_list",
            "args": [ { "string": [ { "field": "user.str" },
                                    { "string": "@"},
                                    { "field": "host.str" } ] } ]
          }
        }
      }
    }
  }
}
```

前面的过滤器根据当前用户是否在 `audit_log_include_accounts` 系统变量中找到来确定是否记录 `general` 类 `status` 事件。该用户是使用事件中的字段构建的。

以下列表描述了 `function` 项的允许预定义函数：

+   `audit_log_exclude_accounts_is_null()`

    检查 `audit_log_exclude_accounts` 系统变量是否为 `NULL`。在定义与传统审计日志实现对应的过滤器时，此函数可能会有所帮助。

    参数：

    无。

+   `audit_log_include_accounts_is_null()`

    检查 `audit_log_include_accounts` 系统变量是否为 `NULL`。在定义与传统审计日志实现对应的过滤器时，此函数可能会有所帮助。

    参数：

    无。

+   `debug_sleep(millisec)`

    休眠指定的毫秒数。此函数用于性能测量。

    `debug_sleep()` 仅适用于调试版本。

    参数：

    +   *`millisec`*：指定要休眠的毫秒数的无符号整数。

+   `find_in_exclude_list(account)`

    检查账户字符串是否存在于审计日志排除列表中（`audit_log_exclude_accounts` 系统变量的值）。

    参数：

    +   *`account`*：指定用户账户名称的字符串。

+   `find_in_include_list(account)`

    检查账户字符串是否存在于审计日志包含列表中（`audit_log_include_accounts` 系统变量的值）。

    参数：

    +   *`account`*：指定用户账户名称的字符串。

+   `query_digest([str])`

    此函数具有不同的行为，具体取决于是否给出参数：

    +   没有参数时，`query_digest` 返回与当前事件中语句文本对应的语句摘要值。

    +   有参数时，`query_digest` 返回一个布尔值，指示参数是否等于当前语句摘要。

    参数：

    +   *`str`*：此参数是可选的。如果提供，则指定要与当前事件中语句的摘要进行比较的语句摘要。

    例子：

    此`function`项不包含参数，因此`query_digest`将当前语句摘要作为字符串返回：

    ```sql
    "function": {
      "name": "query_digest"
    }
    ```

    此`function`项包含一个参数，因此`query_digest`返回一个布尔值，指示参数是否等于当前语句摘要：

    ```sql
    "function": {
      "name": "query_digest",
      "args": "SELECT ?"
    }
    ```

    此函数是在 MySQL 8.0.26 中添加的。

+   `string_find(text, substr)`

    检查`substr`值是否包含在`text`值中。此搜索区分大小写。

    参数：

    +   *`text`*：要搜索的文本字符串。

    +   *`substr`*：要在*`text`*中搜索的子字符串。

##### 事件字段值的替换

截至 MySQL 8.0.26，审计过滤器定义支持替换某些审计事件字段，以便记录的事件包含替换值而不是原始值。此功能使记录的审计记录可以包含语句摘要而不是文字语句，这对于可能暴露敏感值的 MySQL 部署非常有用。

审计事件中的字段替换工作原理如下：

+   字段替换在审计过滤器定义中指定，因此必须按照第 8.4.5.7 节，“审计日志过滤”中描述的方式启用审计日志过滤。

+   并非所有字段都可以被替换。表 8.43，“可替换事件字段”显示了哪些字段可以在哪些事件类中被替换。

    **表 8.43 可替换事件字段**

    | 事件类 | 字段名称 |
    | --- | --- |
    | `general` | `general_query.str` |
    | `table_access` | `query.str` |

+   替换是有条件的。过滤器定义中的每个替换规范都包括一个条件，使得可根据条件结果更改或保持替换字段不变。

+   如果发生替换，替换规范将使用允许用于此目的的函数指示替换值。

如表 8.43，“可替换事件字段”所示，目前唯一可替换的字段是包含语句文本的字段（出现在`general`和`table_access`类事件中）。此外，唯一允许用于指定替换值的函数是`query_digest`。这意味着唯一允许的替换操作是将语句文字文本替换为其对应的摘要。

因为字段替换发生在早期的审计阶段（在过滤期间），所以无论稍后写入的日志格式是 XML 还是 JSON 输出（即，审计日志插件生成的日志格式），都需要选择是写入语句文字文本还是摘要值。

字段替换可以在不同级别的事件粒度上进行：

+   要为类中的所有事件执行字段替换，请在类级别过滤事件。

+   要更细粒度地执行替换，可以包括额外的事件选择项。例如，您可以仅针对给定事件类的特定子类执行字段替换，或仅在具有特定特征的字段的事件中执行。

在过滤器定义中，通过包含`print`项来指定字段替换，其语法如下：

```sql
"print": {
  "field": {
    "name": "*field_name*",
    "print": *condition*,
    "replace": *replacement_value*
  }
}
```

在`print`项中，其`field`项使用这三个项目来指示替换是如何发生的：

+   `name`：替换（可能）发生的字段。*`field_name`*必须是 Table 8.43，“可替换事件字段”中显示的字段之一。

+   `print`：确定是保留原始字段值还是替换它的条件：

    +   如果*`condition`*评估为`true`，则该字段保持不变。

    +   如果*`condition`*评估为`false`，则发生替换，使用`replace`项的值。

    要无条件替换字段，请指定条件如下：

    ```sql
    "print": false
    ```

+   `replace`：当`print`条件评估为`false`时要使用的替换值。使用`function`项指定*`replacement_value`*。

例如，此过滤器定义适用于`general`类中的所有事件，将语句文字替换为其摘要：

```sql
{
  "filter": {
    "class": {
      "name": "general",
      "print": {
        "field": {
          "name": "general_query.str",
          "print": false,
          "replace": {
            "function": {
              "name": "query_digest"
            }
          }
        }
      }
    }
  }
}
```

前面的过滤器使用这个`print`项来无条件地通过其摘要值替换`general_query.str`中包含的语句文字：

```sql
"print": {
  "field": {
    "name": "general_query.str",
    "print": false,
    "replace": {
      "function": {
        "name": "query_digest"
      }
    }
  }
}
```

`print`项可以以不同的方式编写以实现不同的替换策略。刚刚显示的`replace`项使用这个`function`结构指定替换文本，以返回表示当前语句摘要的字符串：

```sql
"function": {
  "name": "query_digest"
}
```

`query_digest`函数也可以以另一种方式使用，作为返回布尔值的比较器，从而使其在`print`条件中可用。为此，提供一个参数，指定一个比较语句摘要：

```sql
"function": {
  "name": "query_digest",
  "args": "*digest*"
}
```

在这种情况下，`query_digest`根据当前语句摘要是否与比较摘要相同返回`true`或`false`。以这种方式使用`query_digest`使得过滤器定义能够检测与特定摘要匹配的语句。以下结构中的条件仅对具有等于`SELECT ?`的摘要的语句为真，因此仅对不匹配摘要的语句进行替换：

```sql
"print": {
  "field": {
    "name": "general_query.str",
    "print": {
      "function": {
        "name": "query_digest",
        "args": "SELECT ?"
      }
    },
    "replace": {
      "function": {
        "name": "query_digest"
      }
    }
  }
}
```

仅为匹配摘要的语句执行替换，使用`not`来反转条件：

```sql
"print": {
  "field": {
    "name": "general_query.str",
    "print": {
      "not": {
        "function": {
          "name": "query_digest",
          "args": "SELECT ?"
        }
      }
    },
    "replace": {
      "function": {
        "name": "query_digest"
      }
    }
  }
}
```

假设您希望审计日志仅包含语句摘要而不是文字语句。为实现此目的，您必须对包含语句文本的所有事件执行替换；即`general`和`table_access`类别中的事件。较早的过滤器定义显示了如何无条件地替换`general`事件的语句文本。要对`table_access`事件执行相同操作，请使用类似但将类别从`general`更改为`table_access`，字段名称从`general_query.str`更改为`query.str`的过滤器：

```sql
{
  "filter": {
    "class": {
      "name": "table_access",
      "print": {
        "field": {
          "name": "query.str",
          "print": false,
          "replace": {
            "function": {
              "name": "query_digest"
            }
          }
        }
      }
    }
  }
}
```

结合`general`和`table_access`过滤器会产生一个执行所有包含语句文本事件替换的单一过滤器：

```sql
{
  "filter": {
    "class": [
      {
        "name": "general",
        "print": {
          "field": {
            "name": "general_query.str",
            "print": false,
            "replace": {
              "function": {
                "name": "query_digest"
              }
            }
          }
        }
      },
      {
        "name": "table_access",
        "print": {
          "field": {
            "name": "query.str",
            "print": false,
            "replace": {
              "function": {
                "name": "query_digest"
              }
            }
          }
        }
      }
    ]
  }
}
```

仅对类别中的某些事件执行替换，向过滤器添加指示更具体替换发生时间的项目。以下过滤器适用于`table_access`类中的事件，但仅对`insert`和`update`事件执行替换（保持`read`和`delete`事件不变）：

```sql
{
  "filter": {
    "class": {
      "name": "table_access",
      "event": {
        "name": [
          "insert",
          "update"
        ],
        "print": {
          "field": {
            "name": "query.str",
            "print": false,
            "replace": {
              "function": {
                "name": "query_digest"
              }
            }
          }
        }
      }
    }
  }
}
```

该过滤器对应于列出的账户管理语句的`general`类事件执行替换（效果是隐藏语句中的凭据和数据值）：

```sql
{
  "filter": {
    "class": {
      "name": "general",
      "event": {
        "name": "status",
        "print": {
          "field": {
            "name": "general_query.str",
            "print": false,
            "replace": {
              "function": {
                "name": "query_digest"
              }
            }
          }
        },
        "log": {
          "or": [
            {
              "field": {
                "name": "general_sql_command.str",
                "value": "alter_user"
              }
            },
            {
              "field": {
                "name": "general_sql_command.str",
                "value": "alter_user_default_role"
              }
            },
            {
              "field": {
                "name": "general_sql_command.str",
                "value": "create_role"
              }
            },
            {
              "field": {
                "name": "general_sql_command.str",
                "value": "create_user"
              }
            }
          ]
        }
      }
    }
  }
}
```

有关可能的`general_sql_command.str`值的信息，请参阅测试事件字段值。

##### 替换用户过滤器

在某些情况下，过滤器定义可以动态更改。为此，在现有的`filter`中定义一个`filter`配置。例如：

```sql
{
  "filter": {
    "id": "main",
    "class": {
      "name": "table_access",
      "event": {
        "name": [ "update", "delete" ],
        "log": false,
        "filter": {
          "class": {
            "name": "general",
            "event" : { "name": "status",
                        "filter": { "ref": "main" } }
          },
          "activate": {
            "or": [
              { "field": { "name": "table_name.str", "value": "temp_1" } },
              { "field": { "name": "table_name.str", "value": "temp_2" } }
            ]
          }
        }
      }
    }
  }
}
```

当子过滤器中的`activate`项评估为`true`时，将激活新的过滤器。在顶层`filter`中使用`activate`是不允许的。

通过在子过滤器中使用`ref`项引用原始过滤器`id`，可以将新过滤器替换为原始过滤器。

所示的过滤器的操作方式如下：

+   `main`过滤器等待`table_access`事件，无论是`update`还是`delete`。

+   如果在`temp_1`或`temp_2`表上发生`update`或`delete` `table_access`事件，则将过滤器替换为内部过滤器（没有`id`，因为没有必要显式引用它）。

+   如果命令结束被标记（`general` / `status`事件），则会向审计日志文件写入条目，并将过滤器替换为`main`过滤器。

该过滤器用于记录更新或删除`temp_1`或`temp_2`表中任何内容的语句，例如：

```sql
UPDATE temp_1, temp_3 SET temp_1.a=21, temp_3.a=23;
```

该语句生成多个`table_access`事件，但审计日志文件仅包含`general` / `status`条目。

注意

在定义中使用的任何`id`值仅与该定义相关。它们与`audit_log_filter_id`系统变量的值无关。
