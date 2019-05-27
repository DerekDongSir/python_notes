# MySQL_03

### 一、关联关系

#### 1、概述

在实际的项目开发中，会有很多数据表，而且表之间不是孤立的，是存在关联关系的。关联关系的搭建是通过 **外键** 完成的。

设计部门表和员工表 （部门包含员工，员工从属于部门）。



#### 2、外键使用

外键语法：

```mysql
FOREIGN KEY(自己的列名) REFERENCES 对方表名（对方主键名）
            dept_id               t_department(id)
```

示例

```mysql
create table t_department(
	id int auto_increment primary key,
	...
)ENGINE InnoDB default charset utf8;

create table t_employee(
	id int auto_increment primary key,
	...
	dept_id int,
	FOREIGN key(dept_id) REFERENCES t_department(id)   # 约束外键的值只能是t_department表中已存在的id值
)ENGINE InnoDB default charset utf8
```
![foreign_key](.\Mysql-notes-pic\foreign_key.png) 

1.  **外键列的类型要和对方主键列类型保持一致**

2.  **外键列中的值必须是对方主键列中值的子集**

3.  **插入数据时先插入主表**

4.  **删除 数据时先删除从表** (ops:有外键的一方为从表) 

  

#### 3、关系种类

##### 3.1 一对一

```mysql
-- 外键 + 唯一  --   

create table person(
	id int primary key auto_increment,
	name varchar(20) unique,
	age tinyint,
)ENGINE=INNODB DEFAULT CHARSET=utf8;

create table passport(
	id int primary key auto_increment,
	note varchar(20),
	expire TINYINT not null,
	persion_id int,
	FOREIGN KEY(person_id) REFERENCES person(id), #外键
	unique(persion_id) #唯一
)ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=10000;
```



##### 3.2 一对多 

```mysql
-- 外键 --
create table t_department( 1
	id int primary key auto_increment,
	title varchar(20) unique,
	note varchar(20)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

create table t_employee( *
	id int primary key auto_increment,
	name varchar(20) unique,
	age TINYINT not null,
	dept_id int not null,
	FOREIGN KEY(dept_id) REFERENCES t_department(id)
)ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=10000;
```



##### 3.3 多对多

```mysql
-- 第三方表+双外键+联合主键 --

create table student( 
	id int PRIMARY key auto_increment,
	name varchar(20),
	age TINYINT
)ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=10000;

create table course(
	id int PRIMARY key auto_increment,
	title varchar(20) unique,
	expire TINYINT
)ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=10000;

-- 第三方表 关系表
create table relation(
	student_id int,
	course_id int,
	FOREIGN KEY(student_id) REFERENCES student(id),
	FOREIGN KEY(course_id) REFERENCES course(id),
	PRIMARY key(student_id,course_id)
)ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=10000;
```



### 二、事务

#### 1、概念

- 在一个复杂的业务处理中，会有复杂的多次的数据操作，这些操作全部成功才意味着业务的完成。但是如果其中的某一步数据通信出错，那整个业务也就失败。  
- 保证一个业务中的诸多数据操作一旦有任何一处的出错可以立即全盘回退，使数据库不出现非法数据，至关重要！
- 我们把多个数据操作打包成一个事务，事务内的所有操作，要么都成功，要么全盘回滚。



#### 2、事务的使用

- 表选项中使用**InnoDB**引擎，支持事务 

- 事务如何开启
  - **begin;** 语句执行，会开启一个事务
  - **start  transaction;**   语句执行，也可以开启一个事务
- 事务如何，何时结束
  - **commit;**   语句执行，事务提交 -- 如果事务处理的过程中遇到错误，MySQL会自动回滚数据。

  - **rollback;**   语句执行，事务回滚（手动回滚，由程序员决定什么时候需要回滚数据，以及是否需要回滚）

    **细节：如果事务内部，出现了数据操作失败，自动回滚。如果需要手动回滚，可以执行：rollback;**

```mysql
begin;
insert into order10(price,note,user_id) values(123.55,'zzz',1);
commit;
或者
rollback;
```



#### 3、事务的特性ACID

- Atomicity（原子性）：一个事务是一个不可分割的工作单位，事务中包括的诸多操作要么都做，要么都不做。
- Consistency（一致性）：事务的干涉下，数据库总是从一个一致性的状态转换到另一个一致性的状态，数据的逻辑是完整和正确的。

- Isolation（隔离性）：一个事务中的数据，对其他事务是隔离的。 
- Durability（持久性）：事务一旦提交，处理结束后，事务中的数据就持久化到数据库中。



#### 4、事务并发问题  

- **脏读：**事务A读取到了事务B中的数据，但是事务B回滚了或再次更新了数据，事务A读取到的为脏数据（一个事务读取了另一个未提交的事务中的数据）   

  `A开启了事务，读数据，此时B更新了数据，但未提交，此时A再读数据，读到的就是脏数据。dirty data`   

- **不可重复读：**事务A多次读取同一数据的过程中，事务B更新了此数据并提交了事务，则事务A多次读取到不同的数据(一个事物内多次读取同一数据，但是结果不一致)

  `A开启了事务，读数据，此时B更新了数据并提交了事务，此时A再读数据，读到的不一致。`

- **幻影读：**事务A多次读取同一张表的数据的过程中，事务B更新了表中的数据(增加或删除)，则事务A发现出现了莫名其妙的额外的数据。 

  - 一个事务多次查询，数据行数不一致 (在低于 ‘repeatable-read中出现’)

  - 一个事务通过一次查询的结果，决定可以添加一个不重复的数据，但实际添加时却发现表中已有数据重复

    (在低于‘serializable’中出现)

