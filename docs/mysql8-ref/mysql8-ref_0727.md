> 原文：[`dev.mysql.com/doc/refman/8.0/en/ldml-collation-example.html`](https://dev.mysql.com/doc/refman/8.0/en/ldml-collation-example.html)

#### 12.14.4.1 使用 LDML 语法定义 UCA 整理

要为 Unicode 字符集添加 UCA 整理而无需重新编译 MySQL，请使用以下过程。如果您不熟悉用于描述整理排序特性的 LDML 规则，请参阅第 12.14.4.2 节，“MySQL 支持的 LDML 语法”。

该示例向`utf8mb4`字符集添加了一个名为`utf8mb4_phone_ci`的整理。该整理设计用于涉及用户发布其姓名和电话号码的 Web 应用程序的场景。电话号码可以以非常不同的格式给出：

```sql
+7-12345-67
+7-12-345-67
+7 12 345 67
+7 (12) 345 67
+71234567
```

处理这些类型值时引发的问题是，不同的允许格式使得搜索特定电话号码变得非常困难。解决方案是定义一个重新排序标点字符的新整理，使它们可忽略。

1.  选择一个整理 ID，如第 12.14.2 节，“选择整理 ID”所示。以下步骤使用 ID 为 1029。

1.  修改`Index.xml`配置文件。该文件位于由`character_sets_dir`系统变量命名的目录中。您可以按照以下方式检查变量值，尽管路径名可能在您的系统上有所不同：

    ```sql
    mysql> SHOW VARIABLES LIKE 'character_sets_dir';
    +--------------------+-----------------------------------------+
    | Variable_name      | Value                                   |
    +--------------------+-----------------------------------------+
    | character_sets_dir | /user/local/mysql/share/mysql/charsets/ |
    +--------------------+-----------------------------------------+
    ```

1.  为整理选择一个名称，并在`Index.xml`文件中列出它。此外，您需要提供整理排序规则。找到正在添加整理的字符集的`<charset>`元素，并添加一个指示整理名称和 ID 的`<collation>`元素，以将名称与 ID 关联起来。在`<collation>`元素内，提供包含排序规则的`<rules>`元素：

    ```sql
    <charset name="utf8mb4">
      ...
      <collation name="utf8mb4_phone_ci" id="1029">
        <rules>
          <reset>\u0000</reset>
          <i>\u0020</i> <!-- space -->
          <i>\u0028</i> <!-- left parenthesis -->
          <i>\u0029</i> <!-- right parenthesis -->
          <i>\u002B</i> <!-- plus -->
          <i>\u002D</i> <!-- hyphen -->
        </rules>
      </collation>
      ...
    </charset>
    ```

1.  如果您希望为其他 Unicode 字符集添加类似的整理，添加其他`<collation>`元素。例如，要定义`ucs2_phone_ci`，请向`<charset name="ucs2">`元素添加一个`<collation>`元素。请记住，每个整理必须有其自己的唯一 ID。

1.  重新启动服务器并使用此语句验证整理是否存在：

    ```sql
    mysql> SHOW COLLATION WHERE Collation = 'utf8mb4_phone_ci';
    +------------------+---------+------+---------+----------+---------+
    | Collation        | Charset | Id   | Default | Compiled | Sortlen |
    +------------------+---------+------+---------+----------+---------+
    | utf8mb4_phone_ci | utf8mb4 | 1029 |         |          |       8 |
    +------------------+---------+------+---------+----------+---------+
    ```

现在测试整理，确保它具有所需的属性。

创建一个包含一些使用新整理的示例电话号码的表：

```sql
mysql> CREATE TABLE phonebook (
         name VARCHAR(64),
         phone VARCHAR(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_phone_ci
       );
Query OK, 0 rows affected (0.09 sec)

mysql> INSERT INTO phonebook VALUES ('Svoj','+7 912 800 80 02');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO phonebook VALUES ('Hf','+7 (912) 800 80 04');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO phonebook VALUES ('Bar','+7-912-800-80-01');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO phonebook VALUES ('Ramil','(7912) 800 80 03');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO phonebook VALUES ('Sanja','+380 (912) 8008005');
Query OK, 1 row affected (0.00 sec)
```

运行一些查询，查看忽略的标点字符是否实际上被忽略以进行比较和排序：

```sql
mysql> SELECT * FROM phonebook ORDER BY phone;
+-------+--------------------+
| name  | phone              |
+-------+--------------------+
| Sanja | +380 (912) 8008005 |
| Bar   | +7-912-800-80-01   |
| Svoj  | +7 912 800 80 02   |
| Ramil | (7912) 800 80 03   |
| Hf    | +7 (912) 800 80 04 |
+-------+--------------------+
5 rows in set (0.00 sec)

mysql> SELECT * FROM phonebook WHERE phone='+7(912)800-80-01';
+------+------------------+
| name | phone            |
+------+------------------+
| Bar  | +7-912-800-80-01 |
+------+------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM phonebook WHERE phone='79128008001';
+------+------------------+
| name | phone            |
+------+------------------+
| Bar  | +7-912-800-80-01 |
+------+------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM phonebook WHERE phone='7 9 1 2 8 0 0 8 0 0 1';
+------+------------------+
| name | phone            |
+------+------------------+
| Bar  | +7-912-800-80-01 |
+------+------------------+
1 row in set (0.00 sec)
```
