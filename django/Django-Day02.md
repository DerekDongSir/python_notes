# Django视图和URL配置

### 一、MTV模式

#### 1、概述

Django的MTV模式本质上和MVC是一样的，也是为了各组件间保持松耦合关系，只是定义上有些许不同，Django的MTV分别是值：

- **M 代表模型（Model）：**负责业务对象和数据库的关系映射(ORM)。
- **T 代表模板 (Template)：**负责如何把页面展示给用户(html)。
- **V 代表视图（View）**：负责业务逻辑，并在适当时候调用Model和Template。

Django中提供了View组件用来充当MVC中的C-controller。



#### 2、view的主要职责

- 接收请求
- 调度Model，处理业务逻辑
- 响应请求



### 二、基本开发流程

#### 1、创建App，定义view中的函数

在每个App的views.py中，定义一个hello函数

```python
def hello(request):  # 必须要有一个request参数 
	print("函数")
```

#### 2、函数返回响应

每个view必须要有响应，即返回一个HttpResponse

```python
from django.shortcuts import HttpResponse

def hello(request):
	return HttpResponse("<h1>view基本开发流程</h1>")
或者
    return render(request,'xxx.html')
```

#### 3、定义访问路径

在urls.py中，为每个view定义访问路径

```python
from django.urls import path
from view_app.views import hello   # 引入views模块中的hello

urlpatterns = [
	path("admin/",admin.site.urls),
	path("hello/",hello)            # 添加view中的配置
]
```

#### 4、启动服务

django内置了一个供开发使用的  web-服务器，虽然不适合生产环境，但用户开发测试，特别便利。

```python
python manage.py runserver  #默认ip和端口   localhost:8000

python manage.py runserver 0.0.0.0:8000 #修改默认的ip和端口
python manage.py runserver 192.168.0.3:8899
```

**注意**：修改了ip和port后，需要到settings.py中增加一个设置：ALLOWED_HOSTS = ["*"]

#### 5、访问view

```
http://localhost:8000/hello/

htpp://本机ip:8000/hello/
```

总结：启动服务器的几种方式

```python
# 1、python manage.py runserver == 在菜单项中点击run   localhost:8000
# 2、可以修改默认端口 python manage.py runserver 8899
# 3、以第一种方式或第二种方式启动的服务器，只能使用 127.0.0.1 来访问  其它客户端无法访问
# 4、如果想让其它电脑能够访问服务器，python manage.py runserver  自己电脑的ip:端口号
#    还需要在settings.py文件中修改allowed_hosts=["自己电脑的ip"]
#   或者 python manage.py runserver 0.0.0.0:8899  -- allowed_hosts=["*"]
```

### 三、URL配置

URL配置的本质是 URL 模式以及要为该 URL 模式调用的视图函数之间的映射表。

该配置用于告诉 Django，对于这个 URL 调用这段代码，对于那个 URL 调用那段代码。

例如，当用户访问xxx/hello时，调用视图函数hello()，这个视图函数存在于Python模块文件views.py中。

#### 1、一般配置

```python
from django.contrib import admin
from django.urls import path
from view_app.views import hello
urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/',xxx)              # http://localhost:8000/hello/
    path('hello/world/',xxx)        # http://localhost:8000/hello/world/
    path('hello/world/a/b/',xxx)    # http://localhost:8000/hello/world/a/b/
]
```



#### 2、正则配置 

```python
from django.contrib import admin
from django.urls import path, re_path   # 引入 re_path 函数
from view_app.views import hello
urlpatterns = [
	path('admin/', admin.site.urls),
	re_path("hello/\\d{3}/",hello),   # localhost:8000/hello/三个整数/
	re_path(r"hello/\d{3}/",hello),   
	re_path(r'hello/\d{2,5}/\w+/',hello), # locahost:8000/hello/12415/dfasdf
   									# locahost:8000/hello/12415/abc/def/
    re_path(r"hello/[a-d]*/$"),    # localhost:8000/abchello/abcaa/
    re_path(r"^hello/[a-d]*/$"),    # localhost:8000/hello/abcaa/
  ]
```

**注意：** 在使用正则时，`^` 和` $` 可以保证精准匹配。 ^表示以什么开头， $ 表示以什么结尾。



#### 3、URL技巧（path管理）

一个Project会有很多App，每个App会有很多View函数，如果都在一个urls.py中配置，则项目的管理和维护成本将急剧上升。

每个App不仅逻辑独立，也希望在url-pattern的配置上也相互隔离。

因此我们希望每个App都应该有一个独立的url-conf，在每个app下创建一个新的urls.py文件，然后在全局的urls.py中来包含各个app的url-conf。

- 在App目录下，新建一个urls.py，并配置自己的url路径：

```python
from django.contrib import admin
from django.urls import path,re_path
from view_app.views import hello

urlpatterns = [
    path('hello/',hello),
    re_path(r'hello/\d{2,5}/\w+/',hello),
]
```

