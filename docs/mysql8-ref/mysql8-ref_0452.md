> 原文：[`dev.mysql.com/doc/refman/8.0/en/firewall-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/firewall-usage.html)

#### 8.4.7.3 使用 MySQL 企业防火墙

在使用 MySQL 企业防火墙之前，请根据 Section 8.4.7.2, “安装或卸载 MySQL 企业防火墙”中提供的说明进行安装。

本节描述了如何使用 SQL 语句配置 MySQL 企业防火墙。另外，MySQL Workbench 6.3.4 或更高版本提供了用于防火墙控制的图形界面。参见 MySQL 企业防火墙界面。

+   启用或禁用防火墙

+   分配防火墙权限

+   防火墙概念

+   注册防火墙组配置文件

+   注册防火墙帐户配置文件

+   监控防火墙

+   将帐户配置文件迁移到组配置文件

##### 启用或禁用防火墙

要启用或禁用防火墙，请设置`mysql_firewall_mode`系统变量。默认情况下，当安装防火墙时，此变量已启用。要明确控制初始防火墙状态，可以在服务器启动时设置该变量。例如，要在选项文件中启用防火墙，请使用以下行：

```sql
[mysqld]
mysql_firewall_mode=ON
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时设置和持久化防火墙设置：

```sql
SET PERSIST mysql_firewall_mode = OFF;
SET PERSIST mysql_firewall_mode = ON;
```

`SET PERSIST`为正在运行的 MySQL 实例设置一个值。它还保存该值，导致其在后续服务器重新启动时保留。要更改正在运行的 MySQL 实例的值，而不使其在后续重新启动时保留，使用`GLOBAL`关键字而不是`PERSIST`。参见 Section 15.7.6.1, “变量赋值的 SET 语法”。

##### 分配防火墙权限

安装防火墙后，向用于管理它的 MySQL 帐户或帐户授予适当的权限。权限取决于帐户应被允许执行哪些防火墙操作：

+   将`FIREWALL_EXEMPT`权限（自 MySQL 8.0.27 起可用）授予任何应免于防火墙限制的帐户。例如，对于配置防火墙的数据库管理员来说，这很有用，以避免配置错误导致甚至管理员也被锁定并无法执行语句的可能性。

+   将`FIREWALL_ADMIN`权限授予任何应具有完全管理防火墙访问权限的帐户。（某些管理防火墙功能可以由具有`FIREWALL_ADMIN`或已弃用的`SUPER`权限的帐户调用，如各个功能描述中所示。）

+   将`FIREWALL_USER`权限授予任何应仅具有其自身防火墙规则的管理访问权限的帐户。

+   为`mysql`系统数据库中的防火墙存储过程授予`EXECUTE`权限。这些存储过程可能调用管理功能，因此存储过程访问还需要前面所需的用于这些功能的权限。

注意

只能在安装了防火墙时授予`FIREWALL_EXEMPT`、`FIREWALL_ADMIN`和`FIREWALL_USER`权限，因为`MYSQL_FIREWALL`插件定义了这些权限。

##### 防火墙概念

MySQL 服务器允许客户端连接并接收他们发送的要执行的 SQL 语句。如果启用了防火墙，服务器会将每个未立即因语法错误而失败的传入语句传递给它。根据防火墙是否接受该语句，服务器会执行它或向客户端返回错误。本节描述了防火墙如何完成接受或拒绝语句的任务。

+   防火墙配置文件

+   防火墙语句匹配

+   配置文件操作模式

+   多个配置文件应用时的防火墙语句处理

###### 防火墙配置文件

防火墙使用确定是否允许语句执行的配置文件注册表。配置文件具有以下属性：

+   允许列表。允许列表是定义哪些语句可接受到配置文件的规则集。

+   当前的操作模式。该模式使得可以以不同方式使用配置文件。例如：配置文件可以置于训练模式以建立白名单；白名单可用于限制语句执行或入侵检测；配置文件可以完全禁用。

+   适用范围。范围指示配置文件适用于哪些客户端连接：

    +   防火墙支持基于账户的配置文件，使得每个配置文件都匹配特定的客户端账户（客户端用户名和主机名组合）。例如，您可以注册一个账户配置文件，其中白名单适用于源自`admin@localhost`的连接，以及另一个账户配置文件，其中白名单适用于源自`myapp@apphost.example.com`的连接。

    +   截至 MySQL 8.0.23 版本，防火墙支持可以有多个账户作为成员的组配置文件，配置文件白名单对所有成员均等适用。组配置文件使得对于需要将给定的一组白名单规则应用于多个账户的部署，管理更加简便且灵活。

最初，不存在任何配置文件，因此默认情况下，防火墙接受所有语句，并不影响 MySQL 账户可以执行哪些语句。要应用防火墙的保护功能，需要采取明确的行动：

+   向防火墙注册一个或多个配置文件。

+   通过为每个配置文件建立白名单来训练防火墙；也就是说，配置文件允许客户端执行的语句类型。

+   将训练过的配置文件置于保护模式，以加固 MySQL 防止未经授权的语句执行：

    +   MySQL 将每个客户端会话与特定的用户名和主机名组合关联。这个组合就是*会话账户*。

    +   对于每个客户端连接，防火墙使用会话账户来确定哪些配置文件适用于处理客户端发出的语句。

        防火墙仅接受适用于适用配置文件白名单的语句。

大多数防火墙原则在组配置文件和账户配置文件上都适用。这两种类型的配置文件在以下方面有所不同：

+   账户配置文件白名单仅适用于单个账户。组配置文件白名单适用于会话账户匹配任何属于该组的账户时。

+   使用账户配置文件为多个账户应用白名单时，需要为每个账户注册一个配置文件，并在每个配置文件中复制白名单。这意味着必须单独训练每个账户配置文件，因为每个配置文件必须使用适用于其的单个账户进行训练。

    组配置文件白名单适用于多个账户，无需为每个账户重复。组配置文件可以使用任何或所有组成员账户进行训练，或者训练可以限制为任何单个成员。无论哪种方式，白名单都适用于所有成员。

+   帐户配置文件名称基于连接到 MySQL 服务器的客户端取决于特定用户名称和主机名组合。组配置文件名称由防火墙管理员选择，除了长度必须在 1 到 288 个字符之外，没有其他约束。

注意

由于组配置文件比帐户配置文件具有优势，并且因为具有单个成员帐户的组配置文件在逻辑上等同于该帐户的帐户配置文件，建议所有新的防火墙配置文件都创建为组配置文件。从 MySQL 8.0.26 开始，帐户配置文件已被弃用，并且将在未来的 MySQL 版本中被移除。有关转换现有帐户配置文件的帮助，请参阅 Migrating Account Profiles to Group Profiles。

防火墙提供的基于配置文件的保护使得可以实施以下策略：

+   如果应用程序具有独特的保护要求，请配置它使用不用于任何其他目的的帐户，并为该帐户设置组配置文件或帐户配置文件。

+   如果相关应用程序共享保护要求，请将每个应用程序与其自己的帐户关联，然后将这些应用程序帐户添加为同一组配置文件的成员。或者，配置所有应用程序使用相同的帐户，并将它们与该帐户的帐户配置文件关联。

###### 防火墙语句匹配

防火墙执行的语句匹配不使用从客户端接收的 SQL 语句。相反，服务器将传入的语句转换为规范化的摘要形式，防火墙操作使用这些摘要。语句规范化的好处在于它使相似的语句能够被分组并使用单个模式识别。例如，这些语句彼此不同：

```sql
SELECT first_name, last_name FROM customer WHERE customer_id = 1;
select first_name, last_name from customer where customer_id = 99;
SELECT first_name, last_name FROM customer WHERE customer_id = 143;
```

但它们都具有相同的规范化摘要形式：

```sql
SELECT `first_name` , `last_name` FROM `customer` WHERE `customer_id` = ?
```

通过规范化，防火墙白名单可以存储与从客户端接收的许多不同语句匹配的摘要。有关规范化和摘要的更多信息，请参阅第 29.10 节，“性能模式语句摘要和抽样”。

警告

将`max_digest_length`系统变量设置为零会禁用摘要生成，这也会禁用需要摘要的服务器功能，例如 MySQL Enterprise Firewall。

###### 配置文件操作模式

注册到防火墙的每个配置文件都有自己的操作模式，可以从以下值中选择：

+   `OFF`：此模式禁用配置文件。防火墙将其视为无效并忽略它。

+   `RECORDING`：这是防火墙的训练模式。从客户端接收的与配置文件匹配的传入语句被视为配置文件可接受的一部分，并成为其“指纹”的一部分。防火墙记录每个语句的规范化摘要形式，以学习配置文件的可接受语句模式。每个模式都是一个规则，规则的并集是配置文件白名单。

    组和账户配置文件之间的区别在于，组配置文件的语句记录可以限制为仅接收来自单个组成员（培训成员）的语句。

+   `PROTECTING`：在此模式下，配置文件允许或阻止语句执行。防火墙将传入的语句与配置文件白名单进行匹配，仅接受与之匹配的语句，并拒绝不匹配的语句。在`RECORDING`模式下训练配置文件后，将其切换到`PROTECTING`模式以加固 MySQL 防止被偏离白名单的语句访问。如果启用了`mysql_firewall_trace`系统变量，防火墙还会将被拒绝的语句写入错误日志。

+   `DETECTING`：此模式检测但不阻止入侵（与配置文件白名单中没有匹配的语句因为可疑而被拦截）。在`DETECTING`模式下，防火墙将可疑语句写入错误日志，但接受它们而不拒绝访问。

当配置文件被分配任何前述模式值时，防火墙会将模式存储在配置文件中。防火墙模式设置操作还允许使用`RESET`模式值，但不会存储此值：将配置文件设置为`RESET`模式会导致防火墙删除配置文件的所有规则并将其模式设置为`OFF`。

注意

在`DETECTING`模式下写入错误日志的消息或因为启用了`mysql_firewall_trace`而写入的消息被写入为 Notes，这些是信息消息。为确保这些消息出现在错误日志中并且不被丢弃，请确保错误日志的详细程度足以包括信息消息。例如，如果您正在使用基于优先级的日志过滤，如第 7.4.2.5 节，“基于优先级的错误日志过滤（log_filter_internal）”中描述的那样，请将`log_error_verbosity`系统变量设置为值 3。

###### 当多个配置文件适用时的防火墙语句处理

为简单起见，后续描述如何设置配置文件的部分将从防火墙将客户端传入语句与仅一个配置文件（组配置文件或账户配置文件）匹配的角度进行。但是防火墙操作可能更加复杂：

+   组配置文件可以包括多个账户作为成员。

+   一个账户可以是多个组配置文件的成员。

+   多个配置文件可以匹配给定的客户端。

以下描述涵盖了防火墙在可���有多个配置文件适用于传入语句时的一般操作情况。

正如之前提到的，MySQL 将每个客户端会话与特定的用户名和主机名组合（称为*会话账户*）关联起来。防火墙将会话账户与注册的配置文件进行匹配，以确定哪些配置文件适用于处理来自会话的传入语句：

+   防火墙会忽略不活动的配置文件（模式为`OFF`的配置文件）。

+   会话账户与包含具有相同用户和主机的成员的每个活动组配置文件匹配。可能会有多个这样的组配置文件。

+   会话账户与具有相同用户和主机的活动账户配置文件匹配，如果有的话。最多只有一个这样的账户配置文件。

换句话说，会话账户可以匹配 0 或多个活动组配置文件，以及 0 或 1 个活动账户配置文件。这意味着对于给定会话，防火墙可以适用 0、1 或多个防火墙配置文件，防火墙将处理每个传入语句如下：

+   如果没有适用的配置文件，防火墙不会施加任何限制并接受该语句。

+   如果有适用的配置文件，它们的模式决定语句处理方式：

    +   防火墙将语句记录在每个处于`RECORDING`模式的适用配置文件的允许列表中。

    +   对于每个处于`DETECTING`模式的适用配置文件，如果语句可疑（不匹配配置文件允许列表），防火墙将将语句写入错误日志。

    +   如果至少有一个适用配置文件处于`RECORDING`或`DETECTING`模式（这些模式接受所有���句），或者如果语句与至少一个处于`PROTECTING`模式的适用配置文件的允许列表匹配，则防火墙将接受该语句。否则，防火墙将拒绝该语句（如果启用了`mysql_firewall_trace`系统变量，则将其写入错误日志）。

将这个描述记在心中，接下来的部分将回到单个组配置文件或单个账户配置文件适用的简单情况，并介绍如何设置每种类型的配置文件。

##### 注册防火墙组配置文件

MySQL Enterprise Firewall 支持从 MySQL 8.0.23 开始注册组配置文件。组配置文件可以有多个账户作为其成员。要使用防火墙组配置文件来保护 MySQL 免受来自特定账户的传入语句的影响，请按照以下步骤操作：

1.  注册组配置文件并将其置于`RECORDING`模式。

1.  将成员账户添加到组配置文件中。

1.  使用成员账户连接到 MySQL 服务器并执行要学习的语句。这将训练组配置文件并建立形成配置文件允许列表的规则。

1.  将其他要作为组成员的账户添加到组配置文件中。

1.  将组配置文件切换到`PROTECTING`模式。当客户端使用组配置文件的任何成员帐户连接到服务器时，配置文件允许列表会限制语句执行。

1.  如果需要额外的培训，请将组配置文件再次切换到`RECORDING`模式，使用新的语句模式更新其允许列表，然后将其切换回`PROTECTING`模式。

防火墙相关帐户引用的准则如下：

+   注意帐户引用出现的上下文。为防火墙操作命名帐户时，请将其指定为单引号字符串（`'*`user_name`*@*`host_name`*'`）。这与通常的 MySQL 语句约定不同，例如`CREATE USER`和`GRANT`，其中您分别引用帐户名称的用户和主机部分（`'*`user_name`*'@'*`host_name`*'`）。

    为防火墙操作命名帐户的要求意味着您不能使用用户名称中嵌入`@`字符的帐户。

+   防火墙根据服务器验证的实际用户和主机名称来评估语句。在配置文件中注册帐户时，请勿使用通配符字符或网络掩码：

    +   假设存在一个名为`me@%.example.org`的帐户，并且客户端使用它从主机`abc.example.org`连接到服务器。

    +   帐户名称包含`%`通配符字符，但服务器将客户端验证为具有用户名`me`和主机名`abc.example.com`，这就是防火墙看到的内容。

    +   因此，用于防火墙操作的帐户名称是`me@abc.example.org`而不是`me@%.example.org`。

以下过程显示了如何向防火墙注册组配置文件，训练防火墙了解该配置文件的可接受语句（其允许列表），使用配置文件保护 MySQL 免受不可接受语句执行，并添加和删除组成员。示例使用`fwgrp`作为组配置文件名称。假定示例配置文件用于访问`sakila`数据库中的表的应用程序的客户端（可在`dev.mysql.com/doc/index-other.html`找到）。

使用管理 MySQL 帐户执行此过程中的步骤，除了由防火墙组配置文件的成员帐户执行的步骤。对于由成员帐户执行的语句，默认数据库应为`sakila`。（您可以通过相应调整指令来使用不同的数据库。）

1.  如有必要，请创建要成为`fwgrp`组配置文件成员的帐户，并授予它们适当的访问权限。这里显示了一个成员的语句（选择适当的密码）：

    ```sql
    CREATE USER 'member1'@'localhost' IDENTIFIED BY '*password*';
    GRANT ALL ON sakila.* TO 'member1'@'localhost';
    ```

1.  使用`sp_set_firewall_group_mode()`存储过程向防火墙注册组配置文件，并将配置文件置于`RECORDING`（培训）模式：

    ```sql
    CALL mysql.sp_set_firewall_group_mode('fwgrp', 'RECORDING');
    ```

1.  使用`sp_firewall_group_enlist()`存储过程添加一个初始成员帐户，用于训练组配置文件的允许列表：

    ```sql
    CALL mysql.sp_firewall_group_enlist('fwgrp', 'member1@localhost');
    ```

1.  使用初始成员帐户训练组配置文件，从服务器主机连接到服务器，以便防火墙看到`member1@localhost`的会话帐户。然后执行一些语句，以便被视为配置文件的合法内容。例如：

    ```sql
    SELECT title, release_year FROM film WHERE film_id = 1;
    UPDATE actor SET last_update = NOW() WHERE actor_id = 1;
    SELECT store_id, COUNT(*) FROM inventory GROUP BY store_id;
    ```

    防火墙接收来自`member1@localhost`帐户的语句。因为该帐户是`fwgrp`配置文件的成员，该配置文件处于`RECORDING`模式，防火墙将语句解释为适用于`fwgrp`并将语句的规范化摘要形式记录为`fwgrp`允许列表中的规则。然后，这些规则适用于所有是`fwgrp`成员的帐户。

    注意

    直到`fwgrp`组配置文件以`RECORDING`模式接收语句，其允许列表为空，相当于“拒绝所有”。没有语句可以匹配空的允许列表，这带来以下影响：

    +   组配置文件无法切换到`PROTECTING`模式。它会拒绝每个语句，有效地禁止组成员执行任何语句。

    +   组配置文件可以切换到`DETECTING`模式。在这种情况下，配置文件接受每个语句，但将其记录为可疑。

1.  此时，组配置文件信息已缓存，包括其名称、成员资格和允许列表。要查看此信息，请查询性能模式防火墙表：

    ```sql
    mysql> SELECT MODE FROM performance_schema.firewall_groups
           WHERE NAME = 'fwgrp';
    +-----------+
    | MODE      |
    +-----------+
    | RECORDING |
    +-----------+
    mysql> SELECT * FROM performance_schema.firewall_membership
           WHERE GROUP_ID = 'fwgrp' ORDER BY MEMBER_ID;
    +----------+-------------------+
    | GROUP_ID | MEMBER_ID         |
    +----------+-------------------+
    | fwgrp    | member1@localhost |
    +----------+-------------------+
    mysql> SELECT RULE FROM performance_schema.firewall_group_allowlist
           WHERE NAME = 'fwgrp';
    +----------------------------------------------------------------------+
    | RULE                                                                 |
    +----------------------------------------------------------------------+
    | SELECT @@`version_comment` LIMIT ?                                   |
    | UPDATE `actor` SET `last_update` = NOW ( ) WHERE `actor_id` = ?      |
    | SELECT `title` , `release_year` FROM `film` WHERE `film_id` = ?      |
    | SELECT `store_id` , COUNT ( * ) FROM `inventory` GROUP BY `store_id` |
    +----------------------------------------------------------------------+
    ```

    注意

    `@@version_comment`规则来自连接到服务器时由**mysql**客户端自动发送的语句。

    重要

    根据应用程序使用情况训练防火墙。例如，为了确定服务器特性和功能，给定的 MySQL 连接器可能会在每个会话开始时向服务器发送语句。如果应用程序通常通过该连接器使用，也要使用该连接器训练防火墙。这使得这些初始语句成为与应用程序关联的组配置文件的允许列表的一部分。

1.  再次调用`sp_set_firewall_group_mode()`将组配置文件切换到`PROTECTING`模式：

    ```sql
    CALL mysql.sp_set_firewall_group_mode('fwgrp', 'PROTECTING');
    ```

    重要

    将组配置文件从`RECORDING`模式切换出来，将其缓存数据同步到提供持久底层存储的`mysql`系统数据库表中。如果不为正在记录的配置文件切换模式，则缓存数据不会写入持久存储，并且在服务器重新启动时将丢失。

1.  将应该成为成员的任何其他帐户添加到组配置文件中：

    ```sql
    CALL mysql.sp_firewall_group_enlist('fwgrp', 'member2@localhost');
    CALL mysql.sp_firewall_group_enlist('fwgrp', 'member3@localhost');
    CALL mysql.sp_firewall_group_enlist('fwgrp', 'member4@localhost');
    ```

    使用`member1@localhost`帐户训练的配置文件允许列表现在也适用于其他帐户。

1.  要验证更新后的组成员资格，请再次查询`firewall_membership`表：

    ```sql
    mysql> SELECT * FROM performance_schema.firewall_membership
           WHERE GROUP_ID = 'fwgrp' ORDER BY MEMBER_ID;
    +----------+-------------------+
    | GROUP_ID | MEMBER_ID         |
    +----------+-------------------+
    | fwgrp    | member1@localhost |
    | fwgrp    | member2@localhost |
    | fwgrp    | member3@localhost |
    | fwgrp    | member4@localhost |
    +----------+-------------------+
    ```

1.  使用组中的任何帐户执行一些可接受和不可接受的语句来测试防火墙的组配置文件。防火墙将每个帐户的语句与配置文件的允许列表进行匹配并接受或拒绝：

    +   此语句与训练语句不完全相同，但产生的规范化语句与其中一个相同，因此防火墙接受它：

        ```sql
        mysql> SELECT title, release_year FROM film WHERE film_id = 98;
        +-------------------+--------------+
        | title             | release_year |
        +-------------------+--------------+
        | BRIGHT ENCOUNTERS |         2006 |
        +-------------------+--------------+
        ```

    +   这些语句在允许列表中找不到匹配项，因此防火墙会拒绝每个语句并显示错误：

        ```sql
        mysql> SELECT title, release_year FROM film WHERE film_id = 98 OR TRUE;
        ERROR 1045 (28000): Statement was blocked by Firewall
        mysql> SHOW TABLES LIKE 'customer%';
        ERROR 1045 (28000): Statement was blocked by Firewall
        mysql> TRUNCATE TABLE mysql.slow_log;
        ERROR 1045 (28000): Statement was blocked by Firewall
        ```

    +   如果启用了`mysql_firewall_trace`系统变量，则防火墙还会将被拒绝的语句写入错误日志。例如：

        ```sql
        [Note] Plugin MYSQL_FIREWALL reported:
        'ACCESS DENIED for 'member1@localhost'. Reason: No match in allowlist.
        Statement: TRUNCATE TABLE `mysql` . `slow_log`'
        ```

        这些日志消息可能有助于识别攻击源，如果有必要的话。

1.  如果需要从组配置文件中移除成员，请使用`sp_firewall_group_delist()`存储过程，而不是`sp_firewall_group_enlist()`：

    ```sql
    CALL mysql.sp_firewall_group_delist('fwgrp', 'member3@localhost');
    ```

防火墙组配置文件现在已经为成员帐户进行了训练。当客户端使用组中的任何帐户连接并尝试执行语句时，配置文件将保护 MySQL 免受未在允许列表中匹配的语句的影响。

刚刚展示的过程在训练允许列表之前仅向组配置文件添加了一个成员。这样做可以通过限制哪些帐户可以向允许列表添加新的可接受语句，从而更好地控制训练期间。如果需要进行额外培训，可以将配置文件切换回`RECORDING`模式：

```sql
CALL mysql.sp_set_firewall_group_mode('fwgrp', 'RECORDING');
```

然而，这使得组中的任何成员都可以执行语句并将其添加到允许列表中。为了将额外培训限制在单个组成员身上，请调用`sp_set_firewall_group_mode_and_user()`，它类似于`sp_set_firewall_group_mode()`，但多接受一个参数，指定哪个帐户被允许以`RECORDING`模式训练配置文件。例如，要仅允许`member4@localhost`进行训练，请执行以下操作：

```sql
CALL mysql.sp_set_firewall_group_mode_and_user('fwgrp', 'RECORDING', 'member4@localhost');
```

这使得指定帐户可以进行额外培训，而无需移除其他组成员。他们可以执行语句，但这些语句不会添加到允许列表中。（但请记住，在`RECORDING`模式下，其他成员可以执行*任何*语句。）

注意

为了避免指定特定帐户作为组配置文件的训练帐户时出现意外行为，始终确保该帐户是组的成员。

完成额外培训后，将组配置文件设置回`PROTECTING`模式：

```sql
CALL mysql.sp_set_firewall_group_mode('fwgrp', 'PROTECTING');
```

由`sp_set_firewall_group_mode_and_user()`建立的训练帐户保存在组配置文件中，因此防火墙会在以后需要更多培训时记住它。因此，如果调用`sp_set_firewall_group_mode()`（不接受训练帐户参数），则当前配置文件训练帐户`member4@localhost`保持不变。

如果确实希望启用所有组成员在`RECORDING`模式下执行训练，则清除训练账户，调用`sp_set_firewall_group_mode_and_user()`并为账户参数传递`NULL`值：

```sql
CALL mysql.sp_set_firewall_group_mode_and_user('fwgrp', 'RECORDING', NULL);
```

可以通过将不匹配的语句记录为可疑来检测入侵，而不拒绝访问。首先，将组配置文件设置为`DETECTING`模式：

```sql
CALL mysql.sp_set_firewall_group_mode('fwgrp', 'DETECTING');
```

然后，使用成员账户执行一个不匹配组配置文件允许列表的语句。在`DETECTING`模式下，防火墙允许执行不匹配的语句：

```sql
mysql> SHOW TABLES LIKE 'customer%';
+------------------------------+
| Tables_in_sakila (customer%) |
+------------------------------+
| customer                     |
| customer_list                |
+------------------------------+
```

此外，防火墙会将消息写入错误日志：

```sql
[Note] Plugin MYSQL_FIREWALL reported:
'SUSPICIOUS STATEMENT from 'member1@localhost'. Reason: No match in allowlist.
Statement: SHOW TABLES LIKE ?'
```

要禁用组配置文件，请将其模式更改为`OFF`：

```sql
CALL mysql.sp_set_firewall_group_mode(*group*, 'OFF');
```

要忘记配置文件的所有训练并禁用它，请重置它：

```sql
CALL mysql.sp_set_firewall_group_mode(*group*, 'RESET');
```

重置操作会导致防火墙删除配置文件的所有规则并将其模式设置为`OFF`。

##### 注册防火墙账户配置文件

MySQL 企业防火墙允许注册与个人账户对应的配置文件。要使用防火墙账户配置文件保护 MySQL 免受来自特定账户的传入语句的影响，请按照以下步骤操作：

1.  注册账户配置文件并将其设置为`RECORDING`模式。

1.  使用账户连接到 MySQL 服务器并执行要学习的语句。这将训练账户配置文件并建立形成配置文件允许列表的规则。

1.  将账户配置文件切换到`PROTECTING`模式。当客户端使用该账户连接到服务器时，账户配置文件允许列表将限制语句执行。

1.  如果需要额外的训练，请再次将账户配置文件切换到`RECORDING`模式，使用新的语句模式更新其允许列表，然后将其切换回`PROTECTING`模式。

遵循以下与防火墙相关的账户引用指南：

+   注意账户引用出现的上下文。要为防火墙操作命名一个账户，请将其指定为单引号字符串（`'*`user_name`*@*`host_name`*'`）。这与通常的 MySQL 语句约定不同，例如`CREATE USER`和`GRANT`，其中您分别引用账户名称的用户和主机部分（`'*`user_name`*'@'*`host_name`*'`）。

    为防火墙操作命名账户时，要求将账户命名为单引号字符串意味着您不能使用用户名称中嵌入`@`字符的账户。

+   防火墙根据服务器验证的实际用户和主机名称对语句进行评估。在配置文件中注册账户时，不要使���通配符字符或网络掩码：

    +   假设存在一个名为`me@%.example.org`的账户，并且客户端使用它从主机`abc.example.org`连接到服务器。

    +   账户名称包含`%`通配符字符，但服务器将客户端验证为具有用户名`me`和主机名`abc.example.com`，这就是防火墙看到的内容。

    +   因此，用于防火墙操作的账户名称是`me@abc.example.org`而不是`me@%.example.org`。

以下过程展示了如何向防火墙注册账户配置文件，训练防火墙了解该配置文件的可接受语句（其允许列表），并使用配置文件保护 MySQL 免受账户执行不可接受语句的影响。假定示例账户`fwuser@localhost`用于由访问`sakila`数据库中的表的应用程序（可在`dev.mysql.com/doc/index-other.html`找到）。

使用管理 MySQL 账户执行此过程中的步骤，除了那些指定由与防火墙注册的账户配置文件对应的`fwuser@localhost`账户执行的步骤。对于使用此账户执行的语句，默认数据库应为`sakila`。（您可以通过相应调整指令来使用不同的数据库。）

1.  如有必要，创建用于执行语句的账户（选择适当的密码）并为`sakila`数据库授予权限：

    ```sql
    CREATE USER 'fwuser'@'localhost' IDENTIFIED BY '*password*';
    GRANT ALL ON sakila.* TO 'fwuser'@'localhost';
    ```

1.  使用`sp_set_firewall_mode()`存储过程向防火墙注册账户配置文件并将配置文件置于`RECORDING`（训练）模式：

    ```sql
    CALL mysql.sp_set_firewall_mode('fwuser@localhost', 'RECORDING');
    ```

1.  要训练已注册的账户配置文件，请从服务器主机作为`fwuser`连接到服务器，以便防火墙看到`fwuser@localhost`的会话账户。然后使用该账户执行一些被视为配置文件合法的语句。例如：

    ```sql
    SELECT first_name, last_name FROM customer WHERE customer_id = 1;
    UPDATE rental SET return_date = NOW() WHERE rental_id = 1;
    SELECT get_customer_balance(1, NOW());
    ```

    因为配置文件处于`RECORDING`模式，防火墙将语句的规范化摘要形式记录为配置文件允许列表中的规则。

    注意

    直到`fwuser@localhost`账户配置文件在`RECORDING`模式下接收语句，其允许列表为空，相当于“拒绝所有”。空的允许列表无法匹配任何语句，这带来以下影响：

    +   账户配置文件无法切换到`PROTECTING`模式。它会拒绝每个语句，有效地禁止账户执行任何语句。

    +   账户配置文件可以切换到`DETECTING`模式。在这种情况下，配置文件接受每个语句但将其记录为可疑。

1.  此时，账户配置文件信息已被缓存。要查看此信息，请查询`INFORMATION_SCHEMA`防火墙表：

    ```sql
    mysql> SELECT MODE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS
           WHERE USERHOST = 'fwuser@localhost';
    +-----------+
    | MODE      |
    +-----------+
    | RECORDING |
    +-----------+
    mysql> SELECT RULE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_WHITELIST
           WHERE USERHOST = 'fwuser@localhost';
    +----------------------------------------------------------------------------+
    | RULE                                                                       |
    +----------------------------------------------------------------------------+
    | SELECT `first_name` , `last_name` FROM `customer` WHERE `customer_id` = ?  |
    | SELECT `get_customer_balance` ( ? , NOW ( ) )                              |
    | UPDATE `rental` SET `return_date` = NOW ( ) WHERE `rental_id` = ?          |
    | SELECT @@`version_comment` LIMIT ?                                         |
    +----------------------------------------------------------------------------+
    ```

    注意

    `@@version_comment`规则来自连接到服务器时由**mysql**客户端自动发送的语句。

    重要

    根据应用程序使用情况训练防火墙。例如，为了确定服务器特性和功能，给定的 MySQL 连接器可能会在每个会话开始时向服务器发送语句。如果一个应用程序通常通过该连接器使用，也要使用该连接器来训练防火墙。这样可以使这些初始语句成为与应用程序关联的账户配置的白名单的一部分。

1.  再次调用`sp_set_firewall_mode()`，这次将账户配置切换到`PROTECTING`模式：

    ```sql
    CALL mysql.sp_set_firewall_mode('fwuser@localhost', 'PROTECTING');
    ```

    重要提示

    将账户配置从`RECORDING`模式切换出来会将其缓存数据同步到提供持久底层存储的`mysql`系统数据库表中。如果不切换正在记录的配置的模式，则缓存数据不会写入持久存储，并且在服务器重新启动时会丢失。

1.  通过使用账户执行一些可接受和不可接受的语句来测试账户配置。防火墙会将来自账户的每个语句与配置白名单进行匹配并接受或拒绝它：

    +   该语句与训练语句不完全相同，但产生的规范化语句与其中一个相同，因此防火墙接受它：

        ```sql
        mysql> SELECT first_name, last_name FROM customer WHERE customer_id = '48';
        +------------+-----------+
        | first_name | last_name |
        +------------+-----------+
        | ANN        | EVANS     |
        +------------+-----------+
        ```

    +   这些语句在白名单中找不到匹配项，因此防火墙会拒绝每个语句并显示错误：

        ```sql
        mysql> SELECT first_name, last_name FROM customer WHERE customer_id = 1 OR TRUE;
        ERROR 1045 (28000): Statement was blocked by Firewall
        mysql> SHOW TABLES LIKE 'customer%';
        ERROR 1045 (28000): Statement was blocked by Firewall
        mysql> TRUNCATE TABLE mysql.slow_log;
        ERROR 1045 (28000): Statement was blocked by Firewall
        ```

    +   如果启用了`mysql_firewall_trace`系统变量，防火墙还会将被拒绝的语句写入错误日志。例如：

        ```sql
        [Note] Plugin MYSQL_FIREWALL reported:
        'ACCESS DENIED for fwuser@localhost. Reason: No match in allowlist.
        Statement: TRUNCATE TABLE `mysql` . `slow_log`'
        ```

        这些日志消息可能有助于识别攻击源，如果有必要的话。

防火墙账户配置现在已经针对`fwuser@localhost`账户进行了训练。当客户端使用该账户连接并尝试执行语句时，该配置会保护 MySQL 免受未被配置白名单匹配的语句的影响。

可以通过将不匹配的语句记录为可疑来检测入侵，而不拒绝访问。首先，将账户配置置于`DETECTING`模式：

```sql
CALL mysql.sp_set_firewall_mode('fwuser@localhost', 'DETECTING');
```

然后，使用该账户执行一个不匹配账户配置白名单的语句。在`DETECTING`模式下，防火墙允许执行不匹配的语句：

```sql
mysql> SHOW TABLES LIKE 'customer%';
+------------------------------+
| Tables_in_sakila (customer%) |
+------------------------------+
| customer                     |
| customer_list                |
+------------------------------+
```

此外，防火墙会将一条消息写入错误日志：

```sql
[Note] Plugin MYSQL_FIREWALL reported:
'SUSPICIOUS STATEMENT from 'fwuser@localhost'. Reason: No match in allowlist.
Statement: SHOW TABLES LIKE ?'
```

要禁用账户配置，请将其模式更改为`OFF`：

```sql
CALL mysql.sp_set_firewall_mode(*user*, 'OFF');
```

要忘记配置的所有训练并禁用它，请重置它：

```sql
CALL mysql.sp_set_firewall_mode(*user*, 'RESET');
```

重置操作会导致防火墙删除配置的所有规则并将其模式设置为`OFF`。

##### 监控防火墙

要评估防火墙的活动，请检查其状态变量。例如，在执行早期显示的过程来训练和保护`fwgrp`组配置后，变量如下所示：

```sql
mysql> SHOW GLOBAL STATUS LIKE 'Firewall%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Firewall_access_denied     | 3     |
| Firewall_access_granted    | 4     |
| Firewall_access_suspicious | 1     |
| Firewall_cached_entries    | 4     |
+----------------------------+-------+
```

变量分别表示被拒绝的语句数、被接受的语句数、被记录为可疑的语句数以及被添加到缓存中的语句数。`Firewall_access_granted`计数为 4，因为通过注册账户连接时，**mysql**客户端发送了`@@version_comment`语句，每次连接都发送了三次，再加上未在`DETECTING`模式下被阻止的`SHOW TABLES`语句。

##### 将账户配置文件迁移到组配置文件

在 MySQL 8.0.23 之前，MySQL 企业防火墙仅支持每个应用于单个账户的账户配置文件。从 MySQL 8.0.23 开始，防火墙还支持每个可以应用于多个账户的组配置文件。当需要将相同的允许列表应用于多个账户时，组配置文件能够更轻松地进行管理：不需要为每个账户创建一个账户配置文件并在所有这些配置文件中复制允许列表，只需创建一个组配置文件并将账户作为其成员。然后该组允许列表将应用于所有账户。

一个只有一个成员账户的组配置文件在逻辑上等同于该账户的账户配置文件，因此可以完全使用组配置文件来管理防火墙，而不是混合使用账户和组配置文件。对于新的防火墙安装，通过统一创建新的组配置文件而避免账户配置文件，可以实现这一点。

由于组配置文件提供了更大的灵活性，建议所有新的防火墙配置文件都以组配置文件的形式创建。从 MySQL 8.0.26 开始，账户配置文件已被弃用，并可能在未来的 MySQL 版本中被移除。对于已包含账户配置文件的防火墙安装升级，MySQL 8.0.26 及更高版本的 MySQL 企业防火墙包含一个名为`sp_migrate_firewall_user_to_group()`的存储过程，帮助您将账户配置文件转换为组配置文件。要使用它，请按照具有`FIREWALL_ADMIN`权限的用户执行以下过程：

1.  运行`firewall_profile_migration.sql`脚本来安装`sp_migrate_firewall_user_to_group()`存储过程。该脚本位于您的 MySQL 安装的`share`目录中。

    ```sql
    $> mysql -u root -p < firewall_profile_migration.sql
    Enter password: *(enter root password here)*
    ```

1.  通过查询信息模式`MYSQL_FIREWALL_USERS`表来识别存在哪些账户配置文件。例如：

    ```sql
    mysql> SELECT USERHOST FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS;
    +-------------------------------+
    | USERHOST                      |
    +-------------------------------+
    | admin@localhost               |
    | local_client@localhost        |
    | remote_client@abc.example.com |
    +-------------------------------+
    ```

1.  对于上一步骤识别出的每个账户配置文件，将其转换为一个组配置文件：

    ```sql
    CALL mysql.sp_migrate_firewall_user_to_group('admin@localhost', 'admins');
    CALL mysql.sp_migrate_firewall_user_to_group('local_client@localhost', 'local_clients');
    CALL mysql.sp_migrate_firewall_user_to_group('remote_client@localhost', 'remote_clients');
    ```

    在每种情况下，账户配置文件必须存在且不能处于`RECORDING`模式，并且群组配置文件不能已经存在。生成的群组配置文件以命名的账户作为其唯一的成员，并且也被设置为群组培训账户。群组配置文件的操作模式取自账户配置文件的操作模式。

1.  （可选）移除`sp_migrate_firewall_user_to_group()`：

    ```sql
    DROP PROCEDURE IF EXISTS mysql.sp_migrate_firewall_user_to_group;
    ```

有关`sp_migrate_firewall_user_to_group()`的更多详细信息，请参见防火墙杂项存储过程。
