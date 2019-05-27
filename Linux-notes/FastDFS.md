# FastDFS安装配置

### 一、简介

#### 1、 教程说明

本教程整合FastDFS、Nginx、Django、uWSGI，客户端访问Nginx服务器。Nginx负责将上传的文件通过Nginx插件模块`ngx_fastdfs_module`分发给FastDFS分布式文件系统，通过`upstream`模块将动态请求转发给uWSGI服务器处理。



#### 2、为什么要使用FastDFS

（1）不使用FastDFS时，在Django中，文件上传处理如下：

- model中声明ImageField字段或FileField字段：

```python
class User(models.Model):
    name = models.CharField(max_length=30)
    imagepath = models.ImageField(upload_to='Mr_lee')
```

- 在settings.py中配置：

```python
MEDIA_ROOT = os.path.join(BASE_DIR,'media')
STATIC_URL = '/static/'
STATICFILES_DIRS = [ os.path.join(BASE_DIR, 'static'), MEDIA_ROOT ]  #将media目录添加为静态资源访问路径
```

则当上传文件（图片）后，会被保存在项目根目录的media目录的'Mr_lee'目录下。



在使用Nginx做负载均衡时，配置一台Nginx服务器，多台uWSGI服务器，并在uWSGI服务器中部署相同的Django项目。

同一用户在访问Web服务器Nginx时，会根据负载均衡策略分发给各个uWSGI服务器，此时用户上传的文件也会分布在多个uWSGI服务器中，此时将会导致用户在访问文件时缺少的情况发生。



（2）解决`(1)`中的问题，Nginx的作用之一：动静分离，即将静态请求交给Nginx自己处理，转发动态请求给uWSGI服务器。在Nginx中配置如下：

```nginx
location /static { 
	alias /usr/local/static;   # 当请求的url中有static时，会在Nginx服务器的/usr/local/static目录下                                # 查找静态资源
}
```

那么我们只需要将上传的文件存放在Nginx的`/usr/local/static`目录下即可。

在Nginx中提供如下配置：

     upstream ems {
         server 192.168.157.151:9001;
     }
     server {
           ...
             location /upload {
                    # 转到后台处理URL,表示Nginx接收完上传的文件后，然后交给后端处理的地址
                    upload_pass @python;
    
                    # 临时保存路径, 可以使用散列
                    # 上传模块接收到的文件临时存放的路径， 1 表示方式，该方式是需要在/tmp/nginx_upload下创建以0到9为目录名称的目录，上传时候会进行一个散列处理。
                    #upload_store /tmp/nginx_upload;
    
                    # 上传文件的权限，rw表示读写 r只读
                    upload_store_access user:rw group:rw all:rw;
                    set $upload_field_name "file";
    
                    # upload_resumable on;
                    # 这里写入http报头，pass到后台页面后能获取这里set的报头字段
                    upload_set_form_field "${upload_field_name}_name" $upload_file_name;
                    upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;
                    upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;
                    # Upload模块自动生成的一些信息，如文件大小与文件md5值
    
                    upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;
                    upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;
    
                    # 允许的字段，允许全部可以 "^.*$"
                    upload_pass_form_field "^.*$";
                    # upload_pass_form_field "^submit$|^description$";
    
                    # 每秒字节速度控制，0表示不受控制，默认0, 128K
                    upload_limit_rate 0;
    
                    # 如果pass页面是以下状态码，就删除此次上传的临时文件
                    upload_cleanup 400 404 499 500-505;
    
                    # 打开开关，意思就是把前端脚本请求的参数会传给后端的脚本语言，比如：http://192.168.1.251:9000/upload/?k=23,后台可以通过POST['k']来访问。
                    upload_pass_args on;  
                }
                
                location @python {
                    proxy_pass http://localhost:9999;
                    # return 200;  # 如果不需要后端程序处理，直接返回200即可
                }
                
                location / {
                    uwsgi_pass ems;
                    include  /usr/local/nginx/conf/uwsgi_params;
                }
                
                location /static {
                    alias /usr/local/static;
                }
    }
测试文件上传成功，但在Nginx做负载均衡分发给uWSGI时出错（/upload和/冲突？）

因此在这里推出了FastDFS来解决上述问题。



#### 3、什么是FastDFS

FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。 
FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服服。



### 二、安装FastDFS

#### 1、安装依赖

-  `yum install gcc-c++  `   安装gcc

- `yum –y install libevent libevent-devel  `  安装libevent