- 在全局urls.py中，使用include包含各个app的url-conf：

```python
from django.contrib import admin
from django.urls import path,re_path,inlcude
from view_app.views import hello

urlpatterns = [
    path('admin/', admin.site.urls),
    path('view_app/',include("view_app.urls")),  # 导入外部的urls模块
    
    # view_app为 view_app.urls每一个path路径的前缀
]

# 如上的两个路径在访问时，都要加上全局路径中的view_app前缀
# http://localhost:8000/view_app/hello
# http://localhost:8000/view_app/hello/1243/abc
```

**补充**：

​	如果在一个app的众多path中，又有关系逻辑关联紧密的path，可以进一步管理：

```python
from django.contrib import admin
from django.urls import path,re_path
from view_app.views import hello

urlpatterns = [
    path(...),
    ...
    path('user/vip',include([
        path('a/',view.helloa),   # http://.../view_app/user/vip/a/
        path('b/',view.hellob),   # http://.../view_app/user/vip/b/
        path('c/',view.helloc),   # http://.../view_app/user/vip/c/
    ])),
    ...
    path(...),
]
```

**至此：View的url配置则有了管理方案：**

1. 每个App定义自己的urls.py，其中定义自己的url-conf。
2. Project的全局urls.py中引入各个App的url-conf,并定义访问前缀，区分、隔离各个App
3. App内部的url-conf可以根据逻辑进一步区分、隔离。



### 四、命名路径

#### 1、基本使用

**语法：**

```python
path("xxx/<参数名1>/<参数名2>/../", xxx)    # <参数名1> 此种路径可以匹配任何内容
```

**示例：**

```python
path("hello/<name>/<age>/",views.hello)       # localhost:8000/hello/mr_lee/18/

def hello(request，name,age):
    print(name,age)
    ...
```

**注意：**有了命名路径，view的函数中必须有对应的接收参数。



#### 2、命名路径转换器

非正则命名路径中可以使用转换器，以约束每个路径捕获到的数据的类型

- int：接收整数
- str：字符串（了解）
- path：可以接收包含"/"的路径（了解）
- slug：接收【数字、字母、下划线、中划线】四种字符（了解）

如果路径上的值不符合转换器类型，则访问不到对应的view函数（404页面）

```python
path("hello/<int: age>/<str: name>/<slug:nick>",hello)

# http://localhost:8000/hello/18/Mrlee/ab_c12-def/
def hello(request,age,name,nick):
    print(age,name,nick)
    return HttpResponse('Done!')
```

#### 3、正则命名路径

##### 1、正则路径匹配且捕获数据

使用正则的“捕获组”的格式（regex）也可捕获到url路径中的数据。

即用括号包围的正则，就是一个捕获组，不仅匹配路径，还可以捕获到数据。

```python
re_path("hello/(\d{3})/",hello1)                    # http://localhost:8000/hello/125/
re_path("hello2/(d{2,5})/(\w+)/",hello2)		   # http://localhost:8000/hello2/43/abfa2_ad
re_path("hello3/([a-d]*)/",hello3)			       # http://localhost:8000/hello3/abcdef/
re_path("^hello4/([a-d]*)/$",hello4)			   # http://localhost:8000/hello4/abc
											    # 注意hello3与hello4的区别
def hello1(request,a):
    print(a)
    return HttpResponse("Done!")

def hello2(request,a,b):
    print(a,b)
    return HttpResponse("Done!")

def hello3(request,a):
    print(a)
    return HttpResponse("Done!")

def hello4(request,a):
    print(a)
    return HttpResponse("Done!")
```



##### 2、正则命名路径且捕获数据

```
re_path("hello/(?P<age>\d{3})/",hello1)             # http://localhost:8000/hello/125/
re_path("hello2/(?P<age>d{2,5})/(?P<name>\w+)/",hello2)	 # http://localhost:8000/hello2/43/ab_f
re_path("hello3/(?P<nick>[a-d]*)/",hello3)		   # http://localhost:8000/hello3/abcdef/
re_path("^hello4/(?P<nick>[a-d]*)/$",hello4)	   # http://localhost:8000/hello4/abc
											    # 注意hello3与hello4的区别
def hello(request,age):
    print(age)
    return HttpResponse("Done!")

def hello2(request,age,name):
    print(age,name
    return HttpResponse("Done!")

def hello3(request,nick)
    print(nick)
    return HttpResponse("Done!")

def hello4(request,nick)
    print(nick)
    return HttpResponse("Done!")
```



