:Date: 2013-04-17 20:52:01

===============================
Stackoverflow上的Python问题精选
===============================

:Author: hit9
:注1: 以下问题来自Stackoverflow, 但不完全一致
:注2: 欢迎fork向本文添加内容, 文章在Github上，地址见首页。

.. Contents::

不能直接给对象设置属性?
-----------------------

::

    >>> obj = object()
    >>> obj.name = "whatever"
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'object' object has no attribute 'name'

但是为什么这样就可以呢::

    >>> class Object(object):pass
    ...
    >>> Obj = Object()
    >>> Obj.name = "whatever"
    >>> Obj.name
    'whatever'
    >>>

答: 现在你给第二个代码块中的Object加上属性 ``__slots__`` 试试::

    >>> class Object(object):
    ...     __slots__ = {}
    ...
    >>> Obj = Object()
    >>> Obj.name = "whatever"
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Object' object has no attribute 'name'

会发现抛出了同样的异常。 ``object`` 、 ``list``  、 ``dict`` 等内置函数都如此。

拥有 ``__slots__`` 属性的类在实例化对象时不会自动分配 ``__dict__`` ，而 ``obj.attr`` 即 ``obj.__dict__['attr']``,
所以会引起 ``AttributeError``

对于拥有 ``__slots__`` 属性的类的实例 ``Obj`` 来说，只能对 ``Obj`` 设置 ``__slots__`` 中有的属性::

    >>> class Object(object):
    ...     __slots__ = {"a","b"}
    ...
    >>> Obj = Object()
    >>> Obj.a = 1
    >>> Obj.a
    1
    >>> Obj.c = 1
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Object' object has no attribute 'c'

详细见 Python-slots-doc_

.. _Python-slots-doc: http://docs.python.org/2/reference/datamodel.html#__slots__

如何打印一个对象的所有属性和值的对
----------------------------------

原问题: http://stackoverflow.com/questions/1251692/how-to-enumerate-an-objects-properties-in-python

答:

::

    for property, value in vars(theObject).iteritems():
        print property, ": ", value

这个做法其实就是 ``theObject.__dict__`` ,  也就是 ``vars(obj)`` 其实就是返回了 ``o.__dict__``

另一个做法: ``inspect.getmembers(object[, predicate])`` ::

    >>> import inspect
    >>> for attr, value in inspect.getmembers(obj):
    ...     print attr, value

两者不同的是， ``inspect.getmembers``  返回的是元组 ``(attrname, value)`` 的列表。而且是所有的属性,  包括 ``__class__`` , ``__doc__`` ,
``__dict__`` ,  ``__init__`` 等特殊命名的属性和方法。而 ``vars()`` 只返回 ``__dict__``. 对于一个空的对象来说， ``__dict__`` 会是 ``{}``
,  而 ``inspect.getmembers`` 返回的不是空的。


类的__dict__无法更新?
---------------------

::

    >>> class O(object):pass
    ...
    >>> O.__dict__["a"] = 1
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: 'dictproxy' object does not support item assignment

答: 是的, class的 ``__dict__`` 是只读的::

    >>> class O(object):pass
    ...
    >>> O.__dict__
    <dictproxy object at 0xb76d8ac4>
    >>> O.__dict__.items()
    [('__dict__', <attribute '__dict__' of 'O' objects>), ('__module__', '__main__'), ('__weakref__', <attribute '__weakref__' of 'O' objects>), ('__doc__', None)]
    >>> O.func = lambda: 0
    >>> O.__dict__.items()
    [('__dict__', <attribute '__dict__' of 'O' objects>), ('__module__', '__main__'), ('__weakref__', <attribute '__weakref__' of 'O' objects>), ('__doc__', None), ('func', <function <lambda> at 0xb76de224>)]
    >>> O.func
    <unbound method O.<lambda>>

可以看到 ``O.__dict__`` 是一个 ``dictproxy`` 对象，而不是一个 ``dict`` . (你可以 ``dir(O.__dict__)`` ，但不会发现有它有属性 ``__setitem__`` )

那我们怎么给类设置属性呢? 用 ``setattr`` ::

    >>> setattr(O, "a", 1)
    >>> O.a
    1

