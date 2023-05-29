# 提供参考的数据表
## dept部门表
  部门编号    部门名称      位置  
+--------+------------+----------+
| DEPTNO |   DNAME    |   LOC    |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
## emp员工表
 员工编号  员工姓名  工作       领导编号  就职日期      工资      补贴      部门编号
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
## salgrade薪资表
 薪资等级 最低工资 最高工资
+-------+-------+-------+
| GRADE | LOSAL | HISAL |
+-------+-------+-------+
|     1 |   700 |  1200 |
|     2 |  1201 |  1400 |
|     3 |  1401 |  2000 |
|     4 |  2001 |  3000 |
|     5 |  3001 |  9999 |
+-------+-------+-------+
# 经典Mysql34道面试题
## 1、取得每个部门 / 最高薪水的 / 人员名称
第一步：先按部门分组，从每个部门组中查找最高薪水（每个最高薪水一定只有一个，但获得最高薪水的人员可能有多个）
错误点：先按部门分组，从每个部门组中查找最高薪水，直接查找出对应的员工姓名（ 此时只能查找出一个最高薪资的员工 ）
select ename,max(sal),deptno from emp group by deptno;
mysql> select max(sal),deptno    // 不能在第一步直接查找出员工（数据不完全）
    -> from emp
    -> group by deptno
    -> ;
  +----------+--------+
  | max(sal) | deptno |
  +----------+--------+
  |  5000.00 |     10 |
  |  3000.00 |     20 |
  |  2850.00 |     30 |
  +----------+--------+
第二步：将以上的查询结果当做一张临时表t，最后再查找出对应的员工姓名（t和emp表连接查询，条件：部门编号相等，员工薪资相等）
mysql> select e.ename,t.*
    -> from emp e
    -> join (select max(sal) as maxsal,deptno from emp group by deptno) t  //先将部门最高薪资取出作为临时表
    -> on t.deptno = e.deptno  //再使用条件过滤出最高薪资的员工（最高薪资员工有多个）
    -> and t.maxsal = e.sal ;
  +-------+--------+---------+
  | ename | deptno | maxsal  |
  +-------+--------+---------+
  | BLAKE |     30 | 2850.00 |
  | SCOTT |     20 | 3000.00 |
  | KING  |     10 | 5000.00 |
  | FORD  |     20 | 3000.00 |
  +-------+--------+---------+
## 2、哪些人的薪水在部门的平均薪水之上
第一步：先根据部门分组，找出每个部门组的平均薪水
mysql> select deptno,avg(sal) as avgsal
    -> from emp
    -> group by deptno;
+--------+-------------+
| deptno | avgsal      |
+--------+-------------+
|     10 | 2916.666667 |
|     20 | 2175.000000 |
|     30 | 1566.666667 |
+--------+-------------+
第二步：将以上的查询结果当做一张临时表t，最后查找出部门中薪资大于部门平均薪资的员工（t和emp表连接查询，条件：部门编号相同，同部门的额员工薪资 大于部门平均薪资）
mysql> select t.*, e.ename, e.sal
    -> from emp e
    -> join (select deptno,avg(sal) as avgsal from emp group by deptno) t
    -> on e.deptno = t.deptno and e.sal > t.avgsal;
+--------+-------------+-------+---------+
| deptno | avgsal      | ename | sal     |
+--------+-------------+-------+---------+
|     30 | 1566.666667 | ALLEN | 1600.00 |
|     20 | 2175.000000 | JONES | 2975.00 |
|     30 | 1566.666667 | BLAKE | 2850.00 |
|     20 | 2175.000000 | SCOTT | 3000.00 |
|     10 | 2916.666667 | KING  | 5000.00 |
|     20 | 2175.000000 | FORD  | 3000.00 |
+--------+-------------+-------+---------+
## 3、取得各部门中 平均的薪水等级
平均的 薪水等级：先计算每一个部门 每一个员工薪水的等级，然后找出薪水等级的平均值；
平均薪水 的等级：先计算平均薪水，然后找出每个平均薪水的等级值。
第一步：找出每个人的薪水等级，员工表和薪资等级表连接，连接条件是员工薪资处于薪资表薪资的等级划分。
mysql> select e.ename,e.sal,e.deptno,s.grade
    -> from emp e
    -> join salgrade s
    -> on e.sal between s.losal and s.hisal;
    +--------+---------+--------+-------+
    | ename  | sal     | deptno | grade |
    +--------+---------+--------+-------+
    | CLARK  | 2450.00 |     10 |     4 |
    | KING   | 5000.00 |     10 |     5 |
    | MILLER | 1300.00 |     10 |     2 |
    | SMITH  |  800.00 |     20 |     1 |
    | ADAMS  | 1100.00 |     20 |     1 |
    | SCOTT  | 3000.00 |     20 |     4 |
    | FORD   | 3000.00 |     20 |     4 |
    | JONES  | 2975.00 |     20 |     4 |
    | MARTIN | 1250.00 |     30 |     2 |
    | TURNER | 1500.00 |     30 |     3 |
    | BLAKE  | 2850.00 |     30 |     4 |
    | ALLEN  | 1600.00 |     30 |     3 |
    | JAMES  |  950.00 |     30 |     1 |
    | WARD   | 1250.00 |     30 |     2 |
    +--------+---------+--------+-------+
