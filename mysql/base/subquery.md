子查询就是嵌套的 select 语句，可以理解为子查询是一张表

### where 语句中使用子查询

**在 where 语句中使用子查询，也就是在 where 语句中加入 select 语句**

查询员工信息，查询哪些人是管理者，要求显示出其员工编号和员工姓名

```bash
1、首先取得管理者的编号，去除重复的
select distinct mgr from emp where mgr is not null;

+------+
| mgr  |
+------+
| 7902 |
| 7698 |
| 7839 |
| 7566 |
| 7788 |
| 7782 |
+------+

2、查询员工编号包含管理者编号
select empno, ename from emp where empno in (select distinct mgr from emp where mgr is not null);
+-------+-------+
| empno | ename |
+-------+-------+
|  7902 | FORD  |
|  7698 | BLAKE |
|  7839 | KING  |
|  7566 | JONES |
|  7788 | SCOTT |
|  7782 | CLARK |
+-------+-------+

```

查询哪些人的薪水高于员工的平均薪水，需要显示员工编号，员工姓名，薪水

```bash
1、首先取得员工的平均薪水
select avg(sal) from emp;

+-------------+
| avg(sal)    |
+-------------+
| 2073.214286 |
+-------------+

2、 取得大于平均薪水的员工
select empno, ename, sal from emp where sal > (select avg(sal) from emp);

+-------+-------+---------+
| empno | ename | sal     |
+-------+-------+---------+
|  7566 | JONES | 2975.00 |
|  7698 | BLAKE | 2850.00 |
|  7782 | CLARK | 2450.00 |
|  7788 | SCOTT | 3000.00 |
|  7839 | KING  | 5000.00 |
|  7902 | FORD  | 3000.00 |
+-------+-------+---------+

```

### from 语句中使用子查询

**在 from 语句中使用子查询，可以将该子查询看做一张表**

查询员工信息，查询哪些人是管理者，要求显示出其员工编号和员工姓名 

```bash
1、首先取得管理者的编号，去除重复的
select distinct mgr from emp where mgr is not null;

将以上查询作为一张表，放到 from 语句的后面

select e.empno, e.ename from emp e join (select distinct mgr from emp where mgr is not null) m on e.empno = m.mgr;

+-------+-------+
| empno | ename |
+-------+-------+
|  7902 | FORD  |
|  7698 | BLAKE |
|  7839 | KING  |
|  7566 | JONES |
|  7788 | SCOTT |
|  7782 | CLARK |
+-------+-------+

```

查询各个部门的平均薪水所属等级，需要显示部门编号，平均薪水，等级编号

```bash
1、首先取得各个部门的平均薪水
select deptno, avg(sal) avg_sal from emp group by deptno;

+--------+-------------+
| deptno | avg_sal     |
+--------+-------------+
|     20 | 2175.000000 |
|     30 | 1566.666667 |
|     10 | 2916.666667 |
+--------+-------------+

将以上查询作为一张表，放到 from 语句的后面, 与等级表关联查询

select m.deptno, m.avg_sal, s.grade from SALGRADE s join (select deptno, avg(sal) avg_sal from emp group by deptno) m on m.avg_sal between s.losal and hisal;

+--------+-------------+-------+
| deptno | avg_sal     | grade |
+--------+-------------+-------+
|     20 | 2175.000000 |     4 |
|     30 | 1566.666667 |     3 |
|     10 | 2916.666667 |     4 |
+--------+-------------+-------+

```

### 在 select 语句中使用子查询

查询员工信息，并显示出员工所属的部门名称

```bash
select e.ename, (select d.dname from dept d where e.deptno = d.deptno) as dname from emp e;
    

+--------+------------+
| ename  | dname      |
+--------+------------+
| SMITH  | RESEARCH   |
| ALLEN  | SALES      |
| WARD   | SALES      |
| JONES  | RESEARCH   |
| MARTIN | SALES      |
| BLAKE  | SALES      |
| CLARK  | ACCOUNTING |
| SCOTT  | RESEARCH   |
| KING   | ACCOUNTING |
| TURNER | SALES      |
| ADAMS  | RESEARCH   |
| JAMES  | SALES      |
| FORD   | RESEARCH   |
| MILLER | ACCOUNTING |
+--------+------------+

```