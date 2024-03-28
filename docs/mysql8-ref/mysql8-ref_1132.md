# 15.8.3 HELP 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/help.html`](https://dev.mysql.com/doc/refman/8.0/en/help.html)

```sql
HELP '*search_string*'
```

`HELP`语句从 MySQL 参考手册返回在线信息。其正确操作要求在`mysql`数据库中初始化帮助主题信息（参见 Section 7.1.17, “服务器端帮助支持”）。

`HELP`语句搜索给定搜索字符串的帮助表，并显示搜索结果。搜索字符串不区分大小写。

搜索字符串可以包含通配符字符`%`和`_`。这些字符的含义与使用`LIKE`运算符执行的模式匹配操作相同。例如，`HELP 'rep%'`返回以`rep`开头的主题列表。

`HELP`语句不需要像`；`或`\G`这样的终止符。

`HELP`语句理解几种类型的搜索字符串：

+   在最一般的级别上，使用`contents`检索顶级帮助类别的列表：

    ```sql
    HELP 'contents'
    ```

+   要获取给定帮助类别（如`Data Types`）中主题的列表，请使用类别名称：

    ```sql
    HELP 'data types'
    ```

+   要获取有关特定帮助主题（如`ASCII()`函数或`CREATE TABLE`语句）的帮助，请使用相关的关键字或关键字：

    ```sql
    HELP 'ascii'
    HELP 'create table'
    ```

换句话说，搜索字符串匹配一个类别、多个主题或一个主题。以下描述指示结果集可能采用的形式。

+   空结果

    未找到与搜索字符串匹配的结果。

    示例：`HELP 'fake'`

    结果：

    ```sql
    Nothing found
    Please try to run 'help contents' for a list of all accessible topics
    ```

+   包含单行结果集

    这意味着搜索字符串找到了帮助主题。结果包括以下项目：

    +   `name`：主题名称。

    +   `description`：主题的描述性帮助文本。

    +   `example`：一个或多个用法示例。（可能为空。）

    示例：`HELP 'log'`

    结果：

    ```sql
    Name: 'LOG'
    Description:
    Syntax:
    LOG(X), LOG(B,X)

    If called with one parameter, this function returns the natural
    logarithm of X. If X is less than or equal to 0.0E0, the function
    returns NULL and a warning "Invalid argument for logarithm" is
    reported. Returns NULL if X or B is NULL.

    The inverse of this function (when called with a single argument) is
    the EXP() function.

    URL: https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html

    Examples:
    mysql> SELECT LOG(2);
            -> 0.69314718055995
    mysql> SELECT LOG(-2);
            -> NULL
    ```

+   主题列表。

    这意味着搜索字符串匹配了多个帮助主题。

    示例：`HELP 'status'`

    结果：

    ```sql
    Many help items for your request exist.
    To make a more specific request, please type 'help <item>',
    where <item> is one of the following topics:
       FLUSH
       SHOW
       SHOW ENGINE
       SHOW FUNCTION STATUS
       SHOW MASTER STATUS
       SHOW PROCEDURE STATUS
       SHOW REPLICA STATUS
       SHOW SLAVE STATUS
       SHOW STATUS
       SHOW TABLE STATUS
    ```

+   主题列表。

    如果搜索字符串匹配一个类别，也会显示列表。

    示例：`HELP 'functions'`

    结果：

    ```sql
    You asked for help about help category: "Functions"
    For more information, type 'help <item>', where <item> is one of the following
    categories:
       Aggregate Functions and Modifiers
       Bit Functions
       Cast Functions and Operators
       Comparison Operators
       Date and Time Functions
       Encryption Functions
       Enterprise Encryption Functions
       Flow Control Functions
       GROUP BY Functions and Modifiers
       GTID
       Information Functions
       Internal Functions
       Locking Functions
       Logical Operators
       Miscellaneous Functions
       Numeric Functions
       Performance Schema Functions
       Spatial Functions
       String Functions
       Window Functions
       XML
    ```
