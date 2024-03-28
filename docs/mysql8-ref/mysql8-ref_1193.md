> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-deadlock-example.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlock-example.html)

#### 17.7.5.1 InnoDB 死锁示例

以下示例说明了当锁请求导致死锁时会发生错误的情况。该示例涉及两个客户端，A 和 B。

InnoDB 状态包含最后一个死锁的详细信息。对于频繁发生的死锁，请启用全局变量`innodb_print_all_deadlocks`。这会将死锁信息添加到错误日志中。

客户端 A 启用`innodb_print_all_deadlocks`，创建两个表，'Animals' 和 'Birds'，并向每个表插入数据。客户端 A 开始一个事务，并以共享模式选择 Animals 表中的一行：

```sql
mysql> SET GLOBAL innodb_print_all_deadlocks = ON;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE Animals (name VARCHAR(10) PRIMARY KEY, value INT) ENGINE = InnoDB;
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE TABLE Birds (name VARCHAR(10) PRIMARY KEY, value INT) ENGINE = InnoDB;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO Animals (name,value) VALUES ("Aardvark",10);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO Birds (name,value) VALUES ("Buzzard",20);
Query OK, 1 row affected (0.00 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT value FROM Animals WHERE name='Aardvark' FOR SHARE;
+-------+
| value |
+-------+
|    10 |
+-------+
1 row in set (0.00 sec)
```

接下来，客户端 B 开始一个事务，并以共享模式选择 Birds 表中的一行：

```sql
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT value FROM Birds WHERE name='Buzzard' FOR SHARE;
+-------+
| value |
+-------+
|    20 |
+-------+
1 row in set (0.00 sec)
```

性能模式显示了两个 select 语句后的锁定情况：

```sql
mysql> SELECT ENGINE_TRANSACTION_ID as Trx_Id, 
              OBJECT_NAME as `Table`, 
              INDEX_NAME as `Index`, 
              LOCK_DATA as Data, 
              LOCK_MODE as Mode, 
              LOCK_STATUS as Status, 
              LOCK_TYPE as Type 
        FROM performance_schema.data_locks;
+-----------------+---------+---------+------------+---------------+---------+--------+
| Trx_Id          | Table   | Index   | Data       | Mode          | Status  | Type   |
+-----------------+---------+---------+------------+---------------+---------+--------+
| 421291106147544 | Animals | NULL    | NULL       | IS            | GRANTED | TABLE  |
| 421291106147544 | Animals | PRIMARY | 'Aardvark' | S,REC_NOT_GAP | GRANTED | RECORD |
| 421291106148352 | Birds   | NULL    | NULL       | IS            | GRANTED | TABLE  |
| 421291106148352 | Birds   | PRIMARY | 'Buzzard'  | S,REC_NOT_GAP | GRANTED | RECORD |
+-----------------+---------+---------+------------+---------------+---------+--------+
4 rows in set (0.00 sec)
```

然后客户端 B 更新 Animals 表中的一行：

```sql
mysql> UPDATE Animals SET value=30 WHERE name='Aardvark';
```

客户端 B 必须等待。性能模式显示了等待锁的情况：

```sql
mysql> SELECT REQUESTING_ENGINE_LOCK_ID as Req_Lock_Id,
              REQUESTING_ENGINE_TRANSACTION_ID as Req_Trx_Id,
              BLOCKING_ENGINE_LOCK_ID as Blk_Lock_Id, 
              BLOCKING_ENGINE_TRANSACTION_ID as Blk_Trx_Id
        FROM performance_schema.data_lock_waits;
+----------------------------------------+------------+----------------------------------------+-----------------+
| Req_Lock_Id                            | Req_Trx_Id | Blk_Lock_Id                            | Blk_Trx_Id      |
+----------------------------------------+------------+----------------------------------------+-----------------+
| 139816129437696:27:4:2:139816016601240 |      43260 | 139816129436888:27:4:2:139816016594720 | 421291106147544 |
+----------------------------------------+------------+----------------------------------------+-----------------+
1 row in set (0.00 sec)

mysql> SELECT ENGINE_LOCK_ID as Lock_Id, 
              ENGINE_TRANSACTION_ID as Trx_id, 
              OBJECT_NAME as `Table`, 
              INDEX_NAME as `Index`, 
              LOCK_DATA as Data, 
              LOCK_MODE as Mode, 
              LOCK_STATUS as Status, 
              LOCK_TYPE as Type 
        FROM performance_schema.data_locks;
+----------------------------------------+-----------------+---------+---------+------------+---------------+---------+--------+
| Lock_Id                                | Trx_Id          | Table   | Index   | Data       | Mode          | Status  | Type   |
+----------------------------------------+-----------------+---------+---------+------------+---------------+---------+--------+
| 139816129437696:1187:139816016603896   |           43260 | Animals | NULL    | NULL       | IX            | GRANTED | TABLE  |
| 139816129437696:1188:139816016603808   |           43260 | Birds   | NULL    | NULL       | IS            | GRANTED | TABLE  |
| 139816129437696:28:4:2:139816016600896 |           43260 | Birds   | PRIMARY | 'Buzzard'  | S,REC_NOT_GAP | GRANTED | RECORD |
| 139816129437696:27:4:2:139816016601240 |           43260 | Animals | PRIMARY | 'Aardvark' | X,REC_NOT_GAP | WAITING | RECORD |
| 139816129436888:1187:139816016597712   | 421291106147544 | Animals | NULL    | NULL       | IS            | GRANTED | TABLE  |
| 139816129436888:27:4:2:139816016594720 | 421291106147544 | Animals | PRIMARY | 'Aardvark' | S,REC_NOT_GAP | GRANTED | RECORD |
+----------------------------------------+-----------------+---------+---------+------------+---------------+---------+--------+
6 rows in set (0.00 sec)
```

