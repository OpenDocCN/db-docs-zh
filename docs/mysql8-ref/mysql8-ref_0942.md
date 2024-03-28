# 15.2.6 IMPORT TABLE Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/import-table.html`](https://dev.mysql.com/doc/refman/8.0/en/import-table.html)

```sql
IMPORT TABLE FROM *sdi_file* [, *sdi_file*] ...
```

`IMPORT TABLE`语句根据`.sdi`（序列化字典信息）元数据文件中包含的信息导入`MyISAM`表。`IMPORT TABLE`需要`FILE`权限来读取`.sdi`和表内容文件，以及`CREATE`权限用于创建表。

表可以使用**mysqldump**从一个服务器导出以编写 SQL 语句文件，并使用**mysql**在另一个服务器上导入以处理转储文件。`IMPORT TABLE`提供了一个更快的替代方案，使用“原始”表文件。

在导入之前，提供表内容的文件必须放置在适当的模式目录中，而`.sdi`文件必须位于服务器可访问的目录中。例如，`.sdi`文件可以放置在由`secure_file_priv`系统变量命名的目录中，或者（如果`secure_file_priv`为空）放置在服务器数据目录下的一个目录中。

以下示例描述了如何从一个服务器的`hr`模式中导出名为`employees`和`managers`的`MyISAM`表，并将其导入到另一个服务器的`hr`模式中。该示例使用以下假设（要在自己的系统上执行类似操作，请根据需要修改路径名）：

+   对于导出服务器，*`export_basedir`*代表其基本目录，其数据目录为`*`export_basedir`*/data`。

+   对于导入服务器，*`import_basedir`*代表其基本目录，其数据目录为`*`import_basedir`*/data`。

+   表文件从导出服务器导出到`/tmp/export`目录，该目录是安全的（不可被其他用户访问）。

+   导入服务器使用`/tmp/mysql-files`作为其`secure_file_priv`系统变量命名的目录。

要从导出服务器导出表，请使用以下过程：

1.  通过执行此语句锁定表以确保一致的快照，以防在导出过程中对其进行修改：

    ```sql
    mysql> FLUSH TABLES hr.employees, hr.managers WITH READ LOCK;
    ```

    在锁定期间，表仍然可以使用，但仅用于读取访问。

1.  在文件系统级别，将`.sdi`和表内容文件从`hr`模式目录复制到安全导出目录：

    +   `.sdi`文件位于`hr`模式目录中，但可能与表名不完全相同。例如，`employees`和`managers`表的`.sdi`文件可能被命名为`employees_125.sdi`和`managers_238.sdi`。

    +   对于`MyISAM`表，内容文件是其`.MYD`数据文件和`.MYI`索引文件。

    鉴于这些文件名，复制命令如下：

    ```sql
    $> cd *export_basedir*/data/hr
    $> cp employees_125.sdi /tmp/export
    $> cp managers_238.sdi /tmp/export
    $> cp employees.{MYD,MYI} /tmp/export
    $> cp managers.{MYD,MYI} /tmp/export
    ```

1.  解锁表：

    ```sql
    mysql> UNLOCK TABLES;
    ```

要将表导入导入服务器，请使用以下过程：

1.  导入模式必须存在。如有必要，执行此语句以创建它：

    ```sql
    mysql> CREATE SCHEMA hr;
    ```

1.  在文件系统级别，将`.sdi`文件复制到导入服务器`secure_file_priv`目录`/tmp/mysql-files`。同时，将表内容文件复制到`hr`模式目录：

    ```sql
    $> cd /tmp/export
    $> cp employees_125.sdi /tmp/mysql-files
    $> cp managers_238.sdi /tmp/mysql-files
    $> cp employees.{MYD,MYI} *import_basedir*/data/hr
    $> cp managers.{MYD,MYI} *import_basedir*/data/hr
    ```

1.  通过执行一个命名了`.sdi`文件的`IMPORT TABLE`语句来导入表：

    ```sql
    mysql> IMPORT TABLE FROM
           '/tmp/mysql-files/employees.sdi',
           '/tmp/mysql-files/managers.sdi';
    ```

