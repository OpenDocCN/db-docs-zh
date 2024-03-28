# 22.4.3 文档和集合

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-collections.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-documents-collections.html)

22.4.3.1 创建、列出和删除集合

22.4.3.2 使用集合

22.4.3.3 查找文档

22.4.3.4 修改文档

22.4.3.5 删除文档

22.4.3.6 创建和删除索引

当您将 MySQL 用作文档存储时，集合是您可以创建、列出和删除的模式内的容器。集合包含您可以添加、查找、更新和删除的 JSON 文档。

本节示例使用`world_x`模式中的`countryinfo`集合。有关设置`world_x`模式的说明，请参见第 22.4.2 节，“下载和导入 world_x 数据库”。

#### 文档

在 MySQL 中，文档被表示为 JSON 对象。在内部，它们以一种高效的二进制格式存储，可以实现快速查找和更新。

+   Python 的简单文档格式：

    ```sql
    {"field1": "value", "field2" : 10, "field 3": null}

    ```

一组文档由一组由逗号分隔并包含在`[`和`]`字符中的文档组成。

+   Python 的简单文档数组：

    ```sql
    [{"Name": "Aruba", "Code:": "ABW"}, {"Name": "Angola", "Code:": "AGO"}]

    ```

MySQL 支持 JSON 文档中以下 Python 值类型：

+   数字（整数和浮点数）

+   字符串

+   布尔值（False 和 True）

+   无

+   更多 JSON 值的数组

+   更多 JSON 值的嵌套（或嵌入）对象

#### 集合

集合是共享目的并可能共享一个或多个索引的文档的容器。每个集合都有一个唯一的名称，并存在于单个模式中。

术语模式等同于数据库，意味着一组数据库对象，而不是用于强制数据结构和约束的关系模式。模式不会对集合中的文档强制一致性。

在这个快速入门指南中：

+   基本对象包括：

    | 对象形式 | 描述 |
    | --- | --- |
    | `db` | `db` 是分配给当前活动模式的全局变量。当您想对模式运行操作时，例如检索集合，您可以使用`db`变量可用的方法。 |
    | `db.get_collections()` | db.get_collections() 返回模式中集合的列表。使用列表获取对集合对象的引用，对其进行迭代等。 |

+   由集合范围内的基本操作包括：

    | 操作形式 | 描述 |
    | --- | --- |
    | `db.*`name`*.add()` | add() 方法将一个或多个文档插入到指定集合中。 |
    | `db.*`name`*.find()` | find() 方法返回指定集合中的一些或所有文档。 |
    | `db.*`name`*.modify()` | modify() 方法更新指定集合中的文档。 |
    | `db.*`name`*.remove()` | remove() 方法从指定集合中删除一个或多个文档。 |

#### 相关信息

+   查看操作集合以获取一般概述。

+   CRUD EBNF 定义 提供了操作的完整列表。