```python
总结：
# 目的：捕获url中的参数值
# 1. 基本的命名路径    path("xxx/<name>/<age>/",xxx)   可以加转换器：限制参数的类型和取值范围
#                     def hello(request,name,age)  视图函数中的参数名要和path中的名字保持一致

# 2. 正则路径捕获参数  re_path("xxx/(\d{3})",xxx)
#                    def hello(request,a)         视图函数中的参数名字无要求


# 3. 正则命名路径且捕获参数    re_path("xxx/(?P<id>\d{3}/",xxx)
#                           def hello(request,id)   视图函数中的参数名要和path中的名字保持一致
```



### 五、请求处理   

（1）用户发送请求  （2）服务器接收请求 （3）服务器处理请求并返回响应

#### 1、发送请求的方式

在Web中发送页面请求的方式一般有三种：

- `<a href="xxx">`            超链接
- `location.href="xxx" `       <button onclick=function>
- `<form action="xxx">`  表单

```html
<html>
<head>
	<title>发送请求的三种方式</title>
	<script type="text/javascript"> 
		function myclick() {
			location.href = "http://localhost:8000/hello/123"
		}
	</script>
</head>
<body>
	<p><a href="http://e.dangdang.com/products/1900470914.html">打开</a></p>

<form action="http://e.dangdang.com/products/1900671721.html">
	<input type="submit" name="" value="提交">
</form>

<input type="button" name="" value="点我" onclick="myclick()">

</body>
</html>
```



#### 2、接收请求参数

客户端（浏览器）访问服务器时，需要向服务器传递数据（请求参数），以让服务器可以区别处理。

比如：点击多个商品详情链接，会看到不同的商品的详情页面，服务器如何知道查询哪个商品的数据？

- **方式一：接收路径中的参数**
  - 命名路径
  - 正则路径
  - 正则命名路径
- **方式二：常规的参数接收，通过request对象接收**

#### 3、超链接传参   

超链接传参格式： `url/?key=value&key2=value2`       

```python
<a href="http://localhost:8000/hello5/?id=115&name=mr_lee">超链接传参</a>  # html超链接

path('hello5/',views.hello5)                        # url配置

def hello5(request):                                # view函数
    return HttpResponse("Done")
```

附：location.href传参与超链接传参类似。



#### 4、表单传参

表单中的控件，name属性值为参数名，value属性值为参数值。

```html
<form action="http://localhost:8000/hello6/" method="get">
	用户名：<input type="text" name="name" >
	密码：<input type="password" name="pwd">
	<input type="submit" name="" value="提交">
</form>
```



#### 5、request接收请求参数

##### 5.1 接收get请求参数

- request.GET['参数名']   
- request.GET.get("参数名")

```python
path('hello6/', views.hello6)

def hello6(request):
    print(request.GET['name'],request.GET['pwd'])
    return HttpResponse("Done")
```

##### 5.2 接收post请求参数

- request.POST['参数名']   
- request.POST.get("参数名")

```html
<form action="http://localhost:8000/hello6/" method="post">
    {% csrf_token %}
	用户名：<input type="text" name="name" >
	密码：<input type="password" name="pwd">
	<input type="submit" name="" value="提交">
</form>
```

```python
path('hello6/', views.hello6)

def hello6(request):
    print(request.POST['name'],request.POST['pwd'])
    return HttpResponse("Done")
```

**注意：**

- **GET['age'] 如果没有键则报错**
- **GET.get('age') 如果没有键返回None**



#### 6、Get和Post请求区别

- POST安全性好于GET，GET中的请求参数直接暴露于url上；传递密码等敏感数据时，不使用GET。
- GET对数据长度有限制,URL 的最大长度是 2048 个字符，POST无限制。
- HTTP的语义中GET请求用来查询数据，POST用来提交数据
- GET只允许 ASCII 字符。POST可以传递各种数据，二进制也可以。
- GET可以缓存，POST不能缓存

**实用技巧：请求中有隐私数据(如密码) 或 有较多数据时不建议使用Get**

**实用场景：Get常用来做查询和删除请求，Post常用来做增加和更新请求**



#### 7、接收多值参数

- 当请求参数中同一个key上有多个值，View中如何接收？

`http://localhost:8000/test1/user/vip/a/?age=18&age=19&name=Mr_lee`  

```html
<form action="http://localhost:8000/hello7" method="GET">
    <input type="checkbox" name="hobby" value="football"/>足球
    <input type="checkbox" name="hobby" value="basketball"/>篮球
    <input type="checkbox" name="hobby" value="volleyball"/>排球
    <input type="submit" value="提交"/>
</form>
```

hobby有多个值，GET['hobby']得到的值是什么 ？

`http://localhost:8000/hello7/?hobby=football&hobby=basketball&hobby=volleyball`

```python
def hello7(request):
    print(request.GET['hobby'])   #将只会得到最后一个值
    return HttpResponse("")
```

- 使用GET.getlist("xxx")或POST.getlist("xxx")来接收多个相同key的值：

```python
def hello7(request):
    print(request.GET.getlist('hobby'))
    return HttpResponse("")
```

