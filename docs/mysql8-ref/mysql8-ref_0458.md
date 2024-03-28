> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-component-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-component-usage.html)

#### 8.5.2.2 使用 MySQL 企业数据蒙版和去识别组件

在使用 MySQL 企业数据蒙版和去识别之前，请根据提供的说明进行安装，详情请参阅第 8.5.2.1 节，“MySQL 企业数据蒙版和去识别组件安装”。

要在应用程序中使用 MySQL 企业数据蒙版和去识别，请调用适用于您希望执行的操作的函数。有关详细的函数描述，请参阅第 8.5.2.4 节，“MySQL 企业数据蒙版和去识别组件函数描述”。本节演示如何使用这些函数执行一些代表性任务。首先概述可用函数，然后展示一些函数在实际环境中的使用示例：

+   蒙版数据以消除识别特征

+   生成具有特定特征的随机数据

+   使用字典生成随机数据

+   使用蒙版数据进行客户识别

+   创建显示蒙版数据的视图

##### 蒙版数据以消除识别特征

MySQL 提供了通用蒙版组件函数，用于蒙版任意字符串，以及专用蒙版函数，用于蒙版特定类型的值。

###### 通用蒙版组件函数

`mask_inner()` 和 `mask_outer()` 是通用函数，根据字符串内的位置蒙版任意字符串的部分。这两个函数支持以任何字符集编码的输入字符串：

