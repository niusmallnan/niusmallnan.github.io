=======================================
Pythonic Singleton的实现
=======================================
所谓单例，就是保证一个类仅有一个实例。所有引用（实例、对象）拥有相同的状态（属性）和行为（方法）；

同一个类的所有实例天然拥有相同的行为（方法）；

只需要保证同一个类的所有实例具有相同的状态（属性）即可；

所有实例共享属性的最简单最直接的方法就是__dict__属性指向（引用）同一个字典（dict）


单例模式要点:

- 一是某个类只能有一个实例；
- 二是它必须自行创建这个实例；
- 三是它必须自行向整个系统提供这个实例。


从具体实现角度来说，就是以下三点：

- 一是单例模式的类只提供私有的构造函数；
- 二是类定义中含有一个该类的静态私有对象；
- 三是该类提供了一个静态的共有的函数用于创建或获取它本身的静态私有对象。



代码实现
=================
我们通常的实现方式是这样的:


.. code-block:: python

    class Singleton(object):   
        def __new__(cls, *args, **kw):   
            if not hasattr(cls, '_instance'):   
                orig = super(Singleton, cls)   
                cls._instance = orig.__new__(cls, *args, **kw)   
            return cls._instance   
    



实现在__new__方法里，这个方法只有当Python的类创建对象时候才调用， 
并在将一个类的实例绑定到类变量_instance上，
如果cls._instance为None说明该类还没有实例化过，实例化该类，
并返回，如果cls._instance不为None,直接返回cls._instance。

但是这种方法还是不够Pythonic，如果我们的系统中有很多类要单例化，那么岂不是要在各自的类中都要写__new__方法，这个肯定不符合Python的哲学。所以看下面的代码:


.. code-block:: python

    def singleton(cls, *args, **kw):  
        instances = {}  
        def _singleton():  
            if cls not in instances:  
                instances[cls] = cls(*args, **kw)  
            return instances[cls]  
        return _singleton  
 
    @singleton  
    class MyClass4(object):  
        def __init__(self):  
            pass



利用Python的装饰器，完美的解决了复用的问题，点个赞！
不要忘了Python实现单例比较简洁也是多亏了GIL，全局解释器锁，你多线程的影响。
当然Python的并发不是用多线程实现的，你可以用多进程或者协程实现，这个其它文章再做讨论。








