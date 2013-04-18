:Date: 2013-04-17 20:52:01

===============================
Stackoverflow上的一些Python问题
===============================

:Author: hit9
:注1: 以下问题来自Stackoverflow, 但不完全一致
:注2: 欢迎fork向本文添加内容

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



