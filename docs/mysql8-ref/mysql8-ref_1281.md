> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-porting-mysql.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-porting-mysql.html)

#### 17.20.6.1 调整现有的 MySQL 模式以适应 InnoDB memcached 插件

在调整现有的 MySQL 模式或应用程序以使用`daemon_memcached`插件时，请考虑这些**memcached**应用程序的方面：

+   **memcached** 键不能包含空格或换行符，因为这些字符在 ASCII 协议中用作分隔符。如果您使用包含空格的查找值，请在将其用作`add()`、`set()`、`get()`等调用中的键之前将其转换或哈希为无空格的值。尽管在使用二进制协议的程序中理论上允许这些字符作为键，但为了确保与广泛范围的客户端兼容性，您应限制键中使用的字符。

+   如果在`InnoDB`表中有一个短的数字主键列，请将其转换为字符串值，以将其用作**memcached**的唯一查找键。如果**memcached**服务器用于多个应用程序，或与多个`InnoDB`表一起使用，请考虑修改名称以确保其唯一性。例如，在数字值之前添加表名，或数据库名和表名。

    注意

    `daemon_memcached`插件支持在具有`INTEGER`定义为主键的映射`InnoDB`表上进行插入和读取。

+   不能使用分区表来查询或存储使用**memcached**的数据。

+   **memcached** 协议将数字值作为字符串传递。为了在底层的`InnoDB`表中存储数字值，实现可以用于 SQL 函数如`SUM()`或`AVG()`的计数器，例如：

    +   使用足够字符的`VARCHAR`列来容纳预期最大数字的所有数字（如果适用，还有负号、小数点或两者的其他字符）。

    +   在执行使用列值进行算术运算的任何查询中，使用`CAST()`函数将值从字符串转换为整数，或转换为其他数值类型。例如：

        ```sql
        # Alphabetic entries are returned as zero.

        SELECT CAST(c2 as unsigned integer) FROM demo_test;

        # Since there could be numeric values of 0, can't disqualify them.
        # Test the string values to find the ones that are integers, and average only those.

        SELECT AVG(cast(c2 as unsigned integer)) FROM demo_test
          WHERE c2 BETWEEN '0' and '9999999999';

        # Views let you hide the complexity of queries. The results are already converted;
        # no need to repeat conversion functions and WHERE clauses each time.

        CREATE VIEW numbers AS SELECT c1 KEY, CAST(c2 AS UNSIGNED INTEGER) val
          FROM demo_test WHERE c2 BETWEEN '0' and '9999999999';
        SELECT SUM(val) FROM numbers;
        ```

        注意

        结果集中的任何字母值在调用`CAST()`时都会转换为 0。在使用诸如`AVG()`之类依赖于结果集中的行数的函数时，包括`WHERE`子句以过滤非数字值。

+   如果用作键的`InnoDB`列可能具有超过 250 字节的值，请将该值哈希为少于 250 字节。

+   要在`daemon_memcached`插件中使用现有表，需在`innodb_memcache.containers`表中为其定义一个条目。要使该表成为所有**memcached**请求的默认表，需在`name`列中指定一个值为`default`，然后重新启动 MySQL 服务器以使更改生效。如果您为不同类别的**memcached**数据使用多个表，请在`innodb_memcache.containers`表中设置多个条目，其中`name`值由您选择，然后在应用程序中发出形式为`get @@*`name`*`或`set @@*`name`*`的**memcached**请求，以指定用于后续**memcached**请求的表。

    要查看使用预定义的`test.demo_test`表之外的表的示例，请参阅示例 17.13，“使用自己的表与 InnoDB memcached 应用程序”。有关所需表布局，请参阅第 17.20.8 节，“InnoDB memcached 插件内部”。

+   要使用多个`InnoDB`表列值与**memcached**键值对，需在`innodb_memcache.containers`条目的`value_columns`字段中指定以逗号、分号、空格或竖线字符分隔的列名。例如，在`value_columns`字段中指定`col1,col2,col3`或`col1|col2|col3`。

    在将字符串传递给**memcached**的`add`或`set`调用之前，使用竖线字符作为分隔符将列值连接成单个字符串。字符串会自动解包为正确的列。每个`get`调用返回一个包含以竖线字符分隔的列值的单个字符串。您可以使用适当的应用程序语法解包这些值。

**示例 17.13 使用自己的表与 InnoDB memcached 应用程序**

该示例展示了如何使用自己的表与一个使用`memcached`进行数据操作的示例 Python 应用程序。

该示例假定`daemon_memcached`插件已按照第 17.20.3 节，“设置 InnoDB memcached 插件”中描述的方式安装。还假定您的系统已配置为运行使用`python-memcache`模块的 Python 脚本。

