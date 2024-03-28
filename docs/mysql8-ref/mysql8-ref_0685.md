# 12.5 配置应用程序字符集和校对规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-applications.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-applications.html)

对于使用默认 MySQL 字符集和校对规则（`utf8mb4`、`utf8mb4_0900_ai_ci`）存储数据的应用程序，不需要特殊配置。如果应用程序需要使用不同的字符集或校对规则进行数据存储，可以通过多种方式配置字符集信息：

+   为每个数据库指定字符设置。例如，使用一个数据库的应用程序可能使用`utf8mb4`的默认设置，而使用另一个数据库的应用程序可能使用`sjis`。

+   在服务器启动时指定字符设置。这将导致服务器对所有未做其他安排的应用程序使用给定设置。

+   如果从源代码构建 MySQL，请在配置时指定字符设置。这将导致服务器将给定设置用作所有应用程序的默认设置，而无需在服务器启动时指定它们。

当不同应用程序需要不同的字符设置时，每个数据库的技术提供了很大的灵活性。如果大多数或所有应用程序使用相同的字符集，则在服务器启动时或配置时指定字符设置可能是最方便的。

对于每个数据库或服务器启动技术，这些设置控制数据存储的字符集。应用程序还必须告诉服务器在客户端/服务器通信中使用哪种字符集，如下面的说明所述。

这里展示的示例假定在特定情境下使用`latin1`字符集和`latin1_swedish_ci`校对规则，作为`utf8mb4`和`utf8mb4_0900_ai_ci`默认设置的替代方案。

+   **为每个数据库指定字符设置。** 要创建一个数据库，使其表使用给定的默认字符集和校对规则进行数据存储，请使用类似于以下的`CREATE DATABASE`语句：

    ```sql
    CREATE DATABASE mydb
      CHARACTER SET latin1
      COLLATE latin1_swedish_ci;
    ```

    数据库中创建的表默认使用`latin1`和`latin1_swedish_ci`作为任何字符列的字符集。

    使用数据库的应用程序在每次连接时也应配置与服务器的连接。这可以通过连接后执行`SET NAMES 'latin1'`语句来完成。该语句可用于任何连接方法（**mysql**客户端、PHP 脚本等）。

    在某些情况下，可能可以通过其他方式配置连接以使用所需的字符集。例如，要使用**mysql**连接，可以指定`--default-character-set=latin1`命令行选项，以实现与`SET NAMES 'latin1'`相同的效果。

    有关配置客户端连接的更多信息，请参见第 12.4 节，“连接字符集和校对规则”。

    注意

    如果您使用`ALTER DATABASE`更改数据库默认字符集或校对规则，则必须删除并重新创建数据库中使用这些默认设置的现有存储过程，以便它们使用新的默认设置。（在存储过程中，如果未明确指定字符集或校对规则，则具有字符数据类型的变量将使用数据库默认设置。请参见第 15.1.17 节，“CREATE PROCEDURE 和 CREATE FUNCTION 语句”。）

+   **在服务器启动时指定字符设置。** 要在服务器启动时选择字符集和校对规则，请使用`--character-set-server`和`--collation-server`选项。例如，要在选项文件中指定这些选项，请包含以下行：

    ```sql
    [mysqld]
    character-set-server=latin1
    collation-server=latin1_swedish_ci
    ```

    这些设置适用于整个服务器，并作为任何应用程序创建的数据库以及在这些数据库中创建的表的默认设置。

    应用程序仍然需要在连接后使用`SET NAMES`或等效方法配置其连接，如前所述。您可能会尝试使用`--init_connect="SET NAMES 'latin1'"`选项启动服务器，以使`SET NAMES`自动为每个连接的客户端执行。但是，这可能会产生不一致的结果，因为具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的用户不会执行`init_connect`值。

+   **在 MySQL 配置时间指定字符设置。** 如果您从源代码配置和构建 MySQL，可以使用`DEFAULT_CHARSET`和`DEFAULT_COLLATION` **CMake**选项来选择字符集和校对规则：

    ```sql
    cmake . -DDEFAULT_CHARSET=latin1 \
      -DDEFAULT_COLLATION=latin1_swedish_ci
    ```

    结果服务器使用`latin1`和`latin1_swedish_ci`作为数据库和表以及客户端连接的默认设置。在服务器启动时不需要使用`--character-set-server`和`--collation-server`来指定这些默认设置。应用程序也无需在连接到服务器后使用`SET NAMES`或等效方法配置其连接。

无论您如何为应用程序配置 MySQL 字符集，您还必须考虑这些应用程序执行的环境。例如，如果您打算使用从编辑器中创建的文件中的 UTF-8 文本发送语句，您应该将文件的编辑环境设置为 UTF-8，以便文件编码正确，并且操作系统正确处理它。如果您在终端窗口中使用**mysql**客户端，窗口必须配置为使用 UTF-8，否则字符可能无法正确显示。对于在 Web 环境中执行的脚本，脚本必须正确处理字符编码以便与 MySQL 服务器交互，并且必须生成正确指示编码的页面，以便浏览器知道如何显示页面内容。例如，您可以在`<head>`元素中包含此`<meta>`标签：

```sql
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
```
