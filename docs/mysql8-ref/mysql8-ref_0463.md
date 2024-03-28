> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-usage.html)

#### 8.5.3.2 使用 MySQL 企业数据脱敏和去标识化插件

在使用 MySQL 企业数据脱敏和去标识化之前，请根据提供的说明安装它，详情请参阅第 8.5.3.1 节，“MySQL 企业数据脱敏和去标识化插件安装”。

要在应用程序中使用 MySQL 企业数据脱敏和去标识化，请调用适用于所需操作的函数。有关详细的函数描述，请参阅第 8.5.3.4 节，“MySQL 企业数据脱敏和去标识化插件函数描述”。本节演示如何使用这些函数执行一些代表性任务。首先概述可用函数，然后展示这些函数在实际环境中的使用示例：

+   脱敏数据以删除识别特征

+   生成具有特定特征的随机数据

+   使用字典生成随机数据

+   使用脱敏数据进行客户识别

+   创建显示脱敏数据的视图

##### 脱敏数据以删除识别特征

MySQL 提供通用脱敏函数，用于脱敏任意字符串，以及特定类型值的专用脱敏函数。

###### 通用脱敏函数

`mask_inner()` 和 `mask_outer()` 是根据字符串内部位置脱敏任意字符串的通用函数：

+   `mask_inner()` 脱敏其字符串参数的内部，保留末尾未脱敏。其他参数指定未脱敏末尾的大小。

    ```sql
    mysql> SELECT mask_inner('This is a string', 5, 1);
    +--------------------------------------+
    | mask_inner('This is a string', 5, 1) |
    +--------------------------------------+
    | This XXXXXXXXXXg                     |
    +--------------------------------------+
    mysql> SELECT mask_inner('This is a string', 1, 5);
    +--------------------------------------+
    | mask_inner('This is a string', 1, 5) |
    +--------------------------------------+
    | TXXXXXXXXXXtring                     |
    +--------------------------------------+
    ```

+   `mask_outer()` 做相反的操作，遮盖其字符串参数的末尾，保留内部不遮盖。其他参数指定了遮盖末尾的大小。

    ```sql
    mysql> SELECT mask_outer('This is a string', 5, 1);
    +--------------------------------------+
    | mask_outer('This is a string', 5, 1) |
    +--------------------------------------+
    | XXXXXis a strinX                     |
    +--------------------------------------+
    mysql> SELECT mask_outer('This is a string', 1, 5);
    +--------------------------------------+
    | mask_outer('This is a string', 1, 5) |
    +--------------------------------------+
    | Xhis is a sXXXXX                     |
    +--------------------------------------+
    ```

默认情况下，`mask_inner()` 和 `mask_outer()` 使用`'X'`作为遮盖字符，但允许一个可选的遮盖字符参数：

```sql
mysql> SELECT mask_inner('This is a string', 5, 1, '*');
+-------------------------------------------+
| mask_inner('This is a string', 5, 1, '*') |
+-------------------------------------------+
| This **********g                          |
+-------------------------------------------+
mysql> SELECT mask_outer('This is a string', 5, 1, '#');
+-------------------------------------------+
| mask_outer('This is a string', 5, 1, '#') |
+-------------------------------------------+
| #####is a strin#                          |
+-------------------------------------------+
```

###### 特殊用途的遮盖函数

其他遮盖函数期望一个表示特定类型值的字符串参数，并对其进行遮盖以消除识别特征。

注意

这里的示例使用返回适当类型值的随机值生成函数来提供函数参数。有关生成函数的更多信息，请参阅使用特定特征生成随机数据。

**支付卡主帐号遮盖。** 遮盖函数提供主帐号数字的严格和宽松遮盖。

+   `mask_pan()` 遮盖号码除最后四位外的所有内容：

    ```sql
    mysql> SELECT mask_pan(gen_rnd_pan());
    +-------------------------+
    | mask_pan(gen_rnd_pan()) |
    +-------------------------+
    | XXXXXXXXXXXX2461        |
    +-------------------------+
    ```

+   `mask_pan_relaxed()` 类似，但不遮盖表示支付卡发行者的前六位数字：

    ```sql
    mysql> SELECT mask_pan_relaxed(gen_rnd_pan());
    +---------------------------------+
    | mask_pan_relaxed(gen_rnd_pan()) |
    +---------------------------------+
    | 770630XXXXXX0807                |
    +---------------------------------+
    ```

**美国社会安全号码遮盖。** `mask_ssn()` 遮盖号码除最后四位外的所有内容：