+   `mask_inner()` 对其字符串参数的内部进行蒙版处理，保留末尾未蒙版。其他参数指定未蒙版末尾的大小。

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
    mysql> SELECT mask_inner("かすみがうら市", 3, 1);
    +----------------------------------+
    | mask_inner("かすみがうら市", 3, 1) |
    +----------------------------------+
    | かすみXXX 市                       |
    +----------------------------------+
    mysql> SELECT mask_inner("かすみがうら市", 1, 3);
    +----------------------------------+
    | mask_inner("かすみがうら市", 1, 3) |
    +----------------------------------+
    | かXXXうら市                       |
    +----------------------------------+
    ```

+   `mask_outer()` 则相反，掩码其字符串参数的两端，保留内部不掩码。其他参数指定掩码端的大小。

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

默认情况下，`mask_inner()` 和 `mask_outer()` 使用 `'X'` 作为掩码字符，但允许使用可选的掩码字符参数：

```sql
mysql> SELECT mask_inner('This is a string', 5, 1, '*');
+-------------------------------------------+
| mask_inner('This is a string', 5, 1, '*') |
+-------------------------------------------+
| This **********g                          |
+-------------------------------------------+
mysql> SELECT mask_inner("かすみがうら市", 2, 2, "#");
+---------------------------------------+
| mask_inner("かすみがうら市", 2, 2, "#") |
+---------------------------------------+
| かす###ら市                            |
+---------------------------------------+
```

###### 特殊用途掩码组件函数

其他掩码函数期望一个表示特定类型值的字符串参数，并对其进行掩码以消除识别特征。

注意

这里的示例使用返回适当类型值的随机值生成函数来提供函数参数。有关生成函数的更多信息，请参阅生成具有特定特征的随机数据。

**支付卡主帐号号码掩码。** 掩码函数提供主帐号号码的严格和宽松掩码。

+   `mask_pan()` 掩码号码的除最后四位之外的所有数字：

    ```sql
    mysql> SELECT mask_pan(gen_rnd_pan());
    +-------------------------+
    | mask_pan(gen_rnd_pan()) |
    +-------------------------+
    | XXXXXXXXXXXX2461        |
    +-------------------------+
    ```

+   `mask_pan_relaxed()` 类似，但不掩码指示支付卡发卡方的前六位数字：

    ```sql
    mysql> SELECT mask_pan_relaxed(gen_rnd_pan());
    +---------------------------------+
    | mask_pan_relaxed(gen_rnd_pan()) |
    +---------------------------------+
    | 770630XXXXXX0807                |
    +---------------------------------+
    ```

**国际银行账号号码掩码。** `mask_iban()` 掩码号码的除前两个字母（表示国家）之外的所有数字：

```sql
mysql> SELECT mask_iban(gen_rnd_iban());
+---------------------------+
| mask_iban(gen_rnd_iban()) |
+---------------------------+
| ZZ** **** **** ****       |
+---------------------------+
```

**通用唯一标识符掩码。** `mask_uuid()` 掩码所有有意义的字符：

```sql
mysql> SELECT mask_uuid(gen_rnd_uuid());
+--------------------------------------+
| mask_uuid(gen_rnd_uuid())            |
+--------------------------------------+
| ********-****-****-****-************ |
+--------------------------------------+
```

**美国社会安全号码掩码。** `mask_ssn()` 掩码号码的除最后四位之外的所有数字：

```sql
mysql> SELECT mask_ssn(gen_rnd_ssn());
+-------------------------+
| mask_ssn(gen_rnd_ssn()) |
+-------------------------+
| ***-**-1723             |
+-------------------------+
```

**加拿大社会保险号码掩码。** `mask_canada_sin()` 掩码号码的有意义数字：

```sql
mysql> SELECT mask_canada_sin(gen_rnd_canada_sin());
+---------------------------------------+
| mask_canada_sin(gen_rnd_canada_sin()) |
+---------------------------------------+
| XXX-XXX-XXX                           |
+---------------------------------------+
```

**英国国民保险号码掩码。** `mask_uk_nin()` 掩码号码的除前两位之外的所有数字：

```sql
mysql> SELECT mask_uk_nin(gen_rnd_uk_nin());
+-------------------------------+
| mask_uk_nin(gen_rnd_uk_nin()) |
+-------------------------------+
| ZH*******                     |
+-------------------------------+
```

##### 生成具有特定特征的随机数据

几个组件函数生成随机值。这些值可用于测试、模拟等用途。

`gen_range()` 从给定范围返回一个随机整数：

```sql
mysql> SELECT gen_range(1, 10);
+------------------+
| gen_range(1, 10) |
+------------------+
|                6 |
+------------------+
```

`gen_rnd_canada_sin()` 返回从未用于合法号码的范围中选择的随机数：

```sql
mysql> SELECT gen_rnd_canada_sin();
+----------------------+
| gen_rnd_canada_sin() |
+----------------------+
```

(由于其返回值仅用于测试目的，而非出版，因此不显示`gen_rnd_canada_sin()`函数的结果。不能保证该数字未分配给合法的加拿大 SIN。)

`gen_rnd_email()`返回一个随机的电子邮件地址，名称和姓氏部分在指定域`mynet.com`中具有指定数量的数字，如下例所示：

```sql
mysql> SELECT gen_rnd_email(6, 8, 'mynet.com');
+------------------------------+
| gen_rnd_email(6, 8, 'mynet') |
+------------------------------+
| ayxnqu.xmkpvvyr@mynet.com    |
+------------------------------+
```

`gen_rnd_iban()`返回一个从未用于合法数字的范围中选择的数字：

```sql
mysql> SELECT gen_rnd_iban('XO', 24);
+-------------------------------+
| gen_rnd_iban('XO', 24)        |
+-------------------------------+
| XO25 SL7A PGQR B9NN 6IVB RFE8 |
+-------------------------------+
```

`gen_rnd_pan()`返回一个随机的支付卡主帐号：

```sql
mysql> SELECT gen_rnd_pan();
```

(由于其返回值仅用于测试目的，而非出版，因此不显示`gen_rnd_pan()`函数的结果。不能保证该数字未分配给合法的支付账户。)

`gen_rnd_ssn()`返回一个随机的美国社会安全号码，第一部分和第二部分分别从未用于合法数字的范围中选择：

```sql
mysql> SELECT gen_rnd_ssn();
+---------------+
| gen_rnd_ssn() |
+---------------+
| 912-45-1615   |
+---------------+
```

`gen_rnd_uk_nin()`返回一个从未用于合法数字的范围中选择的数字：

```sql
mysql> SELECT gen_rnd_uk_nin();
+------------------+
| gen_rnd_uk_nin() |
+------------------+
```

(由于其返回值仅用于测试目的，而非出版，因此不显示`gen_rnd_uk_nin()`函数的结果。不能保证该数字未分配给合法的 NIN。)

`gen_rnd_us_phone()`返回一个在未用于合法数字的 555 区号中的随机美国电话号码：

```sql
mysql> SELECT gen_rnd_us_phone();
+--------------------+
| gen_rnd_us_phone() |
+--------------------+
| 1-555-747-5627     |
+--------------------+
```

`gen_rnd_uuid()`返回一个从未用于合法标识符的范围中选择的数字：

```sql
mysql> SELECT gen_rnd_uuid();
+--------------------------------------+
| gen_rnd_uuid()                       |
+--------------------------------------+
| 68946384-6880-3150-6889-928076732539 |
+--------------------------------------+
```

##### 使用字典生成随机数据

MySQL 企业数据脱敏和去标识化使得可以将字典用作称为*术语*的随机值的来源。要使用字典，必须首先将其添加到`masking_dictionaries`系统表中并赋予一个名称。这些字典在组件初始化期间（服务器启动时）从表中读取并加载到缓存中。然后可以向字典中添加、移除和选择术语，并将其用作随机值或替换其他值。

注意

始终使用字典管理函数编辑字典，而不是直接修改表格。如果手动操作表格，则字典缓存将与表格不一致。

一个有效的`masking_dictionaries`表具有以下特征：

+   管理员在`mysql`模式中创建了`masking_dictionaries`系统表如下：

    ```sql
    CREATE TABLE IF NOT EXISTS
    masking_dictionaries(
        Dictionary VARCHAR(256) NOT NULL,
        Term VARCHAR(256) NOT NULL,
        UNIQUE INDEX dictionary_term_idx (Dictionary, Term),
        INDEX dictionary_idx (Dictionary)
    ) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4;
    ```

+   需要 MASKING_DICTIONARY_ADMIN 权限来添加和删除术语，或者删除整个字典。

+   表中可能包含多个字典及其术语。

+   任何用户账户都可以查看字典。通过足够的查询，所有字典中的术语都是可检索的。避免向字典表中添加敏感数据。

假设一个名为`DE_cities`的字典包含德国这些城市的名称：

```sql
Berlin
Munich
Bremen
```

使用`masking_dictionary_term_add()`来分配一个字典名称和一个术语：

```sql
mysql> SELECT masking_dictionary_term_add('DE_Cities', 'Berlin');
+----------------------------------------------------+
| masking_dictionary_term_add('DE_Cities', 'Berlin') |
+----------------------------------------------------+
|                                                  1 |
+----------------------------------------------------+
mysql> SELECT masking_dictionary_term_add('DE_Cities', 'Munich');
+----------------------------------------------------+
| masking_dictionary_term_add('DE_Cities', 'Munich') |
+----------------------------------------------------+
|                                                  1 |
+----------------------------------------------------+
mysql> SELECT masking_dictionary_term_add('DE_Cities', 'Bremen');
+----------------------------------------------------+
| masking_dictionary_term_add('DE_Cities', 'Bremen') |
+----------------------------------------------------+
|                                                  1 |
+----------------------------------------------------+
```

还假设一个名为`US_Cities`的字典包含美国这些城市的名称：

```sql
Houston
Phoenix
Detroit
```

```sql
mysql> SELECT masking_dictionary_term_add('US_Cities', 'Houston');
+-----------------------------------------------------+
| masking_dictionary_term_add('US_Cities', 'Houston') |
+-----------------------------------------------------+
|                                                   1 |
+-----------------------------------------------------+
mysql> SELECT masking_dictionary_term_add('US_Cities', 'Phoenix');
+-----------------------------------------------------+
| masking_dictionary_term_add('US_Cities', 'Phoenix') |
+-----------------------------------------------------+
|                                                   1 |
+-----------------------------------------------------+
mysql> SELECT masking_dictionary_term_add('US_Cities', 'Detroit');
+-----------------------------------------------------+ 
| masking_dictionary_term_add('US_Cities', 'Detroit') |
+-----------------------------------------------------+
|                                                   1 |
+-----------------------------------------------------+
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