1.  创建`multicol`表，其中存储包括人口、面积和驾驶侧数据（右侧为`'R'`，左侧为`'L'`）的国家信息。

    ```sql
    mysql> USE test;

    mysql> CREATE TABLE `multicol` (
            `country` varchar(128) NOT NULL DEFAULT '',
            `population` varchar(10) DEFAULT NULL,
            `area_sq_km` varchar(9) DEFAULT NULL,
            `drive_side` varchar(1) DEFAULT NULL,
            `c3` int(11) DEFAULT NULL,
            `c4` bigint(20) unsigned DEFAULT NULL,
            `c5` int(11) DEFAULT NULL,
            PRIMARY KEY (`country`)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
    ```

1.  将记录插入`innodb_memcache.containers`表中，以便`daemon_memcached`插件可以访问`multicol`表。

    ```sql
    mysql> INSERT INTO innodb_memcache.containers
           (name,db_schema,db_table,key_columns,value_columns,flags,cas_column,
           expire_time_column,unique_idx_name_on_key)
           VALUES
           ('bbb','test','multicol','country','population,area_sq_km,drive_side',
           'c3','c4','c5','PRIMARY');

    mysql> COMMIT;
    ```

    +   `innodb_memcache.containers`记录中的`multicol`表指定了一个`name`值为`'bbb'`，这是表标识符。

        注意

        如果单个`InnoDB`表用于所有**memcached**应用程序，则可以将`name`值设置为`default`，以避免使用`@@`符号切换表。

    +   `db_schema`列设置为`test`，这是`multicol`表所在的数据库的名称。

    +   `db_table`列设置为`multicol`，这是`InnoDB`表的名称。

    +   `key_columns`设置为唯一的`country`列。`country`列在`multicol`表定义中被定义为主键。

    +   不是使用单个`InnoDB`表列来保存复合数据值，数据分布在三个表列（`population`，`area_sq_km`和`drive_side`）之间。为了容纳多个值列，`value_columns`字段中指定了一个逗号分隔的列列表。在存储或检索值时，`value_columns`字段中定义的列将被使用。

    +   `flags`，`expire_time`和`cas_column`字段的值基于`demo.test`示例表中使用的值。在使用`daemon_memcached`插件的应用程序中，这些字段通常不重要，因为 MySQL 会保持数据同步，不需要担心数据过期或变得陈旧。

    +   `unique_idx_name_on_key`字段设置为`PRIMARY`，指的是在`multicol`表中唯一的`country`列上定义的主索引。

1.  将示例 Python 应用程序复制到一个文件中。在本例中，示例脚本被复制到一个名为`multicol.py`的文件中。

    示例 Python 应用程序将数据插入`multicol`表，并检索所有键的数据，演示如何通过`daemon_memcached`插件访问`InnoDB`表。

    ```sql
    import sys, os
    import memcache

    def connect_to_memcached():
      memc = memcache.Client(['127.0.0.1:11211'], debug=0);
      print "Connected to memcached."
      return memc

    def banner(message):
      print
      print "=" * len(message)
      print message
      print "=" * len(message)

    country_data = [
    ("Canada","34820000","9984670","R"),
    ("USA","314242000","9826675","R"),
    ("Ireland","6399152","84421","L"),
    ("UK","62262000","243610","L"),
    ("Mexico","113910608","1972550","R"),
    ("Denmark","5543453","43094","R"),
    ("Norway","5002942","385252","R"),
    ("UAE","8264070","83600","R"),
    ("India","1210193422","3287263","L"),
    ("China","1347350000","9640821","R"),
    ]

    def switch_table(memc,table):
      key = "@@" + table
      print "Switching default table to '" + table + "' by issuing GET for '" + key + "'."
      result = memc.get(key)

    def insert_country_data(memc):
      banner("Inserting initial data via memcached interface")
      for item in country_data:
        country = item[0]
        population = item[1]
        area = item[2]
        drive_side = item[3]

        key = country
        value = "|".join([population,area,drive_side])
        print "Key = " + key
        print "Value = " + value

        if memc.add(key,value):
          print "Added new key, value pair."
        else:
          print "Updating value for existing key."
          memc.set(key,value)

    def query_country_data(memc):
      banner("Retrieving data for all keys (country names)")
      for item in country_data:
        key = item[0]
        result = memc.get(key)
        print "Here is the result retrieved from the database for key " + key + ":"
        print result
        (m_population, m_area, m_drive_side) = result.split("|")
        print "Unpacked population value: " + m_population
        print "Unpacked area value      : " + m_area
        print "Unpacked drive side value: " + m_drive_side

    if __name__ == '__main__':

      memc = connect_to_memcached()
      switch_table(memc,"bbb")
      insert_country_data(memc)
      query_country_data(memc)

      sys.exit(0)
    ```

    示例 Python 应用程序注意事项：

    +   运行应用程序不需要数据库授权，因为数据操作是通过**memcached**接口执行的。唯一需要的信息是**memcached**守护程序在本地系统上监听的端口号。

    +   为确保应用程序使用`multicol`表，调用`switch_table()`函数，该函数使用`@@`符号执行虚拟的`get`或`set`请求。请求中的`name`值为`bbb`，这是在`innodb_memcache.containers.name`字段中定义的`multicol`表标识符。

        在实际应用程序中可能会使用更具描述性的`name`值。这个例子只是说明了在`get @@...`请求中指定了一个表标识符，而不是表名。

    +   用于插入和查询数据的实用函数演示了如何将 Python 数据结构转换为用于通过`add`或`set`请求将数据发送到 MySQL 的管道分隔值，以及如何解压`get`请求返回的管道分隔值。只有在将单个**memcached**值映射到多个 MySQL 表列时才需要进行这种额外处理。

