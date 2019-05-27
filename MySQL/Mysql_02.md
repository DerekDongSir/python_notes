# MySQL_02

### 一、表连接  

#### 1、概述

两张表或多张表联合查询，共同提供数据，最终得出结果。 用户表 订单表

本章节我们将向大家介绍如何使用 MySQL 的 JOIN 在两个或多个表中查询数据。 



#### 2、表连接使用

##### 2.1 连接方式

- 内连接  `inner join`          数据必须一一对应  不会出现null数据
- 外连接
  - 左外连接    `left outer join`            左表优先   左表数据全显  右表无数据显示null
  - 右外连接    `right outer join`          右表优先   右表数据全显  左表无数据显示null
- 自连接   多用于类别表 ，不区分连接方式，只不过是 连接的双方是同一张表而已



##### 2.2 内连接

语法：

```sql
select ... from 表1 as 别名1 join 表2 as 别名2 on 别名1.列名 = 别名2.列名    
-- on表示连接条件 --
```

使用示例：

- 查询员工的信息和所在部门的名称

```mysql
select  t1.*,t2.title from t_employee as t1 inner join t_department as t2 on t1.dept_id = t2.id;
```

内连接的 ‘**inner**’ 可省略为 '**join**'

技巧：在表连接时，为各个表，定义别名，防止列名命名冲突



##### 2.3 表连接执行过程

- 如果是 A inner join B则A为主表，B为从表

- 主表中的任何一条数据，都要试图和B中的每一行取连接，是否能够真正连接，取决于on是否被满足

- on条件满足后，主从双方将自己对应的行，放入最终的大表中

  ![table_join](Mysql-notes-pic\table_join.png)




##### 2.4 外连接

- 如上将 ‘**inner join**’ 换成 ‘**right outer join**’ 换成‘**left outer join**’ 即是右外连接 或 左外连接。
- 其中外连接的 ‘**outer**’  可以省略为  ‘**left/ritht join**’ 



##### 2.5 外连接与内连接的区别

- 外连接：左外连接的左表（主表），右外连接的右表（主表） 数据全显，如果没有对方数据，用null填充。
- 内连接：只有存在匹配数据的数据才会显示在最终结果中。



##### 2.6 自连接

自连接即自己连接自己，多用于类别表中 

![1550039357541](Mysql_02.assets/1550039357541.png) 

```mysql
select 
	t1.id,t1.level,t1.title,t2.title,t2.level
from 
	t_cate  t1
JOIN
	t_cate t2
ON
	t1.id=t2.parent_id
```



##### 2.7 多表连接

示例：员工、部门、地区

- 查询所有员工的信息及所在部门的信息和部门所在的地区的信息

```mysql
SELECT
	emp.*,dept.*,loc.*
FROM	
	t_employee emp
left JOIN
	t_department dept
on 
	emp.dept_id = dept.id
right JOIN
	t_location loc
on 
	loc.id = dept.loc_id;
```



### 二、建表操作

#### 1、数据类型

- 数字类型   

| 类型名称     | 大小                                   | 用途            |
| ------------ | -------------------------------------- | --------------- |
| tinyint      | 1字节                                  | 极小整数值      |
| smallint     | 2字节                                  | 小整数值        |
| mediumint    | 3字节                                  | 大整数值        |
| int或integer | 4字节    -2147483648   2147483647      | 大整数值        |
| bigint       | 8字节                                  | 极大整数值      |
| float        | 4字节                                  | 单精度 浮点数值 |
| double       | 8字节                                  | 双精度 浮点数值 |
| decimal      | decimal(m,n) 无固定大小   decimal(8,3) | 小数值          |

- 字符类型  

| 类型名称   | 大小             | 用途                                 |
| ---------- | ---------------- | ------------------------------------ |
| char       | 0-255字节        | 定长字符串 char(10) 固定占用10字节   |
| varchar    | 0-65535字节      | 变长字符串 varchar(20)  表示最大长度 |
| tinyblob   | 0-255字节        | 不超过255个字符的二进制字符串        |
| tinytext   | 0-255字节        | 短文本字符串                         |
| blob       | 0-65535字节      | 二进制形式的长文本数据               |
| text       | 0-65535字节      | 长文本数据                           |
| mediumblob | 0-16777215字节   | 二进制形式的中等长度文本数据         |
| mediumtext | 0-16777215字节   | 中等长度文本数据                     |
| longblob   | 0-4294967295字节 | 二进制形式的极大文本数据             |
| longtext   | 0-4294967295字节 | 极大文本数据                         |