```sql
mysql> SELECT mask_ssn(gen_rnd_ssn());
+-------------------------+
| mask_ssn(gen_rnd_ssn()) |
+-------------------------+
| XXX-XX-1723             |
+-------------------------+
```

##### 使用特定特征生成随机数据

几个函数生成随机值。这些值可用于测试、模拟等用途。

`gen_range()` 返回从给定范围中选择的随机整数：

```sql
mysql> SELECT gen_range(1, 10);
+------------------+
| gen_range(1, 10) |
+------------------+
|                6 |
+------------------+
```

`gen_rnd_email()` 返回一个在`example.com`域中的随机电子邮件地址：

```sql
mysql> SELECT gen_rnd_email();
+---------------------------+
| gen_rnd_email()           |
+---------------------------+
| ayxnq.xmkpvvy@example.com |
+---------------------------+
```

`gen_rnd_pan()` 返回一个随机的支付卡主帐号数字：

```sql
mysql> SELECT gen_rnd_pan();
```

（`gen_rnd_pan()` 函数的结果未显示，因为其返回值仅应用于测试目的，而不用于发布。不能保证该号码未分配给合法支付帐户。）

`gen_rnd_ssn()` 返回一个随机的美国社会安全号码，第一部分和第二部分分别从未用于合法号码的范围中选择：

```sql
mysql> SELECT gen_rnd_ssn();
+---------------+
| gen_rnd_ssn() |
+---------------+
| 912-45-1615   |
+---------------+
```

`gen_rnd_us_phone()` 返回一个在未用于合法号码的 555 区号中的随机美国电话号码：

```sql
mysql> SELECT gen_rnd_us_phone();
+--------------------+
| gen_rnd_us_phone() |
+--------------------+
| 1-555-747-5627     |
+--------------------+
```

##### 使用字典生成随机数据

MySQL 企业数据蒙面和去标识化使得可以将字典用作随机值的来源。要使用字典，必须首先从文件加载它并赋予一个名称。每个加载的字典都成为字典注册表的一部分。然后可以从注册的字典中选择项目并用作随机值或替换其他值。

有效的字典文件具有以下特征：

+   文件内容为纯文本，每行一个术语。

+   空行将被忽略。

+   文件必须至少包含一个术语。

假设名为`de_cities.txt`的文件包含德国这些城市名称：

```sql
Berlin
Munich
Bremen
```

还假设名为`us_cities.txt`的文件包含美国这些城市名称：

```sql
Chicago
Houston
Phoenix
El Paso
Detroit
```

假设`secure_file_priv`系统变量设置为`/usr/local/mysql/mysql-files`。在这种情况下，将字典文件复制到该目录，以便 MySQL 服务器可以访问它们。然后使用`gen_dictionary_load()`加载字典到字典注册表并分配名称：

```sql
mysql> SELECT gen_dictionary_load('/usr/local/mysql/mysql-files/de_cities.txt', 'DE_Cities');
+--------------------------------------------------------------------------------+
| gen_dictionary_load('/usr/local/mysql/mysql-files/de_cities.txt', 'DE_Cities') |
+--------------------------------------------------------------------------------+
| Dictionary load success                                                        |
+--------------------------------------------------------------------------------+
mysql> SELECT gen_dictionary_load('/usr/local/mysql/mysql-files/us_cities.txt', 'US_Cities');
+--------------------------------------------------------------------------------+
| gen_dictionary_load('/usr/local/mysql/mysql-files/us_cities.txt', 'US_Cities') |
+--------------------------------------------------------------------------------+
| Dictionary load success                                                        |
+--------------------------------------------------------------------------------+
```

要从字典中选择一个随机术语，请使用`gen_dictionary()`：

```sql
mysql> SELECT gen_dictionary('DE_Cities');
+-----------------------------+
| gen_dictionary('DE_Cities') |
+-----------------------------+
| Berlin                      |
+-----------------------------+
mysql> SELECT gen_dictionary('US_Cities');
+-----------------------------+
| gen_dictionary('US_Cities') |
+-----------------------------+
| Phoenix                     |
+-----------------------------+
```

要从多个字典中选择一个随机术语，随机选择其中一个字典，然后从中选择���个术语：

```sql
mysql> SELECT gen_dictionary(ELT(gen_range(1,2), 'DE_Cities', 'US_Cities'));
+---------------------------------------------------------------+
| gen_dictionary(ELT(gen_range(1,2), 'DE_Cities', 'US_Cities')) |
+---------------------------------------------------------------+
| Detroit                                                       |
+---------------------------------------------------------------+
mysql> SELECT gen_dictionary(ELT(gen_range(1,2), 'DE_Cities', 'US_Cities'));
+---------------------------------------------------------------+
| gen_dictionary(ELT(gen_range(1,2), 'DE_Cities', 'US_Cities')) |
+---------------------------------------------------------------+
| Bremen                                                        |
+---------------------------------------------------------------+
```