1.  运行示例 Python 应用程序。

    ```sql
    $> python multicol.py
    ```

    如果成功，示例应用程序将返回以下输出：

    ```sql
    Connected to memcached.
    Switching default table to 'bbb' by issuing GET for '@@bbb'.

    ==============================================
    Inserting initial data via memcached interface
    ==============================================
    Key = Canada
    Value = 34820000|9984670|R
    Added new key, value pair.
    Key = USA
    Value = 314242000|9826675|R
    Added new key, value pair.
    Key = Ireland
    Value = 6399152|84421|L
    Added new key, value pair.
    Key = UK
    Value = 62262000|243610|L
    Added new key, value pair.
    Key = Mexico
    Value = 113910608|1972550|R
    Added new key, value pair.
    Key = Denmark
    Value = 5543453|43094|R
    Added new key, value pair.
    Key = Norway
    Value = 5002942|385252|R
    Added new key, value pair.
    Key = UAE
    Value = 8264070|83600|R
    Added new key, value pair.
    Key = India
    Value = 1210193422|3287263|L
    Added new key, value pair.
    Key = China
    Value = 1347350000|9640821|R
    Added new key, value pair.

    ============================================
    Retrieving data for all keys (country names)
    ============================================
    Here is the result retrieved from the database for key Canada:
    34820000|9984670|R
    Unpacked population value: 34820000
    Unpacked area value      : 9984670
    Unpacked drive side value: R
    Here is the result retrieved from the database for key USA:
    314242000|9826675|R
    Unpacked population value: 314242000
    Unpacked area value      : 9826675
    Unpacked drive side value: R
    Here is the result retrieved from the database for key Ireland:
    6399152|84421|L
    Unpacked population value: 6399152
    Unpacked area value      : 84421
    Unpacked drive side value: L
    Here is the result retrieved from the database for key UK:
    62262000|243610|L
    Unpacked population value: 62262000
    Unpacked area value      : 243610
    Unpacked drive side value: L
    Here is the result retrieved from the database for key Mexico:
    113910608|1972550|R
    Unpacked population value: 113910608
    Unpacked area value      : 1972550
    Unpacked drive side value: R
    Here is the result retrieved from the database for key Denmark:
    5543453|43094|R
    Unpacked population value: 5543453
    Unpacked area value      : 43094
    Unpacked drive side value: R
    Here is the result retrieved from the database for key Norway:
    5002942|385252|R
    Unpacked population value: 5002942
    Unpacked area value      : 385252
    Unpacked drive side value: R
    Here is the result retrieved from the database for key UAE:
    8264070|83600|R
    Unpacked population value: 8264070
    Unpacked area value      : 83600
    Unpacked drive side value: R
    Here is the result retrieved from the database for key India:
    1210193422|3287263|L
    Unpacked population value: 1210193422
    Unpacked area value      : 3287263
    Unpacked drive side value: L
    Here is the result retrieved from the database for key China:
    1347350000|9640821|R
    Unpacked population value: 1347350000
    Unpacked area value      : 9640821
    Unpacked drive side value: R
    ```

1.  查询`innodb_memcache.containers`表以查看您之前为`multicol`表插入的记录。第一条记录是在初始`daemon_memcached`插件设置期间创建的`demo_test`表的示例条目。第二条记录是您为`multicol`表插入的条目。

    ```sql
    mysql> SELECT * FROM innodb_memcache.containers\G
    *************************** 1\. row ***************************
                      name: aaa
                 db_schema: test
                  db_table: demo_test
               key_columns: c1
             value_columns: c2
                     flags: c3
                cas_column: c4
        expire_time_column: c5
    unique_idx_name_on_key: PRIMARY
    *************************** 2\. row ***************************
                      name: bbb
                 db_schema: test
                  db_table: multicol
               key_columns: country
             value_columns: population,area_sq_km,drive_side
                     flags: c3
                cas_column: c4
        expire_time_column: c5
    unique_idx_name_on_key: PRIMARY
    ```

