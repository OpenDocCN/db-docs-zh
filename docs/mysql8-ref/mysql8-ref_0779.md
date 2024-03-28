# 13.4.8 获取空间数据

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fetching-spatial-data.html`](https://dev.mysql.com/doc/refman/8.0/en/fetching-spatial-data.html)

存储在表中的几何值可以以内部格式获取。您还可以将它们转换为 WKT 或 WKB 格式。

+   获取内部格式的空间数据：

    使用内部格式获取几何值在表与表之间的转移中可能很有用：

    ```sql
    CREATE TABLE geom2 (g GEOMETRY) SELECT g FROM geom;
    ```

+   获取 WKT 格式的空间数据：

    `ST_AsText()` 函数将几何从内部格式转换为 WKT 字符串。

    ```sql
    SELECT ST_AsText(g) FROM geom;
    ```

+   获取 WKB 格式的空间数据：

    `ST_AsBinary()` 函数将几何从内部格式转换为包含 WKB 值的 `BLOB`。

    ```sql
    SELECT ST_AsBinary(g) FROM geom;
    ```
