================
Python描述器引导
================

:作者: Raymond Hettinger

:联系: <python at rcn dot com>

:译者注:  原文链接- http://docs.python.org/2/howto/descriptor.html

.. Contents::

摘要
----

定义描述器, 总结协定，并展示描述器怎么被调用的。测试一个普通的描述器和包括函数，属性(property), 静态方法(static method), 类方法在内的几个Python内置描述器。通过给出一个纯Python的等价和应用来展示每个是怎么工作的。

学习描述器不仅让你接触到更多的工具，描述器让我们更深入地了解Python是如何工作的，体现了Python设计的优雅之处。

定义和介绍
----------

一般来说，一个描述器是一个有“监听行为”的属性，它的控制被描述器的方法重写。这些方法是 :meth:`__get__`, :meth:`__set__`, 和 :meth:`__delete__`.有这些方法的对象叫做描述器。

默认对属性的控制是从对象的字典里面(__dict__)中获取(get), 设置(set)和删除(delete)它.对于实例 ``a`` 来说， ``a.x``  的查找顺序是, ``a.__dict__['x']`` , 然后 ``type(a).__dict__['x']`` , 然后找 ``type(a)`` 的父类(不包括元类).如果查找到的值是一个描述器, Python就会用装饰器的方法来重写默认的控制行为。这个重写发生在这个查找环节的哪里取决于哪个定义了哪个描述器方法。注意, 描述器只有当它是个新式类的实例时才被调用(新式类是继承自 ``type`` 或者 ``object`` 的类) (注：亦即描述器的类应为新式类)

描述器是强大的，应用广泛的。描述器正是属性, 实例方法, 静态方法, 类方法,和 ``super`` 的实现原理.描述器在Python被用于实现Python 2.2中引入的新式类。

描述器协定
----------

``descr.__get__(self, obj, type=None) --> value``

``descr.__set__(self, obj, value) --> None``

``descr.__delete__(self, obj) --> None``

这是所有描述器方法。一个对象被定义了这些任一个方法就会成为描述器，可以重写默认的查找属性的行为。

如果一个对象同时定义了 :meth:`__get__` 和 :meth:`__set__`,它叫做资料装饰器(data descriptor)。仅定义了 :meth:`__get__` 的描述器叫非资料描述器(特别用于方法，当然其他用途也是可能的)

资料描述器和非资料描述器的区别在于：相对于实例的字典的优先级。如果实例字典中有与描述器同名的属性，如果描述器是资料描述器，优先使用资料描述器，如果是非资料描述器，优先使用字典中的属性。(译者注：这就是为何实例 ``a`` 的函数和属性重名时，比如都叫 ``foo`` Python会在访问 ``a.foo`` 的时候优先访问实例字典中的属性，因为实例函数的实现是个非资料描述器)

要想制作一个只读的资料描述器，需要同时定义 ``__set__`` 和 ``__get__``,并在 ``__set__`` 中引发一个 ``AttributeError`` 异常。

描述器的调用
------------

描述器可以直接这么调用：    ``d.__get__(obj)``

然而更多的时候描述器用来拦截对实例属性的访问。举例来说， ``obj.d`` 会在 ``obj`` 的字典中找 ``d`` ,如果 ``d`` 定义了 ``__get__`` 方法，那么 ``d.__get__(obj)`` 会被调用。

调用的细节取决于 ``obj`` 是一个类还是一个实例。另外，描述器只在它的类是个新式类的时候起作用。继承于 ``object`` 的类叫做新式类。

对于对象来讲，方法 :meth:`type.__getattribute__` 把 ``B.x`` 变成 ``B.__dict__['x'].__get__(None, B)`` 。用Python来描述就是::

    def __getattribute__(self, key):
        "Emulate type_getattro() in Objects/typeobject.c"
        v = object.__getattribute__(self, key)
        if hasattr(v, '__get__'):
           return v.__get__(None, self)
        return v

其中重要的几点：

* 描述器的调用是因为 :meth:`__getattribute__`
* 重写 :meth:`__getattribute__` 方法会阻止正常的描述器调用
* :meth:`__getattribute__` 只对新式类和实例可用
* :meth:`object.__getattribute__` 和 :meth:`type.__getattribute__` 对 :meth:`__get__` 的调用不一样
* 资料描述器总是比实例字典优先。
* 非资料描述器可能被实例字典重写。(非资料描述器不如实例字典优先)

``super()`` 返回的对象同样有用来调用描述器的  :meth:`__getattribute__` 方法。调用 ``super(B, obj).m()`` 时会去在 ``obj.__class__.__mro__`` 中查找B的父类,然后返回 ``A.__dict__['m'].__get__(obj, A)`` 。如果没有描述器， 原样返回 ``m`` 。如果连实例字典中都找不到 ``m`` ，继续调用 :meth:`object.__getattribute__`.

注意:在Python 2.2中，如果 ``m`` 是一个描述器, ``super(B, obj).m()`` 只会调用方法 :meth:`__get__` 。在Python 2.3中，非资料描述器(除非是个旧式类)也会被调用。 :c:func:`super_getattro()` 的实现细节在： 
`Objects/typeobject.c <http://svn.python.org/view/python/trunk/Objects/typeobject.c?view=markup>`_
，一个等价的Python实现在 `Guido's Tutorial`_.

.. _`Guido's Tutorial`: http://www.python.org/2.2.3/descrintro.html#cooperation


以上展示了描述器的机理是在  :class:`object`, :class:`type`, 和 :func:`super` 的 :meth:`__getattribute__()` 方法中实现的。由:class:`object` 派生出的类自动的继承这个机理，或者它们有个有类似机理的元类。同样，可以重写类的:meth:`__getattribute__()` 方法来关闭这个类的描述器行为。

描述器例子
----------

下面的代码中的类中定义了资料描述器，每次 ``get`` 和 ``set`` 都会打印一条消息。重写 :meth:`__getattribute__` 是另一个可以使所有属性拥有这个行为的方法。但是，描述器对于只是几个属性的时候是很有用的。

::

    class RevealAccess(object):
        """A data descriptor that sets and returns values
           normally and prints a message logging their access.
        """

        def __init__(self, initval=None, name='var'):
            self.val = initval
            self.name = name

        def __get__(self, obj, objtype):
            print 'Retrieving', self.name
            return self.val

        def __set__(self, obj, val):
            print 'Updating' , self.name
            self.val = val

    >>> class MyClass(object):
        x = RevealAccess(10, 'var "x"')
        y = 5

    >>> m = MyClass()
    >>> m.x
    Retrieving var "x"
    10
    >>> m.x = 20
    Updating var "x"
    >>> m.x
    Retrieving var "x"
    20
    >>> m.y
    5
