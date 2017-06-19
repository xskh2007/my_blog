---
title: DQL、DML、DDL、DCL的概念与区别 
date: 2017-06-15 14:17:27
categories:	mysql
tags: 
	- mysql
---

<!-- toc -->



# DQL、DML、DDL、DCL的概念与区别 



SQL(Structure Query Language)语言是数据库的核心语言。


SQL的发展是从1974年开始的，其发展过程如下：
1974年-----由Boyce和Chamberlin提出，当时称SEQUEL。
1976年-----IBM公司的Sanjase研究所在研制RDBMS SYSTEM R
时改为SQL。
1979年-----Oracle公司发表第一个基于SQL的商业化RDBMS产品。
1982年-----IBM公司出版第一个RDBMS语言SQL/DS。
1985年-----IBM公司出版第一个RDBMS语言DB2。
1986年-----美国国家标准化组织ANSI宣布SQL作为数据库工业标准。
SQL是一个标准的数据库语言，是面向集合的描述性非过程化语言。
它功能强，效率高，简单易学易维护（迄今为止，我还没见过比它还好
学的语言）。然而SQL语言由于以上优点，同时也出现了这样一个问题：
它是非过程性语言，即大多数语句都是独立执行的，与上下文无关，而
绝大部分应用都是一个完整的过程，显然用SQL完全实现这些功能是很困
难的。所以大多数数据库公司为了解决此问题，作了如下两方面的工作：
(1)扩充SQL，在SQL中引入过程性结构；(2)把SQL嵌入到高级语言中，
以便一起完成一个完整的应用。


二. SQL语言的分类

SQL语言共分为四大类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL。

1. 数据查询语言DQL
数据查询语言DQL基本结构是由SELECT子句，FROM子句，WHERE
子句组成的查询块：
SELECT <字段名表>
FROM <表或视图名>
WHERE <查询条件>

2 .数据操纵语言DML
数据操纵语言DML主要有三种形式：
1) 插入：INSERT
2) 更新：UPDATE
3) 删除：DELETE

3. 数据定义语言DDL
数据定义语言DDL用来创建数据库中的各种对象-----表、视图、
索引、同义词、聚簇等如：
CREATE TABLE/VIEW/INDEX/SYN/CLUSTER
| | | | |
表 视图 索引 同义词 簇

DDL操作是隐性提交的！不能rollback

4. 数据控制语言DCL
数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制
数据库操纵事务发生的时间及效果，对数据库实行监视等。如：
1) GRANT：授权。


2) ROLLBACK [WORK] TO [SAVEPOINT]：回退到某一点。
回滚---ROLLBACK
回滚命令使数据库状态回到上次最后提交的状态。其格式为：
SQL>ROLLBACK;


3) COMMIT [WORK]：提交。


    在数据库的插入、删除和修改操作时，只有当事务在提交到数据
库时才算完成。在事务提交前，只有操作数据库的这个人才能有权看
到所做的事情，别人只有在最后提交完成后才可以看到。
提交数据有三种类型：显式提交、隐式提交及自动提交。下面分
别说明这三种类型。


(1) 显式提交
用COMMIT命令直接完成的提交为显式提交。其格式为：
SQL>COMMIT；


(2) 隐式提交
用SQL命令间接完成的提交为隐式提交。这些命令是：
ALTER，AUDIT，COMMENT，CONNECT，CREATE，DISCONNECT，DROP，
EXIT，GRANT，NOAUDIT，QUIT，REVOKE，RENAME。


(3) 自动提交
若把AUTOCOMMIT设置为ON，则在插入、修改、删除语句执行后，
系统将自动进行提交，这就是自动提交。其格式为：
SQL>SET AUTOCOMMIT ON；

# mysql上利用通配符模糊匹配数据库进行grant

 给业务搭建数据库时由于采用的时分库策略，导致每个服务器上都有上百个数据库，新用户需要只对这些库有权限读写，由于服务器多，数据库多，如果采用逐个赋权限会很麻烦
