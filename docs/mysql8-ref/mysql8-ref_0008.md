# 1.3 MySQL 8.0 中的新功能

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)

本节总结了 MySQL 8.0 中新增、弃用和删除的内容。一个相关的部分列出了 MySQL 8.0 中新增、弃用或删除的 MySQL 服务器选项和变量；请参见第 1.4 节，“MySQL 8.0 中新增、弃用或删除的服务器和状态变量和选项”。

+   MySQL 8.0 中新增的功能

+   MySQL 8.0 中弃用的功能

+   MySQL 8.0 中删除的功能

### MySQL 8.0 中新增的功能

以下功能已添加到 MySQL 8.0 中：

+   **数据字典。** MySQL 现在包含一个事务性数据字典，用于存储有关数据库对象的信息。在以前的 MySQL 版本中，字典数据存储在元数据文件和非事务性表中。更多信息，请参见第十六章，“MySQL 数据字典”。

+   **原子数据定义语句（原子 DDL）。** 原子 DDL 语句将与 DDL 操作相关的数据字典更新、存储引擎操作和二进制日志写入合并为单个原子事务。更多信息，请参见第 15.1.1 节，“原子数据定义语句支持”。

+   **升级过程。** 以前，在安装新版本的 MySQL 后，MySQL 服务器会在下次启动时自动升级数据字典表，之后 DBA 需要手动调用**mysql_upgrade**来升级`mysql`模式中的系统表，以及其他模式中的对象，如`sys`模式和用户模式。

    从 MySQL 8.0.16 开始，服务器执行了之前由 **mysql_upgrade** 处理的任务。安装新的 MySQL 版本后，服务器现在会在下次启动时自动执行所有必要的升级任务，不再依赖于 DBA 调用 **mysql_upgrade**。此外，服务器会更新帮助表的内容（之前 **mysql_upgrade** 没有做）。新的 `--upgrade` 服务器选项可控制服务器如何执行自动数据字典和服务器升级操作。有关更多信息，请参见 Section 3.4, “What the MySQL Upgrade Process Upgrades”。

+   **会话重用。** MySQL Server 现在默认支持 SSL 会话重用，并设置超时时间以控制服务器维护会话缓存的时间段，该时间段内客户端被允许请求新连接的会话重用。所有 MySQL 客户端程序都支持会话重用。有关服务器端和客户端配置信息，请参见 Section 8.3.5, “Reusing SSL Sessions”。

    此外，C 应用程序现在可以使用 C API 功能来启用加密连接的会话重用（参见 SSL Session Reuse）。

+   **安全性和账户管理。** 这些增强功能旨在提高安全性并增强 DBA 在账户管理方面的灵活性：

    +   MySQL Enterprise Audit 现在支持使用 scheduler 组件来配置和执行定期任务以刷新内存缓存。有关设置说明，请参见 Enabling the Audit Log Flush Task。

    +   新的密码验证系统变量允许配置和强制用户在尝试更换自己的 MySQL 账户密码时必须更改的最小字符数。此新验证设置是当前密码中字符总数的百分比。例如，如果 `validate_password.changed_characters_percentage` 的值为 50，则替换账户密码中至少一半的字符不能出现在当前密码中，否则密码将被拒绝。有关更多信息，请参见 Section 8.4.3, “The Password Validation Component”。

    +   MySQL Enterprise Edition 现在提供基于组件的数据脱敏和去标识化功能，而不是基于 MySQL 8.0.13 中引入的插件库。MySQL Enterprise Data Masking 和 De-Identification 组件支持多字节字符，将脱敏字典存储在数据库表中，并提供几个新功能。更多信息，请参见 第 8.5.1 节，“数据脱敏组件与数据脱敏插件”。

    +   在 MySQL 8.0.33 之前，`mysql` 系统数据库用于 MySQL Enterprise Audit 的持久存储过滤器和用户帐户数据。为了增强灵活性，新的 `audit_log_database` 服务器系统变量现在允许在服务器启动时指定全局模式命名空间中的其他数据库。`mysql` 系统数据库是表存储的默认设置。

    +   `mysql` 系统数据库中的授权表现在是 `InnoDB`（事务性）表。以前，这些是 `MyISAM`（非事务性）表。授权表存储引擎的更改导致了帐户管理语句行为的相应更改。以前，命名多个用户的帐户管理语句（如 `CREATE USER` 或 `DROP USER`）可能对某些用户成功，对其他用户失败。现在，每个语句都是事务性的，要么对所有命名用户成功，要么回滚并且如果发生任何错误则不起作用。如果成功，该语句将写入二进制日志，但如果失败则不会；在这种情况下，回滚发生且不会进行任何更改。更多信息，请参见 第 15.1.1 节，“原子数据定义语句支持”。

    +   新的 `caching_sha2_password` 认证插件现已可用。与 `sha256_password` 插件类似，`caching_sha2_password` 实现了 SHA-256 密码哈希，但使用缓存来解决连接时的延迟问题。它还支持更多传输协议，并且不需要针对 RSA 密钥对密码交换功能链接到 OpenSSL。请参见 第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

        `caching_sha2_password`和`sha256_password`认证插件提供比`mysql_native_password`插件更安全的密码加密，而`caching_sha2_password`比`sha256_password`提供更好的性能。由于`caching_sha2_password`具有更优越的安全性和性能特性，它现在是首选的认证插件，并且默认认证插件不再是`mysql_native_password`。有关此默认插件更改对服务器操作和服务器与客户端和连接器兼容性的影响的信息，请参见将 caching_sha2_password 作为首选认证插件。

    +   MySQL 企业版 SASL LDAP 认证插件现在支持 GSSAPI/Kerberos 作为 Linux 上 MySQL 客户端和服务器的认证方法。这在 Linux 环境中非常有用，其中应用程序使用具有默认启用 Kerberos 的 Microsoft Active Directory 访问 LDAP。请参见 LDAP 认证方法。

    +   MySQL 企业版现在支持一种认证方法，允许用户使用 Kerberos 对 MySQL 服务器进行身份验证，前提是适当的 Kerberos 票证可用或可以获取。有关详细信息，请参见 Section 8.4.1.8, “Kerberos 可插拔认证”。

    +   MySQL 现在支持角色，即命名的权限集合。角色可以创建和删除。可以向角色授予和撤销权限。可以向用户账户授予和撤销角色。账户的活动适用角色可以从授予该账户的角色中选择，并且可以在该账户的会话期间更改。有关更多信息，请参见 Section 8.2.10, “使用角色”。

    +   MySQL 现在引入了用户账户类别的概念，根据是否具有`SYSTEM_USER`权限来区分系统用户和普通用户。参见 Section 8.2.11, “账户类别”。

    +   以前，不可能授予全局适用的权限，除了某些模式。如果启用了`partial_revokes`系统变量，则现在可以实现。请参见 Section 8.2.12, “使用部分撤销进行权限限制”。

    +   `GRANT`语句有一个`AS *`user`* [WITH ROLE]`子句，用于指定关于用于语句执行的权限上下文的附加信息。尽管这种语法在 SQL 级别可见，但其主要目的是通过在二进制日志中显示这些由部分撤销施加的授权限制，以实现对所有授予权限的节点的统一复制。参见第 15.7.1.6 节，“GRANT 语句”。

    +   MySQL 现在维护有关密码历史的信息，从而可以限制重复使用先前密码。数据库管理员可以要求新密码在一定数量的密码更改或一段时间内不从先前密码中选择。可以在全局范围以及按账户基础上建立密码重用策略。

        现在可以要求更改账户密码的尝试通过指定要替换的当前密码进行验证。这使得数据库管理员可以防止用户在未证明知道当前密码的情况下更改密码。可以在全局范围以及按账户基础上建立密码验证策略。

        现在允许账户拥有双密码，这使得在复杂的多服务器系统中无缝执行分阶段密码更改成为可能，而无需停机。

        MySQL 现在允许管理员配置用户账户，以便由于密码错误导致的连续登录失败次数过多时，会导致临时锁定账户。每个账户的所需失败次数和锁定时间都是可配置的。

        这些新功能为数据库管理员提供了更完整的密码管理控制。更多信息，请参见第 8.2.15 节，“密码管理”。

    +   MySQL 现在支持 FIPS 模式，如果使用 OpenSSL 编译，并且在运行时可用的 OpenSSL 库和 FIPS 对象模块。FIPS 模式对加密操作施加条件，例如对可接受的加密算法的限制或对更长密钥长度的要求。参见第 8.8 节，“FIPS 支持”。

    +   服务器现在可以在运行时重新配置用于新连接的 TLS 上下文。例如，这种能力可能很有用，可以避免重新启动运行时间过长以至于 SSL 证书已过期的 MySQL 服务器。参见服务器端运行时配置和加密连接监控。

    +   OpenSSL 1.1.1 支持用于加密连接的 TLS v1.3 协议，MySQL 8.0.16 及更高版本也支持 TLS v1.3，如果服务器和客户端都使用 OpenSSL 1.1.1 或更高版本进行编译。请参见第 8.3.2 节，“加密连接 TLS 协议和密码”。

    +   MySQL 现在将授予客户端在 Windows 上使用命名管道进行成功通信所需的最低访问控制权限。新版 MySQL 客户端软件可以在不需要任何额外配置的情况下打开命名管道连接。如果旧版客户端软件无法立即升级，可以使用新的`named_pipe_full_access_group`系统变量来授予 Windows 组打开命名管道连接所需的权限。完全访问组的成员资格应受限制且临时。

    +   以前，MySQL 用户帐户使用单一认证方法对服务器进行身份验证。从 MySQL 8.0.27 开始，MySQL 支持多因素认证（MFA），这使得可以创建具有最多三种认证方法的帐户成为可能。MFA 支持包括以下更改：

        +   `CREATE USER`和`ALTER USER`语法已扩展，允许指定多个认证方法。

        +   `authentication_policy` 系统变量通过控制可以使用多少因素以及每个因素允许的认证类型，来建立 MFA 策略。这会对`CREATE USER`和`ALTER USER`语句中与认证相关的子句的使用方式施加限制。

        +   客户端程序现在具有新的`--password1`、`--password2`和`--password3`命令行选项，用于指定多个密码。对于使用 C API 的应用程序，`mysql_options4()` C API 函数的新选项 `MYSQL_OPT_USER_PASSWORD` 可以实现相同的功能。

        此外，MySQL 企业版现在支持使用智能卡、安全密钥和生物识别读卡器等设备对 MySQL 服务器进行身份验证。这种身份验证方法基于快速身份在线（FIDO）标准，并使用一对插件，服务器端的`authentication_fido`和客户端的`authentication_fido_client`。服务器端的 FIDO 身份验证插件仅包含在 MySQL 企业版发行版中。它不包含在 MySQL 社区发行版中。然而，客户端插件包含在所有发行版中，包括社区发行版。这使得来自任何发行版的客户端都可以连接到加载了服务器端插件的服务器。

        多因素身份验证可以使用现有的 MySQL 身份验证方法、新的 FIDO 身份验证方法或两者的组合。有关更多信息，请参见第 8.2.18 节，“多因素身份验证”，以及第 8.4.1.11 节，“FIDO 可插拔身份验证”。

+   **资源管理。** MySQL 现在支持创建和管理资源组，并允许将在服务器内运行的线程分配给特定组，以便线程根据组可用的资源执行。组属性可以控制其资源，以启用或限制组内线程的资源消耗。数据库管理员可以根据不同的工作负载适当修改这些属性。目前，CPU 时间是一种可管理的资源，表示为“虚拟 CPU”的概念，包括 CPU 核心、超线程、硬件线程等。服务器在启动时确定有多少虚拟 CPU 可用，具有适当权限的数据库管理员可以将这些 CPU 与资源组关联并将线程分配给组。有关更多信息，请参见第 7.1.16 节，“资源组”。

+   **表加密管理。** 现在可以通过定义和强制执行加密默认值来全局管理表加密。`default_table_encryption`变量为新创建的模式和通用表空间定义了一个加密默认值。创建模式时也可以使用`DEFAULT ENCRYPTION`子句定义模式的加密默认值。默认情况下，表继承所在模式或通用表空间的加密。通过启用`table_encryption_privilege_check`变量来强制执行加密默认值。当创建或更改具有与`default_table_encryption`设置不同的加密设置的模式或通用表空间，或者创建或更改具有与默认模式加密不同的加密设置的表时，将进行权限检查。当启用`table_encryption_privilege_check`时，`TABLE_ENCRYPTION_ADMIN`权限允许覆盖默认加密设置。有关更多信息，请参阅为模式和通用表空间定义加密默认值。

