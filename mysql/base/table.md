### 创建表

#### 语法格式

```bash
create table tableName (
    columnName dataType(length),
    ...
    columnName dataType(length)
)

set character_set_results = 'gbk';
show variables like '%char%';

```

创建表的时候，表中有字段，每一个字段有：  
**字段名**  
**字段数据类型**  
**字段长度限制**  
**字段约束**  

#### mysql常见数据类型

char： 定长字符串，存储空间大小固定，适合作为主键或外键  
varchar：变长字符串，存储空间等于实际数据空间  
double：数值型  
float：数值型  
int：整型  
bigint：长整型  
date：日期型  
blob：二进制大对象  
clob：字符大对象  

#### 建立学生信息表，字段包括（学号、姓名、性别、出生日期、email、班级标识）

```bash
create table t_student (
    student_id     int(10),
    student_name   varchar(20),
    sex            char(2),
    birthday       date,
    email          varchar(30),
    class_id       int(3)
)

desc t_student;
+--------------+-------------+------+-----+---------+-------+
| Field        | Type        | Null | Key | Default | Extra |
+--------------+-------------+------+-----+---------+-------+
| student_id   | int         | YES  |     | NULL    |       |
| student_name | varchar(20) | YES  |     | NULL    |       |
| sex          | char(2)     | YES  |     | NULL    |       |
| birthday     | date        | YES  |     | NULL    |       |
| email        | varchar(30) | YES  |     | NULL    |       |
| classes_id   | int         | YES  |     | NULL    |       |
+--------------+-------------+------+-----+---------+-------+

```

#### 向t_student表中加入数据

```bash
insert into t_student(student_id, student_name, sex, birthday, email, classes_id) values(1001,
'hhh', 'm', '1988-01-01', 'qqq@163.com', 10);

select * from t_student;

+------------+--------------+------+------------+-------------+------------+
| student_id | student_name | sex  | birthday   | email       | classes_id |
+------------+--------------+------+------------+-------------+------------+
|       1002 | zhangsan     | m    | 1988-01-01 | qqq@163.com |         10 |
+------------+--------------+------+------------+-------------+------------+
```


#### 向 t_student 表中加入数据(使用默认值)

```bash

drop table if exists t_student;

create table t_student(
student_id int(10), student_name varchar(20), sex char(2) default 'm', birthday date,
email varchar(30), classes_id int(3)
)


insert into t_student(student_id, student_name, birthday, email, classes_id) values
(1002, 'zhangsan', '1988-01-01', 'qqq@163.com', 10)

select * from t_student;

+------------+--------------+------+------------+-------------+------------+
| student_id | student_name | sex  | birthday   | email       | classes_id |
+------------+--------------+------+------------+-------------+------------+
|       1002 | zhangsan     | m    | 1988-01-01 | qqq@163.com |         10 |
+------------+--------------+------+------------+-------------+------------+
```


### 增加/删除/修改表结构
采用alter table 来增加/删除/修改表结构，不影响表中的数据

#### 添加表字段

需求发生改变，需要向 t_student 中加入联系电话字段，字段名称为:contatct_tel 类型为 varchar(40)

```bash
alter table t_student add contact_tel varchar(40);

+--------------+-------------+------+-----+---------+-------+
| Field        | Type        | Null | Key | Default | Extra |
+--------------+-------------+------+-----+---------+-------+
| student_id   | int         | YES  |     | NULL    |       |
| student_name | varchar(20) | YES  |     | NULL    |       |
| sex          | char(2)     | YES  |     | m       |       |
| birthday     | date        | YES  |     | NULL    |       |
| email        | varchar(30) | YES  |     | NULL    |       |
| classes_id   | int         | YES  |     | NULL    |       |
| contact_tel  | varchar(40) | YES  |     | NULL    |       |
+--------------+-------------+------+-----+---------+-------+

```

#### 修改字段

student_name 无法满足需求，长度需要更改为 100

```bash
alter table t_student modify student_name varchar(100) ;

+--------------+--------------+------+-----+---------+-------+
| Field        | Type         | Null | Key | Default | Extra |
+--------------+--------------+------+-----+---------+-------+
| student_id   | int          | YES  |     | NULL    |       |
| student_name | varchar(100) | YES  |     | NULL    |       |
| sex          | char(2)      | YES  |     | m       |       |
| birthday     | date         | YES  |     | NULL    |       |
| email        | varchar(30)  | YES  |     | NULL    |       |
| classes_id   | int          | YES  |     | NULL    |       |
| contact_tel  | varchar(40)  | YES  |     | NULL    |       |
+--------------+--------------+------+-----+---------+-------+
```

