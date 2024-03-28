> 原文：[`dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html)

#### 7.6.6.4 版本标记参考

以下讨论作为这些版本标记元素的参考：

+   [版本标记函数](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#version-tokens-routines "版本标记函数")

+   [版本标记系统变量](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#version-tokens-system-variables "版本标记系统变量")

##### 版本标记函数

版本标记插件库包含多个函数。一组函数允许操作和检查服务器的版本标记列表。另一组函数允许锁定和解锁版本标记。调用任何版本标记函数都需要[`VERSION_TOKEN_ADMIN`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_version-token-admin)权限（或已弃用的[`SUPER`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_super)权限）。

以下函数允许创建、更改、删除和检查服务器的版本标记列表。对*`name_list`*和*`token_list`*参数的解释（包括空格处理）如[第 7.6.6.3 节“使用版本标记”](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-usage.html "7.6.6.3 使用版本标记")所述，该节提供了关于指定标记语法的详细信息，以及额外的示例。

+   [`version_tokens_delete(*`name_list`*)`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#function_version-tokens-delete)

    使用*`name_list`*参数从服务器的版本标记列表中删除标记，并返回指示操作结果的二进制字符串。*`name_list`*是一个以分号分隔的版本标记名称列表，用于删除。

    ```sql
    mysql> SELECT version_tokens_delete('tok1;tok3');
    +------------------------------------+
    | version_tokens_delete('tok1;tok3') |
    +------------------------------------+
    | 2 version tokens deleted.          |
    +------------------------------------+
    ```

    `NULL`参数被视为空字符串，对标记列表没有影响。

    [`version_tokens_delete()`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#function_version-tokens-delete) 删除其参数中命名的标记（如果存在）。（删除不存在的标记不会报错。）要清除标记列表而不知道列表中有哪些标记，可以传递`NULL`或一个不包含任何标记的字符串给[`version_tokens_set()`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#function_version-tokens-set)：

    ```sql
    mysql> SELECT version_tokens_set(NULL);
    +------------------------------+
    | version_tokens_set(NULL)     |
    +------------------------------+
    | Version tokens list cleared. |
    +------------------------------+
    mysql> SELECT version_tokens_set('');
    +------------------------------+
    | version_tokens_set('')       |
    +------------------------------+
    | Version tokens list cleared. |
    +------------------------------+
    ```

+   [`version_tokens_edit(*`token_list`*)`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#function_version-tokens-edit)

    使用*`token_list`*参数修改服务器的版本标记列表，并返回指示操作结果的二进制字符串。*`token_list`*是一个以分号分隔的`*`name`*=*`value`*`对列表，指定要定义的每个标记的名称及其值。如果标记存在，则其值将使用给定值更新。如果标记不存在，则将使用给定值创建标记。如果参数为`NULL`或一个不包含任何标记的字符串，则标记列表保持不变。

    ```sql
    mysql> SELECT version_tokens_set('tok1=value1;tok2=value2');
    +-----------------------------------------------+
    | version_tokens_set('tok1=value1;tok2=value2') |
    +-----------------------------------------------+
    | 2 version tokens set.                         |
    +-----------------------------------------------+
    mysql> SELECT version_tokens_edit('tok2=new_value2;tok3=new_value3');
    +--------------------------------------------------------+
    | version_tokens_edit('tok2=new_value2;tok3=new_value3') |
    +--------------------------------------------------------+
    | 2 version tokens updated.                              |
    +--------------------------------------------------------+
    ```

+   [`version_tokens_set(*`token_list`*)`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-reference.html#function_version-tokens-set)

    用*`token_list`*参数中定义的令牌替换服务器的版本令牌列表，并返回指示操作结果的二进制字符串。*`token_list`*是一个以分号分隔的`*`name`*=*`value`*`对列表，指定要定义的每个令牌的名称及其值。如果参数为`NULL`或包含零个令牌的字符串，则清除令牌列表。

    ```sql
    mysql> SELECT version_tokens_set('tok1=value1;tok2=value2');
    +-----------------------------------------------+
    | version_tokens_set('tok1=value1;tok2=value2') |
    +-----------------------------------------------+
    | 2 version tokens set.                         |
    +-----------------------------------------------+
    ```

+   `version_tokens_show()`

    将服务器的版本令牌列表作为包含以分号分隔的`*`name`*=*`value`*`对列表的二进制字符串返回。

    ```sql
    mysql> SELECT version_tokens_show();
    +--------------------------+
    | version_tokens_show()    |
    +--------------------------+
    | tok2=value2;tok1=value1; |
    +--------------------------+
    ```

以下函数允许锁定和解锁版本令牌：

+   [`version_tokens_lock_exclusive(*`token_name`*[, *`token_name`*] ..., *`timeout`*)`](version-tokens-reference.html#function_version-tokens-lock-exclusive)

    获取由名称指定为字符串的一个或多个版本令牌的独占锁，在给定的超时值内超时并出现错误，如果在给定的超时值内未获得锁。

    ```sql
    mysql> SELECT version_tokens_lock_exclusive('lock1', 'lock2', 10);
    +-----------------------------------------------------+
    | version_tokens_lock_exclusive('lock1', 'lock2', 10) |
    +-----------------------------------------------------+
    |                                                   1 |
    +-----------------------------------------------------+
    ```

+   [`version_tokens_lock_shared(*`token_name`*[, *`token_name`*] ..., *`timeout`*)`](version-tokens-reference.html#function_version-tokens-lock-shared)

    获取由名称指定为字符串的一个或多个版本令牌的共享锁，在给定的超时值内超时并出现错误，如果在给定的超时值内未获得锁。

    ```sql
    mysql> SELECT version_tokens_lock_shared('lock1', 'lock2', 10);
    +--------------------------------------------------+
    | version_tokens_lock_shared('lock1', 'lock2', 10) |
    +--------------------------------------------------+
    |                                                1 |
    +--------------------------------------------------+
    ```

+   `version_tokens_unlock()`

    释放使用`version_tokens_lock_exclusive()`和`version_tokens_lock_shared()`在当前会话中获取的所有锁。

    ```sql
    mysql> SELECT version_tokens_unlock();
    +-------------------------+
    | version_tokens_unlock() |
    +-------------------------+
    |                       1 |
    +-------------------------+
    ```

锁定函数具有以下特征：

+   返回值为非零表示成功。否则，将发生错误。

+   令牌名称为字符串。

+   与操作服务器令牌列表的函数的参数处理相反，不会忽略围绕令牌名称参数的空白，并且允许使用`=`和`;`字符。

+   可以锁定不存在的令牌名称。这不会创建令牌。

+   超时值是表示在超时前等待获取锁的时间（以秒为单位）的非负整数。如果超时为 0，则不会等待，如果无法立即获取锁，则函数会产生错误。

+   版本令牌锁定函数基于第 7.6.9.1 节，“锁定服务”中描述的锁定服务。

##### 版本令牌系统变量

版本令牌支持以下系统变量。除非安装了版本令牌插件（请参阅第 7.6.6.2 节，“安装或卸载版本令牌”），否则这些变量不可用。

系统变量：

+   `version_tokens_session`

    | 命令行格式 | `--version-tokens-session=value` |
    | --- | --- |
    | 系统变量 | `version_tokens_session` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    此变量的会话值指定客户端版本令牌列表，并指示客户端会话需要服务器版本令牌列表具有的令牌。

    如果`version_tokens_session`变量为`NULL`（默认值）或具有空值，则任何服务器版本令牌列表都匹配。（实际上，空值会禁用匹配要求。）

    如果`version_tokens_session`变量具有非空值，则会导致会话发送到服务器的任何语句的值与服务器版本令牌列表不匹配时出错。在以下情况下会发生不匹配：

    +   在`version_tokens_session`值中的令牌名称不在服务器令牌列表中。在这种情况下，会发生`ER_VTOKEN_PLUGIN_TOKEN_NOT_FOUND`错误。

    +   在`version_tokens_session`值中的令牌值与服务器令牌列表中相应令牌的值不同。在这种情况下，会发生`ER_VTOKEN_PLUGIN_TOKEN_MISMATCH`错误。

    服务器版本令牌列表包含一个未在`version_tokens_session`值中命名的令牌并不构成不匹配。

    假设一个管理应用程序已将服务器令牌列表设置如下：

    ```sql
    mysql> SELECT version_tokens_set('tok1=a;tok2=b;tok3=c');
    +--------------------------------------------+
    | version_tokens_set('tok1=a;tok2=b;tok3=c') |
    +--------------------------------------------+
    | 3 version tokens set.                      |
    +--------------------------------------------+
    ```

    客户端通过设置其`version_tokens_session`值来注册服务器需要匹配的令牌。然后，对于客户端发送的每个后续语句，服务器会将其令牌列表与客户端`version_tokens_session`值进行比较，如果不匹配则产生错误：

    ```sql
    mysql> SET @@SESSION.version_tokens_session = 'tok1=a;tok2=b';
    mysql> SELECT 1;
    +---+
    | 1 |
    +---+
    | 1 |
    +---+

    mysql> SET @@SESSION.version_tokens_session = 'tok1=b';
    mysql> SELECT 1;
    ERROR 3136 (42000): Version token mismatch for tok1\. Correct value a
    ```

    第一个`SELECT`成功，因为客户端令牌`tok1`和`tok2`存在于服务器令牌列表中，并且每个令牌在服务器列表中具有相同的值。第二个`SELECT`失败，因为虽然`tok1`存在于服务器令牌列表中，但其值与客户端指定的值不同。

    此时，除非服务器令牌列表发生变化以匹配，否则客户端发送的任何语句都会失败。假设管理应用程序将服务器令牌列表更改如下：

    ```sql
    mysql> SELECT version_tokens_edit('tok1=b');
    +-------------------------------+
    | version_tokens_edit('tok1=b') |
    +-------------------------------+
    | 1 version tokens updated.     |
    +-------------------------------+
    mysql> SELECT version_tokens_show();
    +-----------------------+
    | version_tokens_show() |
    +-----------------------+
    | tok3=c;tok1=b;tok2=b; |
    +-----------------------+
    ```

    现在客户端的`version_tokens_session`值与服务器令牌列表匹配，客户端可以再次成功执行语句：

    ```sql
    mysql> SELECT 1;
    +---+
    | 1 |
    +---+
    | 1 |
    +---+
    ```

+   `version_tokens_session_number`

    | 命令行格式 | `--version-tokens-session-number=#` |
    | --- | --- |
    | 系统变量 | `version_tokens_session_number` |
    | 范围 | 全局，会话 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |

    此变量仅供内部使用。
