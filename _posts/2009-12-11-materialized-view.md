---
layout:     post
title:      Materialized View(物化视图)
category: Database
keywords: Materialized View
---
###1. Materialized View 概述： 
　　Oracle的实体化视图提供了强大的功能，可以用在不同的环境中。在不同的环境中，实体化视图的作用也不相同。数据仓库中的实体化视图主要用于预先计算并保存表连接或聚集等耗时较多的操作的结果，这样，在执行查询时，就可以避免进行这些耗时的操作，而从快速的得到结果。
　　在数据仓库中，还经常使用查询重写（query rewrite）机制，这样不需要修改原有的查询语句，Oracle会自动选择合适的实体化视图进行查询，完全对应用透明。实体化视图和表一样可以直接进行查询。实体化视图可以基于分区表，实体化视图本身也可以分区。除了在数据仓库中使用，实体化视图还用于复制、移动计算等方面。实体化视图有很多方面和索引很相似：使用实体化视图的目的是为了提高查询性能；实体化视图对应用透明，增加和删除实体化视图不会影响应用程序中SQL语句的正确性和有效性；实体化视图需要占用存储空间；当基表发生变化时，实体化视图也应当刷新。 
　　
　　Materialized View 同Snapshot是同一个概念。但同view是不一样的: 
 1. 物化视图是存储数据的视图，存储了基础表的全部或者一部分数据，主要用作sql语句的优化，查询物化视图比查询表中的数据速度要快; 
 2. MV是自动刷新或者手动刷新的，View不用刷新; 
 3. MV也可以直接update，但是不影响base table，对View的update反映到base table上; 
 4. MV主要用于远程数据访问，mv中的数据需要占用磁盘空间，view中不保存数据 

###2. DDL/DML: 

 1. CREATE MATERIALIZED VIEW 
    语法： 
        CREATE MATERIALIZED VIEW [user.]mview
    例子： 
    
```SQL
CREATE MATERIALIZED VIEW hq_emp  
    REFRESH COMPLETE  
    START WTIH SYSDATE NEXT SYSDATE +1/4096  
     AS SELECT * FROM hq_emp;  
```
 2. ALTER MATERIALIZED VIEW 
     语法： 
      ALTER MATERIALIZED VIEW [user.]mview 
     例子： 
      
```SQL
CREATE MATERIALIZED VIEW hq_emp  
          REFRESH COMPLETE  
          START WTIH SYSDATE NEXT SYSDATE +1/4096  
          AS SELECT * FROM hq_emp;  
          ALTER MATERIALIZED VIEW hq_emp  
          REFRESH FAST;  
```
###3.主要参数说明： 
ON PREBUILD TABLE: 
    描述 将已经存在的表注册为实体化视图。同时还必须提供描述创建该表的查询的 SELECT 子句。可能无法始终保证查询的精度与表的精度匹配。为了克服此问题，应该在规范中包含 WITH REDUCED PRECISION 子句。 

Build Clause 创建方式 
   描述 包括BUILD IMMEDIATE和BUILD DEFERRED两种 
   取值 BUILD IMMEDIATE 在创建实体化视图的时候就生成数据 
   BUILD DEFERRED 在创建时不生成数据，以后根据需要在生成数据 
   默认 BUILD IMMEDIATE 

   Refresh 刷新子句 
   描述 当基表发生了DML操作后，实体化视图何时采用哪种方式和基表进行同步 
   语法 [refresh [fast | complete | force] 
         [on demand | commit] 
         [start with date] 
         [next date] 
         [with {primary key | rowid}] 
       ]      
   取值 FAST 采用增量刷新，只刷新自上次刷新以后进行的修改 
    COMPLETE 对整个实体化视图进行完全的刷新 
    FORCE(默认) Oracle在刷新时会去判断是否可以进行快速刷新，如果可以则采用Fast方式，否则采用Complete的方式，Force选项是默认选项 
   ON DEMAND(默认) 实体化视图在用户需要的时候进行刷新，可以手工通过      DBMS_MVIEW.REFRESH等方法来进行刷新，也可以通过JOB定时进行刷新 
