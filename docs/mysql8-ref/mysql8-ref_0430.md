> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-functions-general-purpose.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-functions-general-purpose.html)

#### 8.4.4.15 通用密钥环密钥管理函数

MySQL 服务器支持一个密钥环服务，使内部组件和插件能够安全地存储敏感信息以供以后检索。

MySQL 服务器还包括一个用于密钥环密钥管理的 SQL 接口，实现为一组访问内部密钥环服务提供的功能的通用函数。密钥环函数包含在一个插件库文件中，该文件还包含一个必须在函数调用之前启用的`keyring_udf`插件。要使用这些函数，必须启用诸如`keyring_file`或`keyring_okv`之类的密钥环插件。

这里描述的函数是通用的，旨在与任何密钥环插件一起使用。特定的密钥环插件可能具有专为仅与该插件一起使用的自己的函数；请参见第 8.4.4.16 节，“特定插件的密钥环密钥管理函数”。

下面的章节提供了关于密钥环函数的安装说明，并演示如何使用它们。有关这些函数调用的密钥环服务函数的信息，请参见第 7.6.9.2 节，“密钥环服务”。有关一般密钥环信息，请参见第 8.4.4 节，“MySQL 密钥环”。

+   安装或卸载通用密钥环函数

+   使用通用密钥环函数

+   通用密钥环函数参考

##### 安装或卸载通用密钥环函数

本节描述了如何安装或卸载密钥环函数，这些函数实现在一个插件库文件中，该文件还包含一个`keyring_udf`插件。有关安装或卸载插件和可加载函数的一般信息，请参见第 7.6.1 节，“安装和卸载插件”，以及第 7.7.1 节，“安装和卸载可加载函数”。

密钥环函数使密钥环密钥管理操作成为可能，但必须安装`keyring_udf`插件，因为没有它，这些函数无法正常工作。尝试在没有`keyring_udf`插件的情况下使用这些函数会导致错误。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来配置插件目录位置。

插件库文件基本名称为`keyring_udf`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为`.so`，对于 Windows 为`.dll`）。

要安装`keyring_udf`插件和钥匙环函数，请使用`INSTALL PLUGIN`和`CREATE FUNCTION`语句，根据需要调整`.so`后缀以适应您的平台：

```sql
INSTALL PLUGIN keyring_udf SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_generate RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_fetch RETURNS STRING
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_length_fetch RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_type_fetch RETURNS STRING
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_store RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_remove RETURNS INTEGER
  SONAME 'keyring_udf.so';
```

如果插件和函数在源复制服务器上使用，则还必须在所有副本上安装它们，以避免复制问题。

一旦按照上述方式安装，插件和函数将保持安装状态直到卸载。要删除它们，请使用`UNINSTALL PLUGIN`和`DROP FUNCTION`语句：

```sql
UNINSTALL PLUGIN keyring_udf;
DROP FUNCTION keyring_key_generate;
DROP FUNCTION keyring_key_fetch;
DROP FUNCTION keyring_key_length_fetch;
DROP FUNCTION keyring_key_type_fetch;
DROP FUNCTION keyring_key_store;
DROP FUNCTION keyring_key_remove;
```

##### 使用通用钥匙环函数

在使用钥匙环通用函数之前，请根据安装或卸载通用钥匙环函数中提供的说明安装它们。

钥匙环函数受以下约束：

+   要使用任何钥匙环函数，必须启用`keyring_udf`插件。否则，会出现错误：

    ```sql
    ERROR 1123 (HY000): Can't initialize function 'keyring_key_generate';
    This function requires keyring_udf plugin which is not installed.
    Please install
    ```

    要安装`keyring_udf`插件，请参阅安装或卸载通用钥匙环函数。

+   钥匙环函数调用钥匙环服务函数（参见第 7.6.9.2 节，“钥匙环服务”）。服务函数反过来使用安装的任何钥匙环插件（例如，`keyring_file`或`keyring_okv`）。因此，要使用任何钥匙环函数，必须启用某个基础钥匙环插件。否则，会出现错误：

    ```sql
    ERROR 3188 (HY000): Function 'keyring_key_generate' failed because
    underlying keyring service returned an error. Please check if a
    keyring plugin is installed and that provided arguments are valid
    for the keyring you are using.
    ```

    要安装钥匙环插件，请参阅第 8.4.4.3 节，“钥匙环插件安装”。