第二步：求得每个人的薪水等级后，继续按照部门编号deptno对员工分组，求每一个薪资等级grade的平均值
mysql> select  e.deptno,avg(s.grade)
    -> from emp e
    -> join salgrade s
    -> on e.sal between s.losal and s.hisal
    -> group by e.deptno;
    +--------+--------------+
    | deptno | avg(s.grade) |
    +--------+--------------+
    |     10 |       3.6667 |
    |     20 |       2.8000 |
    |     30 |       2.5000 |
    +--------+--------------+
## 4、不准用组函数（Max ），取得最高薪水
第一种：对所有员工的薪资sal进行降序，取第一位数据limit 1
mysql> select ename,sal from emp order by sal desc limit 1;
+-------+---------+
| ename | sal     |
+-------+---------+
| KING  | 5000.00 |
+-------+---------+
第二种方案：组函数，select max(sal) from emp;

第三种方案：表的自连接

mysql> select sal
    -> from emp
    -> where sal not in(
    ->    select distinct a.sal
    ->    from emp a
    ->    join emp b
    ->    on a.sal < b.sal) ;
+---------+
| sal     |
+---------+
| 5000.00 |
+---------+
## 5、取得平均薪水最高的部门 的部门编号
第一种方案：降序取第一个
// 第一步：按照部门分组，找出每个部门的平均薪水
mysql> select deptno,avg(sal) as vagsal
    -> from emp
    -> group by deptno;
        +--------+-------------+
        | deptno | avgsal      |
        +--------+-------------+
        |     10 | 2916.666667 |
        |     20 | 2175.000000 |
        |     30 | 1566.666667 |
        +--------+-------------+

// 第二步：对平均薪水降序，选取第一个
mysql> select deptno,avg(sal) as vagsal
    -> from emp
    -> group by deptno
    -> order by avgsal desc 
    -> limit 1;
        +--------+-------------+
        | deptno | avgsal      |
        +--------+-------------+
        |     10 | 2916.666667 |
        +--------+-------------+
第二种方案：使用组函数max()
// 第一步：找出每个部门的平均薪水
mysql> select deptno,avg(sal) as avgsal
    -> from emp
    -> group by deptno;
    +--------+-------------+
    | deptno | avgsal      |
    +--------+-------------+
    |     10 | 2916.666667 |
    |     20 | 2175.000000 |
    |     30 | 1566.666667 |
    +--------+-------------+

// 第二步：从部门平均薪水avgsal中 找出最大的值
mysql> select max(t.avgsal)
    -> from (select avg(sal) as avgsal
    ->       from emp
    ->       group by deptno) t;
    +---------------+
    | max(t.avgsal) |
    +---------------+
    |   2916.666667 |
    +---------------+

/// 第三步：基于第二步找出平均薪资最大值 的部门编号
mysql> select deptno,avg(sal) as avgsal
    -> from emp
    -> group by deptno
    -> having avgsal = (select max(t.avgsal) 
    ->                 from (select avg(sal) as avgsal 
    ->                       from emp 
    ->                       group by deptno) t
    ->                 );   
    +--------+-------------+
    | deptno | avgsal      |
    +--------+-------------+
    |     10 | 2916.666667 |
    +--------+-------------+
## 6、取得平均薪水最高的部门的部门名称
第一步：按照部门分组
第二步：求每个部门的平均薪水，使用组函数max()取最大平均薪水的部门名称
mysql> select d.dname,avg(e.sal) as avgsal
    -> from emp e
    -> join dept d
    -> on e.deptno = d.deptno
    -> group by d.dname
    -> order by avgsal desc
    -> limit 1;
+------------+-------------+
| dname      | avgsal      |
+------------+-------------+
| ACCOUNTING | 2916.666667 |
+------------+-------------+
## 7、求平均薪水的等级 最低的部门的 部门名称
第一步：找出每个部门的平均薪水
mysql> select deptno,avg(sal) as avgsal
    -> from emp
    -> group by deptno;
+--------+-------------+
| deptno | avgsal      |
+--------+-------------+
|     10 | 2916.666667 |
|     20 | 2175.000000 |
|     30 | 1566.666667 |
+--------+-------------+
第二步：找出 部门平均薪水的等级，部门平均薪水表和薪资等级表连接，条件是部门平均薪水在薪资等级表的薪水等级之间
mysql> select  t.*,s.grade
    -> from (select d.dname,avg(sal) as avgsal 
   ->        from emp e 
   ->        join dept d 
   ->        on e.deptno = d.deptno 
   ->        group by d.dname)  t
   -> join salgrade s
   -> on t.avgsal between s.losal and s.hisal;
