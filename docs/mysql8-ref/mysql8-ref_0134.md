> 译文：[`dev.mysql.com/doc/refman/8.0/en/counting-rows.html`](https://dev.mysql.com/doc/refman/8.0/en/counting-rows.html)

#### 5.3.4.8 计算行数

数据库经常用于回答问题，“表中某种类型的数据发生多少次？” 例如，您可能想知道您有多少宠物，或每个所有者有多少宠物，或者您可能想对您的动物执行各种普查操作。

计算您拥有的动物总数与“`pet` 表中有多少行？”是同一个问题，因为每只宠物有一条记录。`COUNT(*)` 计算行数，因此计算您的动物的查询如下所示：

```sql
mysql> SELECT COUNT(*) FROM pet;
+----------+
| COUNT(*) |
+----------+
|        9 |
+----------+
```

之前，您检索了拥有宠物的人的姓名。如果您想知道每个所有者有多少宠物，可以使用 `COUNT()`：

```sql
mysql> SELECT owner, COUNT(*) FROM pet GROUP BY owner;
+--------+----------+
| owner  | COUNT(*) |
+--------+----------+
| Benny  |        2 |
| Diane  |        2 |
| Gwen   |        3 |
| Harold |        2 |
+--------+----------+
```

前面的查询使用 `GROUP BY` 将每个 `owner` 的所有记录分组。与 `GROUP BY` 结合使用 `COUNT()` 有助于对各种分组下的数据进行表征。以下示例展示了执行动物普查操作的不同方法。

每种物种的动物数量：

```sql
mysql> SELECT species, COUNT(*) FROM pet GROUP BY species;
+---------+----------+
| species | COUNT(*) |
+---------+----------+
| bird    |        2 |
| cat     |        2 |
| dog     |        3 |
| hamster |        1 |
| snake   |        1 |
+---------+----------+
```

每种性别的动物数量：

```sql
mysql> SELECT sex, COUNT(*) FROM pet GROUP BY sex;
+------+----------+
| sex  | COUNT(*) |
+------+----------+
| NULL |        1 |
| f    |        4 |
| m    |        4 |
+------+----------+
```

（在此输出中，`NULL` 表示性别未知。）

每种物种和性别组合的动物数量：

```sql
mysql> SELECT species, sex, COUNT(*) FROM pet GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | NULL |        1 |
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
```

当使用 `COUNT()` 时，您无需检索整个表。例如，仅对狗和猫执行上一个查询时，如下所示：

```sql
mysql> SELECT species, sex, COUNT(*) FROM pet
       WHERE species = 'dog' OR species = 'cat'
       GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
+---------+------+----------+
```

或者，如果您只想知道已知性别的动物每种性别的数量：

```sql
mysql> SELECT species, sex, COUNT(*) FROM pet
       WHERE sex IS NOT NULL
       GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
```

如果您在选择要选择的列时还命名了列以外的 `COUNT()` 值，则应该存在一个包含命名相同列的 `GROUP BY` 子句。否则，将发生以下情况：

+   如果启用了 `ONLY_FULL_GROUP_BY` SQL 模式，则会出现错误：

    ```sql
    mysql> SET sql_mode = 'ONLY_FULL_GROUP_BY';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT owner, COUNT(*) FROM pet;
    ERROR 1140 (42000): In aggregated query without GROUP BY, expression
    #1 of SELECT list contains nonaggregated column 'menagerie.pet.owner';
    this is incompatible with sql_mode=only_full_group_by
    ```

+   如果未启用 `ONLY_FULL_GROUP_BY`，则查询将通过将所有行视为单个组来处理，但为每个命名列选择的值是不确定的。服务器可以从任何行中选择值：

    ```sql
    mysql> SET sql_mode = '';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT owner, COUNT(*) FROM pet;
    +--------+----------+
    | owner  | COUNT(*) |
    +--------+----------+
    | Harold |        8 |
    +--------+----------+
    1 row in set (0.00 sec)
    ```

参见 第 14.19.3 节，“MySQL 对 GROUP BY 的处理”。有关 `COUNT(*expr*)` 的行为和相关优化，请参见 第 14.19.1 节，“聚合函数描述”。