+   **InnoDB 增强。** 添加了以下`InnoDB`增强功能：

    +   每次值更改时，当前最大自动增量计数器值都会写入重做日志，并在每次检查点时保存到引擎私有系统表中。这些更改使得当前最大自动增量计数器值在服务器重新启动时持久化。此外：

        +   服务器重新启动不再取消`AUTO_INCREMENT = N`表选项的效果。如果您将自动增量计数器初始化为特定值，或者将自动增量计数器值更改为更大的值，则新值将在服务器重新启动时持久化。

        +   在`ROLLBACK`操作后立即重新启动服务器不再导致自动增量值被回滚事务分配的重复使用。

        +   如果您将`AUTO_INCREMENT`列值修改为大于当前最大自动增量值（例如，在`UPDATE`操作中），新值将被持久化，并且随后的`INSERT`操作将从新的更大值开始分配自动增量值。

        欲了解更多信息，请参阅 第 17.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理” 和 InnoDB AUTO_INCREMENT 计数器初始化。

    +   当遇到索引树损坏时，`InnoDB` 会向重做日志写入损坏标志，使损坏标志具有崩溃安全性。`InnoDB` 还会将内存中的损坏标志数据写入每个检查点上的引擎私有系统表。在恢复过程中，`InnoDB` 从两个位置读取损坏标志并在标记内存表和索引对象为损坏之前合并结果。

    +   `InnoDB` **memcached** 插件支持多个 `get` 操作（在单个 **memcached** 查询中获取多个键值对）和范围查询。请参阅 第 17.20.4 节，“InnoDB memcached 多个 get 和范围查询支持”。

    +   一个新的动态变量，`innodb_deadlock_detect`，可用于禁用死锁检测。在高并发系统中，死锁检测可能会导致大量线程等待同一锁时减速。有时，禁用死锁检测并依赖于 `innodb_lock_wait_timeout` 设置在发生死锁时进行事务回滚可能更有效率。

    +   新的信息模式 `INNODB_CACHED_INDEXES` 表报告了每个索引在 `InnoDB` 缓冲池中缓存的索引页数。

    +   `InnoDB` 临时表现在在共享临时表空间 `ibtmp1` 中创建。

    +   `InnoDB` 表空间加密功能 支持重做日志和撤销日志数据的加密。请参阅 重做日志加密 和 撤销日志加密。

    +   `InnoDB` 支持 `SELECT ... FOR SHARE` 和 `SELECT ... FOR UPDATE` 锁定读语句的 `NOWAIT` 和 `SKIP LOCKED` 选项。`NOWAIT` 会导致语句立即返回，如果请求的行被另一个事务锁定。`SKIP LOCKED` 会从结果集中移除被锁定的行。请参阅 使用 NOWAIT 和 SKIP LOCKED 进行锁定读并发。

        `SELECT ... FOR SHARE`替代了`SELECT ... LOCK IN SHARE MODE`，但`LOCK IN SHARE MODE`仍然可用以保持向后兼容性。这两个语句是等效的。然而，`FOR UPDATE`和`FOR SHARE`支持`NOWAIT`、`SKIP LOCKED`和`OF *`tbl_name`*`选项。参见第 15.2.13 节，“SELECT 语句”。

        `OF *`tbl_name`*`将锁定查询应用于命名表。

    +   `ADD PARTITION`、`DROP PARTITION`、`COALESCE PARTITION`、`REORGANIZE PARTITION`和`REBUILD PARTITION` `ALTER TABLE`选项由本机分区内部 API 支持，并且可以与`ALGORITHM={COPY|INPLACE}`和`LOCK`子句一起使用。

        使用`ALGORITHM=INPLACE`的`DROP PARTITION`会删除存储在分区中的数据并且删除该分区。然而，使用`ALGORITHM=COPY`或`old_alter_table=ON`的`DROP PARTITION`会重建分区表，并尝试将被删除分区的数据移动到具有兼容`PARTITION ... VALUES`定义的另一个分区中。无法移动到另一个分区的数据将被删除。

    +   `InnoDB`存储引擎现在使用 MySQL 数据字典而不是其自己的存储引擎特定数据字典。有关数据字典的信息，请参见第十六章，“MySQL 数据字典”。

    +   `mysql`系统表和数据字典表现在创建在 MySQL 数据目录中名为`mysql.ibd`的单个`InnoDB`表空间文件中。以前，这些表是在`mysql`数据库目录中的单独的`InnoDB`表空间文件中创建的。

    +   MySQL 8.0 引入了以下撤销表空间更改：

        +   默认情况下，撤销日志现在驻留在初始化 MySQL 实例时创建的两个撤销表空间中。撤销日志不再在系统表空间中创建。

        +   从 MySQL 8.0.14 开始，可以使用`CREATE UNDO TABLESPACE`语法在选择的位置在运行时创建额外的撤销表空间。

            ```sql
            CREATE UNDO TABLESPACE *tablespace_name* ADD DATAFILE '*file_name*.ibu';
            ```

            使用`CREATE UNDO TABLESPACE`语法创建的撤销表空间可以使用`DROP UNDO TABLESPACE`语法在运行时删除。

            ```sql
            DROP UNDO TABLESPACE *tablespace_name*;
            ```

            可以使用`ALTER UNDO TABLESPACE`语法将撤销表空间标记为活动或非活动。

            ```sql
            ALTER UNDO TABLESPACE *tablespace_name* SET {ACTIVE|INACTIVE};
            ```

            在 Information Schema `INNODB_TABLESPACES`表中添加了显示表空间状态的`STATE`列。在删除之前，撤销表空间必须处于`empty`状态。

        +   `innodb_undo_log_truncate`变量默认启用。

        +   `innodb_rollback_segments` 变量定义了每个撤销表空间的回滚段数。以前，`innodb_rollback_segments` 指定了 MySQL 实例的总回滚段数。此更改增加了可用于并发事务的回滚段数。更多的回滚段增加了并发事务使用不同回滚段进行撤销日志的可能性，从而减少资源争用。

    +   影响缓冲池预刷和刷新行为的变量的默认值已经修改：

        +   `innodb_max_dirty_pages_pct_lwm` 的默认值现在是 10。以前的默认值为 0，禁用了缓冲池预刷。当缓冲池中脏页的百分比超过 10% 时，值为 10 会启用预刷。启用预刷可以提高性能的一致性。

        +   `innodb_max_dirty_pages_pct` 的默认值从 75 增加到 90。`InnoDB` 试图从缓冲池中刷新数据，以使脏页的百分比不超过此值。增加的默认值允许缓冲池中有更大比例的脏页。

    +   默认的 `innodb_autoinc_lock_mode` 设置现在是 2（交错）。交错锁模式允许并行执行多行插入，提高了并发性和可伸缩性。新的 `innodb_autoinc_lock_mode` 默认设置反映了从基于语句的复制更改为基于行的复制作为 MySQL 5.7 中默认复制类型的变化。基于语句的复制需要连续的自增锁模式（以前的默认值）来确保自增值按照可预测和可重复的顺序分配给给定序列的 SQL 语句，而基于行的复制不受 SQL 语句执行顺序的影响。更多信息，请参见 InnoDB AUTO_INCREMENT Lock Modes。

        对于使用基于语句的复制的系统，新的 `innodb_autoinc_lock_mode` 默认设置可能会破坏依赖顺序自增值的应用程序。要恢复以前的默认值，请将 `innodb_autoinc_lock_mode` 设置为 1。

    +   支持通过 `ALTER TABLESPACE ... RENAME TO` 语法重命名通用表空间。

    +   新的`innodb_dedicated_server`变量，默认情况下已禁用，可用于根据服务器检测到的内存量自动配置以下选项：

        +   `innodb_buffer_pool_size`

        +   `innodb_log_file_size`

        +   `innodb_flush_method`

        此选项适用于在专用服务器上运行的 MySQL 服务器实例。有关更多信息，请参见第 17.8.12 节，“为专用 MySQL 服务器启用自动配置”。

    +   新的信息模式`INNODB_TABLESPACES_BRIEF`视图为`InnoDB`表空间提供空间、名称、路径、标志和空间类型数据。

    +   MySQL 捆绑的[zlib 库](http://www.zlib.net/)版本从 1.2.3 提升到 1.2.11。MySQL 借助 zlib 库实现压缩。

        如果使用`InnoDB`压缩表，请参阅第 3.5 节，“MySQL 8.0 中的更改”以获取相关升级影响。

    +   序列化字典信息（SDI）存在于除全局临时表空间和撤销表空间文件之外的所有`InnoDB`表空间文件中。SDI 是表和表空间对象的序列化元数据。SDI 数据的存在提供了元数据冗余。例如，如果数据字典不可用，则可以从表空间文件中提取字典对象元数据。SDI 提取使用**ibd2sdi**工具进行。SDI 数据以`JSON`格式存储。

        在表空间文件中包含 SDI 数据会增加表空间文件大小。一个 SDI 记录需要一个索引页，默认大小为 16KB。但是，SDI 数据在存储时会进行压缩以减少存储占用。

    +   `InnoDB`存储引擎现在支持原子 DDL，确保 DDL 操作要么完全提交，要么回滚，即使服务器在操作期间停止。有关更多信息，请参见第 15.1.1 节，“原子数据定义语句支持”。

    +   可以使用`innodb_directories`选项在服务器离线时将表空间文件移动或恢复到新位置。有关更多信息，请参见第 17.6.3.6 节，“在服务器离线时移动表空间文件”。

    +   实施了以下重做日志优化：

        +   用户线程现在可以并发写入日志缓冲区而无需同步写入。

        +   用户线程现在可以以松弛顺序将脏页添加到刷新列表中。

        +   现在有一个专用的日志线程负责将日志缓冲区写入系统缓冲区，将系统缓冲区刷新到磁盘，通知用户线程已写入和刷新的重做，维护松弛刷新列表顺序所需的延迟，并写入检查点。

        +   系统变量已添加以配置用户线程在等待刷新重做时使用自旋延迟：

            +   `innodb_log_wait_for_flush_spin_hwm`: 定义了平均日志刷新时间的最大值，超过这个值时，用户线程不再在等待刷新重做时自旋。

            +   `innodb_log_spin_cpu_abs_lwm`: 定义了 CPU 使用率的最小值，低于此值时，用户线程在等待刷新重做时不再自旋。

            +   `innodb_log_spin_cpu_pct_hwm`: 定义了 CPU 使用率的最大值，高于此值时，用户线程在等待刷新重做时不再自旋。

        +   `innodb_log_buffer_size`变量现在是动态的，允许在服务器运行时调整日志缓冲区的大小。

        更多信息，请参见第 10.5.4 节，“优化 InnoDB 重做日志记录”。

    +   截至 MySQL 8.0.12，对大对象（LOB）数据进行小更新支持撤销日志记录，这提高了大小为 100 字节或更小的 LOB 更新的性能。以前，LOB 更新的最小大小为一个 LOB 页，这对于可能只修改几个字节的更新来说不太理想。此增强功能建立在 MySQL 8.0.4 中对 LOB 数据的部分更新支持之上。

    +   截至 MySQL 8.0.12，`ALGORITHM=INSTANT`支持以下`ALTER TABLE`操作：

        +   添加列。此功能也被称为“即时`ADD COLUMN`”。有一些限制。请参见第 17.12.1 节，“在线 DDL 操作”。

        +   添加或删除虚拟列。

        +   添加或删除列的默认值。

        +   修改`ENUM`或`SET`列的定义。

        +   更改索引类型。

        +   重命名表。

        仅支持`ALGORITHM=INSTANT`的操作仅修改数据字典中的元数据。表上不会获取元数据锁，表数据也不受影响，使操作瞬间完成。如果未明确指定，支持`ALGORITHM=INSTANT`的操作将默认使用它。如果指定了`ALGORITHM=INSTANT`但不支持，操作将立即失败并显示错误。

        有关支持`ALGORITHM=INSTANT`的操作的更多信息，请参见第 17.12.1 节，“在线 DDL 操作”。

    +   截至 MySQL 8.0.13，`TempTable`存储引擎支持存储二进制大对象（BLOB）类型列。这个增强功能提高了查询性能，用于包含 BLOB 数据的临时表。以前，包含 BLOB 数据的临时表存储在由`internal_tmp_disk_storage_engine`定义的磁盘存储引擎中。更多信息，请参见第 10.4.4 节，“MySQL 中的内部临时表使用”。

    +   截至 MySQL 8.0.13，`InnoDB`数据静止加密功能支持通用表空间。以前，只有文件表表空间可以加密。为了支持通用表空间的加密，`CREATE TABLESPACE`和`ALTER TABLESPACE`语法被扩展以包括一个`ENCRYPTION`子句。

        信息模式`INNODB_TABLESPACES`表现在包括一个`ENCRYPTION`列，指示表空间是否加密。

        添加了`stage/innodb/alter tablespace (encryption)`性能模式阶段工具，以允许监视通用表空间加密操作。

    +   禁用`innodb_buffer_pool_in_core_file`变量通过排除`InnoDB`缓冲池页面来减小核心文件的大小。要使用这个变量，必须启用`core_file`变量，并且操作系统必须支持`MADV_DONTDUMP`非 POSIX 扩展到`madvise()`，这在 Linux 3.4 及更高版本中支持。更多信息，请参见第 17.8.3.7 节，“从核心文件中排除缓冲池页面”。

    +   截至 MySQL 8.0.13，用户创建的临时表和优化器创建的内部临时表存储在为会话分配的会话临时表空间中，从临时表空间池中分配。当会话断开连接时，其临时表空间被截断并释放回池中。在以前的版本中，临时表被创建在全局临时表空间（`ibtmp1`）中，在临时表被删除后不会将磁盘空间返回给操作系统。

        `innodb_temp_tablespaces_dir`变量定义了会话临时表空间创建的位置。默认位置是数据目录中的`#innodb_temp`目录。

        `INNODB_SESSION_TEMP_TABLESPACES`表提供有关会话临时表空间的元数据。

        全局临时表空间（`ibtmp1`）现在存储对用户创建的临时表所做更改的回滚段。

    +   从 MySQL 8.0.14 开始，`InnoDB`支持并行聚簇索引读取，可以提高`CHECK TABLE`性能。此功能不适用于次要索引扫描。`innodb_parallel_read_threads`会话变量必须设置为大于 1 的值，才能进行并行聚簇索引读取。默认值为 4。用于执行并行聚簇索引读取的实际线程数由`innodb_parallel_read_threads`设置或要扫描的索引子树数量决定，取两者中较小的值。

    +   从 8.0.14 开始，当启用`innodb_dedicated_server`变量时，日志文件的大小和数量将根据自动配置的缓冲池大小进行配置。以前，日志文件大小是根据服务器上检测到的内存量进行配置的，日志文件的数量不是自动配置的。

    +   从 8.0.14 开始，`CREATE TABLESPACE`语句的`ADD DATAFILE`子句是可选的，这允许没有`FILE`权限的用户创建表空间。执行不带`ADD DATAFILE`子句的`CREATE TABLESPACE`语句会隐式创建一个具有唯一文件名的表空间数据文件。

    +   默认情况下，当 TempTable 存储引擎占用的内存量超过`temptable_max_ram`变量定义的内存限制时，TempTable 存储引擎开始从磁盘分配内存映射临时文件。从 MySQL 8.0.16 开始，此行为由`temptable_use_mmap`变量控制。禁用`temptable_use_mmap`会导致 TempTable 存储引擎使用`InnoDB`磁盘上的内部临时表，而不是内存映射文件作为其溢出机制。有关更多信息，请参见内部临时表存储引擎。

    +   自 MySQL 8.0.16 起，`InnoDB`数据静态加密功能支持对`mysql`系统表空间进行加密。`mysql`系统表空间包含`mysql`系统数据库和 MySQL 数据字典表。更多信息，请参见第 17.13 节，“InnoDB 数据静态加密”。

    +   `innodb_spin_wait_pause_multiplier`变量，自 MySQL 8.0.16 起引入，提供了对当线程等待获取互斥锁或读写锁时发生的自旋锁轮询延迟持续时间的更大控制。延迟可以更精细地调整，以考虑不同处理器架构上 PAUSE 指令持续时间的差异。更多信息，请参见第 17.8.8 节，“配置自旋锁轮询”。

    +   `InnoDB`大数据集并行读取线程性能在 MySQL 8.0.17 中得到改善，通过更好地利用读取线程，减少在并行扫描期间发生的预取活动的读取线程 I/O，并支持对分区进行并行扫描。

        并行读取线程功能由`innodb_parallel_read_threads`变量控制。最大设置现在为 256，这是所有客户端连接的线程总数。如果线程限制达到，连接将回退到使用单个线程。

    +   `innodb_idle_flush_pct`变量，自 MySQL 8.0.18 起引入，允许在空闲时期限制页面刷新，有助于延长固态存储设备的寿命。请参见限制空闲时期的缓冲刷新。

    +   从 MySQL 8.0.19 起，支持对`InnoDB`数据进行高效采样，以生成直方图统计信息。请参见直方图统计分析。

    +   自 MySQL 8.0.20 起，双写缓冲存储区位于双写文件中。在之前的版本中，存储区位于系统表空间中。将存储区移出系统表空间可以减少写入延迟，增加吞吐量，并提供关于双写缓冲页放置的灵活性。以下系统变量用于高级双写缓冲配置：

        +   `innodb_doublewrite_dir`

            定义双写缓冲文件目录。

        +   `innodb_doublewrite_files`

            定义双写文件的数量。

        +   `innodb_doublewrite_pages`

            定义了每个批次写入的最大双写页数。

        +   `innodb_doublewrite_batch_size`

            定义一批中要写入的双写页数。

        更多信息，请参见第 17.6.4 节，“双写缓冲区”。

    +   优先考虑等待锁的事务的争用感知事务调度（CATS）算法在 MySQL 8.0.20 中得到改进。现在，事务调度权重计算完全在一个单独的线程中执行，这提高了计算性能和准确性。

        先进先出（FIFO）算法，之前也用于事务调度，已被移除。FIFO 算法被 CATS 算法增强所取代。之前由 FIFO 算法执行的事务调度现在由 CATS 算法执行。

        在`INFORMATION_SCHEMA.INNODB_TRX`表中添加了一个`TRX_SCHEDULE_WEIGHT`列，允许查询 CATS 算法分配的事务调度权重。

        用于监视代码级事务调度事件的以下`INNODB_METRICS`计数器被添加：

        +   `lock_rec_release_attempts`

            释放记录锁的尝试次数。

        +   `lock_rec_grant_attempts`

            授予记录锁的尝试次数。

        +   `lock_schedule_refreshes`

            分析等待图以更新事务调度权重的次数。

        更多信息，请参见第 17.7.6 节，“事务调度”。

    +   截至 MySQL 8.0.21，为了改善需要访问表和行资源的锁队列的操作的并发性，锁系统互斥体（`lock_sys->mutex`）被分片锁替代，并且锁队列被分组为表和页面*锁队列分片*，每个分片由专用互斥体保护。以前，单个锁系统互斥体保护所有锁队列，在高并发系统上是一个争用点。新的分片实现允许更精细地访问锁队列。

        锁系统互斥体（`lock_sys->mutex`）被以下分片锁替代：

        +   全局分片锁（`lock_sys->latches.global_latch`）由 64 个读写锁对象（`rw_lock_t`）组成。访问单个锁队列需要一个共享全局锁和一个锁队列分片的锁。需要访问所有锁队列的操作需要一个独占的全局锁，该锁锁定所有表和页面锁队列分片。

        +   表分片锁（`lock_sys->latches.table_shards.mutexes`），实现为一个包含 512 个互斥体的数组，每个互斥体专用于 512 个表锁队列分片中的一个。

        +   页面分片锁（`lock_sys->latches.page_shards.mutexes`），实现为一个包含 512 个互斥体的数组，每个互斥体专用于 512 个页面锁队列分片中的一个。

        用于监控单个锁系统互斥体的性能模式`wait/synch/mutex/innodb/lock_mutex`仪器已被替换为用于监控新的全局、表分片和页面分片闩锁的仪器：

        +   `wait/synch/sxlock/innodb/lock_sys_global_rw_lock`

        +   `wait/synch/mutex/innodb/lock_sys_table_mutex`

        +   `wait/synch/mutex/innodb/lock_sys_page_mutex`

    +   从 MySQL 8.0.21 开始，使用`DATA DIRECTORY`子句在数据目录之外创建的表和表分区数据文件受限于`InnoDB`已知的目录。此更改允许数据库管理员控制表空间数据文件的创建位置，并确保在恢复期间可以找到数据文件。

        通用和每个表的文件表空间数据文件（`.ibd`文件）不再可以在撤销表空间目录（`innodb_undo_directory`）中创建，除非`InnoDB`已知该目录。

        已知目录是由`datadir`、`innodb_data_home_dir`和`innodb_directories`变量定义的目录。

        截断位于文件表空间中的`InnoDB`表会删除现有的表空间并创建一个新的表空间。从 MySQL 8.0.21 开始，如果当前表空间目录未知，`InnoDB`会在默认位置创建新的表空间，并在错误日志中写入警告。要使`TRUNCATE TABLE`在当前位置创建表空间，请在运行`TRUNCATE TABLE`之前将目录添加到`innodb_directories`设置中。

    +   从 MySQL 8.0.21 开始，可以使用`ALTER INSTANCE {ENABLE|DISABLE} INNODB REDO_LOG`语法启用和禁用重做日志记录。此功能旨在将数据加载到新的 MySQL 实例中。禁用重做日志记录有助于通过避免重做日志写入来加快数据加载速度。

        新的`INNODB_REDO_LOG_ENABLE`权限允许启用和禁用重做日志记录。

        新的`Innodb_redo_log_enabled`状态变量允许监控重做日志记录状态。

        参见禁用重做日志记录。

    +   在启动时，`InnoDB`会校验已知表空间文件的路径与数据字典中存储的表空间文件路径是否匹配，以防表空间文件已移动到不同位置。新引入的`innodb_validate_tablespace_paths`变量允许禁用表空间路径验证。此功能适用于表空间文件未移动的环境。禁用表空间路径验证可以提高在具有大量表空间文件的系统上的启动时间。

        更多信息，请参见第 17.6.3.7 节，“禁用表空间路径验证”。

    +   截至 MySQL 8.0.21 版本，在支持原子 DDL 的存储引擎上，当使用基于行的复制时，`CREATE TABLE ... SELECT`语句将作为一个事务记录在二进制日志中。以前，它被记录为两个事务，一个用于创建表，另一个用于插入数据。通过这个改变，`CREATE TABLE ... SELECT`语句现在对于基于行的复制是安全的，并且允许与基于 GTID 的复制一起使用。更多信息，请参见第 15.1.1 节，“原子数据定义语句支持”。

    +   在繁忙系统上截断撤销表空间可能会影响性能，因为相关的刷新操作会从缓冲池中移除旧的撤销表空间页面，并将新撤销表空间的初始页面刷新到磁盘。为了解决这个问题，从 MySQL 8.0.21 开始移除了刷新操作。

        旧的撤销表空间页面在变得最近未使用时被被动释放，或者在下一个完整检查点时被移除。新撤销表空间的初始页面现在被重做日志记录，而不是在截断操作期间刷新到磁盘，这也提高了撤销表空间截断操作的耐久性。

        为了防止由过多的撤销表空间截断操作引起的潜在问题，现在在检查点之间对同一撤销表空间的截断操作被限制为 64 次。如果超过限制，撤销表空间仍然可以被设置为非活动状态，但直到下一个检查点之后才会被截断。

        与废弃的撤销截断刷新操作相关的`INNODB_METRICS`计数器已被移除。移除的计数器包括：`undo_truncate_sweep_count`、`undo_truncate_sweep_usec`、`undo_truncate_flush_count`和`undo_truncate_flush_usec`。

        请参见第 17.6.3.4 节，“撤销表空间”。

    +   截至 MySQL 8.0.22 版本，新的`innodb_extend_and_initialize`变量允许配置`InnoDB`在 Linux 上为文件表和通用表空间分配空间的方式。默认情况下，当操作需要在表空间中分配额外空间时，`InnoDB`会为表空间分配页面并在这些页面上物理写入 NULL。如果频繁分配新页面，这种行为会影响性能。您可以在 Linux 系统上禁用`innodb_extend_and_initialize`以避免在新分配的表空间页面上物理写入 NULL。当禁用`innodb_extend_and_initialize`时，空间使用`posix_fallocate()`调用进行分配，该调用保留空间而不进行物理写入 NULL。

        `posix_fallocate()`操作不是原子性的，这使得在为表空间文件分配空间和更新文件元数据之间发生故障的可能性。这种故障可能会导致新分配的页面处于未初始化状态，当`InnoDB`尝试访问这些页面时会失败。为了防止这种情况发生，`InnoDB`在分配新表空间页面之前写入重做日志记录。如果页面分配操作被中断，操作将在恢复期间从重做日志记录中重放。

    +   截至 MySQL 8.0.23 版本，`InnoDB`支持对属于加密表空间的双写文件页进行加密。这些页面使用相关表空间的加密密钥进行加密。更多信息，请参见第 17.13 节，“InnoDB 数据静态加密”。

    +   MySQL 8.0.23 版本引入的`temptable_max_mmap`变量定义了 TempTable 存储引擎允许从内存映射（MMAP）文件分配的最大内存量，然后才开始将内部临时表数据存储在磁盘上。设置为 0 会禁用从 MMAP 文件分配。更多信息，请参见第 10.4.4 节，“MySQL 中的内部临时表使用”。

    +   `AUTOEXTEND_SIZE` 选项在 MySQL 8.0.23 中引入，定义了当 `InnoDB` 表空间变满时 `InnoDB` 扩展表空间的量，从而可以以更大的增量扩展表空间大小。`AUTOEXTEND_SIZE` 选项支持 `CREATE TABLE`、`ALTER TABLE`、`CREATE TABLESPACE` 和 `ALTER TABLESPACE` 语句。有关更多信息，请参见 17.6.3.9 节，“表空间 AUTOEXTEND_SIZE 配置”。

        在 Information Schema `INNODB_TABLESPACES` 表中添加了一个 `AUTOEXTEND_SIZE` 大小列。

    +   `innodb_segment_reserve_factor` 系统变量，在 MySQL 8.0.26 中引入，允许配置作为空白页保留的表空间文件段页面的百分比。有关更多信息，请参见 配置保留文件段页面的百分比。

    +   在支持 `fdatasync()` 系统调用的平台上，`innodb_use_fdatasync` 变量，在 MySQL 8.0.26 中引入，允许使用 `fdatasync()` 而不是 `fsync()` 进行操作系统刷新。`fdatasync()` 系统调用不会刷新文件元数据，除非需要进行后续数据检索，从而提供潜在的性能优势。

    +   截至 MySQL 8.0.28 版本，`tmp_table_size` 变量定义了由 TempTable 存储引擎创建的任何单个内存内部临时表的最大大小。适当的大小限制可以防止单个查询消耗过多的全局 TempTable 资源。参见 内部临时表存储引擎。

    +   从 MySQL 8.0.28 开始，`innodb_open_files` 变量，定义了 `InnoDB` 可以同时打开的文件数，可以通过使用 `SELECT innodb_set_open_files_limit(*`N`*)` 语句在运行时进行设置。该语句执行一个存储过程来设置新的限制。

        为防止非 LRU 管理的文件占用整个`innodb_open_files`限制，非 LRU 管理的文件限制为`innodb_open_files`限制的 90％，这为 LRU 管理的文件保留了`innodb_open_files`限制的 10％。

        `innodb_open_files`限制包括临时表空间文件，在此之前不计入限制。

    +   从 MySQL 8.0.28 开始，`InnoDB`支持使用`ALGORITHM=INSTANT`进行`ALTER TABLE ... RENAME COLUMN`操作。

        有关支持`ALGORITHM=INSTANT`的此类 DDL 操作和其他 DDL 操作的更多信息，请参见第 17.12.1 节，“在线 DDL 操作”。

    +   从 MySQL 8.0.29 开始，`InnoDB`支持使用`ALGORITHM=INSTANT`进行`ALTER TABLE ... DROP COLUMN`操作。

        在 MySQL 8.0.29 之前，立即添加的列只能添加为表的最后一列。从 MySQL 8.0.29 开始，立即添加的列可以添加到表中的任何位置。

        立即添加或删除的列会创建受影响行的新版本。最多允许 64 个行版本。Information Schema `INNODB_TABLES`表中添加了一个新的`TOTAL_ROW_VERSIONS`列，用于跟踪行版本的数量。

        有关支持`ALGORITHM=INSTANT`的 DDL 操作的更多信息，请参见第 17.12.1 节，“在线 DDL 操作”。

    +   从 MySQL 8.0.30 开始，`innodb_doublewrite`系统变量支持`DETECT_ONLY`和`DETECT_AND_RECOVER`设置。使用`DETECT_ONLY`设置时，数据库页内容不会写入双写缓冲区，并且恢复不使用双写缓冲区来修复不完整的页写入。此轻量级设置仅用于检测不完整的页写入。`DETECT_AND_RECOVER`设置等同于现有的`ON`设置。有关更多信息，请参见第 17.6.4 节，“双写缓冲区”。

    +   从 MySQL 8.0.30 开始，`InnoDB`支持重做日志容量的动态配置。`innodb_redo_log_capacity`系统变量可以在运行时设置，以增加或减少重做日志文件占用的总磁盘空间量。

        随着这一变化，重做日志文件的数量及其默认位置也发生了变化。从 MySQL 8.0.30 开始，`InnoDB`在数据目录中的`#innodb_redo`目录中维护 32 个重做日志文件。以前，`InnoDB`默认在数据目录中创建两个重做日志文件，并且重做日志文件的数量和大小由`innodb_log_files_in_group`和`innodb_log_file_size`变量控制。这两个变量现在已被弃用。

        当定义了`innodb_redo_log_capacity`设置时，`innodb_log_files_in_group`和`innodb_log_file_size`设置将被忽略；否则，这些设置将用于计算`innodb_redo_log_capacity`设置（`innodb_log_files_in_group` * `innodb_log_file_size` = `innodb_redo_log_capacity`）。如果这些变量都没有设置，则重做日志容量将设置为`innodb_redo_log_capacity`的默认值，即 104857600 字节（100MB）。

        提供了几个状态变量用于监视重做日志和重做日志调整大小操作。

        有关更多信息，请参见第 17.6.5 节，“重做日志”。

    +   在 MySQL 8.0.31 中，为监视在线缓冲池调整大小操作新增了两个新的状态变量。`Innodb_buffer_pool_resize_status_code`状态变量报告一个状态代码，指示在线缓冲池调整大小操作的阶段。`Innodb_buffer_pool_resize_status_progress`状态变量报告一个百分比值，指示每个阶段的进度。

        有关更多信息，请参见第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”。

+   **字符集支持。** 默认字符集已从`latin1`更改为`utf8mb4`。`utf8mb4`字符集有几个新的排序规则，包括`utf8mb4_ja_0900_as_cs`，这是 MySQL 中第一个针对 Unicode 的日语特定排序规则。有关更多信息，请参见第 12.10.1 节，“Unicode 字符集”。

+   **JSON 增强。** MySQL 的 JSON 功能进行了以下增强或添加：

    +   添加了`->>`（内联路径）运算符，等效于对`JSON_EXTRACT()`的结果调用`JSON_UNQUOTE()`。

        这是 MySQL 5.7 中引入的列路径运算符 `->` 的改进；`col->>"$.path"` 等同于 `JSON_UNQUOTE(col->"$.path")`。内联路径运算符可以在任何可以使用 `JSON_UNQUOTE(JSON_EXTRACT())` 的地方使用，例如 `SELECT` 列列表、`WHERE` 和 `HAVING` 子句，以及 `ORDER BY` 和 `GROUP BY` 子句。有关更多信息，请参阅该运算符的描述，以及 JSON 路径语法。

    +   添加了两个 JSON 聚合函数 `JSON_ARRAYAGG()` 和 `JSON_OBJECTAGG()`。`JSON_ARRAYAGG()` 以列或表达式作为其参数，并将结果聚合为单个 `JSON` 数组。表达式可以评估为任何 MySQL 数据类型；这不必是一个 `JSON` 值。`JSON_OBJECTAGG()` 接受两个列或表达式，它将其解释为键和值；它将结果作为单个 `JSON` 对象返回。有关更多信息和示例，请参阅第 14.19 节，“聚合函数”。

    +   添加了 JSON 实用函数 `JSON_PRETTY()`，它以易于阅读的格式输出现有的 `JSON` 值；每个 JSON 对象成员或数组值都打印在单独的行上，子对象或数组相对于其父对象缩进 2 个空格。

        这个函数还可以处理可以解析为 JSON 值的字符串。

        欲了解更详细信息和示例，请参阅第 14.17.8 节，“JSON 实用函数”。

    +   在查询中使用 `ORDER BY` 对 `JSON` 值进行排序时，每个值现在由排序键的可变长度部分表示，而不是固定大小的 1K 部分。在许多情况下，这可以减少过度使用。例如，标量 `INT` 或甚至 `BIGINT` 值实际上只需要很少的字节，因此剩余空间（高达 90% 或更多）被填充。这种变化对性能有以下好处：

        +   现在更有效地使用排序缓冲区空间，因此文件排序不需要像固定长度排序键那样早或频繁地刷新到磁盘。这意味着更多的数据可以在内存中排序，避免不必要的磁盘访问。

        +   较短的键比较起来比较长的键更快，这在性能上有明显的改善。这对于完全在内存中执行的排序以及需要写入和从磁盘读取的排序都是适用的。

    +   在 MySQL 8.0.2 中添加了对`JSON`列值的部分、原地更新的支持，这比完全删除现有 JSON 值并在其位置写入新值更有效，以前更新任何`JSON`列时都是这样做的。要应用此优化，更新必须使用`JSON_SET()`、`JSON_REPLACE()`或`JSON_REMOVE()`。无法向正在更新的 JSON 文档添加新元素；文档内的值不能比更新前占用更多空间。有关要求的详细讨论，请参阅 JSON 值的部分更新。

        JSON 文档的部分更新可以写入二进制日志，占用的空间比记录完整的 JSON 文档少。当使用基于语句的复制时，部分更新总是被记录为这样。要使其与基于行的复制一起工作，必须首先设置`binlog_row_value_options=PARTIAL_JSON`；有关更多信息，请参阅此变量的描述。

    +   添加了 JSON 实用函数`JSON_STORAGE_SIZE()`和`JSON_STORAGE_FREE()`。`JSON_STORAGE_SIZE()`返回在进行任何部分更新之前用于 JSON 文档的二进制表示的存储空间（请参阅上一条）。`JSON_STORAGE_FREE()`显示了在使用`JSON_SET()`或`JSON_REPLACE()`部分更新表列类型为`JSON`后剩余的空间量；如果新值的二进制表示比先前值的小，则此值大于零。

        这些函数还接受 JSON 文档的有效字符串表示。对于这样的值，`JSON_STORAGE_SIZE()`返回其转换为 JSON 文档后的二进制表示所使用的空间。对于包含 JSON 文档字符串表示的变量，`JSON_STORAGE_FREE()`返回零。如果其（非空）参数无法解析为有效的 JSON 文档，则任一函数会产生错误，并且如果参数为`NULL`，则返回`NULL`。

        有关更多信息和示例，请参阅第 14.17.8 节，“JSON 实用函数”。

        `JSON_STORAGE_SIZE()`和`JSON_STORAGE_FREE()`在 MySQL 8.0.2 中实现。

    +   在 MySQL 8.0.2 中添加了对 XPath 表达式中范围（如`$[1 to 5]`）的支持。在此版本中还添加了对`last`关键字和相对寻址的支持，使得`$[last]`始终选择数组中的最后一个（编号最高）元素，而`$[last-1]`选择倒数第二个元素。`last`和使用它的表达式也可以包含在范围定义中。例如，`$[last-2 to last-1]`返回数组中倒数第二个元素。有关更多信息和示例，请参见搜索和修改 JSON 值。

    +   添加了一个旨在符合[RFC 7396](https://tools.ietf.org/html/rfc7396)的 JSON 合并函数。当在 2 个 JSON 对象上使用`JSON_MERGE_PATCH()`时，它将它们合并为一个具有以下集合并集的单个 JSON 对象作为成员：

        +   第一个对象的每个成员，在第二个对象中没有具有相同键的成员。

        +   第二个对象的每个成员，在第一个对象中没有具有相同键的成员，并且其值不是 JSON `null`文字。

        +   每个具有在两个对象中都存在的键的成员，并且第二个对象中的值不是 JSON `null`文字。

        作为这项工作的一部分，`JSON_MERGE()`函数已重命名为`JSON_MERGE_PRESERVE()`。在 MySQL 8.0 中，`JSON_MERGE()`仍然被识别为`JSON_MERGE_PRESERVE()`的别名，但现已被弃用，并可能在将来的 MySQL 版本中被移除。

        有关更多信息和示例，请参见第 14.17.4 节，“修改 JSON 值的函数”。

    +   实现了“最后重复键胜出”的重复键规范化，与[RFC 7159](https://tools.ietf.org/html/rfc7159)和大多数 JavaScript 解析器保持一致。这种行为的示例如下，只保留具有键`x`的最右边成员：

        ```sql
        mysql> SELECT JSON_OBJECT('x', '32', 'y', '[true, false]',
             >                     'x', '"abc"', 'x', '100') AS Result;
        +------------------------------------+
        | Result                             |
        +------------------------------------+
        | {"x": "100", "y": "[true, false]"} |
        +------------------------------------+
        1 row in set (0.00 sec)
        ```

        插入到 MySQL `JSON`列中的值也以这种方式规范化，如下例所示：

        ```sql
        mysql> CREATE TABLE t1 (c1 JSON);

        mysql> INSERT INTO t1 VALUES ('{"x": 17, "x": "red", "x": [3, 5, 7]}');

        mysql> SELECT c1 FROM t1;
        +------------------+
        | c1               |
        +------------------+
        | {"x": [3, 5, 7]} |
        +------------------+
        ```

        这是与 MySQL 先前版本不兼容的更改，在这种情况下使用了“第一个重复键胜出”的算法。

        有关更多信息和示例，请参见 JSON 值的规范化、合并和自动包装。

    +   在 MySQL 8.0.4 中添加了`JSON_TABLE()`函数。此函数接受 JSON 数据并将其作为具有指定列的关系表返回。

        此函数的语法为`JSON_TABLE(*`expr`*, *`path`* COLUMNS *`column_list`*) [AS] *`alias`*)`，其中*`expr`*是返回 JSON 数据的表达式，*`path`*是应用于源的 JSON 路径，*`column_list`*是列定义的列表。这里有一个示例：

        ```sql
        mysql> *SELECT **
            -> *FROM*
            ->   *JSON_TABLE(*
            ->     *'[{"a":3,"b":"0"},{"a":"3","b":"1"},{"a":2,"b":1},{"a":0},{"b":[1,2]}]',*
            ->     *"$[*]" COLUMNS(*
            ->       *rowid FOR ORDINALITY,*
            ->
            ->       *xa INT EXISTS PATH "$.a",*
            ->       *xb INT EXISTS PATH "$.b",*
            ->
            ->       *sa VARCHAR(100) PATH "$.a",*
            ->       *sb VARCHAR(100) PATH "$.b",*
            ->
            ->       *ja JSON PATH "$.a",*
            ->       *jb JSON PATH "$.b"*
            ->     *)*
            ->   *) AS  jt1;*
        +-------+------+------+------+------+------+--------+
        | rowid | xa   | xb   | sa   | sb   | ja   | jb     |
        +-------+------+------+------+------+------+--------+
        |     1 |    1 |    1 | 3    | 0    | 3    | "0"    |
        |     2 |    1 |    1 | 3    | 1    | "3"  | "1"    |
        |     3 |    1 |    1 | 2    | 1    | 2    | 1      |
        |     4 |    1 |    0 | 0    | NULL | 0    | NULL   |
        |     5 |    0 |    1 | NULL | NULL | NULL | [1, 2] |
        +-------+------+------+------+------+------+--------+
        ```

        JSON 源表达式可以是产生有效 JSON 文档的任何表达式，包括 JSON 文字、表列或返回 JSON 的函数调用，例如`JSON_EXTRACT(t1, data, '$.post.comments')`。有关更多信息，请参见第 14.17.6 节，“JSON 表函数”。

+   **数据类型支持。** MySQL 现在支持在数据类型规范中将表达式用作默认值。这包括以前无法分配默认值的`BLOB`、`TEXT`、`GEOMETRY`和`JSON`数据类型的表达式作为默认值。有关详细信息，请参见第 13.6 节，“数据类型默认值”。

+   **优化器。** 这些优化器增强功能已添加：

    +   MySQL 现在支持不可见索引。不可见索引根本不被优化器使用，但在其他方面正常维护。索引默认是可见的。不可见索引使得可以测试删除索引对查询性能的影响，而不必进行破坏性更改，如果索引被证明是必需的，则必须撤消更改。请参见第 10.3.12 节，“不可见索引”。

    +   MySQL 现在支持降序索引：在索引定义中使用`DESC`不再被忽略，而是导致按降序顺序存储键值。以前，索引可以以相反顺序扫描，但会导致性能损失。降序索引可以按正向顺序扫描，这更有效率。降序索引还使优化器能够在最有效的扫描顺序中混合一些列按升序顺序和其他列按降序顺序时使用多列索引。请参见第 10.3.13 节，“降序索引”。

    +   MySQL 现在支持创建功能性索引键部分，用于索引表达式值而不是列值。功能性键部分使得可以索引无法以其他方式索引的值，例如`JSON`值。有关详细信息，请参见第 15.1.15 节，“CREATE INDEX 语句”。

    +   在 MySQL 8.0.14 及更高版本中，由于常量文字表达式导致的琐碎`WHERE`条件在准备阶段被移除，而不是在优化过程中。在处理过程中较早地移除条件使得可以简化具有琐碎条件的外连接查询的连接，例如这样一个：

        ```sql
        SELECT * FROM t1 LEFT JOIN t2 ON *condition_1* WHERE *condition_2* OR 0 = 1
        ```

        优化器现在在准备阶段发现`0 = 1`始终为假，使得`OR 0 = 1`变得多余，并将其移除，留下如下内容：

        ```sql
        SELECT * FROM t1 LEFT JOIN t2 ON *condition_1* where *condition_2*
        ```

        现在优化器可以将查询重写为内连接，如下所示：

        ```sql
        SELECT * FROM t1 LEFT JOIN t2 WHERE *condition_1* AND *condition_2*
        ```

        更多信息，请参阅 Section 10.2.1.9, “Outer Join Optimization”。

    +   在 MySQL 8.0.16 及更高版本中，MySQL 可以在优化时使用常量折叠来处理列与常量值之间的比较，其中常量超出范围或在列类型的范围边界上，而不是在执行时为每行执行此操作。例如，给定一个具有`TINYINT UNSIGNED`列`c`的表`t`，优化器可以将诸如`WHERE c < 256`的条件重写为`WHERE 1`（并完全优化掉条件），或将`WHERE c >= 255`重写为`WHERE c = 255`。

        更多信息，请参阅 Section 10.2.1.14, “Constant-Folding Optimization”。

    +   从 MySQL 8.0.16 开始，与`IN`子查询一起使用的半连接优化现在也可以应用于`EXISTS`子查询。此外，优化器现在将附加到子查询的`WHERE`条件中的琐碎相关等式谓词解耦，以便它们可以类似于`IN`子查询中的表达式处理；这适用于`EXISTS`和`IN`子查询。

        更多信息，请参阅 Section 10.2.2.1, “Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations”。

    +   截至 MySQL 8.0.17，服务器在上下文化阶段内部将任何不完整的 SQL 谓词（即形式为`WHERE *`value`*`的谓词，其中*`value`*是列名或常量表达式且未使用比较运算符）重写为`WHERE *`value`* <> 0`，以便查询解析器、查询优化器和查询执行器只需处理完整的谓词。

        这一变化的一个显著效果是，对于布尔值，`EXPLAIN`输出现在显示`true`和`false`，而不是`1`和`0`。

        此变化的另一个效果是，在 SQL 布尔上下文中对 JSON 值的评估执行隐式比较与 JSON 整数 0。考虑如下所示创建和填充的表：

        ```sql
        mysql> CREATE TABLE test (id INT, col JSON);

        mysql> INSERT INTO test VALUES (1, '{"val":true}'), (2, '{"val":false}');
        ```

        以前，服务器在将提取的`true`或`false`值在 SQL 布尔上下文中进行比较时，尝试将其转换为 SQL 布尔值，如下面使用`IS TRUE`的查询所示：

        ```sql
        mysql> SELECT id, col, col->"$.val" FROM test WHERE col->"$.val" IS TRUE;
        +------+---------------+--------------+
        | id   | col           | col->"$.val" |
        +------+---------------+--------------+
        |    1 | {"val": true} | true         |
        +------+---------------+--------------+
        ```

        在 MySQL 8.0.17 及更高版本中，提取值与 JSON 整数 0 的隐式比较会导致不同的结果：

        ```sql
        mysql> SELECT id, col, col->"$.val" FROM test WHERE col->"$.val" IS TRUE;
        +------+----------------+--------------+
        | id   | col            | col->"$.val" |
        +------+----------------+--------------+
        |    1 | {"val": true}  | true         |
        |    2 | {"val": false} | false        |
        +------+----------------+--------------+
        ```

        从 MySQL 8.0.21 开始，您可以在执行测试之前对提取的值使用`JSON_VALUE()`进行类型转换，如下所示：

        ```sql
        mysql> SELECT id, col, col->"$.val" FROM test
            ->     WHERE JSON_VALUE(col, "$.val" RETURNING UNSIGNED) IS TRUE;
        +------+---------------+--------------+
        | id   | col           | col->"$.val" |
        +------+---------------+--------------+
        |    1 | {"val": true} | true         |
        +------+---------------+--------------+
        ```

        同样从 MySQL 8.0.21 开始，服务器提供警告：在 SQL 布尔上下文中评估 JSON 值会隐式与 JSON 整数 0 进行比较；如果这不是您想要的，请考虑在这种方式中比较提取值时，使用 JSON_VALUE RETURNING 将 JSON 转换为 SQL 数值类型。

    +   在 MySQL 8.0.17 及更高版本中，具有`NOT IN (*`子查询`*)`或`NOT EXISTS (*`子查询`*)`的`WHERE`条件在内部转换为反连接。 （反连接返回表中没有与连接条件匹配的行的表中的所有行。）这会删除子查询，可以加快查询执行速度，因为现在子查询的表在顶层处理。

        这类似于并重用了现有的外连接的`IS NULL`（`Not exists`）优化；请参阅 EXPLAIN Extra Information。

    +   从 MySQL 8.0.21 开始，单表`UPDATE`或`DELETE`语句现在在许多情况下可以使用半连接转换或子查询材料化。这适用于以下形式的语句：

        +   `UPDATE t1 SET t1.a=*`value`* WHERE t1.a IN (SELECT t2.a FROM t2)`

        +   `DELETE FROM t1 WHERE t1.a IN (SELECT t2.a FROM t2)`

        这可以用于满足以下条件的单表`UPDATE`或`DELETE`：

        +   `UPDATE`或`DELETE`语句使用具有`[NOT] IN`或`[NOT] EXISTS`谓词的子查询。

        +   该语句没有`ORDER BY`子句，也没有`LIMIT`子句。

            （`UPDATE`和`DELETE`的多表版本不支持`ORDER BY`或`LIMIT`。）

        +   目标表不支持读取前写入删除（仅适用于`NDB`表）。

        +   基于子查询中包含的任何提示和`optimizer_switch`的值，允许半连接或子查询材料化。

        当半连接优化用于符合条件的单表`DELETE`或`UPDATE`时，在优化器跟踪中可见：对于多表语句，跟踪中有一个`join_optimization`对象，而对于单表语句则没有。转换也可在`EXPLAIN FORMAT=TREE`或`EXPLAIN ANALYZE`的输出中看到；单表语句显示`<not executable by iterator executor>`，而多表语句报告完整计划。

        从 MySQL 8.0.21 开始，使用`InnoDB`表的多表`UPDATE`语句支持半一致性读取，适用于弱于`REPEATABLE READ`的事务隔离级别。

    +   **改进的哈希连接性能。** MySQL 8.0.23 重新实现了用于哈希连接的哈希表，从而改进了哈希连接性能。这项工作包括修复了一个问题（Bug #31516149, Bug #99933），即哈希连接只能使用约分配给连接缓冲区（`join_buffer_size`）的 2/3 内存。

        新的哈希表通常比旧的更快，并且在对齐、键/值以及存在许多相等键的情况下使用更少的内存。此外，当哈希表的大小增加时，服务器现在可以释放旧内存。

+   **公共表达式。** MySQL 现在支持公共表达式，包括非递归和递归。公共表达式使得可以使用命名的临时结果集，通过允许在`SELECT`语句和某些其他语句之前使用`WITH`")子句来实现。有关更多信息，请参见 Section 15.2.20, “WITH (公共表达式)”")。

    截至 MySQL 8.0.19，递归公共表达式（CTE）的`SELECT`部分支持`LIMIT`子句。还支持带有`OFFSET`的`LIMIT`。有关更多信息，请参见递归公共表达式。

+   **窗口函数。** MySQL 现在支持窗口函数，对于查询的每一行，执行使用与该行相关的行的计算。这些函数包括`RANK()`、`LAG()`和`NTILE()`等函数。此外，现在几个现有的聚合函数也可以用作窗口函数（例如，`SUM()`和`AVG()`）。有关更多信息，请参见 Section 14.20, “窗口函数”。

+   **Lateral 派生表。** 现在，派生表前面可以加上`LATERAL`关键字，以指定允许引用（依赖）同一`FROM`子句中之前表的列。Lateral 派生表使得某些 SQL 操作成为可能，这些操作无法使用非 Lateral 派生表完成，或者需要使用效率较低的变通方法。请参见 Section 15.2.15.9, “Lateral 派生表”。

+   **单表 DELETE 语句中的别名。** 在 MySQL 8.0.16 及更高版本中，单表`DELETE`语句支持使用表别名。

+   **正则表达式支持。** 以前，MySQL 使用 Henry Spencer 正则表达式库来支持正则表达式操作符(`REGEXP`, `RLIKE`)。现在，正则表达式支持已经重新实现，使用了提供完整 Unicode 支持且支持多字节的 International Components for Unicode (ICU)。`REGEXP_LIKE()`函数以`REGEXP`和`RLIKE`操作符的方式执行正则表达式匹配，这两者现在是该函数的同义词。此外，`REGEXP_INSTR()`、`REGEXP_REPLACE()`和`REGEXP_SUBSTR()`函数可用于查找匹配位置以及执行子字符串替换和提取。`regexp_stack_limit`和`regexp_time_limit`系统变量可控制匹配引擎的资源消耗。更多信息，请参见第 14.8.2 节，“正则表达式”。关于使用正则表达式的应用程序可能受到实现更改影响的信息，请参见正则表达式兼容性注意事项。

    这一变化的一个影响是，在 MySQL 8.0 中，`[a-zA-Z]`和`[0-9]`的性能比`[[:alpha:]]`和`[[:digit:]]`要好得多。已经使用模式匹配中的字符类的现有应用程序应升级为使用范围。

+   **内部临时表。** `TempTable`存储引擎取代了`MEMORY`存储引擎，成为内存中临时表的默认引擎。`TempTable`存储引擎为`VARCHAR`和`VARBINARY`列提供高效的存储。`internal_tmp_mem_storage_engine`会话变量定义了内存中临时表的存储引擎。允许的值为`TempTable`（默认）和`MEMORY`。`temptable_max_ram`变量定义了`TempTable`存储引擎在数据存储到磁盘之前可以使用的最大内存量。

+   **日志记录。** 这些增强功能旨在改进日志记录：

    +   错误日志已重写以使用 MySQL 组件架构。传统错误日志使用内置组件实现，使用系统日志进行记录则使用可加载组件实现。此外，还提供了可加载的 JSON 日志写入器。更多信息，请参见 Section 7.4.2, “The Error Log”。

    +   从 MySQL 8.0.30 开始，在`InnoDB`存储引擎可用之前，错误日志组件可以在启动时隐式加载。这种新的加载错误日志组件的方法会加载并启用由`log_error_services`变量定义的组件。

        以前，必须首先使用`INSTALL COMPONENT`安装错误日志组件，并且只能在`InnoDB`完全可用后加载，因为要加载的组件列表是从`mysql.components`表中读取的，该表是一个`InnoDB`表。

        隐式加载错误日志组件具有以下优点：

        +   日志组件在启动序列中较早加载，使记录的信息更早可用。

        +   在启动过程中发生故障时，有助于避免缓冲日志信息的丢失。

        +   不需要使用`INSTALL COMPONENT`加载日志组件，简化了错误日志配置。

        使用`INSTALL COMPONENT`显式加载日志组件的方法仍然受支持，以确保向后兼容性。

        更多信息，请参见 Section 7.4.2.1, “Error Log Configuration”。

+   **备份锁。** 新的备份锁类型允许在线备份期间进行 DML 操作，同时阻止可能导致不一致快照的操作。新的备份锁由`LOCK INSTANCE FOR BACKUP`和`UNLOCK INSTANCE`语法支持。需要`BACKUP_ADMIN`权限才能使用这些语句。

+   **复制。** MySQL 复制功能已进行以下增强：

    +   MySQL 复制现在支持使用紧凑的二进制格式对 JSON 文档的部分更新进行二进制日志记录，相比记录完整的 JSON 文档，可以节省日志空间。当使用基于语句的日志记录时，此类紧凑日志记录会自动执行，并且可以通过将新的`binlog_row_value_options`系统变量设置为`PARTIAL_JSON`来启用。更多信息，请参见 Partial Updates of JSON Values，以及`binlog_row_value_options`的描述。

+   **连接管理。** MySQL 服务器现在允许为管理连接专门配置 TCP/IP 端口。这提供了一种替代方案，即使已经建立了 `max_connections` 连接，也可以在用于普通连接的网络接口上配置单个管理连接。请参阅 Section 7.1.12.1, “Connection Interfaces”。

    MySQL 现在提供了更多控制压缩以减少发送到服务器的字节数的功能。以前，给定连接要么未经压缩，要么使用 `zlib` 压缩算法。现在，还可以使用 `zstd` 算法，并为 `zstd` 连接选择压缩级别。允许在服务器端配置压缩算法，以及在客户端程序发起的连接和参与源/副本复制或组复制的服务器端的连接发起端配置压缩算法。有关更多信息，请参阅 Section 6.2.8, “Connection Compression Control”。

+   **配置。** MySQL 中主机名的最大允许长度已从先前的 60 个字符提高到 255 个 ASCII 字符。例如，数据字典、`mysql` 系统模式、性能模式、`INFORMATION_SCHEMA` 和 `sys` 模式中与主机名相关的列；`CHANGE MASTER TO` 语句中的 `MASTER_HOST` 值；`SHOW PROCESSLIST` 语句输出中的 `Host` 列；帐户名称中的主机名（例如在帐户管理语句和 `DEFINER` 属性中使用）；以及与主机名相关的命令选项和系统变量。

    注意事项：

    +   允许的主机名长度增加可能会影响具有主机名列索引的表。例如，现在在索引主机名的 `mysql` 系统模式中的表具有显式的 `ROW_FORMAT` 属性为 `DYNAMIC`，以容纳更长的索引值。

    +   一些文件名值配置设置可能基于服务器主机名构建。允许的值受基础操作系统的限制，可能不允许文件名足够长以包含 255 个字符的主机名。这会影响 `general_log_file`、`log_error`、`pid_file`、`relay_log` 和 `slow_query_log_file` 系统变量及相应选项。如果基于主机名的值对于操作系统来说太长，则必须提供显式较短的值。

    +   尽管服务器现在支持 255 字符的主机名，但使用`--ssl-mode=VERIFY_IDENTITY`选项建立到服务器的连接受 OpenSSL 支持的最大主机名长度的限制。主机名匹配涉及 SSL 证书的两个字段，其最大长度如下：通用名称：最大长度 64；主题备用名称：根据 RFC#1034 的最大长度。

+   **插件。** 以前，MySQL 插件可以用 C 或 C++ 编写。现在插件使用的 MySQL 头文件包含 C++ 代码，这意味着插件必须用 C++ 而不是 C 编写。

+   **C API。** MySQL C API 现在支持用于与 MySQL 服务器进行非阻塞通信的异步函数。每个函数都是现有同步函数的异步对应项。如果从服务器连接读取或写入必须等待，则同步函数会阻塞。异步函数使应用程序能够检查服务器连接上的工作是否准备好继续。如果没有准备好，应用程序可以在稍后再次检查之前执行其他工作。请参阅 C API 异步接口。

+   **强制转换的附加目标类型。** 函数`CAST()`和`CONVERT()`现在支持转换为类型`DOUBLE`、`FLOAT`和`REAL`。在 MySQL 8.0.17 中添加。请参阅第 14.10 节，“强制转换函数和运算符”。

+   **JSON 模式验证。** MySQL 8.0.17 添加了两个函数`JSON_SCHEMA_VALID()`和`JSON_SCHEMA_VALIDATION_REPORT()`用于验证 JSON 文档是否符合 JSON 模式。`JSON_SCHEMA_VALID()`如果文档符合模式则返回 TRUE（1），否则返回 FALSE（0）。`JSON_SCHEMA_VALIDATION_REPORT()`返回一个包含验证结果详细信息的 JSON 文档。以下声明适用于这两个函数：

    +   模式必须符合 JSON Schema 规范的 Draft 4 版本。

    +   支持`required`属性。

    +   不支持外部资源和`$ref`关键字。

    +   支持正则表达式模式；无效模式会被静默忽略。

    有关更多信息和示例，请参阅第 14.17.7 节，“JSON 模式验证函数”。

+   **多值索引。** 从 MySQL 8.0.17 开始，`InnoDB`支持创建多值索引，这是在存储值数组的`JSON`列上定义的二级索引，对于单个数据记录可以有多个索引记录。这样的索引使用类似于`CAST(data->'$.zipcode' AS UNSIGNED ARRAY)`的键部分定义。MySQL 优化器会自动为适当的查询使用多值索引，可以在`EXPLAIN`输出中查看。

    作为这项工作的一部分，MySQL 添加了一个新函数`JSON_OVERLAPS()`和一个新的`MEMBER OF()`运算符，用于处理`JSON`文档，并通过在下面的列表中描述的新`ARRAY`关键字扩展了`CAST()`函数：

    +   `JSON_OVERLAPS()`比较两个`JSON`文档。如果它们包含任何共同的键值对或数组元素，则函数返回 TRUE（1）；否则返回 FALSE（0）。如果两个值都是标量，函数执行简单的相等性测试。如果一个参数是 JSON 数组，另一个是标量，则标量被视为数组元素。因此，`JSON_OVERLAPS()`作为`JSON_CONTAINS()`的补充。

    +   `MEMBER OF()`测试第一个操作数（标量或 JSON 文档）是否是作为第二个操作数传递的 JSON 数组的成员，如果是，则返回 TRUE（1），否则返回 FALSE（0）。不执行操作数的类型转换。

    +   `CAST(`expression` AS `type` ARRAY)`允许通过将位于*`json_path`*中的 JSON 文档中的 JSON 数组转换为 SQL 数组来创建一个功能性索引。类型说明符仅限于`CAST()`已支持的类型，但不包括`BINARY`（不支持）。`CAST()`（和`ARRAY`关键字）仅由`InnoDB`支持，并且仅用于创建多值索引。

    有关多值索引的详细信息，包括示例，请参见多值索引。第 14.17.3 节，“搜索 JSON 值的函数”提供了有关`JSON_OVERLAPS()`和`MEMBER OF()`的信息，以及使用示例。

+   **可提示的 time_zone。** 截至 MySQL 8.0.17，`time_zone`会话变量可以使用`SET_VAR`进行提示。

+   **重做日志归档。** 截至 MySQL 8.0.17 版本，`InnoDB`支持重做日志归档。备份工具复制重做日志记录时，有时可能无法跟上重做日志生成的速度，导致在备份操作进行时丢失重做日志记录，因为这些记录被覆盖。重做日志归档功能通过将重做日志记录顺序写入归档文件来解决此问题。备份工具可以根据需要从归档文件中复制重做日志记录，从而避免数据的潜在丢失。有关更多信息，请参见重做日志归档。

+   **克隆插件。** 截至 MySQL 8.0.17 版本，MySQL 提供了一个克隆插件，允许在本地或从远程 MySQL 服务器实例克隆`InnoDB`数据。本地克隆操作将克隆数据存储在 MySQL 实例运行的同一服务器或节点上。远程克隆操作将克隆数据通过网络从捐赠 MySQL 服务器实例传输到启动克隆操作的接收服务器或节点。

    克隆插件支持复制。除了克隆数据外，克隆操作还从捐赠端提取并传输复制坐标，并在接收端应用这些坐标，从而使得可以使用克隆插件为配置组复制成员和副本提供服务。使用克隆插件进行配置比复制大量事务要快得多且更有效。组复制成员还可以配置为使用克隆插件作为恢复的替代方法，以便成员自动选择从种子成员检索组数据的最有效方式。

    有关更多信息，请参见 Section 7.6.7, “克隆插件”，以及 Section 20.5.4.2, “用于分布式恢复的克隆”。

    截至 MySQL 8.0.27 版本，克隆操作进行时，允许在捐赠 MySQL 服务器实例上进行并发 DDL 操作。以前，在克隆操作期间会持有备份锁，阻止在捐赠端进行并发 DDL。要恢复到在克隆操作期间阻止捐赠端进行并发 DDL 的先前行为，请启用`clone_block_ddl`变量。参见 Section 7.6.7.4, “克隆和并发 DDL”。

    从 MySQL 8.0.29 开始，`clone_delay_after_data_drop`变量允许在远程克隆操作开始时在接收方 MySQL 服务器实例上删除现有数据后立即指定延迟时间。延迟旨在在从捐赠方 MySQL 服务器实例克隆数据之前为接收方主机上的文件系统释放足够的空间。某些文件系统会异步释放空间。在这些文件系统上，在删除现有数据后太快克隆数据可能导致由于空间不足而克隆操作失败。最大延迟时间为 3600 秒（1 小时）。默认设置为 0（无延迟）。

    从 MySQL 8.0.37 开始，克隆允许在不同的点发布版本之间进行。换句话说，以前必须匹配点发布号，现在只需要主版本号和次版本号匹配。

    例如，克隆功能现在允许将 8.0.37 克隆到 8.0.41 或 8.0.51 克隆到 8.0.39。对于早于 8.0.37 的版本仍然存在先前的限制，因此不允许克隆 8.0.36 到 8.0.42 或反之亦不允许。

+   **哈希连接优化。** 从 MySQL 8.0.18 开始，只要连接中的每对表都包含至少一个等值连接条件，并且没有索引适用于任何连接条件，就会使用哈希连接。哈希连接不需要索引，尽管它可以与仅适用于单表谓词的索引一起使用。在大多数情况下，哈希连接比块嵌套循环算法更有效率。这种连接可以通过这种方式进行优化：

    ```sql
    SELECT *
        FROM t1
        JOIN t2
            ON t1.c1=t2.c1;

    SELECT *
        FROM t1
        JOIN t2
            ON (t1.c1 = t2.c1 AND t1.c2 < t2.c2)
        JOIN t3
            ON (t2.c1 = t3.c1)
    ```

    哈希连接也可以用于笛卡尔积——也就是在没有指定连接条件时。

    你可以通过`EXPLAIN FORMAT=TREE`或`EXPLAIN ANALYZE`来查看特定查询中是否使用了哈希连接优化。（在 MySQL 8.0.20 及更高版本中，也可以使用`EXPLAIN`，省略`FORMAT=TREE`。）

    哈希连接可用的内存量受`join_buffer_size`值的限制。如果哈希连接需要的内存超过这么多，则在磁盘上执行；磁盘哈希连接可以使用的磁盘文件数量受`open_files_limit`的限制。

    截至 MySQL 8.0.19 版本，MySQL 8.0.18 中引入的`hash_join`优化器开关不再受支持（hash_join=on 仍然作为 optimizer_switch 值的一部分，但设置它不再产生任何效果）。`HASH_JOIN`和`NO_HASH_JOIN`优化提示也不再受支持。该开关和提示现已被弃用；预计它们将在未来的 MySQL 版本中被移除。在 MySQL 8.0.18 及更高版本中，可以使用`NO_BNL`优化器开关来禁用哈希连接。

    在 MySQL 8.0.20 及更高版本中，MySQL 服务器不再使用块嵌套循环，而是在以前会使用块嵌套循环的任何时候都使用哈希连接，即使查询不包含等值连接条件。这适用于内部非等值连接、半连接、反连接、左外连接和右外连接。`block_nested_loop`标志用于`optimizer_switch`系统变量，以及`BNL`和`NO_BNL`优化提示仍然受支持，但从现在开始仅控制哈希连接的使用。此外，内部和外部连接（包括半连接和反连接）现在都可以使用批量键访问（BKA），该方法逐步分配连接缓冲区内存，以便个别查询不必使用实际上并不需要的大量资源来解决问题。从 MySQL 8.0.18 开始，仅支持内部连接的 BKA。

    MySQL 8.0.20 还用迭代器执行器替换了先前版本 MySQL 中使用的执行器。此工作包括替换了旧的索引子查询引擎，该引擎控制形式为`WHERE *`value`* IN (SELECT *`column`* FROM *`table`* WHERE ...)`的查询，对于那些未被优化为半连接的`IN`查询，以及以相同形式材料化的查询，这些查询以前依赖于旧执行器。

    欲了解更多信息和示例，请参阅第 10.2.1.4 节，“哈希连接优化”。另请参阅批量键访问连接。

+   **EXPLAIN ANALYZE 语句。** MySQL 8.0.18 中实现了一种新形式的`EXPLAIN`语句，`EXPLAIN ANALYZE`，为每个用于处理查询的迭代器提供了关于`SELECT`语句执行的`TREE`格式的扩展信息，使得可以比较查询的预估成本与实际成本。这些信息包括启动成本、总成本、此迭代器返回的行数以及执行的循环次数。

    在 MySQL 8.0.21 及更高版本中，此语句还支持`FORMAT=TREE`指定符。`TREE`是唯一支持的格式。

    查看使用 EXPLAIN ANALYZE 获取信息，获取更多信息。

+   **查询转换注入。** 在版本 8.0.18 及更高版本中，MySQL 将转换操作注入到表达式和条件中的查询项树中，其中参数的数据类型与预期数据类型不匹配。这不会影响查询结果或执行速度，但使得执行的查询等效于符合 SQL 标准的查询，同时保持与之前版本的 MySQL 的向后兼容性。

    这种隐式转换现在在时间类型（`DATE`, `DATETIME`, `TIMESTAMP`, `TIME`) 和数值类型（`SMALLINT`, `TINYINT`, `MEDIUMINT`, `INT`/`INTEGER`, `BIGINT`; `DECIMAL`/`NUMERIC`; `FLOAT`, `DOUBLE`, `REAL`; `BIT`) 之间以及它们使用任何标准数值比较运算符（`=`, `>=`, `>`, `<`, `<=`, `<>`/`!=`, 或 `<=>`) 进行比较时执行。在这种情况下，任何不是`DOUBLE`的值都会被转换为`DOUBLE`。现在还会对`DATE`或`TIME`值与`DATETIME`值之间的比较执行转换注入，其中在必要时将参数转换为`DATETIME`。

    从 MySQL 8.0.21 开始，当将字符串类型与其他类型进行比较时，也会执行这种转换。进行转换的字符串类型包括`CHAR`，`VARCHAR`，`BINARY`，`VARBINARY`，`BLOB`，`TEXT`，`ENUM`和`SET`。将字符串类型的值与数值类型或`YEAR`进行比较时，字符串转换为`DOUBLE`；如果另一个参数的类型不是`FLOAT`，`DOUBLE`或`REAL`，则也将其转换为`DOUBLE`。将字符串类型与`DATETIME`或`TIMESTAMP`值进行比较时，字符串转换为`DATETIME`；将字符串类型与`DATE`进行比较时，字符串转换为`DATE`。

    可以通过查看`EXPLAIN ANALYZE`，`EXPLAIN FORMAT=JSON`或者如下所示的`EXPLAIN FORMAT=TREE`来查看将转换注入到给定查询中的情况：

    ```sql
    mysql> CREATE TABLE d (dt DATETIME, d DATE, t TIME);
    Query OK, 0 rows affected (0.62 sec)

    mysql> CREATE TABLE n (i INT, d DECIMAL, f FLOAT, dc DECIMAL);
    Query OK, 0 rows affected (0.51 sec)

    mysql> CREATE TABLE s (c CHAR(25), vc VARCHAR(25),
        ->     bn BINARY(50), vb VARBINARY(50), b BLOB, t TEXT,
        ->     e ENUM('a', 'b', 'c'), se SET('x' ,'y', 'z'));
    Query OK, 0 rows affected (0.50 sec)

    mysql> EXPLAIN FORMAT=TREE SELECT * from d JOIN n ON d.dt = n.i\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Inner hash join *(cast(d.dt as double) = cast(n.i as double))*
    (cost=0.70 rows=1)
        -> Table scan on n  (cost=0.35 rows=1)
        -> Hash
            -> Table scan on d  (cost=0.35 rows=1)

    mysql> EXPLAIN FORMAT=TREE SELECT * from s JOIN d ON d.dt = s.c\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Inner hash join *(d.dt = cast(s.c as datetime(6)))*  (cost=0.72 rows=1)
        -> Table scan on d  (cost=0.37 rows=1)
        -> Hash
            -> Table scan on s  (cost=0.35 rows=1)

    1 row in set (0.01 sec)

    mysql> EXPLAIN FORMAT=TREE SELECT * from n JOIN s ON n.d = s.c\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Inner hash join *(cast(n.d as double) = cast(s.c as double))*  (cost=0.70 rows=1)
        -> Table scan on s  (cost=0.35 rows=1)
        -> Hash
            -> Table scan on n  (cost=0.35 rows=1)

    1 row in set (0.00 sec)
    ```

    还可以通过执行`EXPLAIN [FORMAT=TRADITIONAL]`来查看这种转换，此时还需要在执行`EXPLAIN`语句后发出`SHOW WARNINGS`。

