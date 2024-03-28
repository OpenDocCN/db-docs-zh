# 8.4.6 审计消息组件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-api-message-emit.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-api-message-emit.html)

截至 MySQL 8.0.14，`audit_api_message_emit` 组件使应用程序能够使用 `audit_api_message_emit_udf()` 函数将自己的消息事件添加到审计日志中。

`audit_api_message_emit` 组件与所有类型为审计的插件合作。为了具体起见，示例使用了第 8.4.5 节，“MySQL 企业审计”中描述的 `audit_log` 插件。

+   安装或卸载审计消息组件

+   审计消息函数

#### 安装或卸载审计消息组件

要使服务器可用，组件库文件必须位于 MySQL 插件目录中（由 `plugin_dir` 系统变量命名的目录）。如果需要，通过在服务器启动时设置 `plugin_dir` 的值来配置插件目录位置。

要安装 `audit_api_message_emit` 组件，请使用以下语句：

```sql
INSTALL COMPONENT "file://component_audit_api_message_emit";
```

组件安装是一次性操作，不需要每次服务器启动都执行。`INSTALL COMPONENT` 加载组件，并在 `mysql.component` 系统表中注册它，以便在后续服务器启动时加载它。

要卸载 `audit_api_message_emit` 组件，请使用以下语句：

```sql
UNINSTALL COMPONENT "file://component_audit_api_message_emit";
```

`UNINSTALL COMPONENT` 卸载组件，并从 `mysql.component` 系统表中注销它，以使其在后续服务器启动时不加载。

因为安装和卸载 `audit_api_message_emit` 组件会安装和卸载组件实现的 `audit_api_message_emit_udf()` 函数，所以不需要使用 `CREATE FUNCTION` 或 `DROP FUNCTION` 来执行此操作。

#### 审计消息函数

本节描述了 `audit_api_message_emit` 组件实现的 `audit_api_message_emit_udf()` 函数。

在使用审计消息函数之前，请根据安装或卸载审计消息组件中提供的说明安装审计消息组件。

+   [`audit_api_message_emit_udf(*`component`*, *`producer`*, *`message`*[, *`key`*, *`value`*] ...)`](audit-api-message-emit.html#function_audit-api-message-emit-udf)

    向审计日志添加消息事件。消息事件包括调用者选择的组件、生产者和消息字符串，以及可选的一组键值对。

    由此函数发布的事件发送到所有已启用的审计类型插件，每个插件根据自己的规则处理事件。如果未启用任何审计类型的插件，则发布事件不会产生任何效果。

    参数：

    +   *`component`*: 一个指定组件名称的字符串。

    +   *`producer`*: 一个指定生产者名称的字符串。

    +   *`message`*: 一个指定事件消息的字符串。

    +   *`key`*, *`value`*: 事件可能包括 0 或多个键值对，指定任意应用程序提供的数据映射。每个 *`key`* 参数是一个指定其紧随其后的 *`value`* 参数名称的字符串。每个 *`value`* 参数指定其紧随其后的 *`key`* 参数的值。每个 *`value`* 可以是字符串或数值，或 `NULL`。

    返回值：

    字符串 `OK` 表示成功。如果函数失败，则会发生错误。

    示例：

    ```sql
    mysql> SELECT audit_api_message_emit_udf('component_text',
                                             'producer_text',
                                             'message_text',
                                             'key1', 'value1',
                                             'key2', 123,
                                             'key3', NULL) AS 'Message';
    +---------+
    | Message |
    +---------+
    | OK      |
    +---------+
    ```

    附加信息：

    由 `audit_api_message_emit_udf()` 接收到的事件由每个启用的审计类型插件以特定于插件的格式记录。例如，`audit_log` 插件（参见 Section 8.4.5, “MySQL Enterprise Audit”）根据由 `audit_log_format` 系统变量配置的日志格式记录消息值，具体取决于配置的日志格式：

    +   JSON 格式（`audit_log_format=JSON`）：

        ```sql
        {
          ...
          "class": "message",
          "event": "user",
          ...
          "message_data": {
            "component": "component_text",
            "producer": "producer_text",
            "message": "message_text",
            "map": {
              "key1": "value1",
              "key2": 123,
              "key3": null
            }
          }
        }
        ```

    +   新式 XML 格式（`audit_log_format=NEW`）：

        ```sql
        <AUDIT_RECORD>
         ...
         <NAME>Message</NAME>
         ...
         <COMMAND_CLASS>user</COMMAND_CLASS>
         <COMPONENT>component_text</COMPONENT>
         <PRODUCER>producer_text</PRODUCER>
         <MESSAGE>message_text</MESSAGE>
         <MAP>
           <ELEMENT>
             <KEY>key1</KEY>
             <VALUE>value1</VALUE>
           </ELEMENT>
           <ELEMENT>
             <KEY>key2</KEY>
             <VALUE>123</VALUE>
           </ELEMENT>
           <ELEMENT>
             <KEY>key3</KEY>
             <VALUE/>
           </ELEMENT>
         </MAP>
        </AUDIT_RECORD>
        ```

    +   旧式 XML 格式（`audit_log_format=OLD`）：

        ```sql
        <AUDIT_RECORD
          ...
          NAME="Message"
          ...
          COMMAND_CLASS="user"
          COMPONENT="component_text"
          PRODUCER="producer_text"
          MESSAGE="message_text"/>
        ```

        注意

        以旧式 XML 格式记录的消息事件不包括键值映射，因为该格式施加的表现约束。

    由 `audit_api_message_emit_udf()` 发布的消息具有 `MYSQL_AUDIT_MESSAGE_CLASS` 的事件类别和 `MYSQL_AUDIT_MESSAGE_USER` 的子类别。（内部生成的审计消息具有相同的类别和 `MYSQL_AUDIT_MESSAGE_INTERNAL` 的子类别；目前未使用此子类别。）要在 `audit_log` 过滤规则中引用此类事件，请使用具有 `name` 值为 `message` 的 `class` 元素。例如：

    ```sql
    {
      "filter": {
        "class": {
          "name": "message"
        }
      }
    }
    ```

    如果需要区分用户生成的和内部生成的消息事件，请将 `subclass` 值与 `user` 或 `internal` 进行测试。

    不支持基于键值映射内容的过滤。

    有关编写过滤规则的信息，请���阅 Section 8.4.5.7, “Audit Log Filtering”。