- **更新丢失：**后续解释



**总结：**  

- 脏读与不可重复读的区别：
  - 脏读是B事务更新但未提交事务
  - 不可重复读：B事务更新并提交了事务，A多次读到的数据不一致

- 幻影读与脏读和不可重复读的区别：
  - 幻影读是对数据行的增加或删除
  - 脏读和不可重复读是对数据内容的修改





#### 5、事务隔离级别

- 获取当前数据库的隔离级别，mysql的默认隔离级别为`repeatable-read`  **可重复读**

​     `select @@tx_isolation;`

- 设置隔离级别 

​      `set session transaction isolation level  read uncommitted;`

​      `set session transaction isolation level  read committed;`

​      `set session transaction isolation level  repeatable read;`

​      `set session transaction isolation level  serializable;`

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 读未提交（read-uncommitted） | 是   | 是         | 是   |
| 读提交（read-committed）     | 否   | 是         | 是   |
| 可重复读（repeatable-read）  | 否   | 否         | 是   |
| 串行化（serializable）       | 否   | 否         | 否   |

**隔离级别由低到高，安全性逐渐加强，并发性能逐渐降低**



#### 6、锁

**更新丢失：**当两个或多个事务选择同一行数据，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题 －－最后的更新覆盖了由其他事务所做的更新。 



增删改语句，会自动对所操作的数据行，加锁。其他事务如果也要增删改相同的数据行，会被阻塞。

拥有锁的事务，在事务结束时，释放锁

基于事务并发中的 **”更新丢失“**展开讨论

场景：大量的事务并发，而且，操作相同的数据

- 悲观锁： `select xxx from xx for update;`   行级锁，排它锁
- 乐观锁：在数据中新增一列 `version`  记录数据更新的版本。并不是真正的锁

```mysql
t_user: id  name     age
	    1   Mr_lee   18
tx1:age+1
	1>select age from t_user where id=1;
      age=18
    3>update t_user set age=(18+1) where id=1;
	  commit;
	  age=19
tx2:age+2
	2>select age from t_user where id=1;
	  age=18
	4>update t_user set age=(18+2) where id=1;
      commit;
	  age=20   
-- 出现如上问题，根本原因是：查询操作不互斥，即查询动作不会对操作的数据加锁(ops:增删改会对操作的数据加锁)
```

```mysql
-- 悲观锁：
tx1:age++
	1>select age from t_user where id=1 for update; #此时会对数据加锁
		age=18
	2>update t_user set age=18+1 where id=1;
	  commit;
	  age=19
tx2:age+2
	3>select age from t_user where id=1 for update; #此时会对数据加锁
		age=19;
	4>update t_user set age=19+2 where id=1;
	  commit;
	  age=21
```

```mysql
-- 乐观锁
t_user   id   name    age   version
          1   Mr_lee  18    0
tx1:age+1
	1>select age,version from t_user where id=1;
        age=18;version=0;
	3>update t_user set age=18+1,version=0+1 where id=1 and version=0;
	  commit;
		age=19;version=1
tx2:age+2
	2>select age,version from t_user where id=1;
        age=18;version=0;
	4>update t_user set age=18+2,version=0+1 where id=1 and version=0;
	  commit;
	--  未更新到数据行   
```



### 三、索引 

#### 1、索引作用

在查询数据表时，默认会从第一行数据依次查到最后一行。所以当数据量较多时，查询很耗时。如果作为查询条件的列有索引，就可以不必遍历每一条数据，就可以很快找到数据。   

`select * from t_user where age = 18`

#### 2、创建索引

- 表中的**主键列**和**外键列** 和**唯一列**自动有索引

##### 2.1 单列索引

```mysql
create table t_user(
	id int primary key,
	name varchar(20),
	index name_ind (name)   # select * from t_user where age = 18;
);
或者
create index ind_name on t_user(name);
*细节：主键列，外键列，唯一列 自带索引
```

如此，再以name为条件查询时，速度会有很大提升

`select .. from .. where name = 'xxx'`



##### 2.2 联合索引

```mysql
create index name_birth_ind on test3(name,birthday);
```

- select ..  from ..  where name = 'Mr_lee' and birthday = '2018/12/12' 

- 用and拼接的多个条件，此时如果有联合索引则会对查询效率有极大促进

- 如果单独对name和birthday两列分别做索引，则因为每次查询时只能用一个索引，不如联合索引效率高



##### 2.3 删除索引

```mysql
drop index name_ind on t_user;
```



##### 2.4 查看索引

```mysql
show index from t_user;
```



##### 2.5 索引特性

- 对查询有增益，使查询速度加快

- 对于增删改会有额外的索引维护的时间消耗   

- 索引不是越多越好

  - 对于修改较多的表，不适合做过多索引
  - 对于查询较多的表，在作为查询条件的列上做索引，提高查询效率
  - 盲目建索引，不仅增加大量的索引维护时间，而且也是对存储空间的严重侵蚀

  

### 四、SQL分类  

- 数据查询语言DQL（Data Query Language）：select、where、order by、group by、having。
- 数据定义语言DDL（Data Definition Language）：create、alter、drop。
- 数据操作语言DML（DataManipulation Language）：insert、update、delete 。
- 事务处理语言TPL（Transaction Process Language）：commit、rollback 。
- 数据控制语言DCL（Data Control Language）：grant、revoke。
  - grant all privileges on \*.\* to 'root'@'%' identified by '222222' with grant option;
  - 所有特权                 库.表           用户名@host                    密码           允许将特权授权给他人
  - flush privileges; #刷新特权，使生效

