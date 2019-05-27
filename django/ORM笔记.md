#### 1、ORM中用到三个类

（1）Model类         自定义的Model类都应该继承自己Model类  --class User(models.Model)  ....

（2）Manager类    User.objects 是一个Manager类型  有很多方法  create()  get() all()   管理者  

（3）QuerySet类   User.objects.all() 返回的是一个QuerySet对象     核心对象  真正做事情的



当使用User.objects.all()时 实际上调用的是QuerySet中的all()方法