+------------+-------------+-------+
| dname      | avgsal      | grade |
+------------+-------------+-------+
| SALES      | 1566.666667 |     3 |
| ACCOUNTING | 2916.666667 |     4 |
| RESEARCH   | 2175.000000 |     4 |
+------------+-------------+-------+

// 第三步：对以上结果 取等级的最小值
## 8、取得比普通员工的最高薪水还要高的领导人姓名(普通员工即 员工编号没有在mgr字段上出现的)
// 第一步：查询所有的领导编号
mysql> select distinct mgr from emp where mgr is not null;
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

// 第二步：找出普通员工中的最高薪水，员工编号不在领导编号表上的就是普通员工
// not in在使用时，后面小括号中记得排除NULL
mysql> select max(sal)
    -> from emp
    -> where empno not in(select distinct mgr
    ->                    from emp
    ->                    where mgr is not null);
+----------+
| max(sal) |
+----------+
|  1600.00 |
+----------+

// 第三步：从领导表中 找出领导薪资高于1600的
mysql> select ename,sal
    -> from emp
    -> where sal > (select max(sal)
    ->              from emp
    ->              where empno not in(select distinct mgr
    ->                                 from emp
    ->                                 where mgr is not null)
    ->              );
+-------+---------+
| ename | sal     |
+-------+---------+
| JONES | 2975.00 |
| BLAKE | 2850.00 |
| CLARK | 2450.00 |
| SCOTT | 3000.00 |
| KING  | 5000.00 |
| FORD  | 3000.00 |
+-------+---------+
## 9、取得薪水最高的前五名员工
// 对所有员工的薪水排序，取前五名
mysql> select ename,sal
    -> from emp
    -> order by sal desc
    -> limit 5;
+-------+---------+
| ename | sal     |
+-------+---------+
| KING  | 5000.00 |
| SCOTT | 3000.00 |
| FORD  | 3000.00 |
| JONES | 2975.00 |
| BLAKE | 2850.00 |
+-------+---------+
## 10、取得薪水最高的第六到第十名员工
// 对所有员工的薪水排序，取第六到第十名
mysql> select ename,sal
    -> from emp
    -> order by sal desc
    -> limit 5,5;
+--------+---------+
| ename  | sal     |
+--------+---------+
| CLARK  | 2450.00 |
| ALLEN  | 1600.00 |
| TURNER | 1500.00 |
| MILLER | 1300.00 |
| MARTIN | 1250.00 |
+--------+---------+
## 11、取得最后入职的 5 名员工
// 对入职日期进行排序
mysql> select ename,hiredate
    -> from emp
    -> order by hiredate desc
    -> limit 5;
    +--------+------------+
    | ename  | hiredate   |
    +--------+------------+
    | ADAMS  | 1987-05-23 |
    | SCOTT  | 1987-04-19 |
    | MILLER | 1982-01-23 |
    | FORD   | 1981-12-03 |
    | JAMES  | 1981-12-03 |
    +--------+------------+
## 12、取得每个薪水等级有多少员工
第一步：找出每个员工的薪水等级
mysql> select e.ename,e.sal,s.grade
    -> from emp e
    -> join salgrade s
    -> on e.sal between s.losal and s.hisal;
+--------+---------+-------+
| ename  | sal     | grade |
+--------+---------+-------+
| SMITH  |  800.00 |     1 |
| ALLEN  | 1600.00 |     3 |
| WARD   | 1250.00 |     2 |
| JONES  | 2975.00 |     4 |
| MARTIN | 1250.00 |     2 |
| BLAKE  | 2850.00 |     4 |
| CLARK  | 2450.00 |     4 |
| SCOTT  | 3000.00 |     4 |
| KING   | 5000.00 |     5 |
| TURNER | 1500.00 |     3 |
| ADAMS  | 1100.00 |     1 |
| JAMES  |  950.00 |     1 |
| FORD   | 3000.00 |     4 |
| MILLER | 1300.00 |     2 |
+--------+---------+-------+
第二步：按薪水等级分组，对每个员工的薪水等级grade 进行统计
mysql> select
    -> s.grade ,count(*)
    -> from emp e
    -> join salgrade s
    -> on e.sal between s.losal and s.hisal
    -> group by s.grade;