1.  查询`multicol`表以查看样本 Python 应用程序插入的数据。这些数据可供 MySQL 查询，演示了如何使用 SQL 或通过应用程序（使用适当的 MySQL 连接器或 API）访问相同的数据。

    ```sql
    mysql> SELECT * FROM test.multicol;
    +---------+------------+------------+------------+------+------+------+
    | country | population | area_sq_km | drive_side | c3   | c4   | c5   |
    +---------+------------+------------+------------+------+------+------+
    | Canada  | 34820000   | 9984670    | R          |    0 |   11 |    0 |
    | China   | 1347350000 | 9640821    | R          |    0 |   20 |    0 |
    | Denmark | 5543453    | 43094      | R          |    0 |   16 |    0 |
    | India   | 1210193422 | 3287263    | L          |    0 |   19 |    0 |
    | Ireland | 6399152    | 84421      | L          |    0 |   13 |    0 |
    | Mexico  | 113910608  | 1972550    | R          |    0 |   15 |    0 |
    | Norway  | 5002942    | 385252     | R          |    0 |   17 |    0 |
    | UAE     | 8264070    | 83600      | R          |    0 |   18 |    0 |
    | UK      | 62262000   | 243610     | L          |    0 |   14 |    0 |
    | USA     | 314242000  | 9826675    | R          |    0 |   12 |    0 |
    +---------+------------+------------+------------+------+------+------+
    ```

    注意

    在定义被视为数字的列的长度时，始终要允许足够的大小来容纳必要的数字、小数点、符号字符、前导零等。对于像`VARCHAR`这样的字符串列中的值过长，通过删除一些字符来截断，可能会产生荒谬的数值。

1.  可选地，在存储**memcached**数据的`InnoDB`表上运行报告类型的查询。

    您可以通过 SQL 查询生成报告，对任何列执行计算和测试，而不仅仅是`country`键列。 （因为以下示例仅使用了少数国家的数据，所以数字仅供说明目的。）以下查询返回驾驶方向为右侧的国家的平均人口和以“U”开头的国家的平均大小：

    ```sql
    mysql> SELECT AVG(population) FROM multicol WHERE drive_side = 'R';
    +-------------------+
    | avg(population)   |
    +-------------------+
    | 261304724.7142857 |
    +-------------------+

    mysql> SELECT SUM(area_sq_km) FROM multicol WHERE country LIKE 'U%';
    +-----------------+
    | sum(area_sq_km) |
    +-----------------+
    |        10153885 |
    +-----------------+
    ```

    因为`population`和`area_sq_km`列存储的是字符数据而不是强类型的数值数据，所以诸如`AVG()`和`SUM()`之类的函数会先将每个值转换为数字。这种方法*不适用*于诸如`<`或`>`之类的运算符，例如，当比较基于字符的值时，`9 > 1000`，这与`ORDER BY population DESC`这样的子句所期望的不符。为了获得最准确的类型处理，请对将数值列转换为适当类型的视图执行查询。这种技术使您可以从数据库应用程序发出简单的`SELECT *`查询，同时确保转换、过滤和排序是正确的。以下示例显示了一个视图，可以查询以按人口降序排列的前三个国家，结果反映了`multicol`表中的最新数据，并将人口和面积数字视为数字：

    ```sql
    mysql> CREATE VIEW populous_countries AS
           SELECT
           country,
           cast(population as unsigned integer) population,
           cast(area_sq_km as unsigned integer) area_sq_km,
           drive_side FROM multicol
           ORDER BY CAST(population as unsigned integer) DESC
           LIMIT 3;

    mysql> SELECT * FROM populous_countries;
    +---------+------------+------------+------------+
    | country | population | area_sq_km | drive_side |
    +---------+------------+------------+------------+
    | China   | 1347350000 |    9640821 | R          |
    | India   | 1210193422 |    3287263 | L          |
    | USA     |  314242000 |    9826675 | R          |
    +---------+------------+------------+------------+

    mysql> DESC populous_countries;
    +------------+---------------------+------+-----+---------+-------+
    | Field      | Type                | Null | Key | Default | Extra |
    +------------+---------------------+------+-----+---------+-------+
    | country    | varchar(128)        | NO   |     |         |       |
    | population | bigint(10) unsigned | YES  |     | NULL    |       |
    | area_sq_km | int(9) unsigned     | YES  |     | NULL    |       |
    | drive_side | varchar(1)          | YES  |     | NULL    |       |
    +------------+---------------------+------+-----+---------+-------+
    ```