在mysql中，当我们想对某个用户赋予权限时，对于数据库可以利用通配符（_和%）指定一类数据库进行操作，这样就可以避免逐个操作啦。
举例如下，假设我们有数据库，

    root@(none) 09:41:16>show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | bp_crm             |
    | dp_0007            |
    | dp_0019            |
    | dp_normandie_0028  |
    | dp_p4p_0082        |
    | dp_p4p_0169        |
    | home               |
    | mysql              |
    | test               |
    +--------------------+
    10 rows in set (0.00 sec)

大家可以看到除了系统db（mysql,information_schema,test）之外，我们有一批报表库是以“dp”开头的，如果我们想创建一个用户，只对这些db可以进行操作，那么可以利用通配符%

    root@(none) 09:52:13>select host,user,password     from mysql.user;
    +------------------------+--------+-------------    ------------------------------+
    | host                   | user   | password                                      |
    +------------------------+--------+-------------    ------------------------------+
    | localhost              | root   |                                               |
    | linezing128042.sqa.cm4 | root   |                                               |
    | 127.0.0.1              | root   |                                               |
    | localhost              |        |                                               |
    | linezing128042.sqa.cm4 |        |                                               |
    | %                      | lzstat |     *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
    | %                      | admin  |     *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
    | %                      | crm    |     *46E75F13B7337A95AAEB7680B6C52280D9CDF5D2 |
    +------------------------+--------+-------------    ------------------------------+
    8 rows in set (0.01 sec)
    
    root@(none) 09:52:17>grant all privileges on     `dp%`.* to dp_admin identified by 'mypasswd';
    Query OK, 0 rows affected (0.00 sec)
    
注意这里不是单引号'，而是反单引号`

    root@(none) 09:53:56>flush privileges;
    Query OK, 0 rows affected (0.01 sec)
    
    root@(none) 09:54:38>select host,user,password     from mysql.user;
    +------------------------+----------+-----------    --------------------------------+
    | host                   | user     | password                                      |
    +------------------------+----------+-----------    --------------------------------+
    | localhost              | root     |                                               |
    | linezing128042.sqa.cm4 | root     |                                               |
    | 127.0.0.1              | root     |                                               |
    | localhost              |          |                                               |
    | linezing128042.sqa.cm4 |          |                                               |
    | %                      | lzstat   |     *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
    | %                      | admin    |     *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
    | %                      | crm      |     *46E75F13B7337A95AAEB7680B6C52280D9CDF5D2 |
    | %                      | dp_admin |     *85E26B8AB29FEE8453201A3511DAE24A24059109 |
    +------------------------+----------+-----------    --------------------------------+
    9 rows in set (0.00 sec)


我们测试一下远程登录，是否可以访问：

    [mysql@testdb2 ~]$ mysql -udp_admin     -h10.232.128.42 -pmypasswd
    
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | dp_0007            |
    | dp_0019            |
    | dp_normandie_0028  |
    | dp_p4p_0082        |
    | dp_p4p_0169        |
    | test               |
    +--------------------+
    7 rows in set (0.00 sec)

