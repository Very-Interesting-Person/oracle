







# 实验1：SQL语句的执行计划分析与优化指导

## 

## 实验目的

分析SQL执行计划，执行SQL语句的优化指导。理解分析SQL语句的执行计划的重要作用。

## 

## 实验内容

- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。

  1.连接上实验需要的Oracle地址：202.115.82.8 

  ​		用户名：system ， 密码123， 数据库名称：pdborcl，端口号：1521

  ![image-连接数据库图示](https://github.com/zenghelin/oracle/blob/main/test1/experience1-1.png)

运行教材的脚本

- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。

查询一：

```oracle
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY d.department_name;
```

查询结果一：

```
已启用自动跟踪
显示执行计划以及语句的统计信息。
DEPARTMENT_NAME                     部门总人数       平均工资
------------------------------ ---------- ----------
IT                                      5       5760 
Sales                                  34 8955.882353 

Plan hash value: 3808327043
 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     1 |    23 |     5  (20)| 00:00:01 |
|   1 |  HASH GROUP BY                |                   |     1 |    23 |     5  (20)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     4   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     4   (0)| 00:00:01 |
|*  4 |     TABLE ACCESS FULL         | DEPARTMENTS       |     2 |    32 |     3   (0)| 00:00:01 |
|*  5 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   6 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   4 - filter("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   5 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Note
-----
   - this is an adaptive plan

   Statistics
-----------------------------------------------------------
               0  user rollbacks
               0  enqueue releases
               0  global enqueue get time
               0  physical read requests optimized
               0  total number of cf enq holders
               0  redo write broadcast ack time
               0  redo write broadcast ack count
               0  redo write broadcast lgwr post count
               0  redo synch time overhead count (  2ms)
               0  redo synch time overhead count (  8ms)

```

查询二：

```oracle
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY d.department_name
HAVING d.department_name in ('IT','Sales');
```

查询结果二：

```oracle
已启用自动跟踪
显示执行计划以及语句的统计信息。
DEPARTMENT_NAME                     部门总人数       平均工资
------------------------------ ---------- ----------
IT                                      5       5760 
Sales                                  34 8955.882353 

Plan hash value: 2128232041
 
----------------------------------------------------------------------------------------------
| Id  | Operation                      | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT               |             |     1 |    23 |     7  (29)| 00:00:01 |
|*  1 |  FILTER                        |             |       |       |            |          |
|   2 |   HASH GROUP BY                |             |     1 |    23 |     7  (29)| 00:00:01 |
|   3 |    MERGE JOIN                  |             |   106 |  2438 |     6  (17)| 00:00:01 |
|   4 |     TABLE ACCESS BY INDEX ROWID| DEPARTMENTS |    27 |   432 |     2   (0)| 00:00:01 |
|   5 |      INDEX FULL SCAN           | DEPT_ID_PK  |    27 |       |     1   (0)| 00:00:01 |
|*  6 |     SORT JOIN                  |             |   107 |   749 |     4  (25)| 00:00:01 |
|   7 |      TABLE ACCESS FULL         | EMPLOYEES   |   107 |   749 |     3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
       filter("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

   Statistics
-----------------------------------------------------------
               0  user rollbacks
               0  enqueue releases
               0  global enqueue get time
               0  physical read requests optimized
               0  total number of cf enq holders
               0  redo write broadcast ack time
               0  redo write broadcast ack count
               0  redo write broadcast lgwr post count
               0  redo synch time overhead count (  2ms)
               0  redo synch time overhead count (  8ms)

```

由对比可发现 查询语句1的CPU占用率要低于查询语句2，故查询语句1执行速度快些。

- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。

自定义查询：

```
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
sum(e.salary)as "总工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY d.department_name
HAVING d.department_name in ('IT','Sales');
```

自定义查询结果:

```
已启用自动跟踪
显示执行计划以及语句的统计信息。
DEPARTMENT_NAME                     部门总人数        总工资
------------------------------ ---------- ----------
IT                                      5      28800 
Sales                                  34     304500 

Plan hash value: 2128232041
 
----------------------------------------------------------------------------------------------
| Id  | Operation                      | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT               |             |     1 |    23 |     7  (29)| 00:00:01 |
|*  1 |  FILTER                        |             |       |       |            |          |
|   2 |   HASH GROUP BY                |             |     1 |    23 |     7  (29)| 00:00:01 |
|   3 |    MERGE JOIN                  |             |   106 |  2438 |     6  (17)| 00:00:01 |
|   4 |     TABLE ACCESS BY INDEX ROWID| DEPARTMENTS |    27 |   432 |     2   (0)| 00:00:01 |
|   5 |      INDEX FULL SCAN           | DEPT_ID_PK  |    27 |       |     1   (0)| 00:00:01 |
|*  6 |     SORT JOIN                  |             |   107 |   749 |     4  (25)| 00:00:01 |
|   7 |      TABLE ACCESS FULL         | EMPLOYEES   |   107 |   749 |     3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
       filter("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

   Statistics
-----------------------------------------------------------
               0  user rollbacks
               0  enqueue releases
               0  global enqueue get time
               0  physical read requests optimized
               0  total number of cf enq holders
               0  redo write broadcast ack time
               0  redo write broadcast ack count
               0  redo write broadcast lgwr post count
               0  redo synch time overhead count (  2ms)
               0  redo synch time overhead count (  8ms)

```



书写查询语句时，涉及到计算的时候用时以及占用的CPU会更多。

## 实验总结



通过这次实验，巩固了sql语句的书写，也学习了新的oracle新知识，如何用SQL developer进行远程连接数据库。以及运行查询脚本。分析sql语句的执行效率，耗时等。也加深了对Oracle数据库的理解。