- 安装libfastcommon依赖环境：

  - 解压、编译、安装

    `tar -zxvf libfastcommon.tar.gz

    `cd 到解压目录`

    `./make.sh`

    `./make.sh install`

  - libfastcommon.so 安装到了/usr/lib64/libfastcommon.so，但是FastDFS主程序设置的lib目录是/usr/local/lib，所以需要创建软链接。 

    ```
    ln -s /usr/lib64/libfastcommon.so  /usr/local/lib/libfastcommon.so
    ln -s /usr/lib64/libfastcommon.so  /usr/lib/libfastcommon.so
    ln -s /usr/lib64/libfdfsclient.so  /usr/local/lib/libfdfsclient.so
    ln -s /usr/lib64/libfdfsclient.so  /usr/lib/libfdfsclient.so 
    ```

#### 2、安装FastDFS

解压，编译，安装

- 解压tar包：`tar -zxvf FastDFS_v5.05.tar.gz`
- 进入解压目录：`cd FastDFS`
- 编译：`./make.sh`
- 安装：`./make.sh install`
- 拷贝FastDFS解压目录中conf下所有的配置文件到/etc/fdfs中：`cp conf/* /etc/fdfs/`



#### 3、安装tracker

配置FastDFS跟踪器(Tracker)：由于tracker运行程序就是fasfdfs，fastDFS安装成功，只需要修改/etc/fdfs/tracker.conf配置文件即可。 

- 修改base_path存储基本路径

  `base_path=/home/fastdfs`               fastdfs需要手动创建，fastdfs在home目录下 （或自己指定目录） 

- 修改存在组

  `store_group=group1 `

- HTTP 服务端口

  `http.server_port=80`

- 创建tracker基础数据目录，即base_path对应的目录 

  `mkdir /home/fastdfs`

- 防火墙中打开跟踪端口（默认的22122） 

  ```
  vi /etc/sysconfig/iptables
  
  添加如下端口行
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT
  
  重启防火墙
  service iptables restart
  ```

- **测试启动tracker** 

  （1）以这种方式启动

  `/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart  `

  （2）或建立软连接

  ```
  ln -s /usr/bin/fdfs_trackerd   /usr/local/bin
  ln -s /usr/bin/fdfs_storaged   /usr/local/bin
  ln -s /usr/bin/stop.sh         /usr/local/bin
  ln -s /usr/bin/restart.sh      /usr/local/bin
  ```

  则可以使用

  ```
  service fdfs_trackerd start
  ```



查看 FastDFS Tracker 是否已成功启动 ，22122端口正在被监听，则算是Tracker服务安装成功。 

```
netstat -unltp|grep fdfs
```

关闭Tracker命令： 

```
service fdfs_trackerd stop
```

设置Tracker开机启动 ：

```
chkconfig fdfs_trackerd on
```

Tracker服务启动成功后，会在base_path下创建data、logs两个目录。目录结构如下： 

```
${base_path}
  |__data
  |   |__storage_groups.dat：存储分组信息
  |   |__storage_servers.dat：存储服务器列表
  |__logs
  |   |__trackerd.log： tracker server 日志文件 
```



#### 4、配置 FastDFS 存储 (Storage)

（1）进入/etc/fdfs/storage.conf  编辑：

```
# 配置文件是否不生效，false 为生效
disabled=false 

-------------------------------需要修改的------------------------------------------
# 指定此 storage server 所在 组(卷)   必须和tracker的组名相同 
group_name=group1

# storage server 服务端口
port=23000

# 心跳间隔时间，单位为秒 (这里是指主动向 tracker server 发送心跳)
heart_beat_interval=30

-------------------------------需要修改的------------------------------------------
# Storage 数据和日志目录地址(根目录必须存在，子目录会自动生成)
base_path=/home/fastdfs/storage

# 存放文件时 storage server 支持多个路径。这里配置存放文件的基路径数目，通常只配一个目录。
store_path_count=1

-------------------------------需要修改的------------------------------------------
# 逐一配置 store_path_count 个路径，索引号基于 0。
# 如果不配置 store_path0，那它就和 base_path 对应的路径一样。
store_path0=/home/fastdfs/file

# FastDFS 存储文件时，采用了两级目录。这里配置存放文件的目录个数。 
# 如果本参数只为 N（如： 256），那么 storage server 在初次运行时，会在 store_path 下自动创建 N * N 个存放文件的子目录。
subdir_count_per_path=256

-------------------------------需要修改的------------------------------------------
# tracker_server 的列表 ，会主动连接 tracker_server
# 有多个 tracker server 时，每个 tracker server 写一行
tracker_server=192.168.157.141:22122

# 允许系统同步的时间段 (默认是全天) 。一般用于避免高峰同步产生一些问题而设定。
sync_start_time=00:00
sync_end_time=23:59
# 访问端口
http.server_port=80
```

（2）创建Storage基础数据目录，对应base_path目录 

```
mkdir -p /home/fastdfs/storage
mkdir -p /home/fastdfs/file
```

（3）防火墙中打开存储器端口（默认的 23000）  

```
vi /etc/sysconfig/iptables

添加如下端口行：
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT

重启防火墙：
service iptables restart
```

（4）启动 Storage 

启动Storage前确保Tracker是启动的。初次启动成功，会在 /home/fastdfs/storage 目录下创建 data、 logs 两个目录。 

```
service fdfs_storaged start
```

（5）查看Storage 是否成功启动，23000 端口正在被监听，就算 Storage 启动成功。 

```
netstat -unltp|grep fdfs
```

![1539844995527](pic\storage启动.png)

（6）设置 Storage 开机启动 

```
chkconfig fdfs_storaged on
```



**查看Storage和Tracker是否在通信：** 

```
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

![1539845124297](pic\1539845124297.png)



（7）Storage 目录 

同 Tracker，Storage 启动成功后，在base_path 下创建了data、logs目录，记录着 Storage Server 的信息。

在 store_path0 目录下，创建了N*N个子目录：

![1539845216036](pic\1539845216036.png)



#### 5、文件上传测试

（1）修改 Tracker 服务器中的客户端配置文件  

`vi /etc/fdfs/client.conf`

修改如下配置：

```
# Client 的数据和日志目录
base_path=/home/fastdfs/client

# Tracker端口
tracker_server=192.168.157.141:22122
```

（2）测试文件上传

在当前目录下`touch 1.txt`

`/usr/bin/fdfs_test /etc/fdfs/client.conf upload 1.txt`

或`/usr/bin/fdfs_upload_file /etc/fdfs/client.conf namei.jpeg`

![1539845554052](pic\1539845554052.png)

返回的文件ID由group、存储目录、两级子目录、fileid、文件后缀名（由客户端指定，主要用于区分文件类型）拼接而成。 

![img](pic\856154-20171011151728965-914197096.png) 



（3） 通过路径访问文件或图片

**注意：如果服务器使用的是外网ip，那么生成的图片路径是无法直接访问到的（虽然已经上传成功）。这时需要结合nginx来访问图片**  



### 三、通过Nginx访问FastDFS文件

#### 1、安装Nginx （略）

#### 2、修改Nginx配置文件 

` vi /usr/local/nginx/conf/nginx.conf`

添加如下：

```nginx
location /group1/M00 {
    alias /home/fastdfs/file/data;
}
```

![1539847470032](pic\1539847470032.png)

在浏览器访问之前上传的文件或图片成功。

```
 http://192.168.157.141/group1/M00/00/00/wKidjVvIpGSAEmeaAAAAEJjQvFw914_big.txt 
```

![1539847547664](pic\1539847547664.png)



### 四、Nginx中配置FastDFS模块

#### 1、安装fastdfs-nginx-module 模块 

（1）将fastdfs-nginx-module_v1.16.tar.gz发送到/usr/local下并解压

​	 修改fastdfs-nginx-module/src/config文件：

```
 CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"   # 去掉local

 CORE_LIBS="$CORE_LIBS -L/usr/lib -lfastcommon -lfdfsclient"        #去掉local
```

建立软连接：

```
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so 
```

#### 2、配置Nginx

```
先停掉nginx服务
# nginx -s stop

进入Nginx解压包目录  如果不存在则重新解压Nginx的tar包
# cd nginx-1.11.1/

# 添加模块
# ./configure --add-module=../fastdfs-nginx-module/src

重新编译、安装
# make && make install
```

![1539850630072](pic\1539850630072.png)

#### 3、配置mod_fastdfs.conf

进入`cd /usr/local/fastdfs-nginx-module/src/`

将mod_fastdfs.conf拷贝到/etc/fdfs目录：`cp mod_fastdfs.conf /etc/fdfs/`

然后：` vi /etc/fdfs/mod_fastdfs.conf`

```
# 连接超时时间
connect_timeout=10

base_path=/home/fastdfs/storage

# Tracker Server
tracker_server=192.168.157.141:22122

# StorageServer 默认端口
storage_server_port=23000

# 如果文件ID的uri中包含/group**，则要设置为true
url_have_group_name = true

# Storage 配置的store_path0路径，必须和storage.conf中的一致
store_path0=/home/fastdfs/file
```

拷贝usr/lib64目录下库文件libfdfsclient.so

`cp /usr/lib64/libfdfsclient.so /usr/lib  `



#### 4、复制 FastDFS 配置文件

复制 FastDFS 的部分配置文件到/etc/fdfs 目录 

```
cd /usr/local/FastDFS/conf/
cp anti-steal.jpg http.conf mime.types /etc/fdfs/
```

#### 5、修改nginx.conf 

```
vi /usr/local/nginx/conf/nginx.conf
```

```
location ~/group([0-9])/M00 {
    ngx_fastdfs_module;
}
```

注意：

　　listen 80 端口值是要与 /etc/fdfs/storage.conf 中的 http.server_port=80 (前面改成80了)相对应。如果改成其它端口，则需要统一，同时在防火墙中打开该端口。

　　location 的配置，如果有多个group则配置location ~/group([0-9])/M00 ，没有则不用配group。



在/home/fastdfs/file 文件存储目录下创建软连接，将其链接到实际存放数据的目录，这一步可以省略。 

```
ln -s /home/fastdfs/file/data/ /home/fastdfs/file/data/M00 
```

#### 6、启动nginx 

打印如下则配置成功

![1539851075059](pic\1539851075059.png) 

上传文件：`/usr/bin/fdfs_test /etc/fdfs/client.conf upload a.jpg`

**访问：**

```
http://192.168.157.141/group1/M00/00/00/wKidjVvItgOADHFgAAvqH_kipG8189_big.jpg
```



### 五、Django配置FastDFS

开启一台新的Linux服务器，假设ip为192.168.157.150

在Linux下安装以下Python库：

#### 1、安装Django-FastDFS

- 下载fdfs_client-py  Django项目相当于客户端`https://github.com/jefforeilly/fdfs_client-py `
- 解压 fdfs_client-py-1.2.6.tar.gz 并进入解压包：
- 执行 `python setup.py install `
- `pip install mutagen `
- `pip isntall requests `



#### 2、创建Django项目  

（1）在Windows下使用pycharm创建或在Linux下使用django-admin startproject helloworld都可。

（2）在项目根目录下创建utils目录，并在utils目录下创建python package并命名为fastdfs

（3）在fastdfs下新建client.conf和fdfs_storage.py两个文件

（4）打开client.conf文件，配置如下内容：

```
# connect timeout in seconds
# default value is 30s

connect_timeout=30

# network timeout in seconds
# default value is 30s

network_timeout=120

# the base path to store log files
-------------------------------------需要修改-----------------------------------------------
base_path=/home/fastdfs/storage/logs

# tracker_server can ocur more than once, and tracker_server format is
# "host:port", host can be hostname or ip address
---------------------------------需要修改为FastDFS服务器ip---------------------------------------
tracker_server=192.168.157.141:22122

# standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug

log_level=info

# if use connection pool
# default value is false
# since V4.05

use_connection_pool = false

# connections whose the idle time exceeds this time will be closed
# unit: second
# default value is 3600

# since V4.05

connection_pool_max_idle_time = 3600

# if load FastDFS parameters from tracker server
# since V4.05
# default value is false

load_fdfs_parameters_from_tracker=false

# if use storage ID instead of IP address
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# default value is false
# since V4.05

use_storage_id = false

# specify storage ids filename, can use relative or absolute path
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# since V4.05

storage_ids_filename = storage_ids.conf

# HTTP settings

http.tracker_server_port=80

# use "#include" directive to include HTTP other settiongs
## include http.conf
```

（5）打开fdfs_storage.py文件，添加如下内容：

```python
from django.core.files.storage import Storage
from django.conf import settings
from fdfs_client.client import Fdfs_client

class FDFSStorage(Storage):
    '''fast dfs文件存储类'''
    def __init__(self, client_conf=None, base_url=None):
        '''初始化'''
        if client_conf is None:
            client_conf = settings.FDFS_CLIENT_CONF
        self.client_conf = client_conf

        if base_url is None:
            base_url = settings.FDFS_URL
        self.base_url = base_url

    def _open(self, name, mode='rb'):
        '''打开文件时使用'''
        pass

    def _save(self, name, content):
        '''保存文件时使用'''
        # name:你选择上传文件的名字 test.jpg
        # content:包含你上传文件内容的File对象

        # 创建一个Fdfs_client对象
        client = Fdfs_client(self.client_conf)

        # 上传文件到fast dfs系统中
        res = client.upload_by_buffer(content.read())

        # dict
        # {
        #     'Group name': group_name,
        #     'Remote file_id': remote_file_id,
        #     'Status': 'Upload successed.',
        #     'Local file name': '',
        #     'Uploaded size': upload_size,
        #     'Storage IP': storage_ip
        # }
        if res.get('Status') != 'Upload successed.':
            # 上传失败
            raise Exception('上传文件到fast dfs失败')

        # 获取返回的文件ID
        filename = res.get('Remote file_id')

        return filename

    def exists(self, name):
        '''Django判断文件名是否可用,返回False表示一直可用'''
        return False

    def url(self, name):
        '''返回访问文件的url路径,就是ImageField字段image的url属性的值,image.url,默认的image的url是这样的格式：'/media/001.jpg' '''
        return self.base_url + name
```

（6）在项目的settings.py文件中添加如下配置：

```
DEFAULT_FILE_STORAGE = 'utils.fastdfs.fdfs_storage.FDFSStorage'
FDFS_URL = "http://192.168.157.141:80/"     # FastDFS服务器ip
FDFS_CLIENT_CONF = os.path.join(BASE_DIR, "utils/fastdfs/client.conf")
```

