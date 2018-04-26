---
layout: post
title: "Mysql使用trigger"
description: "Mysql使用trigger"
categories: [Mysql]
tags: [Mysql]
redirect_from:
  - /2016/11/26/
---


* Kramdown table of contents
{:toc .toc}

# 什么是触发器
简单来说，就是一张表发生了某件事（插入，删除，更新操作），然后自动触发了预先编写好的若干SQL语句的执行；

# 特点和作用
数据库触发器有以下的作用：

1.安全性。可以基于数据库的值使用户具有操作数据库的某种权利。

  # 可以基于时间限制用户的操作，例如不允许下班后和节假日修改数据库数据。

  # 可以基于数据库中的数据限制用户的操作，例如不允许股票的价格的升幅一次超过10%。

2.审计。可以跟踪用户对数据库的操作。   

  # 审计用户操作数据库的语句。

  # 把用户对数据库的更新写入审计表。

3.实现复杂的数据完整性规则

  # 实现非标准的数据完整性检查和约束。触发器可产生比规则更为复杂的限制。与规则不同，触发器可以引用列或数据库对象。例如，触发器可回退任何企图吃进超过自己保证金的期货。

  # 提供可变的缺省值。

4.实现复杂的非标准的数据库相关完整性规则。触发器可以对数据库中相关的表进行连环更新。例如，在auths表author_code列上的删除触发器可导致相应删除在其它表中的与之匹配的行。

  # 在修改或删除时级联修改或删除其它表中的与之匹配的行。

  # 在修改或删除时把其它表中的与之匹配的行设成NULL值。

  # 在修改或删除时把其它表中的与之匹配的行级联设成缺省值。

  # 触发器能够拒绝或回退那些破坏相关完整性的变化，取消试图进行数据更新的事务。当插入一个与其主健不匹配的外部键时，这种触发器会起作用。例如，可以在books.author_code 列上生成一个插入触发器，如果新值与auths.author_code列中的某值不匹配时，插入被回退。

5.同步实时地复制表中的数据。

6.自动计算数据值，如果数据的值达到了一定的要求，则进行特定的处理。例如，如果公司的帐号上的资金低于5万元则立即给财务人员发送警告数据。

# 触发器例子
触发器创建的语法如下：
~~~ ruby
CREATE TRIGGER trigger_name trigger_time trigger_event ON tb_name FOR EACH ROW trigger_stmt
trigger_name：触发器的名称
tirgger_time：触发时机，为BEFORE或者AFTER
trigger_event：触发事件，为INSERT、DELETE或者UPDATE
tb_name：表示建立触发器的表明，就是在哪张表上建立触发器
trigger_stmt：触发器的程序体，可以是一条SQL语句或者是用BEGIN和END包含的多条语句
所以可以说MySQL创建以下六种触发器：
BEFORE INSERT,BEFORE DELETE,BEFORE UPDATE
AFTER INSERT,AFTER DELETE,AFTER UPDATE
~~~
其中，触发器名参数指要创建的触发器的名字

BEFORE和AFTER参数指定了触发执行的时间，在事件之前或是之后

FOR EACH ROW表示任何一条记录上的操作满足触发事件都会触发该触发器

其中，BEGIN与END之间的执行语句列表参数表示需要执行的多个语句，不同语句用分号隔开

tips：
一般情况下，mysql默认是以 ; 作为结束执行语句，与触发器中需要的分行起冲突

为解决此问题可用DELIMITER，如：DELIMITER ||，可以将结束符号变成||

当触发器创建完成后，可以用DELIMITER ;来将结束符号变成;

接下来将创建user和user_history表，以及三个触发器tri_insert_user、tri_update_user、tri_delete_user，分别对应user表的增、删、改三件事；

创建user表：
~~~ ruby
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `account` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
~~~

创建对user表的操作历史表：

~~~ ruby
DROP TABLE IF EXISTS `user_history`;
CREATE TABLE `user_history` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL,
  `operatetype` varchar(200) NOT NULL,
  `operatetime` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
~~~

创建user表插入事件对应的触发器tri_insert_user；
注意：

1、DELIMITER：改变输入的结束符，默认情况下输入结束符是分号;，这里把它改成了两个分号;;，这样做的目的是把多条含分号的语句做个封装，全部输入完之后一起执行，而不是一遇到默认的分号结束符就自动执行；

2、new：当触发插入和更新事件时可用，指向的是被操作的记录

3、old： 当触发删除和更新事件时可用，指向的是被操作的记录

4、相同的事件只能创建一个触发器

创建tri_insert_user触发器
~~~ ruby
DROP TRIGGER IF EXISTS `tri_insert_user`;
DELIMITER ;;
CREATE TRIGGER `tri_insert_user` AFTER INSERT ON `user` FOR EACH ROW begin
    INSERT INTO user_history(user_id, operatetype, operatetime) VALUES (new.id, 'add a user',  now());
end
;;
DELIMITER ;
~~~

创建user表更新事件对应的触发器tri_update_user；
~~~ ruby
DROP TRIGGER IF EXISTS `tri_update_user`;
DELIMITER ;;
CREATE TRIGGER `tri_update_user` AFTER UPDATE ON `user` FOR EACH ROW begin
    INSERT INTO user_history(user_id,operatetype, operatetime) VALUES (new.id, 'update a user', now());
end
;;
DELIMITER ;
~~~

创建user表删除事件对应的触发器tri_delete_user；
~~~ ruby
DROP TRIGGER IF EXISTS `tri_delete_user`;
DELIMITER ;;
CREATE TRIGGER `tri_delete_user` AFTER DELETE ON `user` FOR EACH ROW begin
    INSERT INTO user_history(user_id, operatetype, operatetime) VALUES (old.id, 'delete a user', now());
end
;;
DELIMITER ;
~~~

至此，全部表及触发器创建完成，开始验证结果，分别做插入、修改、删除事件，执行以下语句，观察user_history是否自动产生操作记录；
~~~ ruby
INSERT INTO user(account, name, address) VALUES ('user1', 'user1', 'user1');
INSERT INTO user(account, name, address) VALUES ('user2', 'user2', 'user2');

UPDATE user SET name = 'user3', account = 'user3', address='user3' where name='user1';

DELETE FROM `user` where name = 'user2';
~~~

# 通过触发器实现Mysql的Check功能
Mysql中不支持Check语法，可以通过触发器实现对特定字段的检查，触发时机和触发事件为before insert


~~~ ruby
DELIMITER ;;
CREATE TRIGGER `checktrigger` BEFORE INSERT ON `user` FOR EACH ROW BEGIN
DECLARE msg VARCHAR(200);
IF(new.id>20) THEN 
SET msg="Id is above 20.Cannot insert";
signal SQLSTATE 'HY000' SET message_text=msg;
END IF;
END
;;
DELIMITER ;
~~~

# 弊端
增加程序的复杂度，有些业务逻辑在代码中处理，有些业务逻辑用触发器处理，会使后期维护变得困难；


