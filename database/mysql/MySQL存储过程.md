# Mysql 存储过程

## 简介

  mysql 存储过程是可编程的函数，在数据库中创建并保存，可以由SQL语句和控制结构组成。存储过程主要应用于复杂的固定业务逻辑，或定时业务。缺点：bug不易定位、维护成本高、若涉及多表修改和增加，可能会导致锁表等情况。优点： 对于特定功能封装，可以重复利用，简化开发流程和运行成本，增加数据安全性。 



## 基本语法

```mysql
CREATE PROCEDURE 过程名([[IN|OUT|INOUT] 参数名 数据类型], [[IN|OUT|INOUT] 参数名 数据类型],…) [特性 ...] 过程体
DELIMITER //  
CREATE PROCEDURE myproc(OUT s int) BEGIN      
SELECT COUNT(*) INTO s FROM students;
END // DELIMITER ;
```

**参数**

存储过程根据需要可能会有输入、输出、输入输出参数，如果有多个参数用","分割开。MySQL存储过程的参数用在存储过程的定义，共有三种参数类型,IN,OUT,INOUT:

**IN**： 参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值

**OUT**： 该值可在存储过程内部被改变，并可返回

**INOUT**：调用时指定，并且可被改变和返回



**分隔符**

  MySQL默认以";"为分隔符，如果没有声明分割符，则编译器会把存储过程当成SQL语句进行处理，因此编译过程会报错 。DELIMITER // 表示声明当前段分隔符为 // 。

```mysql
#查询存储过程
SHOW PROCEDURE STATUS WHERE db='数据库名';
```



## 游标

游标的应用类似于面向对象中的ArrayList<Map<Key,Value>>,只不过Map的Key是固定的。

```mysql
DELIMITER //
USE mydb //
DROP PROCEDURE IF EXISTS `test`//

CREATE DEFINER=`dev`@`192.168.%` PROCEDURE `test`()
BEGIN
	DECLARE userId INT;
	DECLARE username VARCHAR(20);
	DECLARE u_sex VARCHAR(2);
	DECLARE done INT DEFAULT 0;
	DECLARE userId,username,u_sex CURSOR FOR
	SELECT id,name,sex FROM user;
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1; -- 溢出处理

    OPEN userId,username,u_sex; -- 打开游标
    -- 需要注意的是游标和SELECT查询语句共用同一个溢出参数
    -- 即若在游标循环过程中调用 SELECT 查询语句结果为NULL，则不管游标是否遍历完游标都会中断
    outLoop:LOOP
            FETCH userId,username,u_sex INTO uid,uname,usex;
        IF done = 1
            THEN LEAVE outLoop; -- 退出循环
        ELSE
            INSERT INTO testlog(update_time,params,values) VALUES
            (now(),'uid,uname,usex',CONCAT(uid,',',uname,',',usex)) -- 循环体
        END IF;
    END LOOP;
    CLOSE userId,username,u_sex; -- 关闭游标
END//
DELIMITER;
```


## 存储过程编写注意事项

1. 在游标循环过程中调用 SELECT 查询语句时，需要注意维护游标的溢出参数值，如上文的done参数。
2. 为了方便bug定位，可以在复杂业务逻辑代码前，插入日志。
3. 存储过程编写需要注意避免嵌套循环，防止对数据库服务器资源的过度占用。
4. 经常性迭代的业务代码不可写入存储过程，存储过程只试用于特定业务代码。
5. 嵌套式调用存储过程，除结果参数外其他传入参数均不可修改，防止脏数据不利于定位和维护。
6. 对于不需要查询和修改业务数据的特定代码，应用简单函数替代存储过程。