+   **TIMESTAMP 和 DATETIME 的时区支持。** 从 MySQL 8.0.19 开始，服务器接受插入日期时间（`TIMESTAMP`和`DATETIME`）值的时区偏移量。此偏移量使用与设置`time_zone`系统变量时使用的相同格式，只是当偏移量的小时部分小于 10 时需要前导零，并且不允许`'-00:00'`。包含时区偏移量的日期时间文字示例包括`'2019-12-11 10:40:30-05:00'`，`'2003-04-14 03:30:00+10:00'`和`'2020-01-01 15:35:45+05:30'`。

    选择日期时间值时，不显示时区偏移量。

    可以将包含时区偏移量的日期时间文字用作预备语句参数值。

    作为这项工作的一部分，用于设置`time_zone`系统变量的值现在也限制在`-13:59`到`+14:00`的范围内（仍然可以将名称值分配给`time_zone`，例如`'EST'`，`'Posix/Australia/Brisbane'`和`'Europe/Stockholm'`到这个变量，前提是 MySQL 时区表已加载；参见填充时区表）。

    有关更多信息和示例，请参见第 7.1.15 节，“MySQL 服务器时区支持”，以及第 13.2.2 节，“DATE、DATETIME 和 TIMESTAMP 类型”。

+   **JSON 模式 CHECK 约束失败的精确信息。** 当使用`JSON_SCHEMA_VALID()`指定`CHECK`约束时，MySQL 8.0.19 及更高版本提供有关此类约束失败原因的精确信息。

    有关示例和更多信息，请参见 JSON_SCHEMA_VALID()和 CHECK 约束和 CHECK 约束")。另请参见第 15.1.20.6 节，“CHECK 约束”。

