# 13.4.7 填充空间列

> 原文：[`dev.mysql.com/doc/refman/8.0/en/populating-spatial-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/populating-spatial-columns.html)

在创建空间列之后，您可以用空间数据填充它们。

值应存储在内部几何格式中，但您可以将它们从 Well-Known Text（WKT）或 Well-Known Binary（WKB）格式转换为该格式。以下示例演示了如何通过将 WKT 值转换为内部几何格式将几何值插入表中：

+   直接在`INSERT`语句中执行转换：

    ```sql
    INSERT INTO geom VALUES (ST_GeomFromText('POINT(1 1)'));

    SET @g = 'POINT(1 1)';
    INSERT INTO geom VALUES (ST_GeomFromText(@g));
    ```

+   在`INSERT`之前执行转换：

    ```sql
    SET @g = ST_GeomFromText('POINT(1 1)');
    INSERT INTO geom VALUES (@g);
    ```

以下示例将更复杂的几何形状插入表中：

```sql
SET @g = 'LINESTRING(0 0,1 1,2 2)';
INSERT INTO geom VALUES (ST_GeomFromText(@g));

SET @g = 'POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5))';
INSERT INTO geom VALUES (ST_GeomFromText(@g));

SET @g =
'GEOMETRYCOLLECTION(POINT(1 1),LINESTRING(0 0,1 1,2 2,3 3,4 4))';
INSERT INTO geom VALUES (ST_GeomFromText(@g));
```

前面的示例使用`ST_GeomFromText()`创建几何值。您还可以使用特定类型的函数：

```sql
SET @g = 'POINT(1 1)';
INSERT INTO geom VALUES (ST_PointFromText(@g));

SET @g = 'LINESTRING(0 0,1 1,2 2)';
INSERT INTO geom VALUES (ST_LineStringFromText(@g));

SET @g = 'POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5))';
INSERT INTO geom VALUES (ST_PolygonFromText(@g));

SET @g =
'GEOMETRYCOLLECTION(POINT(1 1),LINESTRING(0 0,1 1,2 2,3 3,4 4))';
INSERT INTO geom VALUES (ST_GeomCollFromText(@g));
```

想要使用 WKB 表示几何值的客户端应用程序负责向服务器发送正确形成的 WKB 查询。有几种方法可以满足此要求。例如：

+   插入一个带有十六进制字面值语法的`POINT(1 1)`值：

    ```sql
    INSERT INTO geom VALUES
    (ST_GeomFromWKB(X'0101000000000000000000F03F000000000000F03F'));
    ```

+   ODBC 应用程序可以发送 WKB 表示，将其绑定到使用`BLOB`类型的参数的占位符：

    ```sql
    INSERT INTO geom VALUES (ST_GeomFromWKB(?))
    ```

    其他编程接口可能支持类似的占位符机制。

+   在 C 程序中，您可以使用`mysql_real_escape_string_quote()`转义二进制值，并将结果包含在发送到服务器的查询字符串中。请参见 mysql_real_escape_string_quote()。
