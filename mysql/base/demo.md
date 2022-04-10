### 查看和指定现有的数据库

```bash
show databases;

+--------------------+
| Database           |
+--------------------+
| bjpowernode        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
```

### 使用数据库

```bash
use bjpowernode;

Database changed
```

### 查看当前使用的库

```bash
select database();

+-------------+
| database()  |
+-------------+
| bjpowernode |
+-------------+
```

### 查看当前库中的表

```bash
show tables;

+-----------------------+
| Tables_in_bjpowernode |
+-----------------------+
| DEPT                  |
| EMP                   |
| SALGRADE              |
+-----------------------+
```

### 查看表结构

```bash
desc emp;

+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| EMPNO    | int         | NO   | PRI | NULL    |       |
| ENAME    | varchar(10) | YES  |     | NULL    |       |
| JOB      | varchar(9)  | YES  |     | NULL    |       |
| MGR      | int         | YES  |     | NULL    |       |
| HIREDATE | date        | YES  |     | NULL    |       |
| SAL      | double(7,2) | YES  |     | NULL    |       |
| COMM     | double(7,2) | YES  |     | NULL    |       |
| DEPTNO   | int         | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
```

### 查看表的创建语句

```bash
show create table emp;

CREATE TABLE `emp` (
  `EMPNO` int NOT NULL,
  `ENAME` varchar(10) DEFAULT NULL,
  `JOB` varchar(9) DEFAULT NULL,
  `MGR` int DEFAULT NULL,
  `HIREDATE` date DEFAULT NULL,
  `SAL` double(7,2) DEFAULT NULL,
  `COMM` double(7,2) DEFAULT NULL,
  `DEPTNO` int DEFAULT NULL,
  PRIMARY KEY (`EMPNO`)
) 
```