+-------+----------+
| grade | count(*) |
+-------+----------+
|     1 |        3 |
|     2 |        3 |
|     3 |        2 |
|     4 |        5 |
|     5 |        1 |
+-------+----------+
## 13、面试题：
有 3 个表 S(学生表)，C（课程表），SC（学生选课表）
S（SNO，SNAME）代表（学号，姓名）
C（CNO，CNAME，CTEACHER）代表（课号，课名，教师）
SC（SNO，CNO，SCGRADE）代表（学号，课号，成绩）
问题：
1，找出没选过“黎明”老师的所有学生姓名。
2，列出 2 门以上（含2 门）不及格学生姓名及平均成绩。
3，即学过 1 号课程又学过 2 号课所有学生的姓名。

## 14、列出所有员工及领导的姓名
// 员工的领导编号 = 领导的员工编号（领导也是员工）
mysql> select a.ename '员工', b.ename '领导'
    -> from emp a
    -> left join emp b
    -> on a.mgr = b.empno;
+--------+-------+
| 员工   | 领导    |
+--------+-------+
| SMITH  | FORD  |
| ALLEN  | BLAKE |
| WARD   | BLAKE |
| JONES  | KING  |
| MARTIN | BLAKE |
| BLAKE  | KING  |
| CLARK  | KING  |
| SCOTT  | JONES |
| KING   | NULL  |
| TURNER | BLAKE |
| ADAMS  | SCOTT |
| JAMES  | BLAKE |
| FORD   | JONES |
| MILLER | CLARK |
+--------+-------+
## 15、列出受雇日期早于其直接上级的 所有员工的编号,姓名,部门名称
// 第一步：找出所有的员工和就职日期
// 第二步：找出所有领导和就职日期
// 第三步：员工就职日 早于 其领导就职日期
// 关键点：a.mgr = b.empno and a.hiredate < b.hiredate
mysql> select  a.ename '员工', a.hiredate '员工就职', b.ename '领导', b.hiredate '领导就职', d.dname '部门'
    -> from emp a
    -> join emp b
    -> on a.mgr = b.empno   // 员工的领导编号 = 领导的员工编号    
    -> join dept d
    -> on a.deptno = d.deptno
    -> where  a.hiredate < b.hiredate;  // 员工就职日期 < 领导就职日期
+-------+------------+-------+------------+------------+
| 员工     | hiredate   | 领导    | hiredate   | dname      |
+-------+------------+-------+------------+------------+
| CLARK | 1981-06-09 | KING  | 1981-11-17 | ACCOUNTING |
| SMITH | 1980-12-17 | FORD  | 1981-12-03 | RESEARCH   |
| JONES | 1981-04-02 | KING  | 1981-11-17 | RESEARCH   |
| ALLEN | 1981-02-20 | BLAKE | 1981-05-01 | SALES      |
| WARD  | 1981-02-22 | BLAKE | 1981-05-01 | SALES      |
| BLAKE | 1981-05-01 | KING  | 1981-11-17 | SALES      |
+-------+------------+-------+------------+------------+
## 16、 列出部门名称和这些部门的员工信息, 同时列出那些没有员工的部门
// 第一步：通过部门编号相等，将部门表连接员工表
// 第二步：以部门表为主
mysql> select e.*,d.dname
    -> from emp e
    -> right join dept d
    -> on e.deptno = d.deptno;
+-------+--------+-----------+------+------------+---------+---------+--------+------------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO | dname      |
+-------+--------+-----------+------+------------+---------+---------+--------+------------+
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 | ACCOUNTING |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 | ACCOUNTING |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 | ACCOUNTING |
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 | RESEARCH   |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 | RESEARCH   |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 | RESEARCH   |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 | RESEARCH   |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 | RESEARCH   |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 | SALES      |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 | SALES      |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 | SALES      |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 | SALES      |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 | SALES      |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 | SALES      |
|  NULL | NULL   | NULL      | NULL | NULL       |    NULL |    NULL |   NULL | OPERATIONS |
+-------+--------+-----------+------+------------+---------+---------+--------+------------+
## 17、列出至少有 5 个员工的所有部门
// 第一步：先按照部门编号分组
// 第二步：对分组后的部门进行统计数量，在此基础上having筛选出 >= 5
mysql> select deptno,count(*) as 员工数量
    -> from emp
    -> group by deptno
    ->  having count(*) >= 5;
+--------+----------+
| deptno | 员工数量 |
+--------+----------+
|     20 |        5 |
|     30 |        6 |
+--------+----------+
## 18、列出薪金比"SMITH" 多的所有员工信息
// 第一步：找出SMITH的薪资信息
mysql> select sal 
    -> from emp 
    -> where ename = 'SMITH';
+--------+
| sal    |
+--------+
| 800.00 |
+--------+
// 第二步：在 SMITH的薪资信息 的基础上找出大于SMITH薪资 的员工信息
mysql> select ename,sal
    -> from emp
    -> where sal > (select sal from emp where ename = 'SMITH');