要从多个字典中选择一个随机术语，随机选择一个字典，然后从中选择一个术语：

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

`gen_blocklist()`函数使得一个字典中的术语可以被另一个字典中的术语替换，从而实现了掩码替换。它的参数是要替换的术语，术语出现的字典，以及要选择替换的字典。例如，要用美国城市替换德国城市，或者反之，可以像这样使用`gen_blocklist()`：

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

如果要替换的术语不在第一个字典中，`gen_blocklist()`会原样返回它：

```sql
mysql> SELECT gen_blocklist('Moscow', 'DE_Cities', 'US_Cities');
+---------------------------------------------------+
| gen_blocklist('Moscow', 'DE_Cities', 'US_Cities') |
+---------------------------------------------------+
| Moscow                                            |
+---------------------------------------------------+
```

##### 使用掩码数据进行客户识别

在客服呼叫中心，一个常见的身份验证技术是要求客户提供他们社会安全号码（SSN）的最后四位数字。例如，一个客户可能说她的名字是乔安娜·邦德，她的最后四位 SSN 数字是`0007`。

假设包含客户记录的`customer`表具有这些列：

+   `id`: 客户 ID 号码。

+   `first_name`: 客户名字。

+   `last_name`: 客户姓氏。

+   `ssn`: 客户社会安全号码。

