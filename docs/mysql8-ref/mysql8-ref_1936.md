# 28.3.41 The INFORMATION_SCHEMA TABLESPACES_EXTENSIONS Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-tablespaces-extensions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tablespaces-extensions-table.html)

`TABLESPACES_EXTENSIONS` 表（自 MySQL 8.0.21 起可用）提供了关于为主要存储引擎定义的表空间属性的信息。

注意

`TABLESPACES_EXTENSIONS` 表保留供将来使用。

`TABLESPACES_EXTENSIONS` 表具有以下列：

+   `TABLESPACE_NAME`

    表空间的名称。

+   `ENGINE_ATTRIBUTE`

    为主要存储引擎定义的表空间属性。保留供将来使用。