+   **带有 ON DUPLICATE KEY UPDATE 的行和列别名。** 从 MySQL 8.0.19 开始，可以使用别名引用要插入的行，以及可选地引用其列。考虑在具有列`a`和`b`的表`t`上执行以下`INSERT`语句：

    ```sql
    INSERT INTO t SET a=9,b=5
        ON DUPLICATE KEY UPDATE a=VALUES(a)+VALUES(b);
    ```

    使用别名`new`表示新行，并且在某些情况下，使用别名`m`和`n`表示此行的列，`INSERT`语句可以以许多不同的方式重写，以下是一些示例：

    ```sql
    INSERT INTO t SET a=9,b=5 AS new
        ON DUPLICATE KEY UPDATE a=new.a+new.b;

    INSERT INTO t VALUES(9,5) AS new
        ON DUPLICATE KEY UPDATE a=new.a+new.b;

    INSERT INTO t SET a=9,b=5 AS new(m,n)
        ON DUPLICATE KEY UPDATE a=m+n;

    INSERT INTO t VALUES(9,5) AS new(m,n)
        ON DUPLICATE KEY UPDATE a=m+n;
    ```

    有关更多信息和示例，请参见第 15.2.7.2 节，“INSERT ... ON DUPLICATE KEY UPDATE 语句”。