例如，表可能定义如下：

```sql
CREATE TABLE customer
(
  id         BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(40),
  last_name  VARCHAR(40),
  ssn        VARCHAR(11)
);
```

客服代表用于检查客户社会安全号码的应用程序可能执行类似这样的查询：

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

然而，这会将 SSN 暴露给客服代表，他们除了最后四位数字外不需要看到任何内容。相反，应用程序可以使用这个查询仅显示掩码的 SSN：

```sql
mysql> SELECT id, mask_ssn(CONVERT(ssn USING binary)) AS masked_ssn
mysql> FROM customer
mysql> WHERE first_name = 'Joanna' AND last_name = 'Bond';
+-----+-------------+
| id  | masked_ssn  |
+-----+-------------+
| 786 | ***-**-0007 |
+-----+-------------+
```

现在代表只看到必要的内容，客户隐私得到保护。

为什么在`CONVERT()`函数中使用参数`mask_ssn()`？因为`mask_ssn()`需要一个长度为 11 的参数。因此，即使`ssn`被定义为`VARCHAR(11)`，如果`ssn`列具有多字节字符集，当传递给可加载函数时可能会看起来比 11 个字节长，并在记录错误时返回`NULL`。将值转换为二进制字符串可以确保函数看到长度为 11 的参数。

当字符串参数没有单字节字符集时，可能需要类似的技术来处理其他数据脱敏函数。

##### 创建显示脱敏数据的视图

如果来自表的脱敏数据用于多个查询，定义一个生成脱敏数据的视图可能会很方便。这样，应用程序可以从视图中选择而无需在单独的查询中执行脱敏。

例如，可以像这样定义前一节中`customer`表的脱敏视图：

```sql
CREATE VIEW masked_customer AS
SELECT id, first_name, last_name,
mask_ssn(CONVERT(ssn USING binary)) AS masked_ssn
FROM customer;
```

然后查找客户的查询变得更简单，但仍返回脱敏数据：

```sql
mysql> SELECT id, masked_ssn
mysql> FROM masked_customer
mysql> WHERE first_name = 'Joanna' AND last_name = 'Bond';
+-----+-------------+
| id  | masked_ssn  |
+-----+-------------+
| 786 | ***-**-0007 |
+-----+-------------+
```
