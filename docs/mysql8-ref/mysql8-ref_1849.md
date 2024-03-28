> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-linear-hash.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-linear-hash.html)

#### 26.2.4.1 线性哈希分区

MySQL 还支持线性哈希，与常规哈希不同之处在于线性哈希利用线性二次幂算法，而常规哈希使用哈希函数值的模。

在语法上，线性哈希分区和常规哈希之间唯一的区别是在`PARTITION BY`子句中添加`LINEAR`关键字，如下所示：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY LINEAR HASH( YEAR(hired) )
PARTITIONS 4;
```

给定表达式*`expr`*，当使用线性哈希时存储记录的分区是从*`num`*个分区中的第*`N`*个分区，其中*`N`*根据以下算法派生：

1.  找到大于*`num`*的下一个 2 的幂。我们称这个值为*`V`*；可以计算如下：

    ```sql
    *V* = POWER(2, CEILING(LOG(2, *num*)))
    ```

    (假设*`num`*为 13。那么`LOG(2,13)`为 3.7004397181411。`CEILING(3.7004397181411)`为 4，*`V`* = `POWER(2,4)`，即 16。)

1.  设置*`N`* = *`F`*(*`column_list`*) & (*`V`* - 1)。

1.  当*`N`* >= *`num`*时：

    +   设置*`V`* = *`V`* / 2。

    +   设置*`N`* = *`N`* & (*`V`* - 1)。

假设使用线性哈希分区并具有 6 个分区的表`t1`是使用以下语句创建的：

```sql
CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATE)
    PARTITION BY LINEAR HASH( YEAR(col3) )
    PARTITIONS 6;
```

现在假设你想要将两条记录插入到具有`col3`列值`'2003-04-14'`和`'1998-10-19'`的`t1`中。第一条记录的分区号计算如下：

```sql
*V* = POWER(2, CEILING( LOG(2,6) )) = 8
*N* = YEAR('2003-04-14') & (8 - 1)
   = 2003 & 7
   = 3

(*3 >= 6 is FALSE: record stored in partition #3*)
```

第二条记录存储在的分区号计算如下：

```sql
*V* = 8
*N* = YEAR('1998-10-19') & (8 - 1)
  = 1998 & 7
  = 6

(*6 >= 6 is TRUE: additional step required*)

*N* = 6 & ((8 / 2) - 1)
  = 6 & 3
  = 2

(*2 >= 6 is FALSE: record stored in partition #2*)
```

通过线性哈希分区的优势在于添加、删除、合并和分割分区变得更快，这在处理包含极大量（TB 级）数据的表时可能会有益。缺点是与使用常规哈希分区获得的分布相比，数据在分区之间的均匀分布性较低。
