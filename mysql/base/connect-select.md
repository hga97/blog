连接查询: 也可以叫做跨表查询，需要关联多个表进行查询  

### SQL92 语法

显示每个员工信息，并显示所属的部门名称

```bash
select ename, dname from emp, dept;

ENAME DNAME
SMITH ACCOUNTING
ALLEN ACCOUNTING
WARD ACCOUNTING
JONES ACCOUNTING
...
ADAMS OPERATIONS
JAMES OPERATIONS
FORD OPERATIONS
MILLER OPERATIONS

```

**其实就是两个表记录的成绩，这种情况我们称为:“笛卡儿乘积”，**  
**出现错误的 原因是:没有指定连接条件**

指定连接条件

```bash
select emp.ename, dept.dname from emp, dept where emp.deptno = dept.deptno;

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

**以上查询也称为 “内连接”，只查询相等的数据(连接条件相等的数据)**

取得员工和所属的领导的姓名

```bash
select e.ename, m.ename from emp e, emp m where e.mgr = m.empno;

+--------+-------+
| ename  | ename |
+--------+-------+
| SMITH  | FORD  |
| ALLEN  | BLAKE |
| WARD   | BLAKE |
| JONES  | KING  |
| MARTIN | BLAKE |
| BLAKE  | KING  |
| CLARK  | KING  |
| SCOTT  | JONES |
| TURNER | BLAKE |
| ADAMS  | SCOTT |
| JAMES  | BLAKE |
| FORD   | JONES |
| MILLER | CLARK |
+--------+-------+

```

**以上称为“自连接”，只有一张表连接，具体的查询方法，把一张表看作两张表即可，**  
**如以上示例:第一个表 emp e 代表了员工表，emp m 代表了领导表，相当于员工表和部门表一样**


### SQL99 语法

(内连接)显示薪水大于2000的员工信息，并显示所属的部门名称

```bash
采用 SQL92 语法:
select e.ename, e.sal, d.dname from emp e, dept d where e.deptno = d.deptno and e.sal > 2000;

采用 SQL99 语法: 
select e.ename, e.sal, d.dname from emp e join dept d on e.deptno=d.deptno where e.sal>2000;
或
select e.ename, e.sal, d.dname from emp e inner join dept d on e.deptno=d.deptno where e.sal>2000;
在实际中一般不加 inner 关键字

+-------+---------+------------+
| ename | sal     | dname      |
+-------+---------+------------+
| JONES | 2975.00 | RESEARCH   |
| BLAKE | 2850.00 | SALES      |
| CLARK | 2450.00 | ACCOUNTING |
| SCOTT | 3000.00 | RESEARCH   |
| KING  | 5000.00 | ACCOUNTING |
| FORD  | 3000.00 | RESEARCH   |
+-------+---------+------------+

```

**99 语法可以做到表的连接和查询条件分离，特别是多个表进行连接的时候，会比 sql92 更清晰**  

(外连接)显示员工信息，并显示所属的部门名称，如果某一个部门没有员工，那么该部门也必须显示出来

```bash
右连接:
select e.ename, e.sal, d.dname from emp e right join dept d on e.deptno=d.deptno;

左连接:
select e.ename, e.sal, d.dname from dept d left join emp e on e.deptno=d.deptno;

以上两个查询效果相同

+-------+---------+------------+
| ename | sal     | dname      |
+-------+---------+------------+
| JONES | 2975.00 | RESEARCH   |
| BLAKE | 2850.00 | SALES      |
| CLARK | 2450.00 | ACCOUNTING |
| SCOTT | 3000.00 | RESEARCH   |
| KING  | 5000.00 | ACCOUNTING |
| FORD  | 3000.00 | RESEARCH   |
+-------+---------+------------+

```

### 连接分类

1、内链接

表1 inner join 表2 on 关联条件

做连接查询的时候一定要写上关联条件
inner 可以省略

2、外连接

**左外连接**

表1 left outer join 表2 on 关联条件

做连接查询的时候一定要写上关联条件
outer 可以省略

**右外连接**

表1 right outer join 表2 on 关联条件

做连接查询的时候一定要写上关联条件
outer 可以省略

**左外连接(左连接)和右外连接(右连接)的区别**

左连接以左面的表为准和右边的表比较，和左表相等的不相等都会显示出来，右表符合条件的显示,不符合条件的不显示

右连接恰恰相反

左连接能完成的功能右连接一定可以完成  