更改字段（sex -> gender）

```bash
alter table t_student change sex gender char(2) not null;

+--------------+--------------+------+-----+---------+-------+
| Field        | Type         | Null | Key | Default | Extra |
+--------------+--------------+------+-----+---------+-------+
| student_id   | int          | YES  |     | NULL    |       |
| student_name | varchar(100) | YES  |     | NULL    |       |
| gender       | char(2)      | NO   |     | NULL    |       |
| birthday     | date         | YES  |     | NULL    |       |
| email        | varchar(30)  | YES  |     | NULL    |       |
| classes_id   | int          | YES  |     | NULL    |       |
| contact_tel  | varchar(40)  | YES  |     | NULL    |       |
+--------------+--------------+------+-----+---------+-------+

```

#### 删除字段

删除联系电话字段

```bash
alter table t_student drop contact_tel;

+--------------+--------------+------+-----+---------+-------+
| Field        | Type         | Null | Key | Default | Extra |
+--------------+--------------+------+-----+---------+-------+
| student_id   | int          | YES  |     | NULL    |       |
| student_name | varchar(100) | YES  |     | NULL    |       |
| gender       | char(2)      | NO   |     | NULL    |       |
| birthday     | date         | YES  |     | NULL    |       |
| email        | varchar(30)  | YES  |     | NULL    |       |
| classes_id   | int          | YES  |     | NULL    |       |
+--------------+--------------+------+-----+---------+-------+

```

### 添加、修改和删除记录

添加、修改和删除都属于 DML，主要包含的语句:insert、update、delete

#### insert

1、insert 语法格式

insert into 表名(字段,...) values (值,...)

2、指定字段的插入(建议使用此种方式)
    
```bash

insert into emp(empno,ename,job,mgr,hiredate,sal, comm,deptno) values(9999,'zhangsan','MANAGER', null, null,3000, 500, 10);

```

3、表复制

```bash

create table emp_bak as select empno,ename,sal from emp;

```

4 查询的数据直接放到已经存在的表中，可以使用条件

```bash

insert into emp_bak select * from emp where sal=3000;

```

#### update
可以修改数据，可以根据条件修改数据

1、语法格式

update 表名 set 字段名称 1=需要修改的值 1, 字段名称 2=需要修改的值 2 where ...

2、修改学生email

```bash
update t_student set email = '123@163.com' where student_id = 1002;

+------------+--------------+--------+------------+-------------+------------+
| student_id | student_name | gender | birthday   | email       | classes_id |
+------------+--------------+--------+------------+-------------+------------+
|       1002 | zhangsan     | m      | 1988-01-01 | 123@163.com |         10 |
+------------+--------------+--------+------------+-------------+------------+
```

#### delete
可以删除数据，可以根据条件删除数据

1、语法格式

delete from 表名 where ...

2、删除email为'qqq@163.com'的员工

```bash
delete from t_student where email = 'qqq@163.com';

+------------+--------------+--------+------------+-------------+------------+
| student_id | student_name | gender | birthday   | email       | classes_id |
+------------+--------------+--------+------------+-------------+------------+
|       1002 | zhangsan     | m      | 1988-01-01 | 123@163.com |         10 |
+------------+--------------+--------+------------+-------------+------------+
```

### 创建表加入约束

常见的约束：  
非空约束，not null  
唯一约束，unique  
主键约束，primary key  
外键约束，foreign key  

#### 非空约束，not null

非空约束，针对某个字段设置其值不为空，如: 学生的姓名不能为空

```bash
student_name varchar(20) not null

Field 'student_name' doesn't have a default value
```

#### 唯一约束，unique

唯一性约束，它可以使某个字段的值不能重复，如:email 不能重复

```bash
email varchar(30) unique,

Duplicate entry 'qqq@163.com' for key 't_student.email'
```

多个列的 UNIQUE 约束

