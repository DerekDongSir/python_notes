# Django环境部署

### 一、MySQL安装 

#### 1、安装方式

##### 1.1 方式一 yum安装（推荐使用该方式）

`yum install -y mysql-server`  # yum安装，需要外网环境      

##### 1.2 方式二 rpm安装（不建议） 

- `rpm -ivh perl-*.rpm`  #安装所有perl依赖
- `rpm -Uvh mysql-libs-5.1.73-7.el6.i686.rpm`  #更新mysql的类库
- `rpm -ivh mysql-5.1.73-7.el6.i686.rpm  mysql-server-5.1.73-7.el6.i686.rpm ` #安装mysql主服务

##### 1.3 方式三 克隆虚拟机

使用虚拟机克隆：打开提供好的克隆虚拟机即可。其中已经安装好所有依赖，可直接使用。

- VMware打开克隆的虚拟机
- 克隆的虚拟机的网络适配,使得虚拟机可以进入局域网
  - `vi  /etc/sysconfig/network-scripts/ifcfg-eth0`
  - 删除 HWADDR所在行、 UUID所在行
  - 将`/etc/udev/rules.d/`目录中的`70-persistent-net.rules`文件删除
  - 重启虚拟机



#### 2、启动MySQL服务

`service mysqld start/stop/restart/status`



#### 3、使用MySQL

初次使用，请设置MySQL密码：`mysqladmin -u root password "123456"`

- 连接数据库
- 选择数据库
- 选择表
- 查询表



#### 4、MySQL远程连接

- 到mysql库的user表中
- `update user set host='%',password=password('123456') where host='127.0.0.1';` #添加可以远程访问的账号
- `flush privileges;` #刷新权限，保证新添加的账号可用
- 关闭linux的防火墙，保证3306可以访问



**注意：防止mysql本地客户端数据乱码**

```
/etc/my.cnf 中添加如下配置，即可
[client]
default-character-set=utf8
```



#### 5、MySQL卸载

- `rpm -e mysql-server` #只需卸载主服务即可                 
- `rm  -rf  /var/lib/mysql` #删除所有mysql的数据



#### 6、MySQL root密码找回（重置）

找到`/etc/my.cnf`

```
[mysqld]
...
skip-grant-tables   //注意，建议在拆除网线的情况下添加  (而且添加配置后，需要重启mysqld服务)
```

```
[root@Server ~] mysql -uroot
mysql> use mysql;
mysql> update user set password=password('123') where host='localhost'; //修改密码
mysql> flush privileges;
```

然后将如上配置删除或注释



### 二、Python安装

#### 1、安装依赖

- yum -y install  python-devel  openssl-devel  bzip2-devel  zlib-devel  expat-devel ncurses-devel sqlite-devel gdbm-devel xz-devel tk-devel readline-devel  gcc 
- yum -y groupinstall "Development tools"

如上两步，汇总安装了python生产环境的各种第三方依赖包



#### 2、安装Python

- 将python的tar包发送给linux (建议位置：/usr/local/xx)

- 解压tar包:`tar -zxvf Python-3.5.2.tgz` 

- cd到解压目录中配置：`./configure --prefix=/usr/local/python3 --enable-optimizations` 

  目的：检测环境中依赖是否完整，设置python的安装位置，
  同时生成一个编译文件，用于进行python编译：make

- cd 到解压目录中：先 `make` 编译  然后  `make  install` 安装

```shell
安装后的日志如下
....
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-8.1.1 setuptools-20.10.1
```



- 将python3 设置为系统默认python解释器

  - 将/usr/bin下的`python`文件改名   `mv /usr/bin/python /usr/bin/python2.6.6`

  - 将python3的执行文件链接到  /usr/bin/python

    `ln -s /usr/local/python3/bin/python3 /usr/bin/python`

    


  

- 设置环境变量：/etc/profile中添加配置

```
在文件末尾追加，不要改动文件的其他内容！！！！！！！
export python_home=/usr/local/python3
export PATH=$PATH:$python_home/bin
```

**注意，设置好后，为了让环境变量生效：**`source /etc/profile`，然后 `python3`即可进入python3的环境

**注意，此时系统自带的python2 依然是默认python解释器**

- 更新pip
  - `pip3 install --upgrade pip`

**补充：**

- 安装依赖库：`yum install mysql-devel`  开发用到的库以及包含文件 
- 由于`yum`用python2编译执行，所以需要单独为`yum`设置为python2，找到`/usr/bin/yum`文件，修改文件头：`#!/usr/bin/python2.6.6`



### 三、Django安装

- 安装数据库驱动：`pip install mysqlclient`

- `pip install django=="2.0.6"`


- 测试使用：
  - `django-admin startproject testproj`  在当前目录下创建一个project:"testproj"
  - cd到testproj目录下的testporj目录下settings.py 修改配置：`ALLOWED_HOSTS = ["*"]`
  - 启动django内置的web服务器。cd到testproj目录下，执行：`python manage.py runserver ip:port`
  - 在浏览器中访问：`ip:port`



### 四、uWSGI服务器

#### 1、WSGI协议

- 使用Django或Flask框架编写的Web应用程序，在`python manage.py runserver`  时都启动的是框架内置的服务器来运行Web应用程序，而内置的服务器遵循了WSGI协议（WSGI Server）。