+   用户必须拥有全局`EXECUTE`权限才能使用任何钥匙环函数。否则，会出现错误：

    ```sql
    ERROR 1123 (HY000): Can't initialize function 'keyring_key_generate';
    The user is not privileged to execute this function. User needs to
    have EXECUTE
    ```

    要向用户授予全局`EXECUTE`权限，请使用此语句：

    ```sql
    GRANT EXECUTE ON *.* TO *user*;
    ```

    或者，如果您希望避免授予全局`EXECUTE`权限，同时仍允许用户访问特定的密钥管理操作，可以定义“包装器”存储程序（这是本节稍后描述的一种技术）。

+   由给定用户在密钥环中存储的密钥后续只能由同一用户操作。也就是说，在密钥操作时`CURRENT_USER()`函数的值必须与密钥存储在密钥环中时的值相同。（此约束排除了使用密钥环函数来操作实例范围密钥的情况，例如`InnoDB`创建的用于支持表空间加密的密钥。）

    为了使多个用户能够对同一密钥执行操作，可以定义“包装器”存储程序（这是本节稍后描述的一种技术）。

+   密钥环函数支持底层密钥环插件支持的密钥类型和长度。有关特定于特定密钥环插件的密钥的信息，请参见第 8.4.4.13 节，“支持的密钥环密钥类型和长度”。

要创建一个新的随机密钥并将其存储在密钥环中，请调用`keyring_key_generate()`，传递给它一个密钥的 ID，以及密钥类型（加密方法）和以字节为单位的长度。以下调用创建一个名为`MyKey`的 2,048 位 DSA 加密密钥：

```sql
mysql> SELECT keyring_key_generate('MyKey', 'DSA', 256);
+-------------------------------------------+
| keyring_key_generate('MyKey', 'DSA', 256) |
+-------------------------------------------+
|                                         1 |
+-------------------------------------------+
```

返回值为 1 表示成功。如果无法创建密钥，则返回值为`NULL`并出现错误。可能的原因之一是底层密钥环插件不支持指定的密钥类型和密钥长度的组合；请参见第 8.4.4.13 节，“支持的密钥环密钥类型和长度”。

为了能够检查返回类型，无论是否发生错误，请使用`SELECT ... INTO @*`var_name`*`并测试变量值：

```sql
mysql> SELECT keyring_key_generate('', '', -1) INTO @x;
ERROR 3188 (HY000): Function 'keyring_key_generate' failed because
underlying keyring service returned an error. Please check if a
keyring plugin is installed and that provided arguments are valid
for the keyring you are using.
mysql> SELECT @x;
+------+
| @x   |
+------+
| NULL |
+------+
mysql> SELECT keyring_key_generate('x', 'AES', 16) INTO @x;
mysql> SELECT @x;
+------+
| @x   |
+------+
|    1 |
+------+
```

这种技术也适用于其他密钥环函数，对于失败返回一个值和一个错误。

传递给`keyring_key_generate()`的 ID 提供了在后续函数调用中引用密钥的方法。例如，使用密钥 ID 将其类型作为字符串或其长度作为整数检索：

```sql
mysql> SELECT keyring_key_type_fetch('MyKey');
+---------------------------------+
| keyring_key_type_fetch('MyKey') |
+---------------------------------+
| DSA                             |
+---------------------------------+
mysql> SELECT keyring_key_length_fetch('MyKey');
+-----------------------------------+
| keyring_key_length_fetch('MyKey') |
+-----------------------------------+
|                               256 |
+-----------------------------------+
```

要检索密钥值，请将密钥 ID 传递给`keyring_key_fetch()`。以下示例使用`HEX()`显示密钥值，因为它可能包含不可打印的字符。示例还使用了一个简短的密钥以简洁起见，但请注意，更长的密钥提供更好的安全性：

```sql
mysql> SELECT keyring_key_generate('MyShortKey', 'DSA', 8);
+----------------------------------------------+
| keyring_key_generate('MyShortKey', 'DSA', 8) |
+----------------------------------------------+
|                                            1 |
+----------------------------------------------+
mysql> SELECT HEX(keyring_key_fetch('MyShortKey'));
+--------------------------------------+
| HEX(keyring_key_fetch('MyShortKey')) |
+--------------------------------------+
| 1DB3B0FC3328A24C                     |
+--------------------------------------+
```

钥匙环函数将密钥 ID、类型和值视为二进制字符串，因此比较区分大小写。例如，`MyKey`和`mykey`的 ID 指的是不同的密钥。

