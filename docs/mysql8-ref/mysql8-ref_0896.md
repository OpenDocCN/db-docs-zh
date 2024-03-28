# 15.1.10 修改表空间语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-tablespace.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-tablespace.html)

```sql
ALTER [UNDO] TABLESPACE *tablespace_name*
  *NDB only:*
    {ADD | DROP} DATAFILE '*file_name*'
    [INITIAL_SIZE [=] size]
    [WAIT]
  *InnoDB and NDB:*
    [RENAME TO *tablespace_name*]
  *InnoDB only:*
    [AUTOEXTEND_SIZE [=] '*value*']
    [SET {ACTIVE | INACTIVE}]
    [ENCRYPTION [=] {'Y' | 'N'}]
  *InnoDB and NDB:*
    [ENGINE [=] *engine_name*]
  *Reserved for future use:*
    [ENGINE_ATTRIBUTE [=] '*string*']
```

此语句用于 `NDB` 和 `InnoDB` 表空间。它可用于向 `NDB` 表空间添加新数据文件，或从中删除数据文件。还可用于重命名 NDB 集群磁盘数据表空间、重命名 `InnoDB` 通用表空间、加密 `InnoDB` 通用表空间，或将 `InnoDB` 撤销表空间标记为活动或非活动。

`UNDO` 关键字，引入于 MySQL 8.0.14 版本，与 `SET {ACTIVE | INACTIVE}` 子句一起用于将 `InnoDB` 撤销表空间标记为活动或非活动。有关更多信息，请参阅 第 17.6.3.4 节，“撤销表空间”。

`ADD DATAFILE` 变体允许您使用 `INITIAL_SIZE` 子句指定 `NDB` 磁盘数据表空间的初始大小，其中 *`size`* 以字节为单位；默认值为 134217728（128 MB）。您可以选择性地在 *`size`* 后面跟上一个表示数量级的单个字母缩写，类似于 `my.cnf` 中使用的那些字母。通常，这些字母之一是 `M`（兆字节）或 `G`（千兆字节）。

在 32 位系统上，`INITIAL_SIZE` 的最大支持值为 4294967296（4 GB）。 (Bug #29186)

`INITIAL_SIZE` 被明确地四舍五入，就像 `CREATE TABLESPACE` 中一样。

一旦创建了数据文件，其大小就无法更改；但是，您可以使用额外的 `ALTER TABLESPACE ... ADD DATAFILE` 语句向 NDB 表空间添加更多数据文件。

当 `ALTER TABLESPACE ... ADD DATAFILE` 与 `ENGINE = NDB` 一起使用时，在每个集群数据节点上创建一个数据文件，但在信息模式 `FILES` 表中只生成一行。有关更多信息，请参阅此表的描述，以及 第 25.6.11.1 节，“NDB 集群磁盘数据对象”。`ADD DATAFILE` 不支持 `InnoDB` 表空间。

使用 `DROP DATAFILE` 与 `ALTER TABLESPACE` 一起，从 NDB 表空间中删除数据文件 '*`file_name`*'。您不能从任何表正在使用的表空间中删除数据文件；换句话说，数据文件必须为空（未使用任何范围）。请参阅 第 25.6.11.1 节，“NDB 集群磁盘数据对象”。此外，要删除的任何数据文件必须先使用 `CREATE TABLESPACE` 或 `ALTER TABLESPACE` 添加到表空间中。`DROP DATAFILE` 不支持 `InnoDB` 表空间。

`WAIT`会被解析但会被忽略。这是为了未来的扩展。

`ENGINE`子句，用于指定表空间使用的存储引擎，已被弃用；预计在未来的版本中将被移除。表空间存储引擎由数据字典管理，使`ENGINE`子句过时。如果指定了存储引擎，它必须与数据字典中定义的表空间存储引擎匹配。与`NDB`表空间兼容的唯一值为`NDB`和`NDBCLUSTER`。

`RENAME TO`操作会隐式在`autocommit`模式下执行，不受`autocommit`设置的影响。

当`LOCK TABLES`或`FLUSH TABLES WITH READ LOCK`对位于表空间中的表生效时，无法执行`RENAME TO`操作。

在重命名表空间时，位于通用表空间中的表会获取独占的元数据锁，这会阻止并发的 DDL。并发的 DML 是支持的。

重命名`InnoDB`通用表空间需要`CREATE TABLESPACE`权限。

`AUTOEXTEND_SIZE`选项定义了当表空间满时`InnoDB`扩展表空间的量。在 MySQL 8.0.23 中引入。设置必须是 4MB 的倍数。默认设置为 0，这会导致表空间根据隐式默认行为进行扩展。更多信息，请参见 Section 17.6.3.9, “Tablespace AUTOEXTEND_SIZE Configuration”。

`ENCRYPTION`子句用于为`InnoDB`通用表空间或`mysql`系统表空间启用或禁用页面级数据加密。通用表空间的加密支持在 MySQL 8.0.13 中引入。`mysql`系统表空间的加密支持在 MySQL 8.0.16 中引入。

在启用加密之前，必须安装和配置一个密钥环插件。

从 MySQL 8.0.16 开始，如果启用了`table_encryption_privilege_check`变量，则需要`TABLE_ENCRYPTION_ADMIN`权限才能更改具有与`default_table_encryption`设置不同的`ENCRYPTION`子句设置的通用表空间。

如果表空间中的任何表属于使用`DEFAULT ENCRYPTION='N'`定义的模式，则无法为通用表空间启用加密。同样，如果通用表空间中的任何表属于使用`DEFAULT ENCRYPTION='Y'`定义的模式，则无法禁用加密。`DEFAULT ENCRYPTION`模式选项是在 MySQL 8.0.16 中引入的。

如果在通用表空间上执行的`ALTER TABLESPACE`语句不包括`ENCRYPTION`子句，则表空间将保留其当前的加密状态，不受`default_table_encryption`设置的影响。

当通用表空间或`mysql`系统表空间被加密时，存储在表空间中的所有表都会被加密。同样，创建在加密表空间中的表也会被加密。

当更改通用表空间或`mysql`系统表空间的`ENCRYPTION`属性时，将使用`INPLACE`算法。`INPLACE`算法允许在表空间中存在的表上进行并发 DML。并发 DDL 将被阻止。

有关更多信息，请参见第 17.13 节，“InnoDB 数据静态加密”。

`ENGINE_ATTRIBUTE`选项（自 MySQL 8.0.21 起可用）用于指定主要存储引擎的表空间属性。该选项保留供将来使用。

允许的值是包含有效`JSON`文档的字符串文字或空字符串（''）。无效的`JSON`将被拒绝。

```sql
ALTER TABLESPACE ts1 ENGINE_ATTRIBUTE='{"*key*":"*value*"}';
```

`ENGINE_ATTRIBUTE`的值可以重复而不会出错。在这种情况下，将使用指定的最后一个值。

服务器不会检查`ENGINE_ATTRIBUTE`的值，也不会在更改表的存储引擎时清除这些值。

不允许更改 JSON 属性值的单个元素。您只能添加或替换属性。
