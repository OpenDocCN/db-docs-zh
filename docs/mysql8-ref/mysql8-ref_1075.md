> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-names.html`](https://dev.mysql.com/doc/refman/8.0/en/set-names.html)

#### 15.7.6.3 SET NAMES 语句

```sql
SET NAMES {'*charset_name*'
    [COLLATE '*collation_name*'] | DEFAULT}
```

这个语句将三个会话系统变量`character_set_client`，`character_set_connection`，和`character_set_results`设置为给定的字符集。将`character_set_connection`设置为`charset_name`也会将`collation_connection`设置为`charset_name`的默认排序规则。参见第 12.4 节，“连接字符集和排序规则”。

可选的`COLLATE`子句可用于显式指定排序规则。如果提供，排序规则必须是*`charset_name`*允许的排序规则之一。

*`charset_name`*和*`collation_name`*可以带引号或不带引号。

默认映射可以通过使用`DEFAULT`值来恢复。默认值取决于服务器配置。

一些字符集不能用作客户端字符集。尝试与`SET NAMES`一起使用它们会产生错误。参见不允许的客户端字符集。
