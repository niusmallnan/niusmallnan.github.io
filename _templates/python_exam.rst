.. niusmallnan documentation master file, created by
   sphinx-quickstart on Tue Feb 18 13:49:43 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


=======================================
python面试题总结
=======================================



文化
======================
``说说你对zen of python的理解，你有什么办法看到它``::
    
    import this

``你在github上都fork过哪些python库，列举一下你经常使用的，每个库用一句话描述下其功能``


``说说你对pythonic的看法，尝试解决下面的小问题``::

    交换两个变量值
    a,b = b,a

    去掉list中的重复元素
    old_list = [1,1,1,3,4]
    new_list = list(set(old_list))

    翻转一个字符串
    s = 'abcde'
    ss = s[::-1]

    用两个元素之间有对应关系的list构造一个dict
    names = ['jianpx', 'yue']
    ages = [23, 40]
    m = dict(zip(names,ages))

    将数量较多的字符串相连，如何效率较高，为什么
    fruits = ['apple', 'banana']
    result = ''.join(fruits)



python字符串效率问题之一就是在连接字符串的时候使用‘+’号，例如 
s = 's1' + 's2' + 's3' + ...+'sN'，总共将N个字符串连接起来，
但是使用+号的话，python需要申请N-1次内存空间，
然后进行字符串拷贝。原因是字符串对象PyStringObject在python当中是不可变
对象，所以每当需要合并两个字符串的时候，就要重新申请一个新的内存空间
（大小为两个字符串长度之和）来给这个合并之后的新字符串，然后进行拷贝。
所以用+号效率非常低。建议在连接字符串的时候使用字符串本身的方法
join（list），这个方法能提高效率，原因是它只是申请了一次内存空间，
因为它可以遍历list中的元素计算出总共需要申请的内存空间的大小，一次申请完。

``你调试python代码的方法有哪些``

基础
======================
``什么是GIL``


``什么是元类(meta_class)``


``对比一下dict中 items 与 iteritems``












高级
======================
``是否遇到过python的模块间循环引用的问题，如何避免它``


``有用过with statement吗？它的好处是什么？``


``说说decorator的用法和它的应用场景，如果可以的话，写一个decorator``


``inspect模块有什么用``



``写一个类，并让它尽可能多的支持操作符``



``说一说你见过比较cool的python实现``