```bash
alter table t_student add constraint uc_PersonID UNIQUE(student_name,email);

+------------+--------------+------+------------+-------------+------------+
| student_id | student_name | sex  | birthday   | email       | classes_id |
+------------+--------------+------+------------+-------------+------------+
|       1003 | zhangsan     | m    | 1988-01-01 | qqq@163.com |         10 |
|       1002 | zhangsan     | m    | 1988-01-01 | 123@163.com |         10 |
|       1001 | lisi         | m    | 1988-01-01 | 123@163.com |         10 |
+------------+--------------+------+------------+-------------+------------+

```

#### 主键约束，primary key

**每个表应该具有主键，主键可以标识记录的唯一性**  
主键分为单一主键和复合(联合)主键，单一主键是由一个字段构成的，复合(联合)主键是由多个字段构成的  

```bash
create table t_student(
    student_id int(10) primary key,/*列级约束*/ 
    student_name varchar(20) not null,
    sex char(2) default 'm',
    birthday date,
    email varchar(30) ,
    classes_id int(3) 
)

insert into t_student(student_id, student_name, birthday, email, classes_id) values (1001, 'lisi', '1988-01-01', '123@163.com', 10);

select * from t_student;

+------------+--------------+------+------------+-------------+------------+
| student_id | student_name | sex  | birthday   | email       | classes_id |
+------------+--------------+------+------------+-------------+------------+
|       1001 | zhangsan     | m    | 1988-01-01 | qqq@163.com |         10 |
+------------+--------------+------+------------+-------------+------------+

insert into t_student(student_id, student_name, birthday, email, classes_id) values (1001, 'lisi', '1988-01-01', '123@163.com', 10);
Duplicate entry '1001' for key 't_student.PRIMARY'
```


#### 外键约束，foreign key

外键主要是维护表之间的关系的，主要是为了保证参照完整性  
如果表中的某个字段为外键字段，那么该字段的值必须来源于参照的表的主键  
如:emp 中的 deptno 值必须来源于 dept 表中的 deptno 字段值。

建立学生和班级表之间的连接

1、首先建立班级表 t_classes

```bash
drop table if exists t_classes;

create table t_classes(
    classes_id int(3),
    classes_name varchar(40),
    constraint pk_classes_id primary key(classes_id)
);
```

2、在t_student中加入外键约束

```bash
drop table if exists t_student;

create table t_student(
    student_id int(10),
    student_name varchar(20),
    sex char(2),
    birthday date,
    email varchar(30),
    classes_id int(3), 
    constraint student_id_pk primary key(student_id),
    constraint fk_classes_id foreign key(classes_id) references t_classes(classes_id)
)
```

3、向 t_student 中加入数据
```bash
insert into t_student(student_id, student_name, sex, birthday, email, classes_id) values(1001, 'zhangsan', 'm', '1988-01-01', 'qqq@163.com', 10);

 Cannot add or update a child row: a foreign key constraint fails (`bjpowernode`.`t_student`, CONSTRAINT `fk_classes_id` FOREIGN KEY (`classes_id`) REFERENCES `t_classes` (`classes_id`))
 ```

出现错误，因为在班级表中不存在班级编号为 10 班级，外键约束起到了作用。
存在外键的表就是子表，参照的表就是父表，所以存在一个父子关系，也就是主从关系，主表就是班级表，从表就是学生表。

```bash
insert into t_student(student_id, student_name, sex, birthday, email, classes_id) values(1001, 'zhangsan', 'm', '1988-01-01', 'qqq@163.com', null);
+------------+--------------+------+------------+-------------+------------+
| student_id | student_name | sex  | birthday   | email       | classes_id |
+------------+--------------+------+------------+-------------+------------+
|       1001 | zhangsan     | m    | 1988-01-01 | qqq@163.com |       NULL |
+------------+--------------+------+------------+-------------+------------+
 ```
当时 classes_id 没有值，这样会影响参照完整性，所以我们建议将外键字段设置为非空

```bash
drop table if exists t_student;

create table t_student(
    student_id int(10),
    student_name varchar(20),
    sex char(2),
    birthday date,
    email varchar(30),
    classes_id int(3) not null, 
    constraint student_id_pk primary key(student_id),
    constraint fk_classes_id foreign key(classes_id) references t_classes(classes_id)
)

insert into t_student(student_id, student_name, sex, birthday, email, classes_id) values(1001, 'zhangsan', 'm', '1988-01-01', 'qqq@163.com', null);

Column 'classes_id' cannot be null
 ```

