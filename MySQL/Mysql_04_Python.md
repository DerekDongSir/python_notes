# Python 连接 MySQL  

### 一、安装MySQL驱动 

#### 1、分类

- mysql-connector-python  - mysql 官方提供，纯python构建
- PyMySQL - 开源作者methane和adamchainz 提供，纯python构建
- cymysql - fork of pymysql，C构建
- mysqlclient - 开源作者methane提供，C构建
- mysqldb 不适用于python3

#### 2、安装模块

- pip install mysqlclient    或
- pip install mysqlclient-1.3.12-cp36-cp36m-win_amd64.whl

pip : 是python的一个第三方的扩展包，作用是，帮助管理对所有第三方包的安装、卸载

**注意：**添加python安装目录下的scripts目录到系统的环境变量中

**注意：**在使用pip前，一定要更新pip到最新版本：`python -m pip install --upgrade pip`

注意：如果pip未安装，或者被卸载。可以 `python get-pip.py` 安装pip

补充： `pip list`   查看已经安装的所有第三方包

​            `pip install 包名`  安装第三方的包

​            `pip download  包名`  下载第三方的包

​            `pip uninstall 包名`  卸载第三方的包

第三方包安装位置：python安装目录下的lib/site-packages中



**总结安装步骤：**

​	1、打开cmd窗口，先更新pip到最新版本`pip install --upgrade pip`  如果提交没有pip指定，需要将Python安装目录下的scripts目录添加到系统的环境变量中

​	2、直接安装：`pip install mysqlclient`  会连网搜索并下载对应的python版本的mysqlclient并安装

​	3、检查是否安装成功：`pip list`查看是否有mysqlclient即可

补充：如果第二步安装失败，下载对应版本的.whl文件 再使用 pip install xxx.whl安装



### 二、数据库操作API

#### 1、获取连接

```python
import MySQLdb
conn = MySQLdb.connect(
    host='localhost',    # mysql所在主机的ip
    port=3306, 		    # mysql的端口号
    user="root",         # mysql 用户名
    password="123456",   # mysql 的密码
    db="testdb",          # 要使用的库名
    charset="utf8"      # 连接中使用的字符集 
)
```

####2、获得Cursor对象

```python
cursor = conn.cursor()  # 游标
```

#### 3、执行SQL

```python
#执行查询语句，返回查询出有多少行
row_count = cursor.execute("select id,name,salary from t_employee where id>5")

id=5
row_count = cursor.execute("select id,name,salary from t_employee where id>"+str(id))

#执行增删改，返回影响的行数
sql = "insert into t_employee (name,salary,age) values('Mr_lee',100.33,18)"
row_count = cursor.execute(sql)

name='Mr_lee'
age=18
salary=50000
sql = "insert into t_employee (name,salary,age) values('%s',%f,%d)"%(name,salary,age)
row_count = cursor.execute(sql)
```

#### 4、执行查询

```python
row_count = cursor.execute("查询的sql语句")     #返回查询到的数据行数，没有数据返回0
cursor.fetchone()       #在查询出的数据中，获取一条数据
cursor.fetchmany(3)     #在查询出的数据中，获取3条数据 
cursor.fetchall()       #获取所有数据
```

- cursor中是所有查到的数据，从中获取数据，需要向下移动游标，游标起始位置是第一行数据上一行

- fetchone()  游标下移一行 ，获取一行数据(tuple)，如果没有数据返回None

- fetchmany(n)   游标下移n行，获取n行数据(tuple of tuple)，如果没有数据返回空tuple：()

- fetchall()   游标移动到底，获取所有数据行(tuple of tuple)，如果没有数据返回空tuple：()


技巧：有值/None    非空tuple/空tuple 可以作为 True/False使用 ，判断是否查询到了数据

从cursor中获取的每一行数据的格式为一个tuple，其中是各列的值

数据表中的列类型和python类型的对应为：

​	int/double/float == int/float

​	varchar/char == str

​	decimal == decimal.Decimal

​	datetime == datetime.datetime

#### 5、执行增删改

```
count = cursor.execute("增删改的sql语句")  返回影响的行数
```

**执行增删改需要控制事务，提交或回滚事务**

```python
 conn.commit() 提交
 conn.rollback() 回滚
```

#### 6、资源回收

```python
cursor.close() #先关闭cursor
conn.close() #后关闭conn

conn.commit()
cursor.close()
conn.close()
```

#### 7、cursor类型选择

```python
cursor = conn.cursor(MySQLdb.cursors.DictCursor) #选择了cursor类型
cursor.execute(sql)
#{'id': 1, 'name': 'Mr_lee', 'birth': datetime.datetime(2018, 4, 24, 21, 26, 3)}
cursor.fetchone() #此时返回的不再是tuple，而是一个dict，key是列名，value是列值

cursor.fetchall()/fetchmany() #返回 tuple of dict，每个dict是一行数据

```



总结：

Python操作Mysql的步骤：

​	（1）连接mysql         conn = MySQLdb.connect(xxx)

​	（2）获取游标对象    conn.cursor(游标的类型)

​	（3）执行SQL指令 -- 指令可以是 增删改查  cursor.execute("SQL语句")

​	（4）根据SQL指令的不同，需要做不同的操作

​		如果是查询指令：cursor.fetchone/fetchmany/fetchall

​		如果是增删改指令：conn.commit()

​	（5）释放资源   关闭curse和conn



### 三、SQL中传递参数

#### 1、SQL注入

所谓SQL注入，就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令，攻击站点，达到入侵目的。 

```python
name="' or '1'='1"
sql = "select id,name,age from user where name='%s'" % name      # name = '' or '1'='1'
total = cursor.execute(sql3) #此时执行sql，可以防止sql注入

出现如上问题，根源：sql语句有字符串拼接
```

防止

```python
name="' or '1'='1"
sql = "select id,name,age from user where name=%s"
total = cursor.execute(sql,[name]) #此时执行sql，其中的参数内容会转义为没有sql语义的片段，可以防止sql注入
```



#### 2、传递参数

用 `%s` 或 `%(key)s` 做占位符，接收参数

```mysql
sql = "select id,name,age from test where name=%s and id>%s"   #%s 做占位符
total = cursor.execute(sql ，["Mr_lee" ，1] )  #列表

sql = "insert into test(name,age) values(%s,%s)"   #%s 做占位符
total = cursor.execute(sql ，（"Mr_lee" ，18） ) #元组
```

```mysql
# %(key)s 做占位符

sql = "select id,name,age from test where name=%(name)s and id>%(id)s" 
total = cursor.execute(sql , {"id":1,"name":"Mr_lee"} ) #dict
```



**总结：在执行带参数的sql语句时，使用cursor.execute(sql,[name]) ,防止sql注入的攻击**