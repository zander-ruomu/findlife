# 关系数据库SQL代码编写规范

## 查询规则

SQL查询原理：

 ![img](https://images2015.cnblogs.com/blog/414640/201706/414640-20170602072256352-1104376539.png) 

从上图可以看到SQL语句执行解析时主要步骤是：先检查语法----->查看缓冲池是否有对应的T-SQL执行计划命中---->若没有则在执行器中进行优化并产生执行计划，存入缓冲池，否则直接进行下一步------>将执行计划输出给查询执行器---->  数据访问方法将执行计划生成SQL可操作数据的代码, 传送给缓冲区管理器来执行  -->  检查缓冲池的数据缓存中是否存在这些数据，如果存在，就把结果返回给存储引擎的数据访问方法；如果不存在，则从磁盘（数据文件）中读出数据并放入数据缓存中，然后将读出的数据返回给存储引擎的数据访问方法 。



了解基本原理之后，我们大概可以从两个大方面进行SQL查询优化：

1. **提高T-SQL命中率**

   1> 参数化SQL查询条件，如hql或者Mybits等框架写查询语句是用 ?param 代替直接 放入参数值。(对于参数化SQL会导致数据高速缓存无法命中，个人觉得参数化SQL是必须的，其可以防止SQL注入同时执行器解析执行计划会占用大部分运行成本。而数据高速缓存无法命中问题可以从存储方式和硬件传输速度上进行优化 )

   2> SELECT PARAM1...PARAMn，尽可能减少*等的用法。

2. **优化SQL语句，提高执行器优化方案命中**

   1> 表结构方面加强索引的使用，避免笛卡尔积，模糊匹配等操作。

   关联查询使用左连接时应将基表提前，利用外键或者唯一键做表关联，减少冗余数据。

   IN， EXISTS，LIKE，REGEXP， ！=， <>等模糊匹配，OR条件应尽可能用UNION ALL代替。

   IN和EXISTS差异(都会用到索引)： in是把外表和内表作hash 连接，而exists 是对外表作loop 循环，每次loop 循环再对内表进行查询。  若查询的两表大小相当，in 和exists 差别不大 ；若子查询表大， exists 速度快；若子查询表小用IN快。

   not extsts 一定比  not in快，因为not in是进行全扫描。可以用外连接代替。

   索引列上应避免进行函数运算、 数据类型转换 、 IS NULL 等。

   创建索引的列尽可能为数值型，或有序字符串，必须设置非空条件。

   2> 善用会话临时表。 TEMPORARY 

   多表关联时，对于复杂的子查询可以建立会话临时表进行数据存储。加快运行速度。

   3> 表连接和查询条件的顺序应遵循查询结果数量级小的放在数量级大的前面，优化的查询条件（如索引列）应放在普通条件前面。

   

   

   ## 批量更新规则

1. 善用会话临时表。 TEMPORARY
2. 禁止使用笛卡尔乘积。
3. 过滤条件优化同查询规则。
4. 应用事务进行更新操作，锁表时支持事务中断数据回滚操作。



# 数据库定位

1. 查看应用服务器日志看检查是否SQL语句语法错误或数据连接中断。

   tomcat项目运行日志，可以根据时间以及类名、方法名进行筛选对应日志。

2. 查看锁表情况。

   mysql： 

   ```mysql
   show status like 'Table%'; #Table_locks_waited 指不能立即获取表级锁而需要等待的次数 
   show OPEN TABLES where In_use > 0; #查看锁表语句  
   show processlist;  #找到锁表的进程 
   kill 进程号;  #删除锁表进程  
   ```

   oracle：

   ```sql
   select SID, SERIAL# from v$session t1, v$locked_object t2 where t1.sid = t2.SESSION_ID; #查看锁表进程
   alter system kill session 'SID,SERIAL#';  #删除锁表进程 
   ```

3. 解析SQL执行计划

   mysql： explain SQL语句;

   oracle：

   ```sql
   EXPLAIN PLAN FOR SELECT * FROM EMP;#解析
   SELECT plan_table_output FROM TABLE(DBMS_XPLAN.DISPLAY('PLAN_TABLE'));#查看解析结果
   SELECT * from table(dbms_xplan.display);#查看解析结果
   ```

4. 根据实际情况优化SQL，对于关联复杂的SQL语句可以分块优化操作。必要时可以使用临时表或者增加索引结构。