unbound method和bound method
-----------------------------

::

    >>> class C(object):
    ...     def foo(self):
    ...         pass
    ...
    >>> C.foo
    <unbound method C.foo>
    >>> C().foo
    <bound method C.foo of <__main__.C object at 0xb76ddcac>>
    >>>

为什么 ``C.foo`` 是一个 ``unbound method`` , ``C().foo`` 是一个 ``bound method`` ？ Python 为什么这样设计?

答：这是问题 http://stackoverflow.com/questions/114214/class-method-differences-in-python-bound-unbound-and-static

来自Armin Ronacher(Flask 作者)的回答:

如果你明白python中描述器(descriptor)是怎么实现的, 方法(method) 是很容易理解的。

上例代码中可以看到，如果你用类 ``C`` 去访问 ``foo`` 方法，会得到 ``unbound`` 方法，然而在class的内部存储中它是个 ``function``, 为什么?
原因就是 ``C`` 的类 (注意是类的类) 实现了一个 ``__getattribute__`` 来解析描述器。听起来复杂，但并非如此。上例子中的 ``C.foo`` 等价于::

    >>> C.__dict__['foo'].__get__(None, C)
    <unbound method C.foo>

这是因为方法 ``foo`` 有个 ``__get__`` 方法，也就是说, 方法是个描述器。如果你用实例来访问的话也是一模一样的::

    >>> c = C()
    >>> C.__dict__['foo'].__get__(c, C)
    <bound method C.foo of <__main__.C object at 0xb76ddd8c>>

只是那个 ``None`` 换成了这个实例。

现在我们来讨论，为什么Python要这么设计?

其实，所谓 ``bound method`` ，就是方法对象的第一个函数参数绑定为了这个类的实例(所谓 ``bind`` )。这也是那个 ``self`` 的由来。

当你不想让类把一个函数作为一个方法，可以使用装饰器 ``staticmethod`` ::

    >>> class C(object):
    ...     @staticmethod
    ...     def foo():
    ...         pass
    ...
    >>> C.foo
    <function foo at 0xb76d056c>
    >>> C.__dict__['foo'].__get__(None, C)
    <function foo at 0xb76d056c>

``staticmethod`` 装饰器会让 ``foo`` 的 ``__get__`` 返回一个函数，而不是一个方法。

那么，function，bound method 和 unbound method 的区别是什么?
---------------------------------------------------------------

一个函数(function)是由 ``def`` 语句或者 ``lambda`` 创建的。

当一个函数(function)定义在了class语句的块中（或者由 ``type`` 来创建的), 它会转成一个 ``unbound method`` , 当我们通过一个类的实例来
访问这个函数的时候，它就转成了 ``bound method`` , ``bound method`` 会自动把这个实例作为函数的地一个参数。

所以， ``bound method`` 就是绑定了一个实例的方法， 否则叫做 ``unbound method`` .它们都是方法(method), 是出现在 ``class`` 中的函数。



什么是metaclass
---------------

这是stackoverflow投票很高的问题: http://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python

回答: (最高得分的答案)

什么叫做元类
============

Metaclass是创建class的东西。

一个class是用来创建对象的是不是?

但是我们知道，Python中的类也是对象。

Metaclass就是用来创建类这些对象的，它们是类的类，你可以形象化地理解为::

    MyClass = MetaClass()
    MyObject = MyClass()

你知道， ``type`` 函数可以这样使用::

    MyClass = type('MyClass', (), {})

这是因为 ``type`` 实际上是个 ``metaclass`` , Python使用 ``type`` 这个元类来创建所有的类。

现在你是不是有疑问了，为什么 ``type`` 是小写开头的，而不是 ``Type`` 呢？既然它是个元类！

我猜，大概是因为和 ``str`` , ``int`` 来保持一致吧， ``str`` 也是一个类，用来创建字符串。

你可以检查下对象的 ``__class__`` 属性来看看它们的类是谁. Python中万物都是对象::

    >>> age = 35
    >>> age.__class__
    <type 'int'>
    >>> name = 'bob'
    >>> name.__class__
    <type 'str'>
    >>> def foo():pass
    ... 
    >>> foo.__class__
    <type 'function'>
    >>> class Bar(object): pass
    ... 
    >>> b = Bar()
    >>> b.__class__
    <class '__main__.Bar'>
    >>> 

