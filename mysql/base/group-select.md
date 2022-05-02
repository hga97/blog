分组查询主要涉及到两个子句，分别是:groupby 和 having

### group by

取得每个工作岗位的工资合计，要求显示岗位名称和工资合计

```bash
select job, sum(sal) from emp group by job;

+-----------+----------+
| job       | sum(sal) |
+-----------+----------+
| CLERK     |  4150.00 |
| SALESMAN  |  5600.00 |
| MANAGER   |  8275.00 |
| ANALYST   |  6000.00 |
| PRESIDENT |  5000.00 |
+-----------+----------+

```

按照工作岗位和部门编码分组，取得的工资合计

```bash
select job,deptno,sum(sal) from emp group by job,deptno;

+-----------+--------+----------+
| job       | deptno | sum(sal) |
+-----------+--------+----------+
| CLERK     |     20 |  1900.00 |
| SALESMAN  |     30 |  5600.00 |
| MANAGER   |     20 |  2975.00 |
| MANAGER   |     30 |  2850.00 |
| MANAGER   |     10 |  2450.00 |
| ANALYST   |     20 |  6000.00 |
| PRESIDENT |     10 |  5000.00 |
| CLERK     |     30 |   950.00 |
| CLERK     |     10 |  1300.00 |
+-----------+--------+----------+

```

### having
如果想对分组数据再进行过滤需要使用 having 子句  

取得每个岗位的平均工资大于 2000

```bash
select job, avg(sal) from emp group by job having avg(sal) > 2000;

+-----------+-------------+
| job       | avg(sal)    |
+-----------+-------------+
| MANAGER   | 2758.333333 |
| ANALYST   | 3000.000000 |
| PRESIDENT | 5000.000000 |
+-----------+-------------+

```

分组函数的执行顺序:   
根据条件查询数据  
分组  
采用 having 过滤，取得正确的数据  


### select 语句总结

一个完整的 select 语句格式如下 

?>
select 字段  
from 表名  
where .......  
group by ........  
having .......(就是为了过滤分组后的数据而存在的—不可以单独的出现)  
orderby........  

以上语句的执行顺序   
1. 首先执行 where 语句过滤原始数据  
2. 执行 group by 进行分组  
3. 执行 having 对分组数据进行操作  
4. 执行 select 选出数据  
5. 执行order by排序  

**原则:能在 where 中过滤的数据，尽量在 where 中过滤，效率较高。having 的过滤是专门对分组之后的数据进行过滤的。**