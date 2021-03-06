### 查询一个字段

查询员工的姓名
```bash
select ename from emp;

+--------+
| ename  |
+--------+
| SMITH  |
| ALLEN  |
| WARD   |
| JONES  |
| MARTIN |
| BLAKE  |
| CLARK  |
| SCOTT  |
| KING   |
| TURNER |
| ADAMS  |
| JAMES  |
| FORD   |
| MILLER |
+--------+
```

### 查询多个字段

查询员工的编号和姓名
```bash
select empno, ename from emp;

+-------+--------+
| empno | ename  |
+-------+--------+
|  7369 | SMITH  |
|  7499 | ALLEN  |
|  7521 | WARD   |
|  7566 | JONES  |
|  7654 | MARTIN |
|  7698 | BLAKE  |
|  7782 | CLARK  |
|  7788 | SCOTT  |
|  7839 | KING   |
|  7844 | TURNER |
|  7876 | ADAMS  |
|  7900 | JAMES  |
|  7902 | FORD   |
|  7934 | MILLER |
+-------+--------+
```

### 查询全部字段

```bash
select * from emp;

+-------+--------+-----------+------+------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+------------+---------+---------+--------+
```
采用 select * from emp，虽然简单，但是*号不是很明确，建议查询全部字段将相关字段写到 select 语句的后面, 可读性强。

### 计算员工的年薪

列出员工的编号，姓名和年薪

```bash
select empno,ename,sal*12 from emp;

可以采用 as 关键字重命名表字段

select empno as '员工编号', ename as '员工姓名', sal*12 as '年薪' from emp;

+--------------+--------------+----------+
| 员工编号     | 员工姓名     | 年薪     |
+--------------+--------------+----------+
|         7369 | SMITH        |  9600.00 |
|         7499 | ALLEN        | 19200.00 |
|         7521 | WARD         | 15000.00 |
|         7566 | JONES        | 35700.00 |
|         7654 | MARTIN       | 15000.00 |
|         7698 | BLAKE        | 34200.00 |
|         7782 | CLARK        | 29400.00 |
|         7788 | SCOTT        | 36000.00 |
|         7839 | KING         | 60000.00 |
|         7844 | TURNER       | 18000.00 |
|         7876 | ADAMS        | 13200.00 |
|         7900 | JAMES        | 11400.00 |
|         7902 | FORD         | 36000.00 |
|         7934 | MILLER       | 15600.00 |
+--------------+--------------+----------+
```