### 一、简介

#### 1、概述

- MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。
- MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件。  
- MySQL是一种关系数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。 
- MySQL系统结构是 “库”  “表”  “行”  “列”，每个库中有多张表，每张表中有多个行，每个行中有多个列。



#### 2、安装

（1）在 [MySQL 下载](http://dev.mysql.com/downloads/mysql/)中下载 Windows 版本的 MySQL 安装包 

（2）双击安装并配置环境变量

​	手动配置环境变量： 右击计算机--属性-高级系统设置-环境变量-系统环境变量-path : 加入 mysql安装目录下的bin路径：`D:\Software\MySQL Server 5.5\bin`

（3）检查是否安装成功  （打开Dos窗口）

```sql
mysqladmin --version                 //查看Mysql服务器版本
```



#### 3、密码设置

##### 3.1 设置初始密码（跳过）

```mysql
mysqladmin -u root password "123456"
```

##### 3.2 修改密码

```mysql
mysqladmin -u用户名 -p旧密码 password 新密码
```

##### 3.3 忘记root密码

```
1. 关闭正在运行的MySQL服务。 net stop mysql
2. 打开DOS窗口，转到mysql\bin目录 -- 如果配置了环境变量，则该步骤可以跳过。 
3. 输入mysqld --skip-grant-tables 回车。--skip-grant-tables 的意思是启动MySQL服务的时候跳过权限表认证。
4. 再开一个DOS窗口（因为刚才那个DOS窗口已经不能动了），输入mysql回车，如果成功，将出现MySQL提示符 >。
5. 连接权限数据库： use mysql; 。
6. 改密码：update user set password=password("新密码") where user="root";（别忘了最后加分号） 。
7. 刷新权限（必须步骤）：flush privileges;　。
8. 退出  quit。
```



#### 4、登录和退出

##### 4.1 登录mysql

```sql
mysql -uroot -p123456
```

##### 4.2 退出mysql

```mysql
mysql> quit;
```



### 二、数据库管理 

#### 1、显示所有的数据库

```mysql
mysql>  show databases;     
```

#### 2、创建数据库

```mysql
mysql> create database 数据库名  charset=utf8;   #万国码 解决中文乱码       
```

#### 3、删除数据库

```mysql
mysql> drop database 数据库名;  
```

#### 4、选择数据库/查看当前选中的库

```mysql
mysql> use db_name;           //使用某个数据库

mysql> select database();     //查看当前选中的数据库
```

#### 5、显示数据库中的表

```mysql
mysql>  show tables;
```

#### 

### 三、简单建表操作 

#### 1、创建表

```mysql
mysql>  create table 表名(列名1 数据类型 约束...，列名2...)
        create table stu_information(id int auto_increment,name varchar(20))     
```

#### 2、简单数据类型

```mysql
int/integer 整数

varchar(n)  字符，n是字符长度

datetime 日期
```

#### 3、简单约束

```mysql
auto_increment 自动增长  由mysql自动维护 默认从1开始

primary key 主键  唯一标识数据库表中的每条记录    不重复且非空   primary key == unique + not null

not null 非空

unique 唯一 但可以为空 
```

#### 4、简单示例

```mysql
mysql> create database userDB charset=utf8;

mysql> use userDB;

mysql> create table userList (id int auto_increment primary key,name varchar(20) not null ,age int,birthday datetime);
```

- 查看表结构

  ```mysql
  mysql> desc table_name;
  ```

- 插入数据

  ```mysql
  insert into userlist(name,age,birthday) values("Mr_lee",18,"2000-07-11");
  ```

- 查询数据

  ```mysql
  select * from userlist;  --  表示所有列  
  ```

  也可以查询指定列

  ```mysql
  select name,age form userlist;
  ```

#### 5、可视化工具navicat



### 四、SQL查询语句   

结构化查询语言SQL（Structured Query Language）：用于存取数据、更新、查询和管理关系数据库系统的程序设计语言。 

#### 1、查询表中的列

- 查询所有列

  ```mysql
  select * from 表名;     # select * from userlist;
  ```

- 查询指定列--部分列

  ```mysql
  select 列名1，列名2 ... from 表名;
  ```



#### 2、列操作

- 列运算：列查询支持算术运算【+ - * /】 运算中出现null 结果也为null

  ```mysql
  select id+1，age * 2 from userlist；
  ```

- 列拼接

  ```mysql
  select concat(age,"岁") from userlist;
  ```

- 列别名

  ```mysql
  select name as 姓名 from userlist；
  -- select 列名 as 别名 from userlist;
  ```

  注意： as可省略

- 列去重（了解）

  ```mysql
  select distinct age from userlist；
  ```
  

#### 3、where子句

##### 3.1 条件查询

如需有条件地从表中选取一部分数据，可将 WHERE 子句添加到 SELECT 语句中。 

基本语法：

```mysql
	select ... from 表名  where 表达式 
```

如：查询年龄大于20的用户信息

```mysql
	select * from userlist where age > 20;
```

- 比较运算符：【=   !=  >  <  >=   <=  <>】
- 逻辑运算符：【and   or】
- 范围 ：【in(xxx,xxx,xxx...)     between.. and】

```mysql
查询“id大于3” 且 “年龄小于19” 或者 “名字是JJ” 的用户的 id,name,age
select id,name,age from userlist where id > 3 and age < 19 or name = 'JJ'

查询生日在‘2018-3-08 00:00:00’之后的用户
select id,name from userlist where birthday > '2018-3-08 00:00:00';

查询年龄大于18 并且id在(1,3,5,7)其中之一的用户
select id,name,age from userlist where age > 18 and id in(1,3,5,7);

查询年龄大于18 并且id在2-4之前的用户
select id,name,age from userlist where id between 2 and 4;
```

##### 3.2 空值判断

语法：

​	select ... from 表名 where 列名 **IS NULL** and 列名  **IS NOT NULL**

```mysql
select id,name,age from userlist where name is null and age is not null;
```

**注意：不能用如下语句进行空值判断**

```sql
select * from t_user where birthday = null;     -- error
select * from t_user where birthday = 'null';   -- error
```



##### 3.3 模糊查询

- % 任意多个字符
- _ 一个字符

语法：

```mysq
select ... from 表名 where 列名 like '%..%'
```

示例：

```mysql
查询名字中包含o的数据

select * from userlist where name like "%o%"
```

- like "%abc%"  含有abc
- like "%abc"   以abc结尾
- like "abc%"  以abc开头
- like "ab%_"  以ab开头，且ab后至少有一个字符
- like "%__%"  至少有2个字符      或  length(列名) >=2

#### 4、分页查询 

```
从第一条开始查询，共查询3条数据  == limit 0,3 
select ... from 表名  limit 3;   

从第一条开始，共查询2条
select ... from 表名 limit 0,2;

从第三条开始，共查询3条
select ... from 表名 limit 2,3;

每页显示n条，查询第m页
select ... from 表名 limit (m-1)*n,n;
```

#### 5、ORDER BY子句：排序

语法：

```
   select ... from 表名  ORDER BY 列名
```

- ASC : 升序(默认)       
- DESC : 降序

```mysql
根据id降序排列
select * from userlist order by id desc;

根据age升序排列，年龄相同按薪水升序排序
select * from userlist order by age asc,salary asc;
```

#### 6、聚合函数：组函数

- MAX( )   最大值
- MIN( )   最小值
- SUM( )  求和
- AVG( )  求平均值
- COUNT( )  总数

```mysql
查询最大id和年龄的平均值（结果只有一行）
select max(id), avg(age) from userlist;

查询共有多少个id（总数）
select count(id) from userlist;

查询年龄总和，和平均年龄
select count(age),avg(age) from userlist;
```

**注意：**

```mysql
select max(age),name from userlist 
-- 得到的结果是逻辑扭曲的 --
```

#### 7、GROUP BY：分组统计 

语法：

```mysql
select ... from 表名  group by 列名  
```

示例：

- 查询每个部门的最高工资，最低工资，最大年龄，部门id

```mysql
select max(salary),min(salary),max(age),detp_id from t_employee group by dept_id;
```

- 查询每个部门的平均工资，最低工资，工资总和，部门id

```mysql
select avg(salary),min(salary),sum(salary),dept_id from t_employee group by dept_id;
```

- 查询每个部门的员工总数

```mysql
select count(id),dept_id from t_employee group by dept_id;
```

- 聚合统计：将dept_id和age都相同的数据分到一组

```mysql
select age,dept_id from t_employee group by dept_id,age;
```

**注意：分组查询中除了组函数外，只能查询分组条件的字段**

![group_liucheng](.\Mysql-notes-pic\group_liucheng.png)

#### 8、having子句：分组限制

语法：

```mysql
select ... from 表名 group by 列名 HAVING .. ;    
```

在group形成临时表之后，最终确定结果之前执行，即对每个临时表做筛选，满足having条件的临时表才可以将一条数据放入最终结果。

示例：

- 查询每个部门的平均工资大于10000的员工的平均年龄和最高工资  

```mysql
select avg(age),max(salary),dept_id from t_employee group by dept_id having avg(salary) >10000;
```

- 查询最小年龄小于20的部门员工总数和部门id

```mysql
select count(id),dept_id from t_employee group by dept_id having min(age) < 20;
```

**重点理解：HAVING的作用时刻**

**注意：和where的区别是：**

- where先执行，用来对总表做数据筛选
- having在where之后的group之后执行，对分组过程中的临时表做筛选，每个临时表筛选一次

**补充：如果一个需求，where和having都可以实现，建议使用where，效率更高**



#### 9、case子句 

语法：

```mysql
select 
case

	when bool表达式 then 结果1
	when bool表达式 then 结果2
	...
	else
		默认结果
end，
列名1，列名2...
from 表名  ...
```

示例：

```mysql
select 
case 
	when age < 20 then '青少年'
	when age >=20 and age < 40 then '中年'
	ELSE	'老年'
end as '年龄段',
age,name
from t_employee order by age;
```



#### 10、子查询

将一个查询结果作为另一个查询的一部分，称为子查询

- 查询工资大于平均工资的员工信息

```mysql
select * from t_employee where salary > (select avg(salary) from t_employee);
```

- 查询工资小于平均值的员工数量

```mysql
select count(id) from t_employee where salary < (select avg(salary) from t_employee)
```

- 查询平均工资大于10000的部门的员工信息

```mysql
select id,name,age,salary from t_employee where dept_id in 
(select dept_id from t_employee group by dept_id having avg(salary)>10000)
-- 平均工资大小10000的部门有哪些 显示的部门id  1 3 5
```

- **注意:子查询的执行效率，很低**



### 五、补充

- LENGTH() 获取长度            select  * from t_user where length(name)>2;
- LCASE() 转为小写字符 abcd
- UCASE() 转为大小字符 ABCD
- TRIM()  去除头部和尾部的空格
- NOW() 当前日期和时间
- CURDATE() 当前日期
- CURTIME() 当前时间
- DATE_FORMAT(NOW(),'%Y/%m/%d %H:%i:%s')  日期格式化
- DATABASE() 当前的数据库名称
- USER() 当前用户名
- VERSION() 当前服务器版本



