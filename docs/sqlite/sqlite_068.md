# 1\. 使用方法

> 原文：[`sqlite.com/sqldiff.html`](https://sqlite.com/sqldiff.html)

`sqldiff.exe` 二进制文件是一个命令行实用程序，用于显示 SQLite 数据库之间的内容差异。例如用法：

```sql
sqldiff [options] database1.sqlite database2.sqlite

```

通常的输出是一个 SQL 脚本，用于将 database1.sqlite（"源"数据库）转换为 database2.sqlite（"目标"数据库）。可以使用命令行开关来改变这种行为：

**--changeset FILE**

不要将更改写入标准输出。而是将（二进制）更改集文件写入 FILE 中。可以使用会话扩展来解释 SQLite 中的更改集。

**--lib LIBRARY**

**-L LIBRARY**

在计算差异之前，加载共享库或 DLL 文件 LIBRARY 到 SQLite。这可用于添加模式所需的应用程序定义的排序序列。

**--primarykey**

使用模式定义的主键而不是 rowid 来配对源和目标数据库中的行。（参见下面的额外解释。）

**--schema**

仅显示模式中列名和表的差异，而不是表的内容

**--summary**

显示每个表中更改了多少行，但不显示实际的更改内容

**--table TABLE**

仅显示 TABLE 的内容差异，而不是整个数据库的差异

**--transaction**

将 SQL 输出包装在一个大的事务中

**--vtab**

增加对处理 FTS3、FTS5 和 rtree 虚拟表的支持。详情请参见下面的内容（#sqldiff_vtab）。

# 2\. 工作原理

`sqldiff.exe` 实用程序的工作原理是通过查找源和目标中逻辑上的“配对”行。默认行为是，如果它们位于同名表中并且具有相同的 rowid，或者在 WITHOUT ROWID 表的情况下具有相同的 PRIMARY KEY，则将两行视为配对。配对行内容的任何差异都会作为 UPDATE 输出。源数据库中无法配对的行会作为 DELETE 输出。目标数据库中无法配对的行会作为 INSERT 输出。

--primarykey 标志会稍微更改配对算法，以便始终使用架构声明的 PRIMARY KEY 进行配对，即使在具有 rowid 的表上也是如此。这通常是寻找差异的更好选择，但是在具有一个或多个主键列设置为 NULL 的行的情况下可能会导致遗漏差异。

# 3\. 限制

1.  `sqldiff.exe` 实用程序不会为以下任何内容计算变更集：无法访问行 id 的 rowid 表；或者没有显式主键的表。在使用--changeset 选项时，sqldiff 会将它们从比较中省略。此类表的示例包括：

    ```sql
    CREATE TABLE NilChangeset (
       -- inaccessible rowid due to hiding its aliases
       "rowid" TEXT,
       "oid" TEXT,
       "_rowid_" TEXT
    );

    ```

    和

    ```sql
    CREATE TABLE NilChangeset (
       -- no explicit primary key
       "authorId" TEXT,
       "bookId" TEXT
    );

    ```

    当`sqldiff` 仅用于比较此类表时，不会发生错误。然而，结果可能出乎意料。例如，此调用的效果：

    ```sql
    sqldiff --changeset CHANGESET_OUT --table NilChangeset db1.sdb db2.sdb

    ```

    将生成一个名为“CHANGESET_OUT”的空文件。有关详细信息，请参阅会话限制。

1.  `sqldiff.exe` 实用程序当前不会显示 TRIGGER 或 VIEW 的差异。

1.  `sqldiff` 实用程序未设计用于支持模式迁移，并且在列定义不同的情况下宽容度较大。通常，在内容比较进行之前仅比较列名及其顺序的类似表。

    然而，单表比较选项，使用命名为"sqlite_schema"的选项，可以用于显示或检测一对数据库之间的详细模式差异。在执行此操作时，输出不应直接用于修改数据库。

1.  默认情况下，不报告虚拟表模式或内容的差异。

    然而，如果 虚拟表 的实现在数据库中创建真实表（有时称为"影子"表）来存储其数据，那么 sqldiff.exe 确实会计算这些表之间的差异。如果生成的 SQL 脚本随后在与源数据库不*完全*相同的数据库上运行，可能会产生意外效果。对于 SQLite 的多个捆绑虚拟表（如 FTS3、FTS5、rtree 等），这些意外效果可能包括损坏虚拟表内容。

    如果将 --vtab 选项传递给 sqldiff.exe，则它将忽略属于 FTS3、FTS5 或 rtree 虚拟表的所有底层影子表，并直接包含虚拟表差异。
