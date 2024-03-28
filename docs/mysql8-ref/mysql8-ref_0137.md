# 5.5 在批处理模式下使用 mysql

> 译文：[`dev.mysql.com/doc/refman/8.0/en/batch-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/batch-mode.html)

在前面的章节中，你以交互方式使用**mysql**输入语句并查看结果。你也可以以批处理模式运行**mysql**。要做到这一点，将你想要运行的语句放在一个文件中，然后告诉**mysql**从文件中读取输入：

```sql
$> mysql < *batch-file*
```

如果你在 Windows 下运行**mysql**并且文件中有一些导致问题的特殊字符，你可以这样做：

```sql
C:\> mysql -e "source *batch-file*"
```

如果你需要在命令行上指定连接参数，命令可能如下所示：

```sql
$> mysql -h *host* -u *user* -p < *batch-file*
Enter password: ********
```

当你以这种方式使用**mysql**时，你正在创建一个脚本文件，然后执行该脚本。

如果你希望脚本在其中的某些语句产生错误时继续运行，你应该使用`--force`命令行选项。

为什么要使用脚本？以下是一些原因：

+   如果你重复运行一个查询（比如，每天或每周一次），将其制作成脚本可以避免每次执行时重新输入它。

+   你可以通过复制和编辑脚本文件从现有类似的查询生成新的查询。

+   在开发查询时，批处理模式也很有用，特别是对于多行语句或多语句序列。如果出现错误，你不必重新输入所有内容。只需编辑你的脚本以纠正错误，然后告诉**mysql**再次执行它。

+   如果你有一个产生大量输出的查询，你可以通过一个分页器运行输出，而不是看着它从屏幕顶部滚动出去：

    ```sql
    $> mysql < *batch-file* | more
    ```

+   你可以将输出捕获到一个文件中以供进一步处理：

    ```sql
    $> mysql < *batch-file* > mysql.out
    ```

+   你可以将你的脚本分发给其他人，这样他们也可以运行这些语句。

+   有些情况不允许交互使用，例如，当你从**cron**作业中运行查询时。在这种情况下，你必须使用批处理模式。

当你以批处理模式运行**mysql**时，默认的输出格式与交互式使用时不��（更简洁）。例如，当在交互式模式下运行**mysql**时，`SELECT DISTINCT species FROM pet`的输出如下：

```sql
+---------+
| species |
+---------+
| bird    |
| cat     |
| dog     |
| hamster |
| snake   |
+---------+
```

在批处理模式下，输出看起来像这样：

```sql
species
bird
cat
dog
hamster
snake
```

如果你想在批处理模式下获得交互式输出格式，使用**mysql -t**。要将执行的语句回显到输出中，使用**mysql -v**。

你也可以在**mysql**提示符下使用`source`命令或`\.`命令来运行脚本：

```sql
mysql> source *filename*;
mysql> \. *filename*
```

查看第 6.5.1.5 节，“从文本文件执行 SQL 语句”，获取更多信息。