+--------+---------+
| ename  | sal     |
+--------+---------+
| ALLEN  | 1600.00 |
| WARD   | 1250.00 |
| JONES  | 2975.00 |
| MARTIN | 1250.00 |
| BLAKE  | 2850.00 |
| CLARK  | 2450.00 |
| SCOTT  | 3000.00 |
| KING   | 5000.00 |
| TURNER | 1500.00 |
| ADAMS  | 1100.00 |
| JAMES  |  950.00 |
| FORD   | 3000.00 |
| MILLER | 1300.00 |
+--------+---------+
## 19、 列出所有从事"CLERK"( 办事员) 的姓名及其部门名称，部门的人数
// 第一步：找出工种为"CLERK"( 办事员) 的员工姓名和部门编号
mysql> select ename,job,deptno
    -> from emp
    -> where job = 'CLERK';
+--------+-------+
| ename  | job   |
+--------+-------+
| SMITH  | CLERK |
| ADAMS  | CLERK |
| JAMES  | CLERK |
| MILLER | CLERK |
+--------+-------+

// 第二步：在以上基础上 连接部门表 找出员工所在的部门名称（员工部门编号 = 部门的部门编号）
mysql> select e.ename,e.job,e.deptno,d.dname
    -> from emp e
    -> join dept d
    -> on e.deptno = d.deptno
    -> where e.job = 'CLERK';
+--------+-------+------------+
| ename  | job   | dname      |
+--------+-------+------------+
| MILLER | CLERK | ACCOUNTING |
| SMITH  | CLERK | RESEARCH   |
| ADAMS  | CLERK | RESEARCH   |
| JAMES  | CLERK | SALES      |
+--------+-------+------------+

// 第三步：对该部门进行人数统计
mysql> select deptno, count(*) as deptcount
    -> from emp
    -> group by deptno;
+--------+-----------+
| deptno | deptcount |
+--------+-----------+
|     10 |         3 |
|     20 |         5 |
|     30 |         6 |
+--------+-----------+

mysql> select t1.*,t2.deptcount  
    -> from (select e.ename,e.job,d.dname,d.deptno
    ->       from emp e
    ->       join dept d
    ->       on e.deptno = d.deptno
    ->       where e.job = 'CLERK') t1    // "CLERK"办事员与其部门的信息表
    ->       join (select deptno, count(*) as deptcount 
    ->             from emp group by deptno) t2    // 部门人数统计表
    ->       on t1.deptno = t2.deptno;

+--------+-------+------------+--------+-----------+
| ename  | job   | dname      | deptno | deptcount |
+--------+-------+------------+--------+-----------+
| MILLER | CLERK | ACCOUNTING |     10 |         3 |
| SMITH  | CLERK | RESEARCH   |     20 |         5 |
| ADAMS  | CLERK | RESEARCH   |     20 |         5 |
| JAMES  | CLERK | SALES      |     30 |         6 |
+--------+-------+------------+--------+-----------+
## 20、列出 最低薪金大于1500的各种工作 及从事此工作的全部雇员人数
// 第一步：先根据工种分组，求最低薪资 >1500 的每一个工种
mysql> select job,min(sal) as minsal
    -> from emp
    -> group by job
    -> having min(Sal) > 1500;
+-----------+---------+
| job       | minsal  |
+-----------+---------+
| ANALYST   | 3000.00 |
| MANAGER   | 2450.00 |
| PRESIDENT | 5000.00 |
+-----------+---------+

// 第二步：统计 从事以上工种的人数
mysql> select job,min(sal) as minsal,count(*)
    -> from emp
    -> group by job
    -> having min(sal) > 1500;
+-----------+---------+----------+
| job       | minsal  | count(*) |
+-----------+---------+----------+
| ANALYST   | 3000.00 |        2 |
| MANAGER   | 2450.00 |        3 |
| PRESIDENT | 5000.00 |        1 |
+-----------+---------+----------+
## 21、列出在部门"SALES"< 销售部> 工作的员工的姓名，假定不知道销售部的部门编号
// 第一步：找出"SALES"< 销售部> 的部门编号
mysql> select deptno
    -> from dept
    -> where dname = 'SALES';
+--------+
| deptno |
+--------+
|     30 |
+--------+

// 第二步：根据"SALES"< 销售部> 的部门编号 找出在该部门工作的员工
mysql> select ename,deptno
    -> from emp
    -> where deptno = (select deptno from dept where dname = 'SALES');
+--------+--------+
| ename  | deptno |
+--------+--------+
| ALLEN  |     30 |
| WARD   |     30 |
| MARTIN |     30 |
| BLAKE  |     30 |
| TURNER |     30 |
| JAMES  |     30 |
+--------+--------+
## 22、列出薪金 高于公司平均薪金的 所有员工、所在部门、上级领导、雇员的工资等级。
// 第一步：先求公司的平均薪金
select avg(sal) as avgsal
from emp;
mysql> select avg(sal) as avgsal
    -> from emp;
+-------------+
| avgsal      |
+-------------+
| 2073.214286 |
+-------------+