ON COMMIT 实体化视图在对基表的DML操作提交的同时进行刷新 

    START WITH 第一次刷新时间 
    NEXT 刷新时间间隔 
    WITH PRIMARY KEY(默认) 生成主键实体化视图,也就是说实体化视图是基于表的主键，而不是ROWID(对应于ROWID子句)。 为了生成PRIMARY KEY子句，应该在表上定义主键，否则应该用基于ROWID的实体化视图。主键实体化视图允许识别实体化视图表而不影响实体化视图增量刷新的可用性。 

WITH ROWID 只有一个单一的主表，不能包括下面任何一项: 
> * Distinct 
> * 聚合函数 
> * Group by 
> * 子查询 
> * 连接 
> * SET操作 
  
    Query Rewrite 查询重写 
    描述 包括ENABLE QUERY REWRITE和DISABLE QUERY REWRITE两种。分别指出创建的实体化视图是否支持查询重写。查询重写是指当对实体化视图的基表进行查询时，Oracle会自动判断能否通过查询实体化视图来得到结果，如果可以，则避免了聚集或连接操作，而直接从已经计算好的实体化视图中读取数据 
    取值 ENABLE QUERY REWRITE 支持查询重写 
    DISABLE QUERY REWRITE 不支持查询重写 
    默认 DISABLE QUERY REWRITE 

3. 实例探究: 
  
```SQL
SQL> create materialized view DUMMY_INSURED_MV  
  2  refresh complete  
  3  start with sysdate next sysdate + 1/4096  
  4  as select * from t_dummy_insured_old;  
   
  Materialized view created  
  
  SQL> view DUMMY_INSURED_MV;  
   CREATE MATERIALIZED VIEW DUMMY_INSURED_MV  
REFRESH COMPLETE ON DEMAND  
START WITH TO_DATE('09-12-2009 16:19:48', 'DD-MM-YYYY HH24:MI:SS') NEXT SYSDATE + 1/4096   
AS  
SELECT "T_DUMMY_INSURED_OLD"."DETAIL_ID" "DETAIL_ID","T_DUMMY_INSURED_OLD"."POLICY_ID" "POLICY_ID","T_DUMMY_INSURED_OLD"."DUMMY_NUM" "DUMMY_NUM","T_DUMMY_INSURED_OLD"."DUMMY_NAME" "DUMMY_NAME","T_DUMMY_INSURED_OLD"."JOB_CLASS" "JOB_CLASS","T_DUMMY_INSURED_OLD"."INSURED_AMOUNT" "INSURED_AMOUNT","T_DUMMY_INSURED_OLD"."AVERAGE_AGE" "AVERAGE_AGE","T_DUMMY_INSURED_OLD"."PREM_AGE" "PREM_AGE","T_DUMMY_INSURED_OLD"."MALE_RATE" "MALE_RATE","T_DUMMY_INSURED_OLD"."GENDER" "GENDER","T_DUMMY_INSURED_OLD"."SMOKING" "SMOKING","T_DUMMY_INSURED_OLD"."REALIZED_AMOUNT" "REALIZED_AMOUNT","T_DUMMY_INSURED_OLD"."REPORT_MONTH" "REPORT_MONTH","T_DUMMY_INSURED_OLD"."CREATE_TIME" "CREATE_TIME" FROM "T_DUMMY_INSURED_OLD" "T_DUMMY_INSURED_OLD";  
  
SQL> insert into t_dummy_insured_old select S_DUMMY_INS_OLD_DETAIL_ID.Nextval,  
  2  DI.*,'200911',Sysdate from t_dummy_insured DI;  
   
1196 rows inserted  
  
SQL> commit;  
   
Commit complete  
SQL> select count(1) from DUMMY_INSURED_MV;  
   
  COUNT(1)  
----------  
      1196  
```
[Source Link](http://hwhuang.iteye.com/blog/546309)