- 日期和时间类型

| 类型名称  | 格式                | 用途                   |
| --------- | ------------------- | ---------------------- |
| DATE      | YYYY-MM-DD          | 日期                   |
| TIME      | HH:MM:SS            | 时间                   |
| DATETIME  | YYYY-MM-DD HH:MM:SS | 混合日期和时间         |
| TIMESTAMP | YYYY-MM-DD HH:MM:SS | 混合日期和时间，时间戳 |

timestamp 时间范围：  1970.1.1  0：0：0   ------	2037

#### 2、表约束

用于对表中的列作出修饰：

- primary key  主键 -- 唯一标记数据库表中的一条记录  不能重复 + 非空
- not null 非空
- null  可空 （默认是可空）
- unique  唯一

```mysql
create table userlist (
    id int primary key auto_increment,
    name varchar(20) not null unique,
    ...
)
```

#### 3、列选项

- auto_increment 自增长，一般用在主键维护上，默认从1开始
- default 列的默认值，插入数据时列的缺省值

```mysql
create table userlist (
    id int primary key,auto_increment,
    name varchar(20) default 'jingjing',
    age tinyint default 18,
    ...
)
```

#### 4、表选项

- engine                   引擎选择--一般使用innodb

- default charset   表中的数据的编码格式

  `1、建库最好选择字符集为utf-8  建表时将会继承库的字符集`

  `2、如果建库时没有选择字符符，则建表必须设置字符集，否则会出现中文乱码`

  `3、建议将my.ini配置文件中的字符集更改为utf8 则数据库将会继承配置中的字符集`

- auto_increment   自增长列的起始值

```mysql
create table user9(
	id int auto_increment primary key,
	name varchar(20) NOT NULL DEFAULT 'Mr_lee',
	nick varchar(15) UNIQUE,
 	birth timestamp
)ENGINE = InnoDB default charset = utf8 auto_increment = 9;

create table user10(
	id int auto_increment  ,
	name varchar(20) NOT NULL DEFAULT 'Mr_lee',
	nick varchar(15),
 	birth timestamp,
   	primary key(id), #表级约束
	unique(nick)     #表级约束
)ENGINE = InnoDB default charset = utf8 auto_increment = 9;
```

#### 5、联合约束

```mysql
create table t_user(
	id int auto_increment,
	name varchar(20),
	age tinyint,
	nick varchar(20),
	primary key(id,name),  # id 和 name的组合作为联合主键
	unique(age,nick)       # age 和 nick 的组合不能重复
)ENGINE = InnoDB default charset = utf8;
```

**删除表：drop table 表名;**



### 三、操作表中的数据

#### 1、插入数据

语法：

```mysql
INSERT INTO table_name ( field1, field2,...fieldN )
                VALUES ( value1, value2,...valueN );
```

插入数据：

```mysql
insert into user9 (id,name,nick,birthday) values (1,'Mr_lee','zz','2000-01-01')
```

**注意：values前后两个括号中的值 数量要一致**



- 主键是自动增长，mysql自动填充值,不建议认为指定id的值！！！

```mysql
insert into user9 (name,nick,birthday) values ('Mr_lee','zz22','2000-01-01')
```

- 一次插入多条数据

```mysql
insert into user9 (name,nick,birthday) values ('Mr_lee','zz22','2000-01-01'),
                                            ('Mr_lee','zz33','2000-01-01'),
                                            ('Mr_lee','zz44','2000-01-01')
```

- 省略列名插入，要求values中必须是完整的列对应的值

```mysql
 insert into user9 values (1,'Mr_lee','zz','2000-01-01')
```



#### 2、更新数据

语法：

```mysql
UPDATE table_name set field1=new_value1,field2=new_value2 ...  where ...
```

**注意**：更新数据时务必指定where条件，否则将会影响所有行的值。

- 更新id为1的用户的姓名为Mr_lee

```mysql
update user9 set name='Mr_lee' where id=1;
```

- 更新所有人的姓名（慎用）

```mysql
update user9 set name='Mr_lee' ;
```



#### 3、删除数据

语法：

```mysql
delete from table_name where ..;
```

- 删除所有数据 - 慎用

```mysql
delete from user9;
```

- 删除birthday是2018/12/12的用户

```mysql
delete from user9 where birthday = '2018/12/12'
```