// 第二步：找出高于 公司平均薪资 的员工、部门、上级领导及员工薪资等级（连接部门表，薪资等级表）
mysql> select e.ename as '员工',d.dname as '部门',l.ename as '领导',s.grade as '薪资等级'
    -> from emp e
    -> join dept d
    -> on e.deptno = d.deptno
    -> left join emp l
    -> on e.mgr = l.empno  // 找出员工的领导
    -> join salgrade s
    -> on e.sal between s.losal and s.hisal  // 找出高于公司平均薪资的员工薪资等级
    -> where e.sal > (select avg(sal) from emp);
+-------+------------+-------+----------+
| 员工  | 部门       | 领导  | 薪资等级 |
+-------+------------+-------+----------+
| JONES | RESEARCH   | KING  |        4 |
| BLAKE | SALES      | KING  |        4 |
| CLARK | ACCOUNTING | KING  |        4 |
| SCOTT | RESEARCH   | JONES |        4 |
| KING  | ACCOUNTING | NULL  |        5 |
| FORD  | RESEARCH   | JONES |        4 |
+-------+------------+-------+----------+
## 23、列出与"SCOTT"从事相同工作的所有员工及部门名称
// 第一步：先找出"SCOTT"从事的工作
mysql> select job
    -> from emp
    -> where ename = 'SCOTT';
+---------+
| job     |
+---------+
| ANALYST |
+---------+

// 找出与"SCOTT"相同工作的员工及部门编号
mysql> select ename,deptno,job
    -> from emp e
    -> where e.job = (select job from emp where ename = 'SCOTT')
    -> and e.ename <> 'SCOTT';
+-------+--------+---------+
| ename | deptno | job     |
+-------+--------+---------+
| FORD  |     20 | ANALYST |
+-------+--------+---------+

// 第三步：基于以上查询结果，找出员工所在的部门名称（连接部门表）
mysql> select e.ename,e.job,d.dname
    -> from emp e
    -> join dept d
    -> on e.deptno = d.deptno
    -> where e.job = (select job from emp where ename = 'SCOTT')
    -> and e.ename <> 'SCOTT';
+-------+---------+----------+
| ename | job     | dname    |
+-------+---------+----------+
| FORD  | ANALYST | RESEARCH |
+-------+---------+----------+
## 24、列出薪金 等于部门30中员工的薪金的 其他员工的姓名和薪金.
// 第一步：找出30部门的员工薪资
mysql> select distinct sal
    -> from emp
    -> where deptno =30;
+---------+
| sal     |
+---------+
|  950.00 |
| 1250.00 |
| 1500.00 |
| 1600.00 |
| 2850.00 |
+---------+

// 第二步：在员工表中找出薪资等于 30部门薪资的员工姓名和姓名
mysql> select ename,deptno,sal
    -> from emp
    -> where sal in(select distinct sal from emp where deptno = 30)
    -> ;
+--------+--------+---------+
| ename  | deptno | sal     |
+--------+--------+---------+
| ALLEN  |     30 | 1600.00 |
| WARD   |     30 | 1250.00 |
| MARTIN |     30 | 1250.00 |
| BLAKE  |     30 | 2850.00 |
| TURNER |     30 | 1500.00 |
| JAMES  |     30 |  950.00 |
+--------+--------+---------+

// 第三步：筛选除30部门之外的部门员工
mysql> select ename,deptno,sal
    -> from emp
    -> where sal in(select distinct sal from emp where deptno = 30)
    -> and deptno <> 30;
Empty set (0.00 sec)
## 25、列出薪金 高于在部门30工作的所有员工的薪金 的员工姓名、薪金和部门名称
实则求大于30部门最高薪资的 其他员工信息。
// 第一步：找出部门编号为30的员工最高薪资
mysql> select max(sal) from emp where deptno = 30;
+----------+
| max(sal) |
+----------+
|  2850.00 |
+----------+

// 第二步：找出大于30部门最高薪资的员工姓名、薪资、部门名称（需要连接部门表，连接条件为部门编号相对）
mysql> select e.ename,e.sal,d.dname
    -> from emp e
    -> join dept d
    -> on e.deptno = d.deptno
    -> where e.sal > (select max(sal) from emp where deptno = 30);
+-------+---------+------------+
| ename | sal     | dname      |
+-------+---------+------------+
| KING  | 5000.00 | ACCOUNTING |
| JONES | 2975.00 | RESEARCH   |
| SCOTT | 3000.00 | RESEARCH   |
| FORD  | 3000.00 | RESEARCH   |
+-------+---------+------------+
## 26、列出在每个部门工作的员工数量，平均工资和平均服务期限
// 第一步：根据部门分组，统计每个部门的员工数量，平均薪资
// 注意超级领导的薪资为null也要算入
mysql> select deptno,count(*) as countNum,ifnull(avg(sal),0) as avgsal, 
    -> from emp
    -> group by deptno;