那么， ``__class__`` 的 ``__class__`` 属性又是谁? ::

    >>> a = 1
    >>> a.__class__.__class__
    <type 'type'>
    >>> name = 'bob'
    >>> name.__class__.__class__
    <type 'type'>

所以，元类是用来创建类的。

你可以叫元类为 *类工厂*

``type`` 是Python使用的内建元类，当然，Python允许大家建立自己的元类.

__metaclass__ 属性
==================

你可以在写一个类的时候加上这个属性 ``__metaclass__`` ::

    class Foo(object):
      __metaclass__ = something...
      [...]

这样的话，Python就会用这个元类(上例中为 ``something`` ) 来创建类 ``Foo``

我们首先写的是 ``class Foo(object)`` ,但是Python跑到这里看到这一行时，并没有在内存中建立类 ``Foo``

**因为Python这么做的：查找它有没有 ``__metaclass__`` 属性，有的话，用指定的类来创建 ``Foo`` ,否则（也就是一般情形下），使用 ``type`` 来创建**

最好还是记住上面那句话 :)

当你这么写的时候::

    class Foo(Bar):
        pass

Python会这么做:

1. 有 ``__metaclass__`` 定义吗？ 如果有，在内存中建立一个类的对象。用 ``__metaclass__`` 指定的类来创建。

2. 如果没有找到这个属性，它会继续在父类 ``Bar`` 中找

3. 这样一直向父类找，父类的父类。。。直到 module 级别的才停止。

4. 如果在任何的父类中都找不到，那就用 ``type`` 创建 ``Foo``

现在一个问题，我们可以给 ``__metaclass__`` 赋值什么呢?

答案当然是，一个可以创建类的东西。

那么，什么才能创建一个类呢？

普通的元类
==========

设计元类的主要目的就是允许我们在类创建的时候动态的修改它，这经常用在API的设计上。

让我们举一个很纯的例子，比如你想要让一个模块中的所有类都共享一些属性，有很多办法可以做到，其中一个就是
在模块中定义一个 ``__metaclass__`` 属性。

这样，模块中所有的类都会被 ``__metaclass__`` 创建。

幸运的是 , ``__metaclass__`` 可以是任何可以被调用的对象。不非要是个class，还可以是个函数。

所以，我们这么做,用一个函数来作为metaclass::

    # the metaclass will automatically get passed the same argument
    # that you usually pass to `type`
    def upper_attr(future_class_name, future_class_parents, future_class_attr):
      """
        Return a class object, with the list of its attribute turned 
        into uppercase.
      """

      # pick up any attribute that doesn't start with '__'
      attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
      # turn them into uppercase
      uppercase_attr = dict((name.upper(), value) for name, value in attrs)
    
      # let `type` do the class creation
      return type(future_class_name, future_class_parents, uppercase_attr)
    
    __metaclass__ = upper_attr # this will affect all classes in the module
    
    class Foo(): # global __metaclass__ won't work with "object" though
      # but we can define __metaclass__ here instead to affect only this class
      # and this will work with "object" children
      bar = 'bip'
    
    print hasattr(Foo, 'bar')
    # Out: False
    print hasattr(Foo, 'BAR')
    # Out: True

    f = Foo()
    print f.BAR
    # Out: 'bip'


现在我们用一个类来作为一个metaclass::

    # remember that `type` is actually a class like `str` and `int`
    # so you can inherit from it
    class UpperAttrMetaclass(type): 
        # __new__ is the method called before __init__
        # it's the method that creates the object and returns it
        # while __init__ just initializes the object passed as parameter
        # you rarely use __new__, except when you want to control how the object
        # is created.
        # here the created object is the class, and we want to customize it
        # so we override __new__
        # you can do some stuff in __init__ too if you wish
        # some advanced use involves overriding __call__ as well, but we won't
        # see this
        def __new__(upperattr_metaclass, future_class_name, 
                    future_class_parents, future_class_attr):

            attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
            uppercase_attr = dict((name.upper(), value) for name, value in attrs)

            return type(future_class_name, future_class_parents, uppercase_attr)