+   **SQL 标准明确的表子句和表值构造函数。** 根据 SQL 标准添加了表值构造函数和明确的表子句。这些分别在 MySQL 8.0.19 中实现为`TABLE`语句和`VALUES`语句。

    `TABLE`语句的格式为`TABLE *`table_name`*`，等同于`SELECT * FROM *`table_name`*`。它支持`ORDER BY`和`LIMIT`子句（后者带有可选的`OFFSET`），但不允许选择单个表列。`TABLE`可以在任何需要等效`SELECT`语句的地方使用；这包括连接、联合、`INSERT ... SELECT`、`REPLACE`、`CREATE TABLE ... SELECT`语句和子查询。例如：

    +   `TABLE t1 UNION TABLE t2`等同于`SELECT * FROM t1 UNION SELECT * FROM t2`

    +   `CREATE TABLE t2 TABLE t1`等同于`CREATE TABLE t2 SELECT * FROM t1`

    +   `SELECT a FROM t1 WHERE b > ANY (TABLE t2)`等同于`SELECT a FROM t1 WHERE b > ANY (SELECT * FROM t2)`。

    `VALUES` 可用于向 `INSERT`、`REPLACE` 或 `SELECT` 语句提供表值，由 `VALUES` 关键字后跟一系列由逗号分隔的行构造函数 (`ROW()`) 组成。例如，语句 `INSERT INTO t1 VALUES ROW(1,2,3), ROW(4,5,6), ROW(7,8,9)` 提供了一个符合 SQL 标准的等效于 MySQL 特定的 `INSERT INTO t1 VALUES (1,2,3), (4,5,6), (7,8,9)`。您也可以像操作表一样从 `VALUES` 表值构造函数中选择，但请记住在这样做时必须提供表别名，并像操作其他表一样使用这个 `SELECT`；这包括连接、联合和子查询。

    有关 `TABLE` 和 `VALUES` 的更多信息，以及它们的使用示例，请参阅本文档的以下部分：

    +   第 15.2.16 节，“TABLE 语句”

    +   第 15.2.19 节，“VALUES 语句”

    +   第 15.1.20.4 节，“CREATE TABLE ... SELECT 语句”

    +   第 15.2.7.1 节，“INSERT ... SELECT 语句”

    +   第 15.2.13.2 节，“JOIN 子句”

    +   第 15.2.15 节，“子查询”

    +   第 15.2.18 节，“UNION 子句”

+   **FORCE INDEX、IGNORE INDEX 的优化器提示。** MySQL 8.0 引入了索引级别的优化器提示，作为传统索引提示的类似物，如 第 10.9.4 节，“索引提示” 中所述。新提示在此列出，以及它们的 `FORCE INDEX` 或 `IGNORE INDEX` 等效项：

    +   `GROUP_INDEX`：等同于 `FORCE INDEX FOR GROUP BY`

        `NO_GROUP_INDEX`：等同于 `IGNORE INDEX FOR GROUP BY`

    +   `JOIN_INDEX`：等同于 `FORCE INDEX FOR JOIN`

        `NO_JOIN_INDEX`：等同于 `IGNORE INDEX FOR JOIN`

    +   `ORDER_INDEX`：等同于 `FORCE INDEX FOR ORDER BY`

        `NO_ORDER_INDEX`：等同于 `IGNORE INDEX FOR ORDER BY`

    +   `INDEX`：与`GROUP_INDEX`、`JOIN_INDEX`和`ORDER_INDEX`相同；相当于没有修饰符的`FORCE INDEX`

        `NO_INDEX`：与`NO_GROUP_INDEX`、`NO_JOIN_INDEX`和`NO_ORDER_INDEX`相同；相当于没有修饰符的`IGNORE INDEX`

    例如，以下两个查询是等效的：

    ```sql
    SELECT a FROM t1 FORCE INDEX (i_a) FOR JOIN WHERE a=1 AND b=2;

    SELECT /*+ JOIN_INDEX(t1 i_a) */ a FROM t1 WHERE a=1 AND b=2;
    ```

    先前列出的优化提示遵循与现有索引级别优化提示相同的基本语法和用法规则。

    这些优化提示旨在取代`FORCE INDEX`和`IGNORE INDEX`，我们计划在未来的 MySQL 版本中弃用，并随后从 MySQL 中删除。它们不实现`USE INDEX`的单个精确等效项；相反，你可以使用`NO_INDEX`、`NO_JOIN_INDEX`、`NO_GROUP_INDEX`或`NO_ORDER_INDEX`中的一个或多个来实现相同的效果。

    欲了解更多信息和使用示例，请参阅索引级别优化提示。

+   **JSON_VALUE()函数。** MySQL 8.0.21 实现了一个新函数`JSON_VALUE()`，旨在简化对`JSON`列的索引。在其最基本形式中，它接受一个 JSON 文档和指向该文档中单个值的 JSON 路径作为参数，还允许你使用`RETURNING`关键字指定返回类型（可选）。`JSON_VALUE(*`json_doc`*, *`path`* RETURNING *`type`*)`等效于这个：

    ```sql
    CAST(
        JSON_UNQUOTE( JSON_EXTRACT(*json_doc*, *path*) )
        AS *type*
    );
    ```

    你还可以指定`ON EMPTY`、`ON ERROR`或两者子句，类似于与`JSON_TABLE()`一起使用的方式。

    你可以使用`JSON_VALUE()`来在`JSON`列上创建表达式索引，就像这样：

    ```sql
    CREATE TABLE t1(
        j JSON,
        INDEX i1 ( (JSON_VALUE(j, '$.id' RETURNING UNSIGNED)) )
    );

    INSERT INTO t1 VALUES ROW('{"id": "123", "name": "shoes", "price": "49.95"}');
    ```

    使用此表达式的查询，如下所示，可以利用索引：

    ```sql
    SELECT j->"$.name" as name, j->"$.price" as price 
        FROM t1
        WHERE JSON_VALUE(j, '$.id' RETURNING UNSIGNED) = 123;
    ```

    在许多情况下，这比从`JSON`列创建生成列，然后在生成列上创建索引更简单。

    有关更多信息和示例，请参阅`JSON_VALUE()`的描述。

+   **用户评论和用户属性。** MySQL 8.0.21 引入了在创建或更新用户帐户时设置用户评论和用户属性的功能。用户评论由作为`CREATE USER`或`ALTER USER`语句中使用的`COMMENT`子句的参数传递的任意文本组成。用户属性由作为这两个语句之一中使用的`ATTRIBUTE`子句的参数传递的 JSON 对象形式的数据组成。属性可以包含 JSON 对象表示法中的任何有效键值对。在单个`CREATE USER`或`ALTER USER`语句中只能使用`COMMENT`或`ATTRIBUTE`中的一个。

    用户评论和用户属性在内部一起存储为 JSON 对象，评论文本作为具有`comment`作为其键的元素的值。此信息可以从 Information Schema `USER_ATTRIBUTES`表的`ATTRIBUTE`列中检索；由于它是以 JSON 格式，您可以使用 MySQL 的 JSON 函数和运算符来解析其内容（参见第 14.17 节，“JSON 函数”）。对用户属性的连续更改与其当前值合并，就像使用`JSON_MERGE_PATCH()`函数一样。

    示例：

    ```sql
    mysql> CREATE USER 'mary'@'localhost' COMMENT 'This is Mary Smith\'s account';
    Query OK, 0 rows affected (0.33 sec)

    mysql> ALTER USER 'mary'@'localhost'
        -≫     ATTRIBUTE '{"fname":"Mary", "lname":"Smith"}';
    Query OK, 0 rows affected (0.14 sec)

    mysql> ALTER USER 'mary'@'localhost'
        -≫     ATTRIBUTE '{"email":"mary.smith@example.com"}';
    Query OK, 0 rows affected (0.12 sec)

    mysql> SELECT
        ->    USER,
        ->    HOST,
        ->    ATTRIBUTE->>"$.fname" AS 'First Name',
        ->    ATTRIBUTE->>"$.lname" AS 'Last Name',
        ->    ATTRIBUTE->>"$.email" AS 'Email',
        ->    ATTRIBUTE->>"$.comment" AS 'Comment'
        -> FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
        -> WHERE USER='mary' AND HOST='localhost'\G
    *************************** 1\. row ***************************
          USER: mary
          HOST: localhost
    First Name: Mary
     Last Name: Smith
         Email: mary.smith@example.com
       Comment: This is Mary Smith's account
    1 row in set (0.00 sec)
    ```

    有关更多信息和示例，请参阅第 15.7.1.3 节，“CREATE USER 语句”，第 15.7.1.1 节，“ALTER USER 语句”和第 28.3.46 节，“INFORMATION_SCHEMA USER_ATTRIBUTES 表”。

+   **新的 optimizer_switch 标志。** MySQL 8.0.21 添加了两个新标志用于`optimizer_switch`系统变量，如下列表所述：

    +   `prefer_ordering_index`标志

        默认情况下，MySQL 尝试对具有`LIMIT`子句的任何`ORDER BY`或`GROUP BY`查询使用有序索引，只要优化器确定这样做会导致更快的执行。因为在某些情况下选择不同的优化对于这些查询实际上执行更好是可能的，现在可以通过将`prefer_ordering_index`标志设置为`off`来禁用此优化。

        此标志的默认值为`on`。

    +   `subquery_to_derived`标志

        当该标志设置为`on`时，优化器将符合条件的标量子查询转换为对派生表的连接。例如，查询`SELECT * FROM t1 WHERE t1.a > (SELECT COUNT(a) FROM t2)`被重写为`SELECT t1.a FROM t1 JOIN ( SELECT COUNT(t2.a) AS c FROM t2 ) AS d WHERE t1.a > d.c`。

        这种优化可以应用于子查询，该子查询是`SELECT`、`WHERE`、`JOIN`或`HAVING`子句的一部分；包含一个或多个聚合函数但没有`GROUP BY`子句；不是相关的；并且不使用任何非确定性函数。

        该优化也可以应用于作为`IN`、`NOT IN`、`EXISTS`或`NOT EXISTS`参数的表子查询，并且不包含`GROUP BY`。例如，查询`SELECT * FROM t1 WHERE t1.b < 0 OR t1.a IN (SELECT t2.a + 1 FROM t2)`被重写为`SELECT a, b FROM t1 LEFT JOIN (SELECT DISTINCT 1 AS e1, t2.a AS e2 FROM t2) d ON t1.a + 1 = d.e2 WHERE t1.b < 0 OR d.e1 IS NOT NULL`。

        从 MySQL 8.0.24 开始，这种优化也可以应用于相关的标量子查询，通过对其应用额外的分组，然后在提升的谓词上进行外连接。例如，像`SELECT * FROM t1 WHERE (SELECT a FROM t2 WHERE t2.a=t1.a) > 0`这样的查询可以重写为`SELECT t1.* FROM t1 LEFT OUTER JOIN (SELECT a, COUNT(*) AS ct FROM t2 GROUP BY a) AS derived ON t1.a = derived.a WHERE derived.a > 0`。MySQL 执行基数检查，以确保子查询不会返回多于一行（`ER_SUBQUERY_NO_1_ROW`）。更多信息请参见第 15.2.15.7 节，“相关子查询”。

        这种优化通常是禁用的，因为在大多数情况下它并不会带来明显的性能优势；该标志默认设置为`off`。

    更多信息，请参见第 10.9.2 节，“可切换的优化”。另请参见第 10.2.1.19 节，“LIMIT 查询优化”，第 10.2.2.1 节，“使用半连接转换优化 IN 和 EXISTS 子查询谓词”和第 10.2.2.4 节，“使用合并或材料化优化派生表、视图引用和公共表达式”。

+   **XML 增强。** 从 MySQL 8.0.21 开始，`LOAD XML`语句现在支持要导入的 XML 中的`CDATA`部分。

+   **支持转换为 YEAR 类型。** 从 MySQL 8.0.22 开始，服务器允许转换为`YEAR`。`CAST()`和`CONVERT()`函数都支持单个数字、两位数字和四位数字的`YEAR`值。对于一位数和两位数的值，允许的范围是 0-99。四位数值必须在 1901-2155 范围内。`YEAR`也可以作为`JSON_VALUE()`函数的返回类型；此函数仅支持四位数年份。

    字符串、时间日期和浮点值都可以转换为`YEAR`。不支持将`GEOMETRY`值转换为`YEAR`。

    更多信息，请参阅`CONVERT()`函数的描述。

+   **以 UTC 检索 TIMESTAMP 值。** MySQL 8.0.22 及更高版本支持将系统时区的`TIMESTAMP`列值转换为 UTC `DATETIME`进行检索，使用`CAST(*`value`* AT TIME ZONE *`specifier`* AS DATETIME)`，其中 specifier 是 `[INTERVAL] '+00:00'` 或 `'UTC'` 中的一个。如果需要，可以指定转换返回的`DATETIME`值的精度高达 6 位小数。此结构不支持`ARRAY`关键字。

    也支持使用时区偏移插入到表中的`TIMESTAMP`值。不支持对`CONVERT()`或任何其他 MySQL 函数或结构使用`AT TIME ZONE`。

    更多信息和示例，请参阅`CAST()`函数的描述。

+   **转储文件输出同步。** MySQL 8.0.22 及更高版本支持通过`SELECT INTO DUMPFILE`和`SELECT INTO OUTFILE`语句写入文件时的定期同步。可以通过将`select_into_disk_sync`系统变量设置为`ON`来启用此功能；写入缓冲区的大小由`select_into_buffer_size`设置的值确定；默认值为 131072（2¹⁷）字节。

    在同步到磁盘后，可以使用`select_into_disk_sync_delay`设置可选的延迟；默认情况下没有延迟（0 毫秒）。

    更多信息，请参阅此项中先前引用的变量的描述。

+   **语句的单次准备。** 从 MySQL 8.0.22 开始，准备好的语句只需准备一次，而不是每次执行时都准备。这是在执行`PREPARE`时完成的。对于存储过程中的任何语句也是如此；当首次执行存储过程时，语句将被准备一次。

    这种变化的一个结果是，准备语句中使用的动态参数解析方式也发生了变化，具体列在以下方式中：

    +   准备语句参数在准备语句时被分配一个数据类型；该类型将持续存在于语句的每次后续执行中（除非语句被重新准备；请参见下文）。

        使用不同的数据类型为准备好的语句中的给定参数或用户变量，在第一次执行后的后续执行可能导致语句被重新准备；因此，建议在重新执行准备好的语句时使用相同的数据类型。

    +   为了与 SQL 标准保持一致，不再接受以下使用窗口函数的构造：

        +   `NTILE(NULL)`

        +   `NTH_VALUE(*`expr`*, NULL)`

        +   `LEAD(*`expr`*, *`nn`*)`和`LAG(*`expr`*, *`nn`*)`，其中*`nn`*是一个负数

        这有助于更好地符合 SQL 标准。有关更多详细信息，请参阅各个函数描述。

    +   在准备语句中引用的用户变量现在在准备语句时确定其数据类型；该类型将持续存在于语句的每次后续执行中。

    +   在存储过程中出现的语句引用的用户变量现在在第一次执行语句时确定其数据类型；该类型将持续存在于包含存储过程的任何后续调用中。

    +   当执行形式为`SELECT *`expr1`*, *`expr2`*, ... FROM *`table`* ORDER BY ?`的准备语句时，为参数传递整数值*`N`*不再导致结果按照选择列表中的第*`N`*个表达式排序；结果不再排序，如在`ORDER BY *`constant`*`中所预期的那样。

    仅在存储过程中使用的准备好的语句只需准备一次，可以增强语句的性能，因为它消除了重复准备的额外成本。这样做还可以避免可能的多次回滚准备结构，这是 MySQL 中许多问题的根源。

    有关更多信息，请参阅第 15.5.1 节，“准备语句”。

+   **将 RIGHT JOIN 视为 LEFT JOIN 处理。** 截至 MySQL 8.0.22，服务器在内部将所有`RIGHT JOIN`实例处理为`LEFT JOIN`，消除了在解析时未执行完全转换的一些特殊情况。

+   **派生条件下推优化。** MySQL 8.0.22（及更高版本）为具有物化派生表的查询实现了派生条件下推。对于诸如`SELECT * FROM (SELECT i, j FROM t1) AS dt WHERE i > *`constant`*`这样的查询，现在在许多情况下可以将外部`WHERE`条件下推到派生表，这种情况下结果为`SELECT * FROM (SELECT i, j FROM t1 WHERE i > *`constant`*) AS dt`。

    以前，如果派生表被物化而不是合并，MySQL 会将整个表物化，然后使用`WHERE`条件限定行。使用派生条件下推优化将`WHERE`条件移动到子查询中，通常可以减少必须处理的行数，从而减少执行查询所需的时间。

    外部`WHERE`条件可以直接下推到一个物化派生表，当派生表不使用任何聚合或窗口函数时。当派生表具有`GROUP BY`并且不使用任何窗口函数时，外部`WHERE`条件可以作为`HAVING`条件下推到派生表。当派生表使用窗口函数且外部`WHERE`引用窗口函数的`PARTITION`子句中使用的列时，`WHERE`条件也可以下推。

    派生条件下推优化默认启用，由`optimizer_switch`系统变量的`derived_condition_pushdown`标志指示。该标志在 MySQL 8.0.22 中添加，默认设置为`on`；要禁用特定查询的优化，可以使用`NO_DERIVED_CONDITION_PUSHDOWN`优化提示（也在 MySQL 8.0.22 中添加）。如果由于`derived_condition_pushdown`设置为`off`而禁用优化，可以使用`DERIVED_CONDITION_PUSHDOWN`为给定查询启用它。

    派生条件下推优化不能用于包含`LIMIT`子句的派生表。在 MySQL 8.0.29 之前，当查询包含`UNION`时，优化也无法使用。在 MySQL 8.0.29 及更高版本中，条件在大多数情况下可以下推到联合的两个查询块；有关更多信息，请参见 Section 10.2.2.5，“派生条件下推优化”。

    此外，一个使用子查询的条件本身无法被下推，而一个 `WHERE` 条件也无法被下推到作为外连接的内连接的派生表。有关更多信息和示例，请参阅 Section 10.2.2.5, “派生条件下推优化”。

+   **MySQL 授权表上的非锁定读取。** 截至 MySQL 8.0.22 版本，为了允许在 MySQL 授权表上进行并发的 DML 和 DDL 操作，先前在 MySQL 授权表上获取行锁的读取操作现在作为非锁定读取执行。

    现在在 MySQL 授权表上作为非锁定读取执行的操作包括：

    +   通过连接列表和子查询从授权表读取数据的 `SELECT` 语句和其他只读语句，包括使用任何事务隔离级别的 `SELECT ... FOR SHARE` 语句。

    +   从授权表读取数据的 DML 操作（通过连接列表或子查询），但不修改它们，使用任何事务隔离级别。

    欲了解更多信息，请参阅 授权表并发性。

+   **FROM_UNIXTIME()、UNIX_TIMESTAMP()、CONVERT_TZ() 的 64 位支持。** 截至 MySQL 8.0.28 版本，函数 `FROM_UNIXTIME()`、`UNIX_TIMESTAMP()` 和 `CONVERT_TZ()` 在支持它们的平台上处理 64 位值。这包括 Linux、MacOS 和 Windows 的 64 位版本。

    在兼容的平台上，`UNIX_TIMESTAMP()` 现在处理的值可达到 `'3001-01-18 23:59:59.999999'` UTC，而 `FROM_UNIXTIME()` 可以将值转换为自 Unix 纪元以来的 32536771199.999999 秒；`CONVERT_TZ()` 现在在转换后接受不超过 `'3001-01-18 23:59:59.999999'` UTC 的值。

    这些函数在 32 位平台上的行为不受这些更改的影响。`TIMESTAMP` 类型的行为也不受影响（在任何平台上）；若要处理 `'2038-01-19 03:14:07.999999'` UTC 之后的日期时间，请改用 `DATETIME` 类型。

    欲了解更多信息，请参阅刚刚讨论的各个函数的描述，在 Section 14.7, “日期和时间函数”。

