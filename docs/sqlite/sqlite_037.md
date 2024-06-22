# 1\. 概述

> 原文：[`sqlite.com/carray.html`](https://sqlite.com/carray.html)

Carray() 是一个表值函数，只有一列（名为 "value"），可以有零个或多个行。carray() 中每行的 "value" 来自应用程序通过参数绑定提供的 C 语言数组。这样，carray() 函数提供了一种便捷的机制来将 C 语言数组绑定到 SQL 查询中。

# 2\. 可用性

carray() 函数默认情况下未编译进 SQLite。它作为可加载扩展出现在 [ext/misc/carray.c](https://www.sqlite.org/src/file/ext/misc/carray.c) 源文件中。

carray() 函数首次出现在 SQLite 的 3.14 版本（2016-08-08）。sqlite3_carray_bind() 接口和 carray() 的单参数变体是在 SQLite 3.34.0 版本（2020-12-01）中添加的。能够绑定作为 BLOB 解释的 `struct iovec` 对象数组的功能是在 SQLite 3.41.0 版本（2023-02-21）中添加的。

# 3\. 详细信息

carray() 函数接受一、二或三个参数。

对于 carray() 的两个或三个参数版本，第一个参数是一个指向数组的指针。由于 SQL 中无法直接指定指针值，第一个参数必须是一个使用 "sqlite3_bind_pointer()" 接口绑定到指针值的参数，并使用 "carray" 类型的指针类型。第二个参数是数组中元素的数量。可选的第三个参数是确定 C 语言数组元素数据类型的字符串。第三个参数的允许值为：

1.  'int32'

1.  'int64'

1.  'double'

1.  'char*'

1.  'struct iovec'

默认数据类型为 'int32'。

用于 BLOB 数据的 'struct iovec' 类型是标准的 POSIX 数据结构，通常使用 "`#include <sys/uio.h>`" 声明。格式如下：

> ```sql
> struct iovec {
>   void  *iov_base; /* Starting address */
>   size_t iov_len;  /* Number of bytes to transfer */
> };
> 
> ```

## 3.1\. 单参数 CARRAY

carray() 的单参数形式需要一个特殊的 C 语言接口名为 "sqlite3_carray_bind()"，用于附加值：

> ```sql
>   int sqlite3_carray_bind(
>     sqlite3_stmt *pStmt,         /* Statement containing the CARRAY */
>     int idx,                     /* Parameter number for CARRAY argument */
>     void *aData,                 /* Data array */
>     int nData,                   /* Number of entries in the array */
>     int mFlags,                  /* Datatype flag */
>     void (*xDestroy)(void*)      /* Destructor for aData */
>   );
> 
> ```

sqlite3_carray_bind() 的 mFlags 参数必须是以下之一：

> ```sql
>   #define CARRAY_INT32   0
>   #define CARRAY_INT64   1
>   #define CARRAY_DOUBLE  2
>   #define CARRAY_TEXT    3
>   #define CARRAY_BLOB    4
> 
> ```

对于 mFlags 参数的高阶位现在必须全部为零，尽管它们可能在将来的增强中使用。指定数据类型的常量定义和 sqlite3_carray_bind() 函数的原型都在辅助头文件 [ext/misc/carray.h](https://www.sqlite.org/src/file/ext/misc/carray.h) 中提供。

sqlite3_carray_bind() 函数的 xDestroy 参数是指向一个释放输入数组的函数的指针。SQLite 在完成数据处理后会调用此函数。xDestroy 参数可以选择是在 "sqlite3.h" 中定义的以下常量之一：

+   SQLITE_STATIC → 这意味着调用 sqlite3_carray_bind()的应用程序将保持对数据数组的所有权，并且应用程序承诺 SQLite 在准备好的语句被 finalize 之后才会修改或释放数据。

+   SQLITE_TRANSIENT → 这个特殊值指示 SQLite 在 sqlite3_carray_bind()接口返回之前，将数据做成它自己的私有拷贝。

# 4\. 使用方法

函数 carray()可以在查询的 FROM 子句中使用。例如，使用从 C 语言数组地址$PTR 取得的 rowid 查询 OBJ 表中的两个条目。

```sql
SELECT obj.* FROM obj, carray($PTR, 10) AS x
 WHERE obj.rowid=x.value;

```

这个查询会得到相同的结果：

```sql
SELECT * FROM obj WHERE rowid IN carray($PTR, 10);

```