4、添加数据到班级表，添加数据到学生表，删除班级数据，将会出现如下错误:

```bash
insert into t_classes(classes_id,classes_name) values (1, '16电子1班');
insert into t_student(student_id, student_name, sex, birthday, email, classes_id ) values(1001, 'zhangsan', 'm', '1988-01-01', 'qqq@163.com', 1);

update t_classes set classes_id = 2 where classes_name = '16电子1班';
Cannot delete or update a parent row: a foreign key constraint fails (`bjpowernode`.`t_student`, CONSTRAINT `fk_classes_id` FOREIGN KEY (`classes_id`) REFERENCES `t_classes` (`classes_id`))
 ```
因为子表(t_student)存在一个外键classes_id，它参照了父表(t_classes)中的主键，所以先删除子表中的引用记录，再修改父表中的数据。


```bash
delete from t_classes where classes_id = 1;
Cannot delete or update a parent row: a foreign key constraint fails (`bjpowernode`.`t_student`, CONSTRAINT `fk_classes_id` FOREIGN KEY (`classes_id`) REFERENCES `t_classes` (`classes_id`))
```
 因为子表(t_student)存在一个外键classes_id，它参照了父表(t_classes)中的主键，所以先删除父表，那么将会影响子表的参照完整性，所以正确的做法是，先删除子表中的数据，再删除父表中的数据，采用drop table也不行，必须先drop子表，再drop父表 我们也可以采取以下措施级联删除。

#### 级联更新与级联删除

1、 on update cascade

mysql 对有些约束的修改比较麻烦，所以我们可以先删除，再添加

```bash
alter table t_student drop foreign key fk_classes_id;
alter table t_student add constraint fk_classes_id_1 foreign key(classes_id) references t_classes(classes_id) on update cascade;

update t_classes set classes_id = 2 where classes_name = '16电子1班';

select * from t_classes;
+------------+--------------+
| classes_id | classes_name |
+------------+--------------+
|          2 | 16电子1班    |
+------------+--------------+

select * from t_student;
+------------+--------------+------+------------+-------------+------------+
| student_id | student_name | sex  | birthday   | email       | classes_id |
+------------+--------------+------+------------+-------------+------------+
|       1001 | zhangsan     | m    | 1988-01-01 | qqq@163.com |          2 |
+------------+--------------+------+------------+-------------+------------+

修改了父表中的数据，但是子表中的数据也会跟着变动。
```

2、 on delete cascade

mysql 对有些约束的修改时不支持的，所以我们可以先删除，再添加

```bash
alter table t_student drop foreign key fk_classes_id_1;
alter table t_student add constraint fk_classes_id_1 foreign key(classes_id) references t_classes(classes_id) on delete cascade;

delete from t_classes where classes_id = 2;

select * from t_classes;
Empty set (0.00 sec)

select * from t_student;
Empty set (0.00 sec)

只删除了父表中的数据，但是子表也会中的数据也会删除。
```

### t_student和t_classes完整实例

```bash
drop table if exists t_classes;


create table t_classes(
    classes_id int(3),
    classes_name varchar(40) not null,
    constraint pk_classes_id primary key(classes_id)
);

drop table if exists t_student;

create table t_student(
    student_id int(10),
    student_name varchar(20),
    sex char(2),
    birthday date,
    email varchar(30),
    classes_id int(3), 
    constraint student_id_pk primary key(student_id),
    constraint fk_classes_id foreign key(classes_id) references t_classes(classes_id)
)
```

### 增加/删除/修改表约束

#### 删除约束

删除外键约束： alter table 表名 drop foreign key 外键（区分大小写）

删除主键约束： alter table 表名 drop primary key

删除约束： alter table 表名 drop key 约束名称

#### 添加约束

添加外键约束：alter table 表名 add constraint 约束名 foreign key(从表字段) references 主表(主键字段)

添加主键约束：alter table 表名 add constraint 约束名 foreign key 表(字段)

添加唯一约束：alter table 表名 add constraint 约束名 unique 表(字段)

#### 修改约束，其实就是修改字段

alter table t_student modify student_name varchar(30) unique;

mysql 对有些约束的修改时不支持，所以我们可以先删除，再添加