当事务尝试修改数据库时，InnoDB 仅使用顺序事务 id。因此，之前的只读事务 id 从 421291106148352 变为 43260。

如果客户端 A 同时尝试更新 Birds 表中的一行，这将导致死锁：

```sql
mysql> UPDATE Birds SET value=40 WHERE name='Buzzard';
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

InnoDB 回滚导致死锁的事务。现在，来自客户端 B 的第一个更新可以继续进行。

信息模式包含死锁数量：

```sql
mysql> SELECT `count` FROM INFORMATION_SCHEMA.INNODB_METRICS
          WHERE NAME="lock_deadlocks";
+-------+
| count |
+-------+
|     1 |
+-------+
1 row in set (0.00 sec)
```

InnoDB 状态包含有关死锁和事务的以下信息。它还显示只读事务 id 421291106147544 变为顺序事务 id 43261。

```sql
mysql> SHOW ENGINE INNODB STATUS;
------------------------
LATEST DETECTED DEADLOCK
------------------------
2022-11-25 15:58:22 139815661168384
*** (1) TRANSACTION:
TRANSACTION 43260, ACTIVE 186 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 19, OS thread handle 139815619204864, query id 143 localhost u2 updating
UPDATE Animals SET value=30 WHERE name='Aardvark'

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 28 page no 4 n bits 72 index PRIMARY of table `test`.`Birds` trx id 43260 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 7; hex 42757a7a617264; asc Buzzard;;
 1: len 6; hex 00000000a8fb; asc       ;;
 2: len 7; hex 82000000e40110; asc        ;;
 3: len 4; hex 80000014; asc     ;;

*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 27 page no 4 n bits 72 index PRIMARY of table `test`.`Animals` trx id 43260 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 8; hex 416172647661726b; asc Aardvark;;
 1: len 6; hex 00000000a8f9; asc       ;;
 2: len 7; hex 82000000e20110; asc        ;;
 3: len 4; hex 8000000a; asc     ;;

*** (2) TRANSACTION:
TRANSACTION 43261, ACTIVE 209 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 18, OS thread handle 139815618148096, query id 146 localhost u1 updating
UPDATE Birds SET value=40 WHERE name='Buzzard'

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 27 page no 4 n bits 72 index PRIMARY of table `test`.`Animals` trx id 43261 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 8; hex 416172647661726b; asc Aardvark;;
 1: len 6; hex 00000000a8f9; asc       ;;
 2: len 7; hex 82000000e20110; asc        ;;
 3: len 4; hex 8000000a; asc     ;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 28 page no 4 n bits 72 index PRIMARY of table `test`.`Birds` trx id 43261 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 7; hex 42757a7a617264; asc Buzzard;;
 1: len 6; hex 00000000a8fb; asc       ;;
 2: len 7; hex 82000000e40110; asc        ;;
 3: len 4; hex 80000014; asc     ;;

*** WE ROLL BACK TRANSACTION (2)
------------
TRANSACTIONS
------------
Trx id counter 43262
Purge done for trx's n:o < 43256 undo n:o < 0 state: running but idle
History list length 0
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421291106147544, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 421291106146736, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 421291106145928, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 43260, ACTIVE 219 sec
4 lock struct(s), heap size 1128, 2 row lock(s), undo log entries 1
MySQL thread id 19, OS thread handle 139815619204864, query id 143 localhost u2
```

错误日志包含有关事务和锁的信息：

```sql
mysql> SELECT @@log_error;
+---------------------+
| @@log_error         |
+---------------------+
| /var/log/mysqld.log |
+---------------------+
1 row in set (0.00 sec)

TRANSACTION 43260, ACTIVE 186 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 19, OS thread handle 139815619204864, query id 143 localhost u2 updating
UPDATE Animals SET value=30 WHERE name='Aardvark'
RECORD LOCKS space id 28 page no 4 n bits 72 index PRIMARY of table `test`.`Birds` trx id 43260 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 7; hex 42757a7a617264; asc Buzzard;;
 1: len 6; hex 00000000a8fb; asc       ;;
 2: len 7; hex 82000000e40110; asc        ;;
 3: len 4; hex 80000014; asc     ;;

RECORD LOCKS space id 27 page no 4 n bits 72 index PRIMARY of table `test`.`Animals` trx id 43260 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 8; hex 416172647661726b; asc Aardvark;;
 1: len 6; hex 00000000a8f9; asc       ;;
 2: len 7; hex 82000000e20110; asc        ;;
 3: len 4; hex 8000000a; asc     ;;

TRANSACTION 43261, ACTIVE 209 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 18, OS thread handle 139815618148096, query id 146 localhost u1 updating
UPDATE Birds SET value=40 WHERE name='Buzzard'
RECORD LOCKS space id 27 page no 4 n bits 72 index PRIMARY of table `test`.`Animals` trx id 43261 lock mode S locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 8; hex 416172647661726b; asc Aardvark;;
 1: len 6; hex 00000000a8f9; asc       ;;
 2: len 7; hex 82000000e20110; asc        ;;
 3: len 4; hex 8000000a; asc     ;;

RECORD LOCKS space id 28 page no 4 n bits 72 index PRIMARY of table `test`.`Birds` trx id 43261 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 7; hex 42757a7a617264; asc Buzzard;;
 1: len 6; hex 00000000a8fb; asc       ;;
 2: len 7; hex 82000000e40110; asc        ;;
 3: len 4; hex 80000014; asc     ;;
```