+   **资源分配控制。** 从 MySQL 8.0.28 开始，您可以通过检查`Global_connection_memory`状态变量来查看所有常规用户发出的查询使用的内存量。（此总数不包括系统用户（如 MySQL root）使用的资源。它还不包括由`InnoDB`缓冲池占用的任何内存。）

    要启用对`Global_connection_memory`的更新，需要设置`global_connection_memory_tracking = 1`；默认情况下为`0`（关闭）。您可以通过设置`connection_memory_chunk_size`来控制多久更新一次`Global_connection_memory`。

    还可以通过设置以下列出的系统变量中的一个或两个，或两者同时，在会话或全局级别为普通用户设置内存使用限制：

    +   `connection_memory_limit`：为每个连接分配的内存量。每当任何用户超出此限制时，来自该用户的新查询将被拒绝。

    +   `global_connection_memory_limit`：为所有连接分配的内存量。每当超出此限制时，来自任何常规用户的新查询将被拒绝。

    这些限制不适用于系统进程或管理帐户。

    有关更多信息，请参阅所引用变量的描述。

+   **分离的 XA 事务。** MySQL 8.0.29 增加了对 XA 事务的支持，一旦准备就绪，就不再与发起连接相关联。这意味着它们可以由另一个连接提交或回滚，并且当前会话可以立即开始另一个事务。

    系统变量`xa_detach_on_prepare`控制 XA 事务是否分离；默认值为`ON`，导致所有 XA 事务被分离。在此情况下，禁止对 XA 事务使用临时表。

    更多信息，请参见第 15.3.8.2 节，“XA 事务状态”。

+   **自动二进制日志清除控制。** MySQL 8.0.29 增加了`binlog_expire_logs_auto_purge`系统变量，提供了一个单一接口来启用和禁用二进制日志的自动清除。默认情况下启用（`ON`）；要禁用二进制日志文件的自动清除，请将此变量设置为`OFF`。

    `binlog_expire_logs_auto_purge`必须为`ON`，才能自动清除二进制日志文件；此变量的值优先于任何其他服务器选项或变量，包括（但不限于）`binlog_expire_logs_seconds`。

    `binlog_expire_logs_auto_purge`的设置对`PURGE BINARY LOGS`没有影响。

+   **条件例程和触发器创建语句。** 从 MySQL 8.0.29 开始，以下语句支持`IF NOT EXISTS`选项：

    +   `CREATE FUNCTION`

    +   `CREATE PROCEDURE`

    +   `CREATE TRIGGER`

    对于`CREATE FUNCTION`用于创建存储函数和`CREATE PROCEDURE`时，此选项可防止出现同名例程的错误。对于`CREATE FUNCTION`用于创建可加载函数时，该选项可防止出现同名可加载函数的错误。对于`CREATE TRIGGER`，该选项可防止出现同名触发器的错误，如果在同一模式和同一表中已经存在同名触发器。

    此增强功能使这些语句的语法更接近于`CREATE DATABASE`、`CREATE TABLE`、`CREATE USER`和`CREATE EVENT`（这些语句已经支持`IF NOT EXISTS`），并且补充了`DROP PROCEDURE`、`DROP FUNCTION`和`DROP TRIGGER`语句支持的`IF EXISTS`选项。

    欲了解更多信息，请参阅指定 SQL 语句的描述，以及函数名称解析。另请参阅第 19.5.1.7 节，“复制 CREATE TABLE ... SELECT 语句”。

+   **包含 FIDO 库升级。** MySQL 8.0.30 将`fido2`库（与`authentication_fido`插件一起使用）从版本 1.5.0 升级到版本 1.8.0。

    查看第 8.4.1.11 节，“FIDO 可插拔认证”，获取更多信息。

+   **字符集：特定语言排序规则。** 以前，当多种语言具有完全相同的排序规则定义时，MySQL 仅为其中一种语言实现了排序规则，这意味着某些语言仅由其他语言特定的`utf8mb4` Unicode 9.0 排序规则覆盖。MySQL 8.0.30（及更高版本）通过为以前仅由其他语言特定排序规则覆盖的语言提供特定语言排序规则来解决此类问题。新排序规则覆盖的语言列在此处：

    +   挪威语（新挪威语）

        和

        挪威语（书面挪威语）

    +   塞尔维亚语（拉丁字符）

    +   波斯尼亚语（拉丁字符）

    +   保加利亚语

    +   加利西亚语

    +   蒙古语（西里尔字母）

    MySQL 为每种语言提供了`*_as_cs`和`*_ai_ci`排序规则。

    欲了解更多信息，请参阅特定语言排序规则。

+   **IF EXISTS 和 IGNORE UNKNOWN USER 选项用于 REVOKE。** MySQL 8.0.30 实现了两个新选项用于`REVOKE`，可用于确定当语句中指定的用户、角色或权限找不到或无法分配时，语句是否产生错误或警告。这里提供了基本语法，显示了这些新选项的放置位置：

    ```sql
    REVOKE [IF EXISTS] *privilege_or_role* 
        ON *object* 
        FROM *user_or_role* [IGNORE UNKNOWN USER]
    ```

    `IF EXISTS` 导致未成功的`REVOKE`语句引发警告，而不是错误，只要语句中指定的目标用户或角色实际存在，尽管语句中可能引用任何找不到的角色或权限。

    `IGNORE UNKNOWN USER` 导致未成功的*`REVOKE`*引发警告，而不是错误，当语句中指定的目标用户或角色找不到时。

    欲了解更多信息和示例，请参阅第 15.7.1.8 节“REVOKE 语句”。

+   **生成的隐形主键。** 从 MySQL 8.0.30 开始，可以运行一个复制源服务器，使得在创建没有显式主键的`InnoDB`表时，会添加一个生成的隐形主键（GIPK）。添加到这样一个表的生成键列定义等同于这里显示的内容：

    ```sql
    my_row_id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT INVISIBLE PRIMARY KEY
    ```

    GIPK 模式默认情况下未启用。要启用它，请将`sql_generate_invisible_primary_key`服务器系统变量设置为`ON`。

    生成的不可见主键通常在诸如 `SHOW CREATE TABLE` 和 `SHOW INDEX` 这样的语句的输出中可见，以及在 MySQL 信息模式表中，如 `COLUMNS` 和 `STATISTICS` 表中。您可以通过将 `show_gipk_in_create_table_and_information_schema` 设置为 `OFF` 来在这些情况下隐藏它们。

    作为这项工作的一部分，为 **mysqldump** 和 **mysqlpump** 添加了一个新的 `--skip-generated-invisible-primary-key` 选项，用于在输出中排除生成的不可见主键、列和列值。

    **GIPKs 和表之间的复制，无论是否有主键。** 在 MySQL 复制中，副本实际上会忽略源端的 `sql_generate_invisible_primary_key` 设置，因此对复制表没有影响。MySQL 8.0.32 及更高版本使得副本可以向任何 `InnoDB` 表中添加一个生成的不可见主键，即使该表在复制时没有主键。您可以通过在副本上调用 `CHANGE REPLICATION SOURCE TO ... REQUIRE_TABLE_PRIMARY_KEY_CHECK = GENERATE` 来实现这一点。

    `REQUIRE_TABLE_PRIMARY_KEY_CHECK = GENERATE` 与 MySQL Group Replication 不兼容。

    欲了解更多信息，请参阅 第 15.1.20.11 节，“生成的不可见主键”。

+   **崩溃安全的 XA 事务。** 以前，XA 事务在意外停止时对于二进制日志并不完全具有弹性，如果这种情况发生在服务器执行 `XA PREPARE`、`XA COMMIT` 或 `XA ROLLBACK` 时，服务器不能保证能够恢复到正确的状态，可能会导致二进制日志中存在未应用的额外 XA 事务，或者缺少一个或多个已应用的 XA 事务。从 MySQL 8.0.30 开始，这不再是一个问题，无论出于何种原因，从复制拓扑中掉出的服务器都可以在重新加入时始终被带回一致的 XA 事务状态。

    *已知问题*：当相同的事务 XID 用于顺序执行 XA 事务，并且在执行`XA COMMIT ... ONE PHASE`期间发生中断时，再次使用此相同的 XID，在此事务在存储引擎中准备好之后，可能不再可能在二进制日志和存储引擎之间同步状态。

    欲了解更多信息，请参阅第 15.3.8.3 节，“XA 事务的限制”。

+   **使用 UNION 进行嵌套。** 从 MySQL 8.0.31 开始，带括号的查询表达式的主体可以与 `UNION` 结合，最多可以嵌套到 63 级深度。此类查询以前会因错误 `ER_NOT_SUPPORTED_YET` 被拒绝，但现在允许。此类查询的 `EXPLAIN` 输出如下：

    ```sql
    mysql> EXPLAIN FORMAT=TREE ( 
        ->   (SELECT a, b, c FROM t ORDER BY a LIMIT 3) ORDER BY b LIMIT 2
        -> ) ORDER BY c LIMIT 1\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Limit: 1 row(s)  (cost=5.55..5.55 rows=1)
        -> Sort: c, limit input to 1 row(s) per chunk  (cost=2.50 rows=0)
            -> Table scan on <result temporary>  (cost=2.50 rows=0)
                -> Temporary table  (cost=5.55..5.55 rows=1)
                    -> Limit: 2 row(s)  (cost=2.95..2.95 rows=1)
                        -> Sort: b, limit input to 2 row(s) per chunk  (cost=2.50 rows=0)
                            -> Table scan on <result temporary>  (cost=2.50 rows=0)
                                -> Temporary table  (cost=2.95..2.95 rows=1)
                                    -> Limit: 3 row(s)  (cost=0.35 rows=1)
                                        -> Sort: t.a, limit input to 3 row(s) per chunk  (cost=0.35 rows=1)
                                            -> Table scan on t  (cost=0.35 rows=1)

    1 row in set (0.00 sec)
    ```

    MySQL 在折叠带括号的查询表达式的主体时遵循 SQL 标准语义，因此外部较高限制不能覆盖内部较低限制。例如，`(SELECT ... LIMIT 5) LIMIT 10` 最多只能返回五行。

    MySQL 优化器的解析器在执行任何简化或合并操作后，才会施加 63 级限制。

    欲了解更多信息，请参阅第 15.2.11 节，“带括号的查询表达式”。

+   **禁用查询重写。** 以前，使用 `Rewriter` 插件时，所有查询都会被重写，而不管用户是谁。在某些情况下，这可能会有问题，例如在管理系统时，或者应用来自复制源或由 **mysqldump** 或其他 MySQL 程序创建的转储文件的语句时。MySQL 8.0.31 通过实现新的用户权限 `SKIP_QUERY_REWRITE` 来解决这些问题；具有此权限的用户发出的语句将被 `Rewriter` 忽略并且不会被重写。

    MySQL 8.0.31 还添加了一个新的服务器系统变量 `rewriter_enabled_for_threads_without_privilege_checks`。当设置为 `OFF` 时，由 `PRIVILEGE_CHECKS_USER` 为 `NULL` 的线程（例如复制应用程序线程）发出的可重写语句不会被 `Rewriter` 插件重写。默认值为 `ON`，这意味着这些语句会被重写。

    欲了解更多信息，请参阅第 7.6.4 节，“Rewriter 查询重写插件”。

+   **XA 语句的复制过滤。** 以前，当使用`--replicate-do-db`或`--replicate-ignore-db`时，默认数据库会过滤掉`XA START`、`XA END`、`XA COMMIT`和`XA ROLLBACK`语句，这可能导致遗漏的事务。从 MySQL 8.0.31 开始，在这种情况下，这些语句不会被过滤，无论`binlog_format`的值是什么。

+   **复制过滤和权限检查。** 从 MySQL 8.0.31 开始，当使用复制过滤时，副本不再因被过滤掉的事件引起与权限检查或`require_row_format`验证相关的复制错误，从而可以过滤掉任何验证失败的事务。

    因为过滤行上的权限检查不再会导致复制停止，所以副本现在可以仅接受给定用户已被授予访问权限的数据库部分；只要对数据库的此部分的更新仅以行格式进行复制。

    当从一个使用表进行管理或其他用途的本地或云服务迁移到 MySQL HeatWave 服务时，此功能也可能对于入站复制用户无法访问的情况有所帮助。

    有关更多信息，请参见第 19.2.5 节，“服务器如何评估复制过滤规则”，以及第 19.5.1.29 节，“复制期间的副本错误”。

+   **INTERSECT 和 EXCEPT 表运算符。** MySQL 8.0.31 添加了对 SQL `INTERSECT` 和 `EXCEPT` 表运算符的支持。其中 *`a`* 和 *`b`* 代表查询结果集，这些运算符的行为如下：

    +   `*`a`* INTERSECT *`b`*`仅包括同时出现在结果集 *`a`* 和 *`b`* 中的行。

    +   `*`a`* EXCEPT *`b`*`仅返回结果集 *`a`* 中那些不同时出现在 *`b`* 中的行。

    `INTERSECT DISTINCT`、`INTERSECT ALL`、`EXCEPT DISTINCT` 和 `EXCEPT ALL` 都受支持；`DISTINCT` 是 `INTERSECT` 和 `EXCEPT` 的默认值（与`UNION`相同）。

    有关更多信息和示例，请参见第 15.2.8 节，“INTERSECT 子句”和第 15.2.4 节，“EXCEPT 子句”。

+   **用户定义的直方图。** 从 MySQL 8.0.31 开始，可以将列的直方图设置为用户指定的 JSON 值。可以使用以下 SQL 语法完成：

    ```sql
    ANALYZE TABLE *tbl_name* 
      UPDATE HISTOGRAM ON *col_name*
      USING DATA '*json_data*'
    ```

    此语句为表*`tbl_name`*的列*`col_name`*创建或覆盖直方图，使用直方图的 JSON 表示*`json_data`*。执行此语句后，您可以通过查询信息模式`COLUMN_STATISTICS`表来验证直方图是否已创建或更新，如下所示：

    ```sql
    SELECT HISTOGRAM FROM INFORMATION_SCHEMA.COLUMN_STATISTICS
      WHERE TABLE_NAME='*tbl_name*' 
      AND COLUMN_NAME='*col_name*';
    ```

    返回的列值应与先前 `ANALYZE TABLE` 语句中使用的*`json_data`*相同。

    在直方图采样过程中，可能会错过被视为重要的值。当发生这种情况时，您可能希望修改直方图或根据完整数据集设置自己的直方图。此外，对大型用户数据集进行采样并构建直方图是消耗资源的操作，可能会影响用户查询。通过这种增强，直方图生成可以从（主）服务器转移到副本上执行；然后将生成的直方图分配给源服务器上的适当表列。

    有关更多信息和示例，请参见直方图统计分析。

+   **服务器构建 ID（Linux）。** MySQL 8.0.31 为 Linux 系统添加了只读`build_id` 系统变量，其中在编译时生成一个 160 位的 `SHA1` 签名；`build_id` 的值是生成的值转换为十六进制字符串的值，为构建提供了唯一标识符。

    每次 MySQL 启动时，`build_id` 都会写入服务器日志。

    如果您从源代码构建 MySQL，您会观察到每次重新编译服务器时此值会更改。有关更多信息，请参见第 2.8 节，“从源代码安装 MySQL”。

    此变量不支持 Linux 以外的平台。

+   **默认 EXPLAIN 输出格式。** MySQL 8.0.32 添加了一个系统变量`explain_format`，用于确定在没有任何`FORMAT`选项的情况下获取查询执行计划的`EXPLAIN`语句输出格式。例如，如果`explain_format`的值为`TREE`，那么任何这样的`EXPLAIN`输出都使用类似树状的格式，就像语句已经指定了`FORMAT=TREE`一样。

    此行为将被`FORMAT`选项中设置的值覆盖。假设*`explain_format`*设置为*`TREE`*；即使如此，`EXPLAIN FORMAT=JSON *`stmt`*` 也会使用 JSON 输出格式显示结果。

    有关更多信息和示例，请参阅 `explain_format` 系统变量的描述，以及 获取执行计划信息。对于 `EXPLAIN ANALYZE` 的行为也有影响；请参阅 使用 EXPLAIN ANALYZE 获取信息。

+   **ST_TRANSFORM() 支持笛卡尔坐标系。** 在 MySQL 8.0.30 之前，`ST_TRANSFORM()` 函数不支持笛卡尔空间参考系统。在 MySQL 8.0.30 及更高版本中，此函数支持流行的可视化伪墨卡托（EPSG 1024）投影方法，用于 WGS 84 伪墨卡托（SRID 3857）。MySQL 8.0.32 及更高版本支持除 EPSG 1042、EPSG 1043、EPSG 9816 和 EPSG 9826 之外的所有笛卡尔空间参考系统。

### MySQL 8.0 中已弃用的特性

以下功能在 MySQL 8.0 中已弃用，并可能在未来的系列中被移除。如果显示了替代方案，则应更新应用程序以使用它们。

对于在 MySQL 8.0 中使用已弃用功能的应用程序，在更高版本的 MySQL 系列中已移除的情况下，从 MySQL 8.0 源到更高系列的复制时可能会导致语句失败，或者在源和副本上产生不同的效果。为避免此类问题，应修订使用 8.0 中已弃用功能的应用程序，尽可能使用替代方案。

+   **数据库授权中的通配符字符。** 在 MySQL 8.0.35 中，使用 `%` 和 `_` 作为数据库授权中的通配符已被弃用。您应该预期通配符功能在未来的 MySQL 版本中被移除，并且这些字符始终被视为文字，就像在 `partial_revokes` 服务器系统变量的值为 `ON` 时一样。

    此外，当检查权限时，服务器将 `%` 视为 `localhost` 的处理方式在 MySQL 8.0.35 中也已弃用，因此可能在未来的 MySQL 版本中被移除。

+   MySQL 8.0.35 及更高版本中已弃用可插拔的 FIDO 认证。

+   `--character-set-client-handshake` 选项最初用于从非常旧版本的 MySQL 进行升级，现在在 MySQL 8.0.35 及更高版本中已弃用，使用时会发出警告。您应该预期此选项在未来的 MySQL 版本中被移除；依赖此选项的应用程序应尽快迁移。

+   在 MySQL 8.0 版本中，开始于 MySQL 8.0.35，`old` 和 `new` 服务器系统变量及相关服务器选项已被弃用。每当设置或读取这些变量时，现在会发出警告。由于这些变量将在未来的 MySQL 版本中被移除，依赖于它们的应用程序应尽快开始迁移。

+   MySQL 8.0.34 版本开始，传统审计日志过滤模式已被弃用。传统审计日志过滤系统变量现在会发出新的弃用警告。这些弃用的变量要么是只读的，要么是动态的。

    (只读) `audit_log_policy` 现在在服务器启动时，当值不为 `ALL`（默认值）时，会向 MySQL 服务器错误日志写入警告消息。

    (动态) `audit_log_include_accounts`，`audit_log_exclude_accounts`，`audit_log_statement_policy`，以及 `audit_log_connection_policy`。动态变量基于使用情况打印警告消息：

    +   在 MySQL 服务器启动期间向 `audit_log_include_accounts` 或 `audit_log_exclude_accounts` 传递非 NULL 值现在会向服务器错误日志写入警告消息。

    +   在 MySQL 服务器启动期间传递非默认值给 `audit_log_statement_policy` 或 `audit_log_connection_policy` 现在会向服务器错误日志写入警告消息。`ALL` 是这两个变量的默认值。

    +   在 MySQL 客户端会话期间使用 `SET` 语法更改现有值现在会向客户端日志写入警告消息。

    +   在 MySQL 客户端会话期间使用 `SET PERSIST` 语法持久化变量现在会向客户端日志写入警告消息。

+   在 MySQL 8.0.34 及更高版本中，`mysql_native_password` 认证插件已被弃用，如果账户尝试使用 `mysql_native_password` 进行认证，现在会在服务器错误日志中产生弃用警告。

+   `ssl_fips_mode` 服务器系统变量，`--ssl-fips-mode` 客户端选项，以及 `MYSQL_OPT_SSL_FIPS_MODE` 选项已被弃用，并将在未来的 MySQL 版本中被移除。

+   `keyring_file`和`keyring_encrypted_file`插件已从 MySQL 8.0.34 开始弃用。这些密钥环插件已被`component_keyring_file`和`component_keyring_encrypted_file`组件取代。有关密钥环组件和插件的简明比较，请参见第 8.4.4.1 节，“密钥环组件与密钥环插件比较”。

+   从 MySQL 8.0.31 开始，`keyring_oci`插件已被弃用，并可能在未来的 MySQL 版本中被移除。相反，考虑使用`component_keyring_oci`组件来存储密钥环数据（参见第 8.4.4.11 节，“使用 Oracle Cloud 基础设施 Vault 密钥环组件”）。

+   `utf8mb3`字符集已被弃用。请改用`utf8mb4`。