- WSGI：全称是`Web Server Gateway Interface`，WSGI不是服务器，python模块，框架，API或者任何软件，只是一种规范，描述web server如何与web application通信的规范。

  - `WSGI server`负责从客户端接收请求，将`request`转发给`application`，将`application`返回的`response`返回给客户端；

  - `WSGI application`接收由`server`转发的`request`，处理请求，并将处理结果返回给`server`。

    ![wsgi.png-22.9kB](Linux_pic\wsgi.png)  

- 要实现WSGI协议，必须同时实现web server和web application，当前运行在`WSGI`协议之上的`web`框架有`Bottle`, `Flask`, `Django`。 

**总结：**WSGI是Web 服务器(uWSGI)与 Web 应用程序或应用框架(Django)之间的一种低级别的接口。 



#### 2、uWSGI服务器安装

WSGI协议下web服务器很多：django内置，uWSGI，gunicorn。

`uWSGI`服务器自己实现了基于`wsgi`协议的`server`部分，我们只需要在`uwsgi`的配置文件中指定`application`的地址，`uWSGI`就能直接和应用框架中的`WSGI application`通信。 

##### 2.1 服务器安装

- 将uWSGI的tar包发送linux

- 解压tar：`tar -zxvf  uwsgi-2.0.17.tar.gz`

- cd到解压目录下，编译：`make`

- 为了可以更方便的执行  uwsgi   启动uWSGI服务器，定制链接：

  `ln -s /usr/local/uwsgi-2.0.17/uwsgi /usr/bin/uwsgi`

  则可以在任意目录下执行 `uwsgi` 去启动uWSGI服务器

- 测试使用python的wsgi服务器-uWSGI

  - 在任意的一个目录中定义一个python脚本：hello.py

    ```python
    def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html;charset=utf-8')])
        return [bytes('你好啊！！','utf-8'),b'Mr_lee']    # 基于wsgi协议规范实现的代码
    ```

  - 启动uWSGI服务器，并部署hello.py程序

    `uwsgi --http 192.168.248.128:8001 --wsgi-file hello.py`   **#注意hilo.py可以写成绝对路径**

  - 浏览器访问：`192.168.248.128:8001`



#### 3、 uWSGI部署django项目

- **设置mysql的引擎默认为：innodb**

  在`/etc/my.cnf`的`[mysqld]`中添加配置：`default-storage-engine=InnoDB`

- **建议设置为严格模式:**     

  在`/etc/my.cnf`的`[mysqld]`中添加配置 :  `sql_mode=STRICT_TRANS_TABLES`

  - `mysql> show variables where variable_name like '%mode%';#可以查看mysql的配置参数`

  修改完配置后，要重启Mysql的服务

- **在数据库中建好项目需要的database：“db9”**

  - 使用Navicat创建即可,注意字符集为 utf8

- **在Django项目的settings.py中修改配置**

  ```python
  DEBUG = False  #去掉开发模式
  ALLOWED_HOSTS = ["*"] #开放访问host
  DATABASES = { #合适数据库参数
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'ems',
          'USER': 'root',
          'HOST': 'localhost',
          'PORT': '3306',
          'PASSWORD': '123456'
      }
  }
  
  ```

- **发送项目到linux并做移植**

  `python manage.py makemigrations`

  `python manage.py migrate`

- **编写uWSGI的配置文件**

  ```python
  #随意找一个目录，创建一个文件：config.ini
  [uwsgi]
  http = 192.168.134.128:9000 # uWSGI服务器访问地址
  #uWSGI和nginx通信的port
  socket = 192.168.134.128:9001
  # the base directory (full path)
  chdir = /usr/local/django_projects/ems #项目所在目录
  # Django's wsgi file
  wsgi-file = ems/wsgi.py #基于项目目录的相对路径
  # maximum number of worker processes
  processes = 4
  #thread numbers startched in each worker process
  threads = 2
  #monitor uwsgi status 通过该端口可以监控 uwsgi 的负载情况
  stats = 192.168.134.128:9002
  # clear environment on exit
  vacuum = true
  pidfile = /usr/local/django_projects/ems/uwsgi.pid #进程ID存放于此文件，位置可以自定义
  #daemonize-run ,file-to-record-log
  daemonize = /usr/local/django_projects/ems/uwsgi.log #后台启动模式，日志文件记录位置自定义
  #http://ip:port/static/...请求会进入该目录找资源，此处可以指向某个app下的static目录
  #或是将所有静态文件汇总到项目的某一个目录下，然后配置在此是更好的选择
  #汇集所有已安装app的静态资源到一个目录下，请参见后续内容
  #http://ip:port/static/a/b/c/d.png   ==>  /usr/local/xxxx/static/a/b/c/d.png
  static-map =/static=/usr/local/xxx/static  # 只在你写的static-map中找静态资源 
  ```

- **根据如上配置启动uWSGI服务器**

  `uwsgi --ini config.ini`  **#注意：config.ini是一个相对路径**

- **关闭服务器**

  `uwsgi --stop /usr/local/django_projects/ems/uwsgi.pid` **#通过进程id文件**

- **部署项目技巧：静态资源管理之汇总**

  - 在project的settings.py中添加配置：`STATIC_ROOT = os.path.join(BASE_DIR,'static')`

    STATIC_ROOT指向project目录下的`static`目录

  - 执行汇总指令：`python manage.py collectstatic`  

    **会将所有已安装   *APP下的静态资源*    以及**

    **额外添加的静态目录   *STATICFILES_DIRS*  汇总到指定目录**

  - 然后在uWSGI的配置文件中：`static-map =/static=/usr/local/xxx/xxx  ==>指定目录`