+--------+----------+-------------+
| deptno | countNum | avgsal      |
+--------+----------+-------------+
|     10 |        3 | 2916.666667 |
|     20 |        5 | 2175.000000 |
|     30 |        6 | 1591.666667 |
+--------+----------+-------------+

// 第二步：使用日期函数timestampdiff()求出总的天数，再求部门中的员工平均服务期限
// 注意部门40没有员工也要输出
mysql> select
    -> d.deptno, count(e.ename) ecount,ifnull(avg(e.sal),0) as avgsal, ifnull(avg(timestampdiff(YEAR, hiredate, now())), 0) as avgservicetime
    -> from emp e
    -> right join dept d
    -> on e.deptno = d.deptno
    -> group by d.deptno;
+--------+--------+-------------+----------------+
| deptno | ecount | avgsal      | avgservicetime |
+--------+--------+-------------+----------------+
|     10 |      3 | 2916.666667 |        38.6667 |
|     20 |      5 | 2175.000000 |        36.8000 |
|     30 |      6 | 1591.666667 |        39.0000 |
|     40 |      0 |    0.000000 |         0.0000 |
+--------+--------+-------------+----------------+
## 27、 列出所有员工的姓名、部门名称和工资。
// 第一步：员工表中找出员工的部门编号，员工姓名，工资
mysql> select ename,deptno,sal
    -> from emp;

// 第二步：以上的结果连接 部门表找出部门名称，连接条件为部门编号相等
mysql> select e.ename,d.dname,e.sal
    -> from emp e
    -> join dept d
    -> on e.deptno = d.deptno
    -> order by sal;
+--------+------------+---------+
| ename  | dname      | sal     |
+--------+------------+---------+
| SMITH  | RESEARCH   |  800.00 |
| JAMES  | SALES      |  950.00 |
| ADAMS  | RESEARCH   | 1100.00 |
| MARTIN | SALES      | 1250.00 |
| WARD   | SALES      | 1250.00 |
| MILLER | ACCOUNTING | 1300.00 |
| ALLEN  | SALES      | 1600.00 |
| TURNER | SALES      | 1650.00 |
| CLARK  | ACCOUNTING | 2450.00 |
| BLAKE  | SALES      | 2850.00 |
| JONES  | RESEARCH   | 2975.00 |
| FORD   | RESEARCH   | 3000.00 |
| SCOTT  | RESEARCH   | 3000.00 |
| KING   | ACCOUNTING | 5000.00 |
+--------+------------+---------+
+--------+------------+---------+
## 28、列出所有部门的详细信息和人数
// 第一步：列出所有部门的详细信息
mysql> select * from dept;
+--------+------------+----------+
| DEPTNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+

// 第二步：统计各个部门的人数
mysql> select deptno,count(*)
    -> from emp
    -> group by deptno
    -> having ifnull(count(*),0);
+--------+----------+
| deptno | count(*) |
+--------+----------+
|     10 |        3 |
|     20 |        5 |
|     30 |        6 |
+--------+----------+

// 第三步：在部门详细信息表的基础上加上一列"部门人数"，将两表连接输出，且以部门详细信息表为主
mysql> select d.*,count(e.ename)
    -> from emp e
    -> right join dept d
    -> on e.deptno = d.deptno
    -> group by d.deptno,d.dname,d.loc;
+--------+------------+----------+----------------+
| deptno | dname      | loc      | count(e.ename) |
+--------+------------+----------+----------------+
|     10 | ACCOUNTING | NEW YORK |              3 |
|     20 | RESEARCH   | DALLAS   |              5 |
|     30 | SALES      | CHICAGO  |              6 |
|     40 | OPERATIONS | BOSTON   |              0 |
+--------+------------+----------+----------------+
## 29、列出各种工作的最低工资及从事此工作的雇员姓名
// 第一步：按工作分组，求每个工作的最低薪资
mysql> select job,min(sal) as minsal
    -> from emp
    -> group by job;
+-----------+----------+
| job       | minsal        |
+-----------+----------+
| ANALYST   |  3000.00 |
| CLERK     |   800.00 |
| MANAGER   |  2450.00 |
| PRESIDENT |  5000.00 |
| SALESMAN  |  1250.00 |
+-----------+----------+

// 第二步：使用员工表与 工种最低薪资表连接，连接条件为job相等，找出该job的员工
mysql> select e.ename,t.*
    -> from emp e
    -> join (select job,min(sal) as minsal
    ->       from emp
    ->       group by job)  t
    -> on e.job = t.job and e.sal = t.minsal;