要移除一个密钥，请将密钥 ID 传递给`keyring_key_remove()`：

```sql
mysql> SELECT keyring_key_remove('MyKey');
+-----------------------------+
| keyring_key_remove('MyKey') |
+-----------------------------+
|                           1 |
+-----------------------------+
```

要混淆和存储您提供的密钥，请将密钥 ID、类型和值传递给`keyring_key_store()`：

```sql
mysql> SELECT keyring_key_store('AES_key', 'AES', 'Secret string');
+------------------------------------------------------+
| keyring_key_store('AES_key', 'AES', 'Secret string') |
+------------------------------------------------------+
|                                                    1 |
+------------------------------------------------------+
```

如前所述，用户必须具有全局`EXECUTE`权限才能调用钥匙环函数，并且最初将密钥存储在钥匙环中的用户必须是稍后执行密钥操作的同一用户，这是根据每个函数调用中生效的`CURRENT_USER()`值确定的。为了允许没有全局`EXECUTE`权限或可能不是密钥“所有者”的用户执行密钥操作，请使用以下技术：

1.  定义“包装”存储程序，封装所需的密钥操作，并且`DEFINER`值等于密钥所有者。

1.  为应该能够调用特定存储程序的个别用户授予`EXECUTE`权限。

1.  如果包装存储程序实现的操作不包括密钥创建，请提前创建任何必要的密钥，使用存储程序定义中命名为`DEFINER`的账户。

该技术使密钥可以在用户之间共享，并为 DBA 提供了更精细的控制权，以确定谁可以对密钥执行什么操作，而无需授予全局权限。

以下示例显示如何设置一个名为`SharedKey`的共享密钥，由 DBA 拥有，并且提供对当前密钥值的访问的`get_shared_key()`存储函数。该值可以被具有`EXECUTE`权限的任何用户检索，该权限是在`key_schema`模式中创建的。

从 MySQL 管理账户（在此示例中为`'root'@'localhost'`）中，创建管理模式和用于访问密钥的存储函数：

```sql
mysql> CREATE SCHEMA key_schema;

mysql> CREATE DEFINER = 'root'@'localhost'
       FUNCTION key_schema.get_shared_key()
       RETURNS BLOB READS SQL DATA
       RETURN keyring_key_fetch('SharedKey');
```

从管理账户中，确保共享密钥存在：

```sql
mysql> SELECT keyring_key_generate('SharedKey', 'DSA', 8);
+---------------------------------------------+
| keyring_key_generate('SharedKey', 'DSA', 8) |
+---------------------------------------------+
|                                           1 |
+---------------------------------------------+
```

从管理账户中，创建一个普通用户账户，以便授予密钥访问权限：

```sql
mysql> CREATE USER 'key_user'@'localhost'
       IDENTIFIED BY 'key_user_pwd';
```

从`key_user`账户中，验证，如果没有适当的`EXECUTE`权限，新账户无法访问共享密钥：

```sql
mysql> SELECT HEX(key_schema.get_shared_key());
ERROR 1370 (42000): execute command denied to user 'key_user'@'localhost'
for routine 'key_schema.get_shared_key'
```

从管理账户中，为存储函数授予`EXECUTE`给`key_user`：

```sql
mysql> GRANT EXECUTE ON FUNCTION key_schema.get_shared_key
       TO 'key_user'@'localhost';
```

从`key_user`账户中，验证该密钥现在是可访问的：

```sql
mysql> SELECT HEX(key_schema.get_shared_key());
+----------------------------------+
| HEX(key_schema.get_shared_key()) |
+----------------------------------+
| 9BAFB9E75CEEB013                 |
+----------------------------------+
```

##### 通用用途钥匙环函数参考

对于每个通用密钥环函数，本节描述了其目的、调用顺序和返回值。有关可以调用这些函数的条件的信息，请参见使用通用密钥环函数。

