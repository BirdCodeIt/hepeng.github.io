---
layout: post
title: "Mysql打开log记录"
description: "Mysql打开log记录"
categories: [Mysql]
tags: [Mysql]
redirect_from:
  - /2016/11/26/
---

> Mysql 5.5版本打开log记录
> 在测试Mysql预编译的时候为了查看Mysql底层的语句执行情况及慢查询日志，需要打开Mysql的日志记录功能，通过修改my.ini里面进行配置发现Mysql无法再次正常启动，因此
> 通过Mysql命令进行修改，具体操作如下：

* Kramdown table of contents
{:toc .toc}

# 打开general_log

~~~ ruby

mysql> show variables LIKE '%general%';
+------------------+----------------------------------------------------------------+
| Variable_name    | Value                                                          |
+------------------+----------------------------------------------------------------+
| general_log      | OFF                                                            |
| general_log_file | C:\ProgramData\MySQL\MySQL Server 5.5\Data\PC-20170609HPDS.log |
+------------------+----------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql>
mysql> set global general_log=ON; //设置为打开记录general_log
Query OK, 0 rows affected (0.01 sec)

mysql> show variables LIKE '%general%';
+------------------+----------------------------------------------------------------+
| Variable_name    | Value                                                          |
+------------------+----------------------------------------------------------------+
| general_log      | ON                                                             |
| general_log_file | C:\ProgramData\MySQL\MySQL Server 5.5\Data\PC-20170609HPDS.log |
+------------------+----------------------------------------------------------------+
2 rows in set (0.00 sec)
~~~

修改成功后执行一些命令可以在记录里面进行查看

~~~ ruby

C:\Program Files (x86)\MySQL\MySQL Server 5.5\bin\mysqld, Version: 5.5.13 (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
180425 15:38:41	    2 Query	show variables LIKE '%general%'
180425 15:39:27	    2 Query	show variables LIKE '%slow%'
180425 15:39:47	    2 Query	set global slow_query_log=ON
180425 15:39:49	    2 Query	show variables LIKE '%slow%'
180425 17:00:53	    3 Connect	root@localhost on 
180425 17:00:55	    3 Query	use `test`
180425 17:00:56	    3 Query	show full tables from `test` where table_type = 'BASE TABLE'
		    3 Query	select `ENGINE`, `SUPPORT` from information_schema.Engines
180425 17:00:59	    3 Query	select `ENGINE`, `SUPPORT` from information_schema.Engines
180425 17:01:00	    3 Query	show full fields from `test`.`user`
		    3 Query	show keys from `test`.`user`
		    3 Query	select * from `test`.`user` limit 0, 50
~~~

# 慢日志记录打开

~~~ ruby
mysql> show variables LIKE '%slow%';
+---------------------+---------------------------------------------------------------------+
| Variable_name       | Value                                                               |
+---------------------+---------------------------------------------------------------------+
| log_slow_queries    | OFF                                                                 |
| slow_launch_time    | 2                                                                   |
| slow_query_log      | OFF                                                                 |
| slow_query_log_file | C:\ProgramData\MySQL\MySQL Server 5.5\Data\PC-20170609HPDS-slow.log |
+---------------------+---------------------------------------------------------------------+
4 rows in set (0.00 sec)

mysql> set global slow_query_log=ON
    -> ;
Query OK, 0 rows affected (0.04 sec)

mysql> show variables LIKE '%slow%';
+---------------------+---------------------------------------------------------------------+
| Variable_name       | Value                                                               |
+---------------------+---------------------------------------------------------------------+
| log_slow_queries    | ON                                                                  |
| slow_launch_time    | 2                                                                   |
| slow_query_log      | ON                                                                  |
| slow_query_log_file | C:\ProgramData\MySQL\MySQL Server 5.5\Data\PC-20170609HPDS-slow.log |
+---------------------+---------------------------------------------------------------------+
4 rows in set (0.00 sec)
~~~

通过sleep命令构造慢查询

~~~ ruby
mysql> select sleep(12);
+-----------+
| sleep(12) |
+-----------+
|         0 |
+-----------+
1 row in set (12.00 sec)
~~~

在慢查询日志中查看log，以方便定位排错

~~~ ruby
C:\Program Files (x86)\MySQL\MySQL Server 5.5\bin\mysqld, Version: 5.5.13 (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
# Time: 180425 17:40:53
# User@Host: root[root] @ localhost [::1]
# Query_time: 12.000038  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1524649253;
select sleep(12);
# Time: 180425 20:29:45
# User@Host: root[root] @ localhost [::1]
# Query_time: 12.000175  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1524659385;
select sleep(12);
# Time: 180425 20:33:01
# User@Host: root[root] @ localhost [::1]
# Query_time: 12.000899  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1524659581;
select sleep(12);
~~~