+--------+-----------+---------+
| ename  | job       | minsal  |
+--------+-----------+---------+
| SMITH  | CLERK     |  800.00 |
| WARD   | SALESMAN  | 1250.00 |
| MARTIN | SALESMAN  | 1250.00 |
| CLARK  | MANAGER   | 2450.00 |
| SCOTT  | ANALYST   | 3000.00 |
| KING   | PRESIDENT | 5000.00 |
| FORD   | ANALYST   | 3000.00 |
+--------+-----------+---------+
## 30、列出各个部门的MANAGER( 领导) 的最低薪金
// 第一步：先按部门分组，找出每个部门的MANAGER
mysql> select deptno,ename
    -> from emp
    -> where job='manager'
    -> group by deptno;
+--------+-------+
| deptno | ename |
+--------+-------+
|     10 | CLARK |
|     20 | JONES |
|     30 | BLAKE |
+--------+-------+
// 第二步：求各个部门领导的最低薪资
mysql> select deptno,ename as manager, min(sal)
    -> from emp
    -> where job = 'MANAGER'
    -> group by deptno;
+--------+---------+----------+
| deptno | manager | min(sal) |
+--------+---------+----------+
|     10 | CLARK   |  2450.00 |
|     20 | JONES   |  2975.00 |
|     30 | BLAKE   |  2850.00 |
+--------+---------+----------+
## 31、列出所有员工的年工资, 按年薪从低到高排序
// 第一步：求出每一位员工的年薪，年薪 = （基本工资+补贴）*12，注意补贴为Null的处理
// 第二步：将求得的年薪从低到高排序 asc
mysql> select
    -> ename,(sal + ifnull(comm,0)) * 12 as yearsal
    -> from
    -> emp
    -> order by
    -> yearsal asc;
+--------+----------+
| ename  | yearsal  |
+--------+----------+
| SMITH  |  9600.00 |
| JAMES  | 11400.00 |
| ADAMS  | 13200.00 |
| MILLER | 15600.00 |
| TURNER | 19800.00 |
| WARD   | 21000.00 |
| ALLEN  | 22800.00 |
| CLARK  | 29400.00 |
| MARTIN | 31800.00 |
| BLAKE  | 34200.00 |
| JONES  | 35700.00 |
| FORD   | 36000.00 |
| SCOTT  | 36000.00 |
| KING   | 60000.00 |
+--------+----------+
## 32、求出员工领导的薪水超过3000 的员工名称与领导
// 第一步：找出领导表中超过3000薪资的领导
mysql> select ename,mgr as leaderno,sal
    -> from emp
    -> where sal>3000;
+-------+----------+---------+
| ename | leaderno | sal     |
+-------+----------+---------+
| KING  |     NULL | 5000.00 |
+-------+----------+---------+

// 第二步：员工表连接 超过3000薪资的领导表
mysql> select a.ename '员工',a.sal '员工薪资',b.ename '领导',b.sal '领导薪资'
    -> from emp a
    -> join (select ename,empno as leaderno,sal from emp where sal>3000) b
    -> on a.mgr = b.leaderno;

mysql> select a.ename '员工',a.sal '员工薪资',b.ename '领导',b.sal '领导薪资'
    -> from emp a
    -> join emp b
    -> on a.mgr = b.empno
    -> where b.sal > 3000;
+-------+----------+------+----------+
| 员工  | 员工薪资 | 领导 | 领导薪资 |
+-------+----------+------+----------+
| JONES |  2975.00 | KING |  5000.00 |
| BLAKE |  2850.00 | KING |  5000.00 |
| CLARK |  2450.00 | KING |  5000.00 |
+-------+----------+------+----------+
33、求出部门名称中, 带'S'字符的部门 员工的工资合计、部门人数
// 第一步：以部门进行分组，找出'S'字符的部门
// 第二步：对该部门的员工薪资进行合计，对该部门员工的人数进行统计
mysql> select d.deptno,d.dname,count(e.ename),ifnull(sum(e.sal),0) as sumsal
    -> from emp e
    -> right join dept d
    -> on e.deptno = d.deptno  //  员工表和部门表连接
    -> where d.dname like '%S%'  //  带'S'字符的部门
    -> group by d.deptno,d.dname,d.loc;
+--------+------------+----------------+----------+
| deptno | dname      | count(e.ename) | sumsal   |
+--------+------------+---------+----------------+----------+
|     20 | RESEARCH   |               5 | 10875.00 |
|     30 | SALES      |               6 |  9400.00 |
|     40 | OPERATIONS |               0 |     0.00 |
+--------+------------+---------+----------------+----------+
34、给任职日期超过30年的员工 加薪10%
// 第一步：先找出任职日期大于30年的员工，使用日期函数timestampdiff()
// 第二步：在 工龄超过30年的基础上，对该员工的薪资增加10%
mysql> update emp
    -> set sal = sal*1.1  // 不能使用 sal*（1+10%）
    -> where timestampdiff(YEAR, hiredate, now()) > 30;  // 以year年为单位，使用当前日期now-就职日期