# 12.14.3 向 8 位字符集添加简单排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/adding-collation-simple-8bit.html`](https://dev.mysql.com/doc/refman/8.0/en/adding-collation-simple-8bit.html)

本节描述了如何通过在 MySQL 的`Index.xml`文件中编写与`<charset>`字符集描述相关联的`<collation>`元素来为 8 位字符集添加简单排序规则的过程。这里描述的过程不需要重新编译 MySQL。示例将一个名为`latin1_test_ci`的排序规则添加到`latin1`字符集中。

1.  选择一个排序规则 ID，如第 12.14.2 节“选择排序规则 ID”所示。以下步骤使用 ID 为 1024。

1.  修改`Index.xml`和`latin1.xml`配置文件。这些文件位于由`character_sets_dir`系统变量命名的目录中。您可以按照以下方式检查变量值，尽管在您的系统上路径名可能不同：

    ```sql
    mysql> SHOW VARIABLES LIKE 'character_sets_dir';
    +--------------------+-----------------------------------------+
    | Variable_name      | Value                                   |
    +--------------------+-----------------------------------------+
    | character_sets_dir | /user/local/mysql/share/mysql/charsets/ |
    +--------------------+-----------------------------------------+
    ```

1.  为排序规则选择一个名称，并在`Index.xml`文件中列出。找到正在添加排序规则的字符集的`<charset>`元素，并添加一个`<collation>`元素，指示排序规则名称和 ID，以将名称与 ID 关联起来。例如：

    ```sql
    <charset name="latin1">
      ...
      <collation name="latin1_test_ci" id="1024"/>
      ...
    </charset>
    ```

1.  在`latin1.xml`配置文件中，添加一个命名排序规则的`<collation>`元素，并包含一个`<map>`元素，为字符代码 0 到 255 定义一个字符代码到权重映射表。`<map>`元素中的每个值必须是十六进制格式的数字。

    ```sql
    <collation name="latin1_test_ci">
    <map>
     00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
     10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F
     20 21 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F
     30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F
     40 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F
     50 51 52 53 54 55 56 57 58 59 5A 5B 5C 5D 5E 5F
     60 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F
     50 51 52 53 54 55 56 57 58 59 5A 7B 7C 7D 7E 7F
     80 81 82 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F
     90 91 92 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F
     A0 A1 A2 A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF
     B0 B1 B2 B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF
     41 41 41 41 5B 5D 5B 43 45 45 45 45 49 49 49 49
     44 4E 4F 4F 4F 4F 5C D7 5C 55 55 55 59 59 DE DF
     41 41 41 41 5B 5D 5B 43 45 45 45 45 49 49 49 49
     44 4E 4F 4F 4F 4F 5C F7 5C 55 55 55 59 59 DE FF
    </map>
    </collation>
    ```

1.  重新启动服务器，并使用此语句验证排序规则是否存在：

    ```sql
    mysql> SHOW COLLATION WHERE Collation = 'latin1_test_ci';
    +----------------+---------+------+---------+----------+---------+
    | Collation      | Charset | Id   | Default | Compiled | Sortlen |
    +----------------+---------+------+---------+----------+---------+
    | latin1_test_ci | latin1  | 1024 |         |          |       1 |
    +----------------+---------+------+---------+----------+---------+
    ```