但这样并不是很 OOP ， 我们可以直接调用 ``type`` 函数，并且不覆盖父亲的 ``__new__`` 方法::

    class UpperAttrMetaclass(type): 
    
        def __new__(upperattr_metaclass, future_class_name, 
                    future_class_parents, future_class_attr):

            attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
            uppercase_attr = dict((name.upper(), value) for name, value in attrs)

            # reuse the type.__new__ method
            # this is basic OOP, nothing magic in there
            return type.__new__(upperattr_metaclass, future_class_name, 
                                future_class_parents, uppercase_attr)

你可能注意到了参数 ``upperattr_metaclass`` ,没什么特殊的，一个方法总是拿那个实例来作为第一个参数。就像寻常的 ``self`` 参数。

当然，可以这么写，我上面的例子命名不那么好:) ::

    class UpperAttrMetaclass(type): 

        def __new__(cls, name, bases, dct):
    
            attrs = ((name, value) for name, value in dct.items() if not name.startswith('__'))
            uppercase_attr = dict((name.upper(), value) for name, value in attrs)

            return type.__new__(cls, name, bases, uppercase_attr)


我们可以使用 ``super``  函数来让这个例子变得更简洁::

    class UpperAttrMetaclass(type): 

        def __new__(cls, name, bases, dct):

            attrs = ((name, value) for name, value in dct.items() if not name.startswith('__'))
            uppercase_attr = dict((name.upper(), value) for name, value in attrs)

            return super(UpperAttrMetaclass, cls).__new__(cls, name, bases, uppercase_attr)

元类是个简单的魔术，只要:

- 注入类的创建

- 修改类

- 返回修改后的类

那么你为什么用类来作为metaclass而不是函数
=========================================

既然 ``__metaclass__`` 可以是任何可以被调用的对象，那么你为什么用类作为metaclass而不是函数呢？

几个原因:

- 更能清楚的表达意图

- 可以使用OOP, metaclass可以继承，重写父类，甚至使用metaclass，可以使用面向对象的特性。

- 更好的组织代码.

- .. 等等,译者不再多写了~ :)

应用场景
========

一个典型例子，Django ORM (译者注,peewee也用metaclass)::

    class Person(models.Model):
        name = models.CharField(max_length=30)
        age = models.IntegerField()

但是你这么做::

    guy = Person(name='bob', age='35')
    print guy.age

并不返回一个 ``IntegerField`` 对象，而是一个 ``int`` 

结束语
======

Python的世界里，万物都是对象

但是 ``type`` 是它自己的元类。

99%的情形下你不需要用这个东西。


为什么Python没有unzip函数
-------------------------

众所周知, ``zip`` 函数可以把多个序列打包到元组中::

    >>> a, b = [1, 2, 3], [4, 5, 6]
    >>> c = zip(a, b)
    >>> c
    [(1, 4), (2, 5), (3, 6)]


那么为什么没有这样的 ``unzip`` 函数来把 ``[(1, 4), (2, 5), (3, 6)]`` 还原呢?

答: Python中有个很神奇的操作符 ``*`` 来 ``unpack`` 参数列表::

    >>> zip(*c)
    [(1, 2, 3), (4, 5, 6)]

Python中的新式类和老式类
------------------------

无论你怎么叫吧， 英文来说是 ``new style class`` 和 ``old style class`` 

问题链接: http://stackoverflow.com/questions/54867/old-style-and-new-style-classes-in-python

新式类是继承自 ``object`` 或其他新式类的类::

    class NewStyleClass(object):
        pass
    
    class AnotherNewStyleClass(NewStyleClass):
        pass

否则是老式类::

    class OldStyleClass():
        pass

**为什么引入新式类?**

> The major motivation for introducing new-style classes is to provide a unified object model with a full meta-model. 

Python为了提供一个更完整的元模型。(好吧，译者也不大明白，不过我知道Python中很神奇的描述器只能在新式类里用)


译者注
------

本文Github地址: https://github.com/HIT-ON-Github/PyZh/blob/master/docs/python-questions-on-stackoverflow.rst

欢迎Fork追加问题。
