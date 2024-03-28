> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-auto-increment.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-auto-increment.html)

#### 19.5.1.1 复制和 AUTO_INCREMENT

基于语句的复制`AUTO_INCREMENT`、`LAST_INSERT_ID()`和`TIMESTAMP`值的执行受以下异常情况的影响：

+   调用触发器或导致`AUTO_INCREMENT`列更新的函数的语句在基于语句的复制中无法正确复制。这些语句被标记为不安全。（Bug #45677）

+   对于具有包含`AUTO_INCREMENT`列但不是这个复合主键的第一列的复合主键的表进行`INSERT`不适用于基于语句的日志记录或复制。这些语句被标记为不安全。（Bug #11754117, Bug #45670）

    这个问题不影响使用`InnoDB`存储引擎的表，因为`InnoDB`表的`AUTO_INCREMENT`列需要至少一个键，其中`AUTO_INCREMENT`列是唯一的或最左边的列。

+   向具有`ALTER TABLE`的表添加`AUTO_INCREMENT`列可能不会在副本和源上产生相同的行顺序。这是因为行编号的顺序取决于用于表的特定存储引擎以及插入行的顺序。如果在源和副本上具有相同的顺序很重要，则在分配`AUTO_INCREMENT`编号之前必须对行进行排序。假设您想向具有列`col1`和`col2`的表`t1`添加`AUTO_INCREMENT`列，以下语句将生成一个与`t1`相同但具有`AUTO_INCREMENT`列的新表`t2`：

    ```sql
    CREATE TABLE t2 LIKE t1;
    ALTER TABLE t2 ADD id INT AUTO_INCREMENT PRIMARY KEY;
    INSERT INTO t2 SELECT * FROM t1 ORDER BY col1, col2;
    ```

    重要提示

    为了保证源和副本的顺序相同，`ORDER BY`子句必须命名`t1`的*所有*列。

    刚刚给出的指令受到`CREATE TABLE ... LIKE`的限制：外键定义被忽略，`DATA DIRECTORY`和`INDEX DIRECTORY`表选项也被忽略。如果表定义包含任何这些特征，使用与创建`t1`相同的`CREATE TABLE`语句创建`t2`，但增加`AUTO_INCREMENT`列。

    无论用于创建和填充具有`AUTO_INCREMENT`列的副本的方法如何，最后一步是删除原始表，然后重命名副本：

    ```sql
    DROP t1;
    ALTER TABLE t2 RENAME t1;
    ```

    另请参阅 Section B.3.6.1, “Problems with ALTER TABLE”。
