> 原文：[`dev.mysql.com/doc/refman/8.0/en/structured-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/structured-system-variables.html)

#### 7.1.9.5 结构化系统变量

结构化变量在两个方面与常规系统变量不同：

+   其值是一个具有指定为密切相关的服务器参数的组件的结构。

+   可能会有几个给定类型的结构化变量实例。每个实例都有一个不同的名称，并引用服务器维护的不同资源。

MySQL 支持一种结构化变量类型，用于指定管理关键缓存操作的参数。关键缓存结构化变量具有以下组件：

+   `key_buffer_size`

+   `key_cache_block_size`

+   `key_cache_division_limit`

+   `key_cache_age_threshold`

本节描述了引用结构化变量的语法。关键缓存变量用于语法示例，但关于关键缓存如何运作的具体细节可以在其他地方找到，在第 10.10.2 节，“MyISAM 关键缓存”中。

要引用结构化变量实例的组件，可以使用*`instance_name.component_name`*格式的复合名称。例如：

```sql
hot_cache.key_buffer_size
hot_cache.key_cache_block_size
cold_cache.key_cache_block_size
```

对于每个结构化系统变量，名称为`default`的实例始终预定义。如果您引用结构化变量的组件而没有任何实例名称，则使用`default`实例。因此，`default.key_buffer_size`和`key_buffer_size`都引用同一系统变量。

结构化变量实例和组件遵循以下命名规则：

+   对于给定类型的结构化变量，每个实例的名称必须在该类型的变量中是唯一的。但是，实例名称在结构化变量类型之间不需要是唯一的。例如，每个结构化变量都有一个名为`default`的实例，因此`default`在变量类型之间不是唯一的。

+   每种结构化变量类型的组件名称必须在所有系统变量名称中是唯一的。如果不是这样（也就是说，如果两种不同类型的结构化变量可以共享组件成员名称），那么对于未由实例名称限定的成员名称的引用将不清楚使用哪个默认结构化变量。

+   如果结构化变量实例名称作为未引用的标识符不合法，则使用反引号引用它作为引用标识符。例如，`hot-cache`不合法，但``hot-cache``是。

+   `global`、`session`和`local`不是合法的实例名称。这避免了与诸如`@@GLOBAL.*`var_name`*`之类的表示非结构化系统变量的符号发生冲突。

目前，前两条规则不可能被违反，因为唯一的结构化变量类型是键缓存类型。如果将来创建其他类型的结构化变量，则这些规则可能变得更加重要。

除了一个例外情况外，在任何简单变量名可以出现的上下文中，您都可以使用复合名称引用结构化变量组件。例如，您可以使用命令行选项为结构化变量赋值：

```sql
$> mysqld --hot_cache.key_buffer_size=64K
```

在选项文件中，请使用以下语法：

```sql
[mysqld]
hot_cache.key_buffer_size=64K
```

如果您使用此选项启动服务器，则会创建一个名为`hot_cache`的键缓存，大小为 64KB，另外还会创建一个默认大小为 8MB 的默认键缓存。

假设您以以下方式启动服务器：

```sql
$> mysqld --key_buffer_size=256K \
         --extra_cache.key_buffer_size=128K \
         --extra_cache.key_cache_block_size=2048
```

在这种情况下，服务器将默认键缓存的大小设置为 256KB。（您也可以编写`--default.key_buffer_size=256K`。）此外，服务器还会创建一个名为`extra_cache`的第二个键缓存，其大小为 128KB，用于缓存表索引块的块缓冲区大小设置为 2048 字节。

以下示例启动具有 3:1:1 比例大小的三个不同键缓存的服务器：

```sql
$> mysqld --key_buffer_size=6M \
         --hot_cache.key_buffer_size=2M \
         --cold_cache.key_buffer_size=2M
```

结构化变量值也可以在运行时设置和检索。例如，要将名为`hot_cache`的键缓存设置为 10MB 的大小，请使用以下任一语句：

```sql
mysql> SET GLOBAL hot_cache.key_buffer_size = 10*1024*1024;
mysql> SET @@GLOBAL.hot_cache.key_buffer_size = 10*1024*1024;
```

要检索缓存大小，请执行以下操作：

```sql
mysql> SELECT @@GLOBAL.hot_cache.key_buffer_size;
```

但是，以下语句不起作用。该变量不被解释为复合名称，而是被解释为用于`LIKE`模式匹配操作的简单字符串：

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'hot_cache.key_buffer_size';
```

这是能够在简单变量名出现的任何地方使用结构化变量名的例外情况。
