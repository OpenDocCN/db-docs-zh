# 下载代码

> 原文：[`sqlite.com/quickstart.html`](https://sqlite.com/quickstart.html)

下面是您开始尝试 SQLite 而不必进行大量繁琐阅读和配置的步骤：

+   获取适用于您的计算机的预构建二进制文件的副本，或者获取源代码的副本并自行编译。访问下载页面获取更多信息。

## 创建一个新的数据库

+   在 shell 或 DOS 提示符下输入：“**sqlite3 test.db**”。这将创建一个名为“test.db”的新数据库。（如果愿意，您可以使用不同的名称。）

+   在提示符下输入 SQL 命令以创建和填充新数据库。

+   额外的文档可以在这里找到。

## 编写使用 SQLite 的程序

+   下面是一个简单的[TCL 程序](http://www.tcl-lang.org)，演示如何使用 TCL 接口访问 SQLite。该程序将第二个参数中给出的 SQL 语句在第一个参数定义的数据库上执行。要关注的命令包括第 7 行的**sqlite3**命令，它打开一个 SQLite 数据库并创建一个名为“**db**”的新对象来访问该数据库，第 8 行上**db**对象的 eval 方法，用于对数据库运行 SQL 命令，以及脚本最后一行的关闭数据库连接。

    > ```sql
    > 01  #!/usr/bin/tclsh
    > 02  if {$argc!=2} {
    > 03    puts stderr "Usage: %s DATABASE SQL-STATEMENT"
    > 04    exit 1
    > 05  }
    > 06  package require sqlite3
    > 07  sqlite3 db [lindex $argv 0]
    > 08  db eval [lindex $argv 1] x {
    > 09    foreach v $x(*) {
    > 10      puts "$v = $x($v)"
    > 11    }
    > 12    puts ""
    > 13  }
    > 14  db close
    > 
    > ```

+   下面是一个简单的 C 程序，演示如何使用 SQLite 的 C/C++接口。数据库的名称由第一个参数给出，第二个参数是要针对数据库执行的一个或多个 SQL 语句。需要注意的函数调用包括第 22 行的 sqlite3_open()，用于打开数据库，第 28 行的 sqlite3_exec()，用于执行针对数据库的 SQL 命令，以及第 33 行的 sqlite3_close()，用于关闭数据库连接。

    查看有关 SQLite C/C++接口的介绍，以获得入门概述和数十个 SQLite 接口函数的路线图。

    > ```sql
    > 01  #include <stdio.h>
    > 02  #include <sqlite3.h>
    > 03  
    > 04  static int callback(void *NotUsed, int argc, char **argv, char **azColName){
    > 05    int i;
    > 06    for(i=0; i<argc; i++){
    > 07      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
    > 08    }
    > 09    printf("\n");
    > 10    return 0;
    > 11  }
    > 12  
    > 13  int main(int argc, char **argv){
    > 14    sqlite3 *db;
    > 15    char *zErrMsg = 0;
    > 16    int rc;
    > 17  
    > 18    if( argc!=3 ){
    > 19      fprintf(stderr, "Usage: %s DATABASE SQL-STATEMENT\n", argv[0]);
    > 20      return(1);
    > 21    }
    > 22    rc = sqlite3_open(argv[1], &db);
    > 23    if( rc ){
    > 24      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
    > 25      sqlite3_close(db);
    > 26      return(1);
    > 27    }
    > 28    rc = sqlite3_exec(db, argv[2], callback, 0, &zErrMsg);
    > 29    if( rc!=SQLITE_OK ){
    > 30      fprintf(stderr, "SQL error: %s\n", zErrMsg);
    > 31      sqlite3_free(zErrMsg);
    > 32    }
    > 33    sqlite3_close(db);
    > 34    return 0;
    > 35  }
    > 
    > ```

    请查看如何编译 SQLite 文档，获取有关如何编译上述程序的说明和提示。