如果`secure_file_priv`系统变量为空，则`.sdi`文件不需要放在由该变量命名的导入服务器目录中；它可以放在服务器可访问的任何目录中，包括导入表的模式目录。但是，如果`.sdi`文件放在该目录中，它可能会被重写；导入操作为表创建一个新的`.sdi`文件，如果操作使用相同的文件名创建新文件，则会覆盖旧��`.sdi`文件。

每个*`sdi_file`*值必须是一个字符串文字，用于命名表的`.sdi`文件或与`.sdi`文件匹配的模式。如果字符串是一个模式，任何前导目录路径和`.sdi`文件名后缀必须以文字形式给出。模式字符仅允许在文件名的基本部分中：

+   `?`匹配任意单个字符

+   `*`匹配任意字符序列，包括无字符

使用模式，先前的`IMPORT TABLE`语句可以这样编写（假设`/tmp/mysql-files`目录不包含与模式匹配的其他`.sdi`文件）：

```sql
IMPORT TABLE FROM '/tmp/mysql-files/*.sdi';
```

要解释`.sdi`文件路径名的位置，服务器使用与`IMPORT TABLE`相同的规则，就像服务器端用于`LOAD DATA`的规则一样（即非`LOCAL`规则）。请参阅 Section 15.2.9, “LOAD DATA Statement”，特别注意用于解释相对路径名的规则。

如果无法定位`.sdi`或表文件，则`IMPORT TABLE`会失败。导入表后，服务器会尝试打开它，并报告检测到的任何问题作为警告。要尝试修复以纠正任何报告的问题，请使用`REPAIR TABLE`。

`IMPORT TABLE`不会写入二进制日志。

#### 限制和限制条件

`IMPORT TABLE`仅适用于非`TEMPORARY` `MyISAM`表。它不适用于使用事务存储引擎创建的表，使用`CREATE TEMPORARY TABLE`创建的表或视图。

在导入操作中使用的`.sdi`文件必须在具有与导入服务器相同的数据字典版本和 sdi 版本的服务器上生成。生成服务器的版本信息可以在`.sdi`文件中找到：

```sql
{
   "mysqld_version_id":80019,
   "dd_version":80017,
   "sdi_version":80016,
   ...
}
```

要确定导入服务器的数据字典和 sdi 版本，您可以检查导入服务器上最近创建的表的`.sdi`文件。

在执行`IMPORT TABLE`语句之前，必须将表数据和索引文件放入导入服务器的模式目录中，除非在导出服务器上定义的表使用`DATA DIRECTORY`或`INDEX DIRECTORY`表选项。在这种情况下，在执行`IMPORT TABLE`语句之前，使用这些替代方案修改导入过程：

+   将数据和索引文件放入导入服务器主机上与导出服务器主机相同的目录中，并在导入服务器模式目录中创建符号链接指向这些文件。

+   将数据和索引文件放入导入服务器主机目录，该目录与导出服务器主机上的目录不同，并在导入服务器模式目录中创建符号链接指向这些文件。此外，修改`.sdi`文件以反映不同的文件位置。

+   将数据和索引文件放入导入服务器主机上的模式目录，并修改`.sdi`文件以删除数据和索引目录表选项。

存储在`.sdi`文件中的任何排序 ID 必须引用导出和导入服务器上相同的排序规则。

表的触发器信息不会序列化到表的`.sdi`文件中，因此触发器不会被导入操作恢复。

在执行`IMPORT TABLE`语句之前，对`.sdi`文件进行一些编辑是允许的，而其他编辑可能会有问题，甚至可能导致导入操作失败：

+   如果数据和索引文件在导出和导入服务器之间的位置不同，则需要��改数据目录和索引目录表选项。

+   要将表导入到导入服务器上与导出服务器上不同的模式中，需要更改模式名称。

+   可能需要更改模式和表名称以适应导出和导入服务器上文件系统区分大小写语义或`lower_case_table_names`设置的差异。在`.sdi`文件中更改表名称可能需要同时重命名表文件。

+   在某些情况下，允许更改列定义。更改数据类型可能会导致问题。
