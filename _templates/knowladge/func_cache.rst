=======================================
Python实现函数级cache
=======================================
cache是我们在优化程序运行性能的一个很直接的手段，我们会使用Redis/MongoDB这种NoSql的K-V cache，也会使用mencache这种专业的cache系统，有时候还会自己在代码中利用数据结构做一些内存cache，本文给大家秀一个比较另类的cache，就是函数级的cache。看看如何用Python实现这个func-cache？


Python装饰器
=================
Python是一种很美的编程语言，而其中的Decorator(一般好像都翻译作"装饰器")则是其优雅语法的一个代表，如果你用Python还没接触过装饰器，那么很遗憾Python还没入门。

如果想了解装饰器，请Google之，这不是本文重点。

推荐阅读:

- http://www.cnblogs.com/rhcad/archive/2011/12/21/2295507.html

- http://www.cnblogs.com/huxi/archive/2011/03/01/1967600.html

- http://youngsterxyf.github.io/2013/01/04/Decorators-and-Functional-Python/


如何实现
=================
话不多说直接上代码:


.. code-block:: python

    import functools

    class memoized(object):
        
        def __init__(self, func):
            self.func = func
            self.cache = {}
        
        def __call__(self, *args):
            print 'args: ', args
            print 'cache: ', self.cache
            try:
                return self.cache[args]
            except KeyError:
                value = self.func(*args)
                self.cache[args] = value
                return value
            except TypeError:
                return self.func(*args)
 
        def __repr__(self):
            return self.func.__doc__ or ''
 
        def __get__(self, obj, objtype):
            '''Support instance methods. Important!!!''' 
            print "obj : ", obj
            print "objtype : " , objtype 
            return functools.partial(self.__call__, obj)
 
        def __str__(self):
            return str(self.func)

我们使用下面的测试代码:


.. code-block:: python

    from memoized import memoized

    @memoized
    def fn_a(a, b):
        print 'no cache, so compute it'
        return a+b

    class A(object):

        @memoized
        def add(self, a, b):
            print 'no cache, so compute it'
            return a+b


    if __name__ == '__main__':
        print fn_a(1,2)
        print '##'*20
        print fn_a(2,4)
        print '##'*20
        print fn_a(1,2)
        
        print "\n\n\n"

        a = A()
        print '##'*20
        a.add(1,2)
        print '##'*20
        a.add(2,4)
        print '##'*20
        a.add(1,2)


输出结果如下::

    args:  (1, 2)
    cache:  {}
    no cache, so compute it
    3
    ########################################
    args:  (2, 4)
    cache:  {(1, 2): 3}
    no cache, so compute it
    6
    ########################################
    args:  (1, 2)
    cache:  {(1, 2): 3, (2, 4): 6}
    3

    ########################################
    obj :  <__main__.A object at 0x10bea79d0>
    objtype :  <class '__main__.A'>
    args:  (<__main__.A object at 0x10bea79d0>, 1, 2)
    cache:  {}
    no cache, so compute it
    ########################################
    obj :  <__main__.A object at 0x10bea79d0>
    objtype :  <class '__main__.A'>
    args:  (<__main__.A object at 0x10bea79d0>, 2, 4)
    cache:  {(<__main__.A object at 0x10bea79d0>, 1, 2): 3}
    no cache, so compute it
    ########################################
    obj :  <__main__.A object at 0x10bea79d0>
    objtype :  <class '__main__.A'>
    args:  (<__main__.A object at 0x10bea79d0>, 1, 2)
    cache:  {(<__main__.A object at 0x10bea79d0>, 2, 4): 6, (<__main__.A object at 0x10bea79d0>, 1, 2): 3}



作为一个程序员，以上能说明一切了。