`gen_blocklist()`函数使得可以用另一个字典中的术语替换一个字典中的术语，从而通过替换实现蒙面。它的参数是要替换的术语，术语出现的字典，以及要选择替换的字典。例如，要用美国城市替换德国城市，或者反之，可以像这样使用`gen_blocklist()`：

```sql
mysql> SELECT gen_blocklist('Munich', 'DE_Cities', 'US_Cities');
+---------------------------------------------------+
| gen_blocklist('Munich', 'DE_Cities', 'US_Cities') |
+---------------------------------------------------+
| Houston                                           |
+---------------------------------------------------+
mysql> SELECT gen_blocklist('El Paso', 'US_Cities', 'DE_Cities');
+----------------------------------------------------+
| gen_blocklist('El Paso', 'US_Cities', 'DE_Cities') |
+----------------------------------------------------+
| Bremen                                             |
+----------------------------------------------------+
```

如果要替换的术语不在第一个字典中，`gen_blocklist()`将不做任何更改返回它：

```sql
mysql> SELECT gen_blocklist('Moscow', 'DE_Cities', 'US_Cities');
+---------------------------------------------------+
| gen_blocklist('Moscow', 'DE_Cities', 'US_Cities') |
+---------------------------------------------------+
| Moscow                                            |
+---------------------------------------------------+
```

##### 使用蒙面数据进行客户识别

在客户服务呼叫中心，一个常见的身份验证技术是要求客户提供他们社会安全号码（SSN）的最后四位数字。例如，一个客户可能说她的名字是乔安娜邦德，她的最后四位 SSN 数字是`0007`。

假设包含客户记录的`customer`表具有以下列：

+   `id`：客户 ID 号码。

+   `first_name`：客户名字。

+   `last_name`：客户姓氏。

+   `ssn`：客户社会安全号码。

例如，表可以定义如下：

```sql
CREATE TABLE customer
(
  id         BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(40),
  last_name  VARCHAR(40),
  ssn        VARCHAR(11)
);
```

由客户服务代表使用的应用程序检查客户社会安全号码可能执行类似这样的查询：

```sql
mysql> SELECT id, ssn
mysql> FROM customer
mysql> WHERE first_name = 'Joanna' AND last_name = 'Bond';
+-----+-------------+
| id  | ssn         |
+-----+-------------+
| 786 | 906-39-0007 |
+-----+-------------+
```

然而，这会将社会安全号码暴露给客户服务代表，他们没有必要看到除最后四位数字之外的任何内容。相反，应用程序可以使用此查询仅显示蒙面的社会安全号码：

```sql
mysql> SELECT id, mask_ssn(CONVERT(ssn USING binary)) AS masked_ssn
mysql> FROM customer
mysql> WHERE first_name = 'Joanna' AND last_name = 'Bond';
+-----+-------------+
| id  | masked_ssn  |
+-----+-------------+
| 786 | XXX-XX-0007 |
+-----+-------------+
```

现在代表只能看到必要的内容，客户隐私得到保护。

为什么在`mask_ssn()`的参数中使用了`CONVERT()`函数？因为`mask_ssn()`需要一个长度为 11 的参数。因此，即使`ssn`被定义为`VARCHAR(11)`，如果`ssn`列具有多字节字符集，当传递给可加载函数时，它可能看起来比 11 个字节长，从而导致错误。将值转换为二进制字符串可以确保函数看到一个长度为 11 的参数。

当字符串参数没有单字节字符集时，可能需要类似的技术来处理其他数据掩码函数。

##### 创建显示掩码数据的视图

如果来自表的掩码数据用于多个查询，定义一个生成掩码数据的视图可能会更方便。这样，应用程序可以从视图中选择，而无需在单个查询中执行掩码操作。

例如，可以像这样定义前一节中`customer`表的掩码视图：

```sql
CREATE VIEW masked_customer AS
SELECT id, first_name, last_name,
mask_ssn(CONVERT(ssn USING binary)) AS masked_ssn
FROM customer;
```

查询客户信息的操作变得更简单，但仍返回掩码数据：

```sql
mysql> SELECT id, masked_ssn
mysql> FROM masked_customer
mysql> WHERE first_name = 'Joanna' AND last_name = 'Bond';
+-----+-------------+
| id  | masked_ssn  |
+-----+-------------+
| 786 | XXX-XX-0007 |
+-----+-------------+
```
