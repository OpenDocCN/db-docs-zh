> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-columns-list.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-columns-list.html)

#### 26.2.3.2 列表列分区

MySQL 8.0 支持`列表列`分区。这是`列表`分区的一种变体，允许将多个列用作分区键，并且可以使用数据类型为整数类型以外的列作为分区列；您可以使用字符串类型、`DATE`和`DATETIME`列。（有关`COLUMNS`分区列允许的数据类型的更多信息，请参见第 26.2.3 节，“列分区”。）

假设您有一个业务，客户分布在 12 个城市，为了销售和营销目的，您将这些城市组织成了每个包含 3 个城市的 4 个地区，如下表所示：

| 地区 | 城市 |
| --- | --- |
| 1 | 奥斯卡沙姆，赫格斯比，蒙斯特罗斯 |
| 2 | 温默比，胡尔特斯弗雷德，韦斯特维克 |
| 3 | 尼舍，艾克绍，维特兰达 |
| 4 | 乌普维丁厄，阿尔韦斯塔，韦克西厄 |

使用`列表列`分区，您可以创建一个客户数据表，根据客户所居住城市的名称将行分配给这些地区中的任何一个分区，如下所示：

```sql
CREATE TABLE customers_1 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS(city) (
    PARTITION pRegion_1 VALUES IN('Oskarshamn', 'Högsby', 'Mönsterås'),
    PARTITION pRegion_2 VALUES IN('Vimmerby', 'Hultsfred', 'Västervik'),
    PARTITION pRegion_3 VALUES IN('Nässjö', 'Eksjö', 'Vetlanda'),
    PARTITION pRegion_4 VALUES IN('Uppvidinge', 'Alvesta', 'Växjo')
);
```

与`范围列`分区一样，您不需要在`COLUMNS()`子句中使用表达式将列值转换为整数。（实际上，除了列名之外，不允许在`COLUMNS()`中使用其他表达式。）

也可以使用`DATE`和`DATETIME`列，如下例所示，使用与之前`customers_1`表相同的名称和列，但根据`renewal`列使用`列表列`分区，根据 2010 年 2 月的某周，将客户账户计划续订的情况存储在 4 个分区中的一个中：

```sql
CREATE TABLE customers_2 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS(renewal) (
    PARTITION pWeek_1 VALUES IN('2010-02-01', '2010-02-02', '2010-02-03',
        '2010-02-04', '2010-02-05', '2010-02-06', '2010-02-07'),
    PARTITION pWeek_2 VALUES IN('2010-02-08', '2010-02-09', '2010-02-10',
        '2010-02-11', '2010-02-12', '2010-02-13', '2010-02-14'),
    PARTITION pWeek_3 VALUES IN('2010-02-15', '2010-02-16', '2010-02-17',
        '2010-02-18', '2010-02-19', '2010-02-20', '2010-02-21'),
    PARTITION pWeek_4 VALUES IN('2010-02-22', '2010-02-23', '2010-02-24',
        '2010-02-25', '2010-02-26', '2010-02-27', '2010-02-28')
);
```

这种方法有效，但如果涉及的日期数量非常大，则定义和维护会变得繁琐；在这种情况下，通常更实用的是使用`范围`或`范围列`分区。在这种情况下，由于我们希望用作分区键的列是一个`DATE`列，我们使用`范围列`分区，如下所示：

```sql
CREATE TABLE customers_3 (
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    street_1 VARCHAR(30),
    street_2 VARCHAR(30),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY RANGE COLUMNS(renewal) (
    PARTITION pWeek_1 VALUES LESS THAN('2010-02-09'),
    PARTITION pWeek_2 VALUES LESS THAN('2010-02-15'),
    PARTITION pWeek_3 VALUES LESS THAN('2010-02-22'),
    PARTITION pWeek_4 VALUES LESS THAN('2010-03-01')
);
```

更多信息请参见第 26.2.3.1 节，“范围列分区”。

此外（与`范围列`分区一样），您可以在`COLUMNS()`子句中使用多个列。

有关`PARTITION BY LIST COLUMNS()`语法的更多信息，请参见第 15.1.20 节，“CREATE TABLE 语句”。
