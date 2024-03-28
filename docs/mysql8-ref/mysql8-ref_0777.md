# 13.4.6 创建空间列

> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-spatial-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-spatial-columns.html)

MySQL 提供了一种标准方法来为几何类型创建空间列，例如，使用`CREATE TABLE`或`ALTER TABLE`。空间列支持`MyISAM`、`InnoDB`、`NDB`和`ARCHIVE`表。另请参阅有关在第 13.4.10 节，“创建空间索引”下的空间索引的注意事项。

具有空间数据类型的列可以具有 SRID 属性，以明确指示存储在列中的值的空间参考系统（SRS）。有关 SRID 受限列的影响，请参见第 13.4.1 节，“空间数据类型”。

+   使用`CREATE TABLE`语句创建具有空间列的表：

    ```sql
    CREATE TABLE geom (g GEOMETRY);
    ```

+   使用`ALTER TABLE`语句向现有表添加或删除空间列：

    ```sql
    ALTER TABLE geom ADD pt POINT;
    ALTER TABLE geom DROP pt;
    ```
