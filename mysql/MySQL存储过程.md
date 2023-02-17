---
title: MySQL存储过程
date: 2023-02-17 16:30:00
categories:
- MySQL
tags:
- mysql
- 存储过程
---

## 概述

存储过程就像一个接口，可以直接调用，不需要重复编写。

与函数的对比：都是事先编译并存储再数据库中的一段`SQL`语句的集合，调用存储过程和函数可以简化开发工作。区别在于函数必须有返回值，存储过程没有。

> 存储过程结构

```sql
delimiter ;;
create procedure 存储过程名称(int parameter)
begin
SQL语句;
end;;
delimiter ;
```

`parameter`为传入函数的参数

### 存储过程的基本使用

>创建测试表

```sql
create table students(
    id int primary key auto_increment,
    age int,
    name varchar(20),
    city varchar(20)
) character set utf8;

insert into students values(null, 22, 'lisa', '杭州');
insert into students values(null, 16, 'rock', '上海');
insert into students values(null, 20, 'jack', '深圳');
insert into students values(null, 21, 'rose', '北京');
```



>不带参数的存储过程

```sql
-- 查询学生个数
drop procedure if exists select_students_count;

delimiter ;; -- 替换分隔符
	create procedure select_students_count() 
		begin 
			select count(id) from students; 
		end ;;
delimiter ;
```

执行存储过程

```sql
call select_students_count();
```

>带参数的存储过程

```sql
-- 根据城市查询总数
delimiter ;;
	create procedure select_students_by_city_count(in _city varchar(255))
		begin
			select count(id) from students where city = _city;
		end;;
delimiter ;
```

执行存储过程

```sql
call select_students_by_city_count('上海')
```

>带有输出参数的存储过程

`MySQL`支持`in(传递给存储过程)`，`out(从存储过程传出)`和`inout(对存储过程传入和传出)`类型的参数。

```sql
-- 根据姓名查询学生信息，返回学生的城市
delimiter ;;
create procedure select_students_by_name(
    in _name varchar(255),
    out _city varchar(255), -- 输出参数
    inout _age int(11)
)
    begin 
    	select city from students where name = _name and age = _age into _city;
    end ;;
delimiter ;
```

执行存储过程

```sql
set @_age = 20;
set @_name = 'jack';
call select_students_by_name(@_name, @_city, @_age);
select @_city as city, @_age as age;
```

带有通配符的存储过程

```sql

delimiter ;;
create procedure select_students_by_likename(
    in _likename varchar(255)
)
    begin
    	select * from students where name like _likename;
    end ;;
delimiter ;
```

执行存储过程

```
call select_students_by_likename('%s%');
call select_students_by_likename('%j%');
```



