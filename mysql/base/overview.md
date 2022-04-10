### SQL概述

SQL，一般发音为 sequel，SQL 的全称 Structured Query Language，SQL 用来和数据库打交道，完成和数据库的通信，SQL 是一套标准。但是每一个数据库都有自己的特性别的数据库没有，当使用这个数据库特性相关的功能,这时 SQL 语句可能就不是标准了。(90%以上的 SQL 都是通用的)

### 什么是数据库

数据库，通常是一个或一组文件，保存了一些符合特定规格的数据，数据库对应的英语单词是 DataBase，简称：DB，数据库软件称为数据库管理系统(DBMS)，全称为 DataBase Management System，如：Oracle、SQL Server、MySql、Sybase、informix、DB2、interbase、PostgreSql。

### mysql概述

MySQL 最初是由“MySQL AB”公司开发的一套关系型数据库管理系统(RDBMS-Relational Database Mangerment System)。 MySQL不仅是最流行的开源数据库，而且是业界成长最快的数据库，每天有超过 7 万次的下载量，其应用范围从大型企业到专有的嵌入应用系统。   
MySQL AB 是由两个瑞典人和一个芬兰人:David Axmark、Allan Larsson 和 Michael “Monty” Widenius 在瑞典创办的。
在 2008 年初，Sun Microsystems 收购了 MySQL AB 公司。在 2009 年，Oracle 收购了 Sun 公司，使 MySQL 并入 Oracle 的 数据库产品线。

### 表

表(table)是一种结构化的文件，可以用来存储特定类型的数据，如:学生信息，课程信息，都可以放到表中。另外表都有特定的名称，而且不能重复。表中具有几个概念:列、行、主键。列叫做字段(Column),行叫做表中的记录,每一个字段都有:字段名称/字段数据类型/字段约束/字段长度

学生信息表

|  学号(主键)  |  姓名  |  性别  |  年龄  |
|  ----   | ---- | ----  | ----  |
|  00001  | 张三  | 男  | 20  |
|  00002  | 李四  | 女  | 20  |

### sql的分类

数据查询语言：select  
数据操纵语言：insert、delete、update  
数据定义语言：create、drop、alter  
事务控制语言：commit、rollback  
数据控制语言：grant、revoke

### 表结构描述

表名称：dept  
描述：部门信息表  

| Field  | Type        | Null | Key | Default | Extra |
|  ----  | ----        | ---- | ----| ----    | ----  |
| DEPTNO | int         | NO   | PRI | NULL    |       |
| DNAME  | varchar(14) | YES  |     | NULL    |       |
| LOC    | varchar(13) | YES  |     | NULL    |       |