+   以下字符集已被弃用：

    +   `ucs2` (参见第 12.9.4 节，“ucs2 字符集（UCS-2 Unicode 编码）”)

    +   `macroman`和`macce` (参见第 12.10.2 节，“西欧字符集”，以及第 12.10.3 节，“中欧字符集”)

    +   `dec` (参见第 12.10.2 节，“西欧字符集”)

    +   `hp8` (参见第 12.10.2 节，“西欧字符集”)

    在 MySQL 8.0.28 及更高版本中，任何这些字符集或它们的排序在以下任一方式中使用时会产生弃用警告：

    +   在启动 MySQL 服务器时使用`--character-set-server`或`--collation-server`

    +   在任何 SQL 语句中指定时，包括但不限于`CREATE TABLE`，`CREATE DATABASE`，`SET NAMES`和`ALTER TABLE`

    你应该使用`utf8mb4`而不是之前列出的任何字符集。

    用户自定义排序已被弃用。从 MySQL 8.0.33 开始，以下任一情况都会导致警告写入日志：

    +   在任何 SQL 语句中与用户自定义排序的名称一起使用`COLLATE`

    +   在`collation_server`，`collation_database`，或`collation_connection`的值中使用用户自定义排序的名称。

    你应该预计在未来的 MySQL 版本中将删除对用户定义排序规则的支持。

+   因为`caching_sha2_password`是 MySQL 8.0 中的默认身份验证插件，并提供了`sha256_password`身份验证插件功能的超集，`sha256_password`已被弃用；预计在未来的 MySQL 版本中将被移除。使用`sha256_password`进行身份验证的 MySQL 账户应该迁移到使用`caching_sha2_password`。

+   `validate_password`插件已被重新实现以使用组件基础架构。`validate_password`插件形式仍然可用，但现在已被弃用；预计在未来的 MySQL 版本中将被移除。使用该插件的 MySQL 安装应该过渡到使用组件。参见第 8.4.3.3 节，“过渡到密码验证组件”。

+   `ALTER TABLESPACE`和`DROP TABLESPACE`语句的`ENGINE`子句已被弃用。

+   `PAD_CHAR_TO_FULL_LENGTH` SQL 模式已被弃用。

+   `AUTO_INCREMENT`支持已被弃用于`FLOAT` - FLOAT, DOUBLE")和`DOUBLE` - FLOAT, DOUBLE")（以及任何同义词）类型的列。考虑从这些列中移除`AUTO_INCREMENT`属性，或将它们转换为整数类型。

+   `UNSIGNED`属性已被弃用于`FLOAT` - FLOAT, DOUBLE")、`DOUBLE` - FLOAT, DOUBLE")和`DECIMAL` - DECIMAL, NUMERIC")（以及任何同义词）类型的列。考虑为这些列使用简单的`CHECK`约束代替。

+   `FLOAT(*`M`*,*`D`*)`和`DOUBLE(*`M`*,*`D`*)`语法用于指定`FLOAT` - FLOAT, DOUBLE")和`DOUBLE` - FLOAT, DOUBLE")（以及任何同义词）类型列的数字位数，这是 MySQL 的非标准扩展。此语法已被弃用。

+   `ZEROFILL`属性已被弃用于数值数据类型，整数数据类型的显示宽度属性也是如此。考虑使用其他方法来实现这些属性的效果。例如，应用程序可以使用`LPAD()`函数将数字零填充到所需的宽度，或者它们可以将格式化的数字存储在`CHAR`列中。

+   对于字符串数据类型，`BINARY`属性是一个非标准的 MySQL 扩展，用于指定列字符集的二进制(`_bin`)排序规则（如果未指定列字符集，则使用表默认字符集）。在 MySQL 8.0 中，这种非标准用法的`BINARY`是模棱两可的，因为`utf8mb4`字符集有多个`_bin`排序规则，所以`BINARY`属性已被弃用；应调整应用程序以使用显式的`_bin`排序规则。

    使用`BINARY`指定数据类型或字符集的做法保持不变。

+   之前的 MySQL 版本支持非标准的简写表达式`ASCII`和`UNICODE`，分别用于`CHARACTER SET latin1`和`CHARACTER SET ucs2`。`ASCII`和`UNICODE`已被弃用（MySQL 8.0.28 及更高版本），现在会产生警告。在这两种情况下，请改用`CHARACTER SET`。

+   作为标准 SQL 的`AND`、`OR`和`NOT`运算符的非标准 C 风格的`&&`、`||`和`!`运算符已被弃用。使用非标准运算符的应用程序应调整为使用标准运算符。

    注意

    除非启用了`PIPES_AS_CONCAT` SQL 模式，否则使用`||`已被弃用（在这种情况下，`||`表示 SQL 标准的字符串连接运算符）。

+   `JSON_MERGE()`函数已被弃用。请改用`JSON_MERGE_PRESERVE()`。

+   `SQL_CALC_FOUND_ROWS`查询修饰符和相应的`FOUND_ROWS()`函数已被弃用。请查看`FOUND_ROWS()`的描述以获取替代策略的信息。

+   截至 MySQL 8.0.13，对于`CREATE TEMPORARY TABLE`，`TABLESPACE = innodb_file_per_table`和`TABLESPACE = innodb_temporary`子句已被弃用。

+   对于`SELECT`语句，在`FROM`之后但不是在`SELECT`的末尾使用`INTO`已被弃用，截至 MySQL 8.0.20。最好将`INTO`放在语句的末尾。

    对于`UNION`语句，截至 MySQL 8.0.20，这两个包含`INTO`的变体已被弃用：

    +   在查询表达式的尾部查询块中，在`FROM`之前使用`INTO`。

    +   在查询表达式的括号尾部块中，无论`INTO`相对于`FROM`的位置如何，都要使用。

    请参阅第 15.2.13.1 节，“SELECT ... INTO 语句”和第 15.2.18 节，“UNION 子句”。

+   自 MySQL 8.0.23 起，`FLUSH HOSTS`已被弃用。取而代之的是截断性能模式`host_cache`表：

    ```sql
    TRUNCATE TABLE performance_schema.host_cache;
    ```

    `TRUNCATE TABLE`操作需要表的`DROP`权限。

+   因为其升级系统模式中的系统表和其他模式中的对象的能力已被移至 MySQL 服务器，所以已弃用**mysql_upgrade**客户端。请参阅第 3.4 节，“MySQL 升级过程升级了什么”。

+   `--no-dd-upgrade`服务器选项已被弃用。它已被`--upgrade`选项取代，该选项提供了对数据字典和服务器升级行为的更精细控制。

+   创建于数据目录中并用于存储 MySQL 版本号的`mysql_upgrade_info`文件已被弃用；预计将在将来的 MySQL 版本中删除。

+   `relay_log_info_file`系统变量和`--master-info-file`选项已被弃用。以前，当设置`relay_log_info_repository=FILE`和`master_info_repository=FILE`时，这些用于指定中继日志信息日志和源信息日志的名称，但这些设置已被弃用。中继日志信息日志和源信息日志的文件使用已被 MySQL 8.0 中的崩溃安全复制表取代，这是默认设置。

+   由于优化器的更改使其过时且无效，`max_length_for_sort_data`系统变量现已被弃用。

+   这些用于压缩与服务器连接的传统参数已被弃用：`--compress`客户端命令行选项；`mysql_options()` C API 函数的`MYSQL_OPT_COMPRESS`选项；`slave_compressed_protocol`系统变量。有关替代参数的信息，请参阅第 6.2.8 节，“连接压缩控制”。

+   使用`MYSQL_PWD`环境变量指定 MySQL 密码已被弃用。

+   从 MySQL 8.0.20 开始，使用`VALUES()`来访问`INSERT ... ON DUPLICATE KEY UPDATE`中的新行值已被弃用。改为使用新行和列的别名。

+   因为在调用`JSON_TABLE()`时在`ON ERROR`之前指定`ON EMPTY`与 SQL 标准相悖，这种语法现在在 MySQL 中已被弃用。从 MySQL 8.0.20 开始，服务器在您尝试这样做时会打印警告。在单个`JSON_TABLE()`调用中指定这两个子句时，请确保首先使用`ON EMPTY`。

+   具有索引前缀的列从未作为表的分区键的一部分得到支持；以前，在创建、修改或升级分区表时允许这样做，但被排除在表的分区函数之外，并且服务器没有发出发生这种情况的警告。这种宽容的行为现在已被弃用，并且在将来的 MySQL 版本中，如果在分区键中使用任何这些列，将导致拒绝`CREATE TABLE`或`ALTER TABLE`语句。

    从 MySQL 8.0.21 开始，每当指定使用索引前缀的列作为分区键的一部分时，将为每个这样的列生成警告。每当因为所有提议的分区键中的列都具有索引前缀而拒绝`CREATE TABLE`或`ALTER TABLE`语句时，生成的错误现在提供了拒绝的确切原因。在任何情况下，这包括使用空的`PARTITION BY KEY()`子句通过隐式定义分区函数中使用的列为表的主键中的列的情况。

    有关更多信息和示例，请参见不支持列索引前缀用于键分区。

+   截至 MySQL 8.0.22，InnoDB memcached 插件已被弃用；预计在将来的 MySQL 版本中将删除对其的支持。

+   `temptable_use_mmap`变量从 MySQL 8.0.26 开始已被弃用；预计在将来的 MySQL 版本中将删除对其的支持。

+   `BINARY`运算符从 MySQL 8.0.27 开始已被弃用，您应该预期在将来的 MySQL 版本中将其移除。现在使用`BINARY`会导致警告。改用`CAST(... AS BINARY)`。

+   `default_authentication_plugin`变量从 MySQL 8.0.27 开始被弃用；预计在未来的 MySQL 版本中将不再支持它。

    `default_authentication_plugin`变量仍然在 MySQL 8.0.27 中使用，但与新的`authentication_policy`系统变量一起，并且优先级低于在 MySQL 8.0.27 中引入的多因素认证功能。详情请参见默认认证插件。

+   `--abort-slave-event-count`和`--disconnect-slave-event-count`服务器选项，由 MySQL 测试套件使用，通常不在生产中使用，从 MySQL 8.0.29 开始被弃用；预计这两个选项将在未来的 MySQL 版本中被移除。

+   `myisam_repair_threads`系统变量和**myisamchk** `--parallel-recover`选项从 MySQL 8.0.29 开始被弃用；预计在未来的 MySQL 版本中将不再支持这两者。

    从 MySQL 8.0.29 开始，`myisam_repair_threads`的值不为 1（默认值）会产生警告。

+   以前，MySQL 接受包含任意数量的（任意）分隔符字符的`DATE`、`TIME`、`DATETIME`和`TIMESTAMP`文字，以及在日期和时间部分之间、之前和之后有任意数量的空白字符的`DATETIME`和`TIMESTAMP`文字。从 MySQL 8.0.29 开始，服务器在文字值包含以下内容时会引发弃用警告：

    +   一个或多个非标准分隔符字符

    +   多余的分隔符字符

    +   除了空格字符（' '，`0x20`）之外的空白字符

    +   多余的空格字符

    每个时间值发出一个弃用警告，即使它有多个问题。在严格模式下，此警告不会升级为错误，因此在严格模式下执行这样一个值的`INSERT`仍然成功。

    你应该期待非标准行为在未来的 MySQL 版本中被移除，并立即采取措施确保你的应用程序不依赖于它。

    有关更多信息和示例，请参见日期和时间上下文中的字符串和数字文字。

+   `replica_parallel_type`系统变量及其相关的服务器选项`--replica-parallel-type`从 MySQL 8.0.29 开始已被弃用。从这个版本开始，读取或设置此值会引发弃用警告；请预期它将在未来的 MySQL 版本中被移除。

+   从 MySQL 8.0.30 开始，将`replica_parallel_workers`系统变量（或等效的服务器选项）设置为 0 已被弃用，并引发警告。当您希望副本使用单线程时，请改用`replica_parallel_workers=1`，这将产生相同的结果，但不会有警告。

+   `--skip-host-cache`服务器选项从 MySQL 8.0.30 开始已被弃用；预计在未来的 MySQL 版本中将被移除。请使用`host_cache_size`系统变量代替。

+   `--old-style-user-limits`选项，用于与非常旧（5.0.3 之前）版本向后兼容，从 MySQL 8.0.30 开始已被弃用；现在使用它会引发警告。您应该预期这个选项将在未来的 MySQL 版本中被移除。

+   `innodb_log_files_in_group`和`innodb_log_file_size`变量从 MySQL 8.0.30 开始已被弃用。这些变量已被`innodb_redo_log_capacity`变量取代。更多信息，请参见第 17.6.5 节，“重做日志”。

+   从 MySQL 8.0.32 开始，“FULL”作为未引用的标识符已被弃用，因为它是 SQL 标准中的保留关键字。这意味着像`CREATE TABLE full (c1 INT, c2 INT)`这样的语句现在会引发警告（`ER_WARN_DEPRECATED_TO_BE_REMOVED_IDENT_FULL`）。为了防止这种情况发生，更改名称或者像这里所示一样用反引号括起来（```sql):

    ```

    CREATE TABLE `full` (c1 INT, c2 INT);

    ```sql

    For more information, see Section 11.3, “Keywords and Reserved Words”.

*   Beginning with MySQL 8.0.32, the use of the dollar sign (`$`) as the leading character of an unquoted identifier is deprecated and produces a warning. Such usage is subject to removal in a future release of MySQL. This includes identifiers used as names of databases, tables, views, columns, or stored programs, as well as aliases for any of these. The dollar sign may still be used as the first character of a quoted identifier. See Section 11.2, “Schema Object Names”, for more information.

*   The `binlog_format` server system variable is deprecated as of MySQL 8.0.34, and is subject to being removed in a future release. Changing the binary logging format, is also deprecated, with the expectation that the removal of `binlog_format` will leave row-based binary logging, already the default in MySQL 8.0, as the only binary logging format used or supported by MySQL. For this reason, new MySQL installations should use *only* row-based binary logging; existing replication setups using `binlog_format=STATEMENT` or `binlog_format=MIXED` logging format should be migrated to the row-based format.

    The system variables `log_bin_trust_function_creators` and `log_statements_unsafe_for_binlog`, are used exclusively for statement-based logging and replication. For this reason, they are now also deprecated, and subject to removal in a future version of MySQL.

    Setting or selecting the value of `binlog_format`, `log_bin_trust_function_creators`, or `log_statements_unsafe_for_binlog` raises a warning in MySQL 8.0.34 and later.

*   The **mysqlpump** client utility program is deprecated beginning with MySQL 8.0.34, and produces a deprecation warning when invoked. This program is subject to removal in a future version of MySQL. Since MySQL provides other means of performing database dumps and backups with the same or additional functionality, including **mysqldump** and MySQL Shell, this program is now considered redundant.

    The associated **lz4_decompress** and **zlib_decompress** utilities are also deprecated as of MySQL 8.0.34.

*   The use of a version number without a whitespace character following (or end of comment) is deprecated as of MySQL 8.0.34, and raises a warning. This statement raises a warning in MySQL 8.0.34 or later, as shown here:

    ```

    mysql> CREATE TABLE t1(a INT, KEY (a)) /*!50110KEY_BLOCK_SIZE=1024*/ ENGINE=MYISAM;

    查询成功，影响行数为 0，警告数为 1（0.01 秒）

    mysql> SHOW WARNINGS\G

    *************************** 1\. 行 ***************************

    级别：警告

    代码：4164

    消息：立即在版本号后开始版本注释

    已弃用并可能在未来版本中更改行为。请插入

    版本号后的空格字符。

    1 行受影响（0.00 秒）

    ```sql

    To avoid such warnings, insert one or more whitespace characters after the version number, like this:

    ```

    mysql> CREATE TABLE t2(a INT, KEY (a)) /*!50110 KEY_BLOCK_SIZE=1024*/ ENGINE=MYISAM;

    查询成功，影响行数为 0（0.00 秒）

    ```sql

    See also Section 11.7, “Comments”.

*   As of MySQL 8.0.34, the `sync_relay_log_info` system variable is deprecated, along with its equivalent server startup option `--sync-relay-log-info`. You should expect support for this variable, and for storing replication applier metadata in a file, to be removed in a future version of MySQL. You are advised to update any of your MySQL applications which may depend on it before this occurs.

*   The `binlog_transaction_dependency_tracking` server system variable is deprecated as of MySQL 8.0.35, and subject to removal in a future version of MySQL. Referencing this variable or the equivalent **mysqld** startup option `--binlog-transaction-dependency-tracking` now triggers a warning. There are no plans to replace this variable or its functionality, which is expected later to be made internal to the server.

### Features Removed in MySQL 8.0

The following items are obsolete and have been removed in MySQL 8.0\. Where alternatives are shown, applications should be updated to use them.

For MySQL 5.7 applications that use features removed in MySQL 8.0, statements may fail when replicated from a MySQL 5.7 source to a MySQL 8.0 replica, or may have different effects on source and replica. To avoid such problems, applications that use features removed in MySQL 8.0 should be revised to avoid them and use alternatives when possible.

*   The `innodb_locks_unsafe_for_binlog` system variable was removed. The `READ COMMITTED` isolation level provides similar functionality.

*   The `information_schema_stats` variable, introduced in MySQL 8.0.0, was removed and replaced by `information_schema_stats_expiry` in MySQL 8.0.3.

    `information_schema_stats_expiry` defines an expiration setting for cached `INFORMATION_SCHEMA` table statistics. For more information, see Section 10.2.3, “Optimizing INFORMATION_SCHEMA Queries”.

*   Code related to obsolete `InnoDB` system tables was removed in MySQL 8.0.3. `INFORMATION_SCHEMA` views based on `InnoDB` system tables were replaced by internal system views on data dictionary tables. Affected `InnoDB` `INFORMATION_SCHEMA` views were renamed:

    **Table 1.1 Renamed InnoDB Information Schema Views**

    | Old Name | New Name |
    | `INNODB_SYS_COLUMNS` | `INNODB_COLUMNS` |
    | `INNODB_SYS_DATAFILES` | `INNODB_DATAFILES` |
    | `INNODB_SYS_FIELDS` | `INNODB_FIELDS` |
    | `INNODB_SYS_FOREIGN` | `INNODB_FOREIGN` |
    | `INNODB_SYS_FOREIGN_COLS` | `INNODB_FOREIGN_COLS` |
    | `INNODB_SYS_INDEXES` | `INNODB_INDEXES` |
    | `INNODB_SYS_TABLES` | `INNODB_TABLES` |
    | `INNODB_SYS_TABLESPACES` | `INNODB_TABLESPACES` |
    | `INNODB_SYS_TABLESTATS` | `INNODB_TABLESTATS` |
    | `INNODB_SYS_VIRTUAL` | `INNODB_VIRTUAL` |

    After upgrading to MySQL 8.0.3 or later, update any scripts that reference previous `InnoDB` `INFORMATION_SCHEMA` view names.

*   The following features related to account management are removed:

    *   Using `GRANT` to create users. Instead, use `CREATE USER`. Following this practice makes the `NO_AUTO_CREATE_USER` SQL mode immaterial for `GRANT` statements, so it too is removed, and an error now is written to the server log when the presence of this value for the `sql_mode` option in the options file prevents **mysqld** from starting.

    *   Using `GRANT` to modify account properties other than privilege assignments. This includes authentication, SSL, and resource-limit properties. Instead, establish such properties at account-creation time with `CREATE USER` or modify them afterward with `ALTER USER`.

    *   `IDENTIFIED BY PASSWORD '*`auth_string`*'` syntax for `CREATE USER` and `GRANT`. Instead, use `IDENTIFIED WITH *`auth_plugin`* AS '*`auth_string`*'` for `CREATE USER` and `ALTER USER`, where the `'*`auth_string`*'` value is in a format compatible with the named plugin.

        Additionally, because `IDENTIFIED BY PASSWORD` syntax was removed, the `log_builtin_as_identified_by_password` system variable is superfluous and was removed.

    *   The `PASSWORD()` function. Additionally, `PASSWORD()` removal means that `SET PASSWORD ... = PASSWORD('*`auth_string`*')` syntax is no longer available.

    *   The `old_passwords` system variable.

*   The query cache was removed. Removal includes these items:

    *   The `FLUSH QUERY CACHE` and `RESET QUERY CACHE` statements.

    *   These system variables: `query_cache_limit`, `query_cache_min_res_unit`, `query_cache_size`, `query_cache_type`, `query_cache_wlock_invalidate`.

    *   These status variables: `Qcache_free_blocks`, `Qcache_free_memory`, `Qcache_hits`, `Qcache_inserts`, `Qcache_lowmem_prunes`, `Qcache_not_cached`, `Qcache_queries_in_cache`, `Qcache_total_blocks`.

    *   These thread states: `checking privileges on cached query`, `checking query cache for query`, `invalidating query cache entries`, `sending cached result to client`, `storing result in query cache`, `Waiting for query cache lock`.

    *   The `SQL_CACHE` `SELECT` modifier.

    These deprecated query cache items remain deprecated, but have no effect; expect them to be removed in a future MySQL release:

    *   The `SQL_NO_CACHE` `SELECT` modifier.

    *   The `ndb_cache_check_time` system variable.

    The `have_query_cache` system variable remains deprecated, and always has a value of `NO`; expect it to be removed in a future MySQL release.

*   The data dictionary provides information about database objects, so the server no longer checks directory names in the data directory to find databases. Consequently, the `--ignore-db-dir` option and `ignore_db_dirs` system variables are extraneous and are removed.

*   The DDL log, also known as the metadata log, has been removed. Beginning with MySQL 8.0.3, this functionality is handled by the data dictionary `innodb_ddl_log` table. See Viewing DDL Logs.

*   The `tx_isolation` and `tx_read_only` system variables have been removed. Use `transaction_isolation` and `transaction_read_only` instead.

*   The `sync_frm` system variable has been removed because `.frm` files have become obsolete.

*   The `secure_auth` system variable and `--secure-auth` client option have been removed. The `MYSQL_SECURE_AUTH` option for the `mysql_options()` C API function was removed.

*   The `multi_range_count` system variable is removed.

*   The `log_warnings` system variable and `--log-warnings` server option have been removed. Use the `log_error_verbosity` system variable instead.

*   The global scope for the `sql_log_bin` system variable was removed. `sql_log_bin` has session scope only, and applications that rely on accessing `@@GLOBAL.sql_log_bin` should be adjusted.

*   The `metadata_locks_cache_size` and `metadata_locks_hash_instances` system variables are removed.

*   The unused `date_format`, `datetime_format`, `time_format`, and `max_tmp_tables` system variables are removed.

*   These deprecated compatibility SQL modes are removed: `DB2`, `MAXDB`, `MSSQL`, `MYSQL323`, `MYSQL40`, `ORACLE`, `POSTGRESQL`, `NO_FIELD_OPTIONS`, `NO_KEY_OPTIONS`, `NO_TABLE_OPTIONS`. They can no longer be assigned to the `sql_mode` system variable or used as permitted values for the **mysqldump** `--compatible` option.

    Removal of `MAXDB` means that the `TIMESTAMP` data type for `CREATE TABLE` or `ALTER TABLE` is treated as `TIMESTAMP`, and is no longer treated as `DATETIME`.

*   The deprecated `ASC` or `DESC` qualifiers for `GROUP BY` clauses are removed. Queries that previously relied on `GROUP BY` sorting may produce results that differ from previous MySQL versions. To produce a given sort order, provide an `ORDER BY` clause.

*   The `EXTENDED` and `PARTITIONS` keywords for the `EXPLAIN` statement have been removed. These keywords are unnecessary because their effect is always enabled.

*   These encryption-related items are removed:

    *   The `ENCODE()` and `DECODE()` functions.

    *   The `ENCRYPT()` function.

    *   The `DES_ENCRYPT()`, and `DES_DECRYPT()` functions, the `--des-key-file` option, the `have_crypt` system variable, the `DES_KEY_FILE` option for the `FLUSH` statement, and the `HAVE_CRYPT` **CMake** option.

    In place of the removed encryption functions: For `ENCRYPT()`, consider using `SHA2()` instead for one-way hashing. For the others, consider using `AES_ENCRYPT()` and `AES_DECRYPT()` instead.

*   In MySQL 5.7, several spatial functions available under multiple names were deprecated to move in the direction of making the spatial function namespace more consistent, the goal being that each spatial function name begin with `ST_` if it performs an exact operation, or with `MBR` if it performs an operation based on minimum bounding rectangles. In MySQL 8.0, the deprecated functions are removed to leave only the corresponding `ST_` and `MBR` functions:

    *   These functions are removed in favor of the `MBR` names: `Contains()`, `Disjoint()`, `Equals()`, `Intersects()`, `Overlaps()`, `Within()`.

    *   These functions are removed in favor of the `ST_` names: `Area()`, `AsBinary()`, `AsText()`, `AsWKB()`, `AsWKT()`, `Buffer()`, `Centroid()`, `ConvexHull()`, `Crosses()`, `Dimension()`, `Distance()`, `EndPoint()`, `Envelope()`, `ExteriorRing()`, `GeomCollFromText()`, `GeomCollFromWKB()`, `GeomFromText()`, `GeomFromWKB()`, `GeometryCollectionFromText()`, `GeometryCollectionFromWKB()`, `GeometryFromText()`, `GeometryFromWKB()`, `GeometryN()`, `GeometryType()`, `InteriorRingN()`, `IsClosed()`, `IsEmpty()`, `IsSimple()`, `LineFromText()`, `LineFromWKB()`, `LineStringFromText()`, `LineStringFromWKB()`, `MLineFromText()`, `MLineFromWKB()`, `MPointFromText()`, `MPointFromWKB()`, `MPolyFromText()`, `MPolyFromWKB()`, `MultiLineStringFromText()`, `MultiLineStringFromWKB()`, `MultiPointFromText()`, `MultiPointFromWKB()`, `MultiPolygonFromText()`, `MultiPolygonFromWKB()`, `NumGeometries()`, `NumInteriorRings()`, `NumPoints()`, `PointFromText()`, `PointFromWKB()`, `PointN()`, `PolyFromText()`, `PolyFromWKB()`, `PolygonFromText()`, `PolygonFromWKB()`, `SRID()`, `StartPoint()`, `Touches()`, `X()`, `Y()`.

    *   `GLength()` is removed in favor of `ST_Length()`.

*   The functions described in Section 14.16.4, “Functions That Create Geometry Values from WKB Values” previously accepted either WKB strings or geometry arguments. Geometry arguments are no longer permitted and produce an error. See that section for guidelines for migrating queries away from using geometry arguments.

*   The parser no longer treats `\N` as a synonym for `NULL` in SQL statements. Use `NULL` instead.

    This change does not affect text file import or export operations performed with `LOAD DATA` or `SELECT ... INTO OUTFILE`, for which `NULL` continues to be represented by `\N`. See Section 15.2.9, “LOAD DATA Statement”.

*   `PROCEDURE ANALYSE()` syntax is removed.

*   The client-side `--ssl` and `--ssl-verify-server-cert` options have been removed. Use `--ssl-mode=REQUIRED` instead of `--ssl=1` or `--enable-ssl`. Use `--ssl-mode=DISABLED` instead of `--ssl=0`, `--skip-ssl`, or `--disable-ssl`. Use `--ssl-mode=VERIFY_IDENTITY` instead of `--ssl-verify-server-cert` options. (The server-side `--ssl` option is still available, but is deprecated as of MySQL 8.0.26 and subject to removal in a future MySQL version.)

    For the C API, `MYSQL_OPT_SSL_ENFORCE` and `MYSQL_OPT_SSL_VERIFY_SERVER_CERT` options for `mysql_options()` correspond to the client-side `--ssl` and `--ssl-verify-server-cert` options and are removed. Use `MYSQL_OPT_SSL_MODE` with an option value of `SSL_MODE_REQUIRED` or `SSL_MODE_VERIFY_IDENTITY` instead.

*   The `--temp-pool` server option was removed.

*   The `ignore_builtin_innodb` system variable is removed.

*   The server no longer performs conversion of pre-MySQL 5.1 database names containing special characters to 5.1 format with the addition of a `#mysql50#` prefix. Because these conversions are no longer performed, the `--fix-db-names` and `--fix-table-names` options for **mysqlcheck**, the `UPGRADE DATA DIRECTORY NAME` clause for the `ALTER DATABASE` statement, and the `Com_alter_db_upgrade` status variable are removed.

    Upgrades are supported only from one major version to another (for example, 5.0 to 5.1, or 5.1 to 5.5), so there should be little remaining need for conversion of older 5.0 database names to current versions of MySQL. As a workaround, upgrade a MySQL 5.0 installation to MySQL 5.1 before upgrading to a more recent release.

