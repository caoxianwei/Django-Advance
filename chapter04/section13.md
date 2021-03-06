## 管理器类(Manager)

Django中的`Manager`类是我们通过模型类去操作数据库的工具，Django给每个定义的模型默认添加一个名为`objects`的`Manager`类的对象。

Djang官方文档对`Manager`的概述如下：
> A Manager is the interface through which database query operations are provided to Django models. At least one Manager exists for every model in a Django application.


先看一个对管理器的基本使用，定义一个基本的图书信息类：
```python
class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateField()
    bread = models.IntegerField(default=0)
    bcomment = models.IntegerField(default=0)
    bisDelete = models.BooleanField(default=False)
    def __str__(self):
        return self.btitle
```

接下来我们要查询BookInfo中的所有图书
```python
BookInfo.objects.all()
```

执行完上述查询语句之后，假设我们现在又接收到一个客户端的要求，要求删除一本书，执行下面的代码对数据库执行逻辑删除操作：
```python
book = BookInfo.object.get(btitle='笑傲江湖')
book.bisDelete = 1
book.save()
```

以后我们再执行查询所有图书的信息时，就需要去掉被删除的图书。一种做法就是每次查询所有的图书信息然后去掉`bisDelete`字段为`1`的记录
```python
BookInfo.objects.all().exclude(bisDelete=1)
```

但是我们第次操作都需要写一次上面的代码未免太麻烦了，并且我们要查询的所有的图书信息时本来就默认去掉被删除的图书的信息，而`objects`这个管理器类中的`all`方法是把所有的信息都返回，显然不能满足我们的需求，这个时候我们就需要去自定义管理器类去重新实现`all`方法。


### 自定义管理器类

```python
class BookManager(models.Manager):
        #重写父类的all方法
        def all(self):
            all_data = super().all()
            return all_data.exclude(bisDelete=1)

class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateField()
    bread = models.IntegerField(default=0)
    bcomment = models.IntegerField(default=0)
    bisDelete = models.BooleanField(default=False)
    def __str__(self):
        return self.btitle
    #实例化自定义管理器类(此时默认的objects管理类对象已经失效)
    bmanager = BookManager()

#使用自定义管理类

>>> BookInfo.bmanager.all()

```

### 扩展管理器类功能

前面我们通过自定义管理器类扩展了原有的`all()`方法，但是有的时候我们想要的功能在管理器类中不仅不具备相应的方法，甚至连类似的可继承扩展的方法都没有，这个时候我们就需要完全自定义一个方法了。比如下面的一个场景，我们不希望每次拿到的返回结果都是`QuerySet`类型的集合对象，我们希望每次查询返回的是一个包含满足条件的图书名的`列表`，这个时候我们可以在自定义管理器类中定义一个方法完全适配我们要求的方法(其实这种自定义扩展方法本质就是把对`Manager`管理类查询结果的过滤操作从类外面搬到了管理类内部)
```python
class BookManager(models.Manager):
        # 查询大于给定id号的图书信息
        def blist(self,id):
            ls = []
            all_data = super().filter(id__gt=id)
            for book in all_data:
                ls.append(book.btitle)
            return ls
    #进行测试
    >>> BookInfo.bmanager.blist(2)
```