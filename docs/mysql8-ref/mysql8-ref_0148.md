# 5.7 使用 MySQL 与 Apache

> 原文：[`dev.mysql.com/doc/refman/8.0/en/apache.html`](https://dev.mysql.com/doc/refman/8.0/en/apache.html)

有些程序可以让您从 MySQL 数据库验证用户，并将日志文件写入 MySQL 表中。

将 Apache 日志格式更改为 MySQL 易读的方法是将以下内容放入 Apache 配置文件中：

```sql
LogFormat \
        "\"%h\",%{%Y%m%d%H%M%S}t,%>s,\"%b\",\"%{Content-Type}o\",  \
        \"%U\",\"%{Referer}i\",\"%{User-Agent}i\""
```

要将以该格式的日志文件加载到 MySQL 中，可以使用类似以下语句的语句：

```sql
LOAD DATA INFILE '*/local/access_log*' INTO TABLE *tbl_name*
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '\\'
```

应创建一个命名表，其列应对应`LogFormat`行写入日志文件的列。