可以看到mysql db是无法看到的，这正符合我们的初衷。

    mysql> use dp_0007
    Reading table information for completion of     table and column names
    You can turn off this feature to get a quicker     startup with -A
    
    Database changed
    mysql> show tables;
    +--------------------------------------+
    | Tables_in_dp_0007                    |
    +--------------------------------------+
    | dpunit_p4p_campaign_d__201006        |
    | dpunit_p4p_effect_contrast_d__201006 |
    | dpunit_p4p_effect_contrast_d__201007 |
    | mytest2                              |
    | mytesttab                            |
    +--------------------------------------+
    5 rows in set (0.00 sec)
    
    mysql> select count(*) from     dpunit_p4p_campaign_d__201006;
    +----------+
    | count(*) |
    +----------+
    |    16622 |
    +----------+
    1 row in set (0.00 sec)
    
    mysql> use dp_p4p_0082
    Reading table information for completion of     table and column names
    You can turn off this feature to get a quicker     startup with -A
    
    Database changed
    mysql> show tables;
    +---------------------------------------------+
    | Tables_in_dp_p4p_0082                       |
    +---------------------------------------------+
    | dim_m_star                                  |
    。。。。。
    | dpunit_p4p_platform_d__201006               |
    | dpunit_p4p_platform_d__201007               |
    | dpunit_p4p_platform_d__201008               |
    | lz_dim_category_level1                      |
    | mytest2                                     |
    | mytesttab                                   |
    +---------------------------------------------+
    74 rows in set (0.00 sec)
    
    mysql> select count(*) from     dpunit_p4p_platform_d__201008;
    +----------+
    | count(*) |
    +----------+
    |    38640 |
    +----------+
    1 row in set (0.00 sec)
    
看到可以访问操作“dp”开头的数据库。

注意：_也是通配符，在你grant "dp_p4p"开头的数据库权限时需要用"\"做一下转义。

    root@(none) 10:18:15>grant all privileges on     `dp\_p4p%`.* to dp_admin2 identified by     'mypasswd';
    Query OK, 0 rows affected (0.00 sec)
    
    root@(none) 10:22:41>flush privileges;
    Query OK, 0 rows affected (0.01 sec)
    
    root@(none) 10:22:46>select host,user,password     from mysql.user;
    +------------------------+-----------+----------    ---------------------------------+
    | host                   | user      | password                                      |
    +------------------------+-----------+----------    ---------------------------------+
    | localhost              | root      |                                               |
    | linezing128042.sqa.cm4 | root      |                                               |
    | 127.0.0.1              | root      |                                               |
    | localhost              |           |                                               |
    | linezing128042.sqa.cm4 |           |                                               |
    | %                      | lzstat    |     *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
    | %                      | admin     |     *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
    | %                      | crm       |     *46E75F13B7337A95AAEB7680B6C52280D9CDF5D2 |
    | %                      | dp_admin  |     *85E26B8AB29FEE8453201A3511DAE24A24059109 |
    | %                      | dp_admin2 |     *85E26B8AB29FEE8453201A3511DAE24A24059109 |
    +------------------------+-----------+----------    ---------------------------------+
    10 rows in set (0.00 sec)
    
我们测试一下远程登录，是否可以访问：

    [mysql@testdb2 ~]$ mysql -udp_admin2     -h10.232.128.42 -pmypasswd
    
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | dp_p4p_0082        |
    | dp_p4p_0169        |
    | test               |
    +--------------------+
    4 rows in set (0.00 sec)
    
注意，dp_admin2只有权限看到"dp_p4p"开头的数据库，dp_p4p_0082和dp_p4p_0169

同样你也可以在hostname中指定通配符，但不可以在user中指定：

    root@(none) 10:36:53>grant all privileges on     `dp\_p4p%`.* to dp_admin3@'10.254.3.%'     identified by 'mypasswd';
    Query OK, 0 rows affected (0.00 sec)
    
    root@(none) 10:37:25>flush privileges;
    
表示10.254.3子网段的服务器都可以访问"dp_p4p"这类数据库，注意这里是单引号


另外，使用反勾号(`)为数据库、表、列和子程序名称加引号。使用单引号(')为hostnames、usernames和password加引号。

    root@(none) 10:58:26>grant select on     dp_p4p_0082.`dpunit_p4p_effect_adgroup_bidword_d    __201006` to dp_admin4@'10.254.3.%' identified     by 'mypasswd';
    Query OK, 0 rows affected (0.00 sec)
    
    root@(none) 10:58:22>flush privileges;
    Query OK, 0 rows affected (0.00 sec)

例子中，表用的是反引号`，而给hostname和密码用的是单引号'    