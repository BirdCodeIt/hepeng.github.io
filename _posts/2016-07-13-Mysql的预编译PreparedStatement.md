---
layout: post
title: "mysql的预编译PreparedStatement"
description: "mysql的预编译PreparedStatement"
categories: [Mysql]
tags: [Mysql]
redirect_from:
  - /2016/08/12/
---

> mysql的预编译PreparedStatement
> *mysql执行脚本的大致过程如下：prepare(准备)->optimize(优化)->exec(物理执行)*{: style="color: red"}，其中，prepare也就是编译过程。对于同一个sql模板，
> 如果能将prepare的结果缓存，以后如果再执行相同模板而测试不同的sql，就可以节省掉prepare(准备)的环节，从而节省sql执行的成本。



* Kramdown table of contents
{:toc .toc}

# Mysql预编译介绍

> 大家平时都使用过JDBC中的PreparedStatement接口，它有预编译功能。什么是预编译功能呢？它有什么好处呢？
> 当客户发送一条SQL语句给服务器后，服务器总是需要校验SQL语句的语法格式是否正确，然后把SQL语句编译成可执行的函数，最后才是执行SQL语句。其中校验语法，和编译所花的时间可能比执行SQL语句花的时间还要多。
> 注意：可执行函数存储在MySQL服务器中，并且当前连接断开后，MySQL服务器会清除已经存储的可执行函数。
> 如果我们需要执行多次insert语句，但只是每次插入的值不同，MySQL服务器也是需要每次都去校验SQL语句的语法格式，以及编译，这就浪费了太多的时间。如果使用预编译功能，那么只对SQL语句进行一次语法校验和编译，所以效率要高。

MySQL Server 4.1之后的版本都是支持预编译的，但是默认是关闭的，需要通过设置useServerPrepStmts=true来开启预编译功能

## 默认关闭下的测试

~~~ ruby

public class PrepareStatementDemo {


    public static void main(String[] args) {

        //数据库连接
        Connection connection = null;

        //预编译的Statement，使用预编译的Statement提高数据库性能
        PreparedStatement preparedStatement = null;

        //结果集
        ResultSet resultSet = null;

        try {
            //加载数据库驱动
            Class.forName("com.mysql.jdbc.Driver");

            //通过驱动管理类获取数据库链接
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test1","root","root");

            //定义SQL语句
            String sql  = "select * from user where name = ?";

            //获取预处理statement
            preparedStatement = connection.prepareStatement(sql);

            //设置参数，第一个参数为sql语句中参数的序号(从1开始)，第二个参数为设置的参数值
            preparedStatement.setString(1,"aaa");

            //向数据库发出sql执行查询，查询出结果集
            resultSet = preparedStatement.executeQuery();

            preparedStatement.setString(1, "bbb");
            resultSet =  preparedStatement.executeQuery();
            //遍历查询结果集
            while(resultSet.next()){
                System.out.println(resultSet.getString("id")+"  "+resultSet.getString("name"));
            }
            resultSet.close();
            preparedStatement.close();

            System.out.println("#############################");

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if(resultSet!=null){
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            if(preparedStatement!=null){
                try {
                    preparedStatement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            if(connection!=null){
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

~~~

通过查看MySQL的执行log如下：

~~~ ruby
18 Query	/* mysql-connector-java-5.1.46 ( Revision: 9cc87a48e75c2d2e87c1a293b2862ce651cb256e ) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_buffer_length AS net_buffer_length, @@net_write_timeout AS net_write_timeout, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@tx_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
18 Query	SET NAMES latin1
18 Query	SET character_set_results = NULL
18 Query	SET autocommit=1
18 Query	select * from user where name = 'aaa'
18 Query	select * from user where name = 'bbb'
18 Quit	
~~~

发现系统并没有使用预编译功能，通过连接时设置useServerPrepStmts=true，DriverManager.getConnection("jdbc:mysql://localhost:3306/test1?&*useServerPrepStmts=true*{: style="color: red"}","root","root")后执行的结果如下：


~~~ ruby
21 Query	/* mysql-connector-java-5.1.46 ( Revision: 9cc87a48e75c2d2e87c1a293b2862ce651cb256e ) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_buffer_length AS net_buffer_length, @@net_write_timeout AS net_write_timeout, @@query_cache_size AS query_cache_size, @@query_cache_type AS query_cache_type, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@tx_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
21 Query	SET NAMES latin1
21 Query	SET character_set_results = NULL
21 Query	SET autocommit=1
21 *Prepare	select * from user where name = ?*{: style="color: red"}
21 Execute	select * from user where name = 'aaa'
21 Execute	select * from user where name = 'bbb'
21 Close stmt	
21 Quit	
~~~


