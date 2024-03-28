> 原文：[`dev.mysql.com/doc/refman/8.0/en/silent-column-changes.html`](https://dev.mysql.com/doc/refman/8.0/en/silent-column-changes.html)

#### 15.1.20.7 悄悄更改列规范

在某些情况下，MySQL 会悄悄地更改列规范，使其与 `CREATE TABLE` 或 `ALTER TABLE` 语句中给出的规范不同。这些更改可能是对数据类型的更改，对与数据类型相关联的属性的更改，或对索引规范的更改。

所有更改都受到 65,535 字节的内部行大小限制的影响，这可能导致某些数据类型更改尝试失败。参见 第 10.4.7 节，“表列数和行大小限制”。

+   `PRIMARY KEY` 的一部分列即使没有声明为 `NOT NULL` 也会被设置为 `NOT NULL`。

+   在创建表时，`ENUM` 和 `SET` 成员值的尾随空格会被自动删除。

+   MySQL 将其他 SQL 数据库供应商使用的某些数据类型映射到 MySQL 类型。参见 第 13.9 节，“使用其他数据库引擎的数据类型”。

+   如果您包含一个 `USING` 子句来指定对于给定存储引擎不允许的索引类型，但是有另一种可用的索引类型，该引擎可以在不影响查询结果的情况下使用该索引类型��

+   如果未启用严格的 SQL 模式，则长度大于 65535 的 `VARCHAR` 列会转换为 `TEXT`，长度大于 65535 的 `VARBINARY` 列会转换为 `BLOB`。否则，在这两种情况下都会发生错误。

+   为字符数据类型指定 `CHARACTER SET binary` 属性会导致列被创建为相应的二进制数据类型：`CHAR` 变为 `BINARY`，`VARCHAR` 变为 `VARBINARY`，`TEXT` 变为 `BLOB`。对于 `ENUM` 和 `SET` 数据类型，不会发生这种情况；它们会按照声明创建。假设您使用以下定义指定表：

    ```sql
    CREATE TABLE t
    (
      c1 VARCHAR(10) CHARACTER SET binary,
      c2 TEXT CHARACTER SET binary,
      c3 ENUM('a','b','c') CHARACTER SET binary
    );
    ```

    结果表的定义如下：

    ```sql
    CREATE TABLE t
    (
      c1 VARBINARY(10),
      c2 BLOB,
      c3 ENUM('a','b','c') CHARACTER SET binary
    );
    ```

若要查看 MySQL 是否使用了您指定之外的数据类型，请在创建或更改表后发出`DESCRIBE`或`SHOW CREATE TABLE`语句。

如果使用**myisampack**对表进行压缩，可能会发生某些其他数据类型的更改。请参阅第 18.2.3.3 节，“压缩表特性”。
