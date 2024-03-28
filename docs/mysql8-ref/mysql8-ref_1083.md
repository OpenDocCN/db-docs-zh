> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-event.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-event.html)

#### 15.7.7.7 显示创建事件语句

```sql
SHOW CREATE EVENT *event_name*
```

此语句显示重新创建给定事件所需的`CREATE EVENT`语句。它需要显示事件的数据库的`EVENT`权限。例如（使用在第 15.7.7.18 节，“显示事件语句”中定义并修改的相同事件`e_daily`）：

```sql
mysql> SHOW CREATE EVENT myschema.e_daily\G
*************************** 1\. row ***************************
               Event: e_daily
            sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                      NO_ZERO_IN_DATE,NO_ZERO_DATE,
                      ERROR_FOR_DIVISION_BY_ZERO,
                      NO_ENGINE_SUBSTITUTION
           time_zone: SYSTEM
        Create Event: CREATE DEFINER=`jon`@`ghidora` EVENT `e_daily`
                        ON SCHEDULE EVERY 1 DAY
                        STARTS CURRENT_TIMESTAMP + INTERVAL 6 HOUR
                        ON COMPLETION NOT PRESERVE
                        ENABLE
                        COMMENT 'Saves total number of sessions then
                                clears the table each day'
                        DO BEGIN
                          INSERT INTO site_activity.totals (time, total)
                            SELECT CURRENT_TIMESTAMP, COUNT(*)
                              FROM site_activity.sessions;
                          DELETE FROM site_activity.sessions;
                        END
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
```

`character_set_client`是事件创建时`character_set_client`系统变量的会话值。`collation_connection`是事件创建时`collation_connection`系统变量的会话值。`数据库排序规则`是事件关联的数据库的排序规则。

输出反映了事件的当前状态（`ENABLE`）而不是创建时的状态。