+   `keyring_key_fetch(*`key_id`*)`

    给定密钥 ID，解密并返回密钥值。

    参数:

    +   *`key_id`*: 一个指定密钥 ID 的字符串。

    返回值:

    返回成功的字符串密钥值，如果密钥不存在则返回`NULL`，或者返回`NULL`和错误。

    注意

    使用`keyring_key_fetch()`检索的密钥值受第 8.4.4.13 节，“支持的密钥类型和长度”中描述的一般密钥函数限制的约束。超过该长度的密钥值可以使用密钥环服务函数存储（参见第 7.6.9.2 节，“密钥环服务”），但如果使用`keyring_key_fetch()`检索，则会被截断为一般密钥环函数限制的长度。

    示例:

    ```sql
    mysql> SELECT keyring_key_generate('RSA_key', 'RSA', 16);
    +--------------------------------------------+
    | keyring_key_generate('RSA_key', 'RSA', 16) |
    +--------------------------------------------+
    |                                          1 |
    +--------------------------------------------+
    mysql> SELECT HEX(keyring_key_fetch('RSA_key'));
    +-----------------------------------+
    | HEX(keyring_key_fetch('RSA_key')) |
    +-----------------------------------+
    | 91C2253B696064D3556984B6630F891A  |
    +-----------------------------------+
    mysql> SELECT keyring_key_type_fetch('RSA_key');
    +-----------------------------------+
    | keyring_key_type_fetch('RSA_key') |
    +-----------------------------------+
    | RSA                               |
    +-----------------------------------+
    mysql> SELECT keyring_key_length_fetch('RSA_key');
    +-------------------------------------+
    | keyring_key_length_fetch('RSA_key') |
    +-------------------------------------+
    |                                  16 |
    +-------------------------------------+
    ```

    该示例使用`HEX()`来显示密钥值，因为它可能包含不可打印字符。该示例还使用了一个简短的密钥以便简洁，但请注意，更长的密钥提供更���的安全性。

+   `keyring_key_generate(*`key_id`*, *`key_type`*, *`key_length`*)`

    生成具有给定 ID、类型和长度的新随机密钥，并将其存储在密钥环中。类型和长度值必须与底层密钥环插件支持的值一致。参见第 8.4.4.13 节，“支持的密钥类型和长度”。

    参数:

    +   *`key_id`*: 一个指定密钥 ID 的字符串。

    +   *`key_type`*: 一个指定密钥类型的字符串。

    +   *`key_length`*: 一个指定以字节为单位的密钥长度的整数。

    返回值:

    返回成功的 1，或者返回`NULL`和错误。

    示例:

    ```sql
    mysql> SELECT keyring_key_generate('RSA_key', 'RSA', 384);
    +---------------------------------------------+
    | keyring_key_generate('RSA_key', 'RSA', 384) |
    +---------------------------------------------+
    |                                           1 |
    +---------------------------------------------+
    ```

+   `keyring_key_length_fetch(*`key_id`*)`

    给定密钥 ID，返回密钥长度。

    参数:

    +   *`key_id`*: 一个指定密钥 ID 的字符串。

    返回值:

    返回成功的整数字节密钥长度，如果密钥不存在则返回`NULL`，或者返回`NULL`和错误。

    示例:

    请参阅`keyring_key_fetch()`的描述。

+   `keyring_key_remove(*`key_id`*)`

    从密钥环中删除具有给定 ID 的密钥。

    参数:

    +   *`key_id`*: 指定密钥 ID 的字符串。

    返回值：

    返回成功时为 1，失败时返回`NULL`。

    示例：

    ```sql
    mysql> SELECT keyring_key_remove('AES_key');
    +-------------------------------+
    | keyring_key_remove('AES_key') |
    +-------------------------------+
    |                             1 |
    +-------------------------------+
    ```

+   `keyring_key_store(*`key_id`*, *`key_type`*, *`key`*)`

    对密钥进行混淆并存储在密钥环中。

    参数：

    +   *`key_id`*: 指定密钥 ID 的字符串。

    +   *`key_type`*: 指定密钥类型的字符串。

    +   *`key`*: 指定密钥值的字符串。

    返回值：

    返回成功时为 1，失败时返回`NULL`和错误。

    示例：

    ```sql
    mysql> SELECT keyring_key_store('new key', 'DSA', 'My key value');
    +-----------------------------------------------------+
    | keyring_key_store('new key', 'DSA', 'My key value') |
    +-----------------------------------------------------+
    |                                                   1 |
    +-----------------------------------------------------+
    ```

+   `keyring_key_type_fetch(*`key_id`*)`

    给定密钥 ID，返回密钥类型。

    参数：

    +   *`key_id`*: 指定密钥 ID 的字符串。

    返回值：

    返回成功时的密钥类型字符串，如果密钥不存在则返回`NULL`，失败时返回`NULL`和错误。

    示例：

    参见`keyring_key_fetch()`的描述。
