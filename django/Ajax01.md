# Ajax   

### 一、Ajax简介

#### 1、概念

- Ajax 即`Asynchronous Javascript and XML`（异步 JavaScript 和 XML） 。运用在前端页面的技术，用于向服务器发送**异步请求**。

  **传统请求：**用户前端页面发送请求 -> 服务器会接收请求并处理请求 -> 返回响应结果（HttpResponse('注册成功')/render() / redirect）-> 显现在前端页面（浏览器中） -- 之前整个页面的内容会被替换掉

- AJAX 请求最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并**更新部分网页内容**。

![regist_check](Ajax_pic\regist_check.gif)

![1](Ajax_pic\queryuser.gif)

#### 2、传统请求

**传统请求过程：**（用户注册）

​	`用户填写表单 --> 提交表单（向服务器发送请求）--> 服务器接收请求并处理请求 --> `

​	`服务器返回响应结果（return HttpResponse('注册失败') / return render(request,'xx.html') ）`

**特点：**请求的响应会刷新当前整张页面。浏览器会用一张新页面承载响应内容，替换掉当前整个页面内容。

​	   `请求 -> 响应 -> 重新加载页面`



#### 3、传统请求与Ajax请求的区别

- 传统请求，在提交一个请求后，服务器返回数据，会更新整个页面。
- Ajax 是一种在无需重新加载整个网页的情况下，可以与服务器交换数据并更新部分网页内容 。 

​    总结：传统请求方式得到是一个页面，而Ajax获取的服务器的数据，而不是整个页面。ajax不会刷新整个页面，只是更新页面的部分内容，效率要比传统请求效率更高。



**Google Suggest**

在 2005 年，Google 通过其 Google Suggest 使 AJAX 变得流行起来。

Google Suggest 使用 AJAX 创造出动态性极强的 web 界面：当您在谷歌的搜索框输入关键字时，JavaScript 会把这些字符发送到服务器，然后服务器会返回一个搜索建议的列表。



### 二、Ajax的使用

#### 1、前言

Ajax是客户端技术，运行在浏览器，向服务器发送异步请求，并更新部分页面内容。

**同步：**指发送一个请求，需要等待服务器返回结果，然后才能继续后面的任务，有个等待过程。  

**异步：**指发送一个请求，不需要等待返回，随时可以做别的操作，不需要等待。



**Ajax技术组成：**

- Javascript 语言环境
- css/dom   刷新页面
- XMLHttpRequest  核心对象，发送异步请求



#### 2、Ajax的使用过程

##### 2.1 创建对象

所有现代浏览器（IE7+、Firefox、Chrome、Safari 以及 Opera）均内建 XMLHttpRequest 对象。 

- 创建 XMLHttpRequest 对象的语法： 

```js
var xhr =new XMLHttpRequest();   
```

- 老版本的 Internet Explorer （IE5 和 IE6）使用 ActiveX 对象： 

```js
var xhr =new ActiveXObject("Microsoft.XMLHTTP"); 
```

- 兼容所有浏览器：

```js
var xhr;
if (window.ActiveXObject )
{
    xhr = new ActiveXObject("Microsoft.XMLHTTP")
}else if(window.XMLHttpRequest)
{
    xhr = new XMLHttpRequest()
}
```



##### 2.2 发送get请求

发送请求到服务器，需要使用open()和send()方法：

```python
xhr.open('get','xxx/?name=xxx&password=123456',true)   # 在url中拼接参数
xhr.send()      #向服务器发送请求  - view function
```

| 方法                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| open(method,url,async) | 规定请求的类型、URL 以及是否异步处理请求。<br />method：请求的类型；GET 或 POST                 <br />url：请求的服务器地址   <br />async：true（异步）或 false（同步） |
| send(string)           | 将请求发送到服务器，string仅用于post请求中。                 |



##### 2.3 服务器接收请求

```python
def checkusername(request):
    name = request.POST.get("userName")
    print("name=",name)
    user = User.objects.filter(name=name)
    if user:
        return HttpResponse("用户名已存在")   # 只需要返回部分内容，不需要返回整个html文件
    return HttpResponse("用户名合法")
```



##### 2.4 客户端接收响应

发送异步请求后，当readyState改变时，会触发onreadystatechange事件。（回调函数）

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| onreadystatechange | 存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。 |
| readyState         | 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。                                               0: 请求未初始化                                                                                                               1: 服务器连接已建立                                                                                                        2: 请求已接收                                                                                                                   3: 请求处理中                                                                                                                   4: 请求已完成，且响应已就绪 |
| status             | 200: "OK"                                                  404: 未找到页面 |

```js
xhr.onreadystatechange=function()
{
    if (xhr.readyState==4 && xhr.status==200)
    {
        var text = xhr.responseText;   //获得字符串形式的响应数据
        console.log(text)
    }
}
```



#### 3、Ajax工作原理

![AJAX](Ajax_pic\Ajax工作原理.jpg) 



#### 4、POST请求

与 POST 相比，GET 更简单也更快，并且在大部分情况下都能用。 下列情况请使用POST请求：

- 向服务器发送大量数据（POST 没有数据量限制）
- 发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠

```python
xhr.open("POST",url,true);      
xhr.setRequestHeader("Content-type","application/x-www-form-urlencoded"); xhr.send("name=Henry&password=123456");   # 在send方法中传参 
```

**CSRF跨站请求伪造：** 

GET 请求不需要 CSRF 认证，POST 请求需要正确认证才能得到正确的返回结果。

- 一般在POST表单中加入 {% csrf_token %} 即可。 

- 在Ajax中设置CSRF：
  - 方式一：`xhr.setRequestHeader("X-CSRFToken", '{{ csrf_token }}');`   
  - 方式二：`xhr.send(content+"&csrfmiddlewaretoken="+"{{ csrf_token }}")`

### 三、请求总结

- 传统请求与Ajax请求区别：浏览器对响应结果处理方式不同。
  - 传统请求：使用一个新的页面来显现响应结果。
  - ajax请求：获取响应结果，更新部分页面内容，不会重新加载整个页面。
- 同步与异步的区别：请求的机制不同，异步请求不会阻塞当前页面，而同步请求会阻塞当前页面。




**作业**：

- 注册（用户名不重复验证，显示loading图片，正确或错误也显示相应图片 + 密码两次输入一致验证）
- 验证码