*   The **mysql_install_db** program has been removed from MySQL distributions. Data directory initialization should be performed by invoking **mysqld** with the `--initialize` or `--initialize-insecure` option instead. In addition, the `--bootstrap` option for **mysqld** that was used by **mysql_install_db** was removed, and the `INSTALL_SCRIPTDIR` `CMake` option that controlled the installation location for **mysql_install_db** was removed.

*   The generic partitioning handler was removed from the MySQL server. In order to support partitioning of a given table, the storage engine used for the table must now provide its own (“native”) partitioning handler. The `--partition` and `--skip-partition` options are removed from the MySQL Server, and partitioning-related entries are no longer shown in the output of `SHOW PLUGINS` or in the Information Schema `PLUGINS` table.

    Two MySQL storage engines currently provide native partitioning support: `InnoDB` and `NDB`. Of these, only `InnoDB` is supported in MySQL 8.0\. Any attempt to create partitioned tables in MySQL 8.0 using any other storage engine fails.

    **Ramifications for upgrades. ** The direct upgrade of a partitioned table using a storage engine other than `InnoDB` (such as `MyISAM`) from MySQL 5.7 (or earlier) to MySQL 8.0 is not supported. There are two options for handling such a table:

    *   Remove the table's partitioning, using `ALTER TABLE ... REMOVE PARTITIONING`.

    *   Change the storage engine used for the table to `InnoDB`, with `ALTER TABLE ... ENGINE=INNODB`.

    At least one of the two operations just listed must be performed for each partitioned non-`InnoDB` table prior to upgrading the server to MySQL 8.0\. Otherwise, such a table cannot be used following the upgrade.

    Due to the fact that table creation statements that would result in a partitioned table using a storage engine without partitioning support now fail with an error (ER_CHECK_NOT_IMPLEMENTED), you must make sure that any statements in a dump file (such as that written by **mysqldump**) from an older version of MySQL that you wish to import into a MySQL 8.0 server that create partitioned tables do not also specify a storage engine such as `MyISAM` that has no native partitioning handler. You can do this by performing either of the following:

    *   Remove any references to partitioning from `CREATE TABLE` statements that use a value for the `STORAGE ENGINE` option other than `InnoDB`.

    *   Specifying the storage engine as `InnoDB`, or allow `InnoDB` to be used as the table's storage engine by default.

    For more information, see Section 26.6.2, “Partitioning Limitations Relating to Storage Engines”.

*   System and status variable information is no longer maintained in the `INFORMATION_SCHEMA`. These tables are removed: `GLOBAL_VARIABLES`, `SESSION_VARIABLES`, `GLOBAL_STATUS`, `SESSION_STATUS`. Use the corresponding Performance Schema tables instead. See Section 29.12.14, “Performance Schema System Variable Tables”, and Section 29.12.15, “Performance Schema Status Variable Tables”. In addition, the `show_compatibility_56` system variable was removed. It was used in the transition period during which system and status variable information in `INFORMATION_SCHEMA` tables was moved to Performance Schema tables, and is no longer needed. These status variables are removed: `Slave_heartbeat_period`, `Slave_last_heartbeat`, `Slave_received_heartbeats`, `Slave_retried_transactions`, `Slave_running`. The information they provided is available in Performance Schema tables; see Migrating to Performance Schema System and Status Variable Tables.

*   The Performance Schema `setup_timers` table was removed, as was the `TICK` row in the `performance_timers` table.

*   The `libmysqld` embedded server library is removed, along with:

    *   The `mysql_options()` `MYSQL_OPT_GUESS_CONNECTION`, `MYSQL_OPT_USE_EMBEDDED_CONNECTION`, `MYSQL_OPT_USE_REMOTE_CONNECTION`, and `MYSQL_SET_CLIENT_IP` options

    *   The **mysql_config** `--libmysqld-libs`, `--embedded-libs`, and `--embedded` options

    *   The **CMake** `WITH_EMBEDDED_SERVER`, `WITH_EMBEDDED_SHARED_LIBRARY`, and `INSTALL_SECURE_FILE_PRIV_EMBEDDEDDIR` options

    *   The (undocumented) **mysql** `--server-arg` option

    *   The **mysqltest** `--embedded-server`, `--server-arg`, and `--server-file` options

    *   The **mysqltest_embedded** and **mysql_client_test_embedded** test programs

*   The **mysql_plugin** utility was removed. Alternatives include loading plugins at server startup using the `--plugin-load` or `--plugin-load-add` option, or at runtime using the `INSTALL PLUGIN` statement.

*   The **resolveip** utility is removed. **nslookup**, **host**, or **dig** can be used instead.

*   The **resolve_stack_dump** utility is removed. Stack traces from official MySQL builds are always symbolized, so there is no need to use **resolve_stack_dump**.

*   The following server error codes are not used and have been removed. Applications that test specifically for any of these errors should be updated.

    ```

    ER_BINLOG_READ_EVENT_CHECKSUM_FAILURE

    ER_BINLOG_ROW_RBR_TO_SBR

    ER_BINLOG_ROW_WRONG_TABLE_DEF

    ER_CANT_ACTIVATE_LOG

    ER_CANT_CHANGE_GTID_NEXT_IN_TRANSACTION

    ER_CANT_CREATE_FEDERATED_TABLE

    ER_CANT_CREATE_SROUTINE

    ER_CANT_DELETE_FILE

    ER_CANT_GET_WD

    ER_CANT_SET_GTID_PURGED_WHEN_GTID_MODE_IS_OFF

    ER_CANT_SET_WD

    ER_CANT_WRITE_LOCK_LOG_TABLE

    ER_CREATE_DB_WITH_READ_LOCK

    ER_CYCLIC_REFERENCE

    ER_DB_DROP_DELETE

    ER_DELAYED_NOT_SUPPORTED

    ER_DIFF_GROUPS_PROC

    ER_DISK_FULL

    ER_DROP_DB_WITH_READ_LOCK

    ER_DROP_USER

    ER_DUMP_NOT_IMPLEMENTED

    ER_ERROR_DURING_CHECKPOINT

    ER_ERROR_ON_CLOSE

    ER_EVENTS_DB_ERROR

    ER_EVENT_CANNOT_DELETE

    ER_EVENT_CANT_ALTER

    ER_EVENT_COMPILE_ERROR

    ER_EVENT_DATA_TOO_LONG

    ER_EVENT_DROP_FAILED

    ER_EVENT_MODIFY_QUEUE_ERROR

    ER_EVENT_NEITHER_M_EXPR_NOR_M_AT

    ER_EVENT_OPEN_TABLE_FAILED

    ER_EVENT_STORE_FAILED

    ER_EXEC_STMT_WITH_OPEN_CURSOR

    ER_FAILED_ROUTINE_BREAK_BINLOG

    ER_FLUSH_MASTER_BINLOG_CLOSED

    ER_FORM_NOT_FOUND

    ER_FOUND_GTID_EVENT_WHEN_GTID_MODE_IS_OFF__UNUSED

    ER_FRM_UNKNOWN_TYPE

    ER_GOT_SIGNAL

    ER_GRANT_PLUGIN_USER_EXISTS

    ER_GTID_MODE_REQUIRES_BINLOG

    ER_GTID_NEXT_IS_NOT_IN_GTID_NEXT_LIST

    ER_HASHCHK

    ER_INDEX_REBUILD

    ER_INNODB_NO_FT_USES_PARSER

    ER_LIST_OF_FIELDS_ONLY_IN_HASH_ERROR

    ER_LOAD_DATA_INVALID_COLUMN_UNUSED

    ER_LOGGING_PROHIBIT_CHANGING_OF

    ER_MALFORMED_DEFINER

    ER_MASTER_KEY_ROTATION_ERROR_BY_SE

    ER_NDB_CANT_SWITCH_BINLOG_FORMAT

    ER_NEVER_USED

    ER_NISAMCHK

    ER_NO_CONST_EXPR_IN_RANGE_OR_LIST_ERROR

    ER_NO_FILE_MAPPING

    ER_NO_GROUP_FOR_PROC

    ER_NO_RAID_COMPILED

    ER_NO_SUCH_KEY_VALUE

    ER_NO_SUCH_PARTITION__UNUSED

    ER_OBSOLETE_CANNOT_LOAD_FROM_TABLE

    ER_OBSOLETE_COL_COUNT_DOESNT_MATCH_CORRUPTED

    ER_ORDER_WITH_PROC

    ER_PARTITION_SUBPARTITION_ERROR

    ER_PARTITION_SUBPART_MIX_ERROR

    ER_PART_STATE_ERROR

    ER_PASSWD_LENGTH

    ER_QUERY_ON_MASTER

    ER_RBR_NOT_AVAILABLE

    ER_SKIPPING_LOGGED_TRANSACTION

    ER_SLAVE_CHANNEL_DELETE

    ER_SLAVE_MULTIPLE_CHANNELS_HOST_PORT

    ER_SLAVE_MUST_STOP

    ER_SLAVE_WAS_NOT_RUNNING

    ER_SLAVE_WAS_RUNNING

    ER_SP_GOTO_IN_HNDLR

    ER_SP_PROC_TABLE_CORRUPT

    ER_SQL_MODE_NO_EFFECT

    ER_SR_INVALID_CREATION_CTX

    ER_TABLE_NEEDS_UPG_PART

    ER_TOO_MUCH_AUTO_TIMESTAMP_COLS

    ER_UNEXPECTED_EOF

    ER_UNION_TABLES_IN_DIFFERENT_DIR

    ER_UNSUPPORTED_BY_REPLICATION_THREAD

    ER_UNUSED1

    ER_UNUSED2

    ER_UNUSED3

    ER_UNUSED4

    ER_UNUSED5

    ER_UNUSED6

    ER_VIEW_SELECT_DERIVED_UNUSED

    ER_WRONG_MAGIC

    ER_WSAS_FAILED

    ```

+   已弃用的`INFORMATION_SCHEMA` `INNODB_LOCKS`和`INNODB_LOCK_WAITS`表已被移除。请改用性能模式`data_locks`和`data_lock_waits`表。

    Note

    在 MySQL 5.7 中，`INNODB_LOCKS`表中的`LOCK_TABLE`列和`sys`模式中的`locked_table`列的`innodb_lock_waits`和`x$innodb_lock_waits`视图包含组合的模式/表名值。在 MySQL 8.0 中，`data_locks`表和`sys`模式视图包含单独的模式名和表名列。参见第 30.4.3.9 节，“innodb_lock_waits 和 x$innodb_lock_waits 视图”。

+   `InnoDB`不再支持压缩临时表。当启用`innodb_strict_mode`（默认情况下）时，如果指定`ROW_FORMAT=COMPRESSED`或`KEY_BLOCK_SIZE`，`CREATE TEMPORARY TABLE`将返回错误。如果禁用`innodb_strict_mode`，则会发出警告并使用非压缩行格式创建临时表。

+   当在 MySQL 数据目录之外创建表空间数据文件时，`InnoDB`不再创建`.isl`文件（`InnoDB`符号链接文件）。现在`innodb_directories`选项支持定位在数据目录之外创建的表空间文件。

    通过这个改变，在服务器离线时通过手动修改`.isl`文件来移动远程表空间不再受支持。现在通过`innodb_directories`选项支持移动远程表空间文件。参见第 17.6.3.6 节，“服务器离线时移动表空间文件”。

+   已移除以下`InnoDB`文件格式变量：

    +   `innodb_file_format`

    +   `innodb_file_format_check`

    +   `innodb_file_format_max`

    +   `innodb_large_prefix`

    在 MySQL 5.1 中，文件格式变量对于创建与早期版本的`InnoDB`兼容的表是必要的。现在 MySQL 5.1 已经到达产品生命周期的尽头，这些选项不再需要。

    `FILE_FORMAT`列已从`INNODB_TABLES`和`INNODB_TABLESPACES`信息模式表中移除。

+   已移除`innodb_support_xa`系统变量，该变量启用 XA 事务中的两阶段提交支持。`InnoDB`对 XA 事务中的两阶段提交始终启用。

+   不再支持 DTrace。

+   `JSON_APPEND()` 函数已被移除。请改用 `JSON_ARRAY_APPEND()`。

+   MySQL 8.0.13 版本中移除了将表分区放置在共享 `InnoDB` 表空间中的支持。共享表空间包括 `InnoDB` 系统表空间和通用表空间。有关识别共享表空间中的分区并将其移动到每个表的文件表空间的信息，请参见 Section 3.6, “Preparing Your Installation for Upgrade”。

+   MySQL 8.0.13 版本中弃用了在除 `SET` 之外的语句中设置用户变量的功能。此功能可能在 MySQL 8.4 中移除。

+   `--ndb` 选项已被移除。请改用 **ndb_perror** 实用程序。

+   `innodb_undo_logs` 变量已被移除。`innodb_rollback_segments` 变量执行相同功能，应该代替使用。

+   `Innodb_available_undo_logs` 状态变量已被移除。每个表空间可用的回滚段数量可以通过 `SHOW VARIABLES LIKE 'innodb_rollback_segments';` 获取。

+   截至 MySQL 8.0.14 版本，先前弃用的 `innodb_undo_tablespaces` 变量不再可配置。更多信息，请参见 Section 17.6.3.4, “Undo Tablespaces”。

+   已移除对 `ALTER TABLE ... UPGRADE PARTITIONING` 语句的支持。

+   截至 MySQL 8.0.16 版本，已移除对 `internal_tmp_disk_storage_engine` 系统变量的支持；磁盘上的内部临时表现在始终使用 `InnoDB` 存储引擎。更多信息，请参见 Storage Engine for On-Disk Internal Temporary Tables。

+   未使用的 `DISABLE_SHARED` **CMake** 选项已被移除。

+   `myisam_repair_threads` 系统变量已在 MySQL 8.0.30 版本中移除。
