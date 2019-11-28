# Mysql 触发器

## 简介

 触发器是基于表操作有关的数据库对象，在满足定义条件时触发，并执行触发器中定义的语句集合。常用于信息通知， 读写日志等，如微信更新一条朋友圈，触发器根据你的朋友圈更新关联朋友的朋友圈消息推送，其他人的朋友圈就会多一条新动态。利用触发器可以实现及时性动态维护数据。



## 基本语法

```mysql
DELIMITER //
CREATE TRIGGER trigger_name trigger_time trigger_event ON tb_name FOR EACH ROW 
BEGIN
	trigger_stmt
END// 
DELIMITER ;
trigger_name：触发器的名称
tirgger_time：触发时机，为BEFORE或者AFTER
trigger_event：触发事件，为INSERT、DELETE或者UPDATE
tb_name：表示建立触发器的表明，就是在哪张表上建立触发器
trigger_stmt：触发器的程序体，可以是一条SQL语句或者是用BEGIN和END包含的多条语句
```



**MySQL可以创建以下六种触发器**：

BEFORE INSERT 插入前； AFTER INSERT 插入后 (INSERT, LOAD ATA)

BEFORE UPDATE 修改前；AFTER UPDATE 修改后

BEFORE DELETE 删除前；AFTER DELETE 删除后