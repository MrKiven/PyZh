================
Python描述器引导
================

:作者: Raymond Hettinger

:联系: <python at rcn dot com>

:翻译: hit9

:译者注:  原文链接- http://docs.python.org/2/howto/descriptor.html

.. Contents::

摘要
----

定义描述器, 总结协定，并展示描述器怎么被调用的。测试一个普通的描述器和包括方法，属性(property), 静态方法(static method), 类方法在内的几个Python内置描述器。通过给出一个纯Python的等价和应用来展示每个是怎么工作的。

学习描述器不仅让你接触到更多的工具，描述器让我们更深入地了解Python是如何工作的，体现了Python设计的优雅之处。

定义和介绍
----------

一般来说，一个描述器是一个有“监听行为”的属性，它的控制被描述器的方法重写。这些方法是 :meth:`__get__`, :meth:`__set__`, 和 :meth:`__delete__`.有这些方法的对象叫做描述器。

默认对属性的控制是从对象的字典里面(__dict__)中获取(get), 设置(set)和删除(delete)它.对于实例 ``a`` 来说， ``a.x``  的查找顺序是, ``a.__dict__['x']`` , 然后 ``type(a).__dict__['x']`` , 然后找 ``type(a)`` 的父类(不包括元类).如果查找到的值是一个描述器, Python就会用描述器的方法来重写默认的控制行为。这个重写发生在这个查找环节的哪里取决于哪个定义了哪个描述器方法。注意, 描述器只有当它是个新式类的实例时才被调用(新式类是继承自 ``type`` 或者 ``object`` 的类) (注：亦即描述器的类应为新式类)

描述器是强大的，应用广泛的。描述器正是属性, 实例方法, 静态方法, 类方法,和 ``super`` 的实现原理.描述器在Python被用于实现Python 2.2中引入的新式类。

描述器协定
----------

``descr.__get__(self, obj, type=None) --> value``

``descr.__set__(self, obj, value) --> None``

``descr.__delete__(self, obj) --> None``

这是所有描述器方法。一个对象被定义了这些任一个方法就会成为描述器，可以重写默认的查找属性的行为。

如果一个对象同时定义了 :meth:`__get__` 和 :meth:`__set__`,它叫做资料描述器(data descriptor)。仅定义了 :meth:`__get__` 的描述器叫非资料描述器(特别用于方法，当然其他用途也是可能的)

资料描述器和非资料描述器的区别在于：相对于实例的字典的优先级。如果实例字典中有与描述器同名的属性，如果描述器是资料描述器，优先使用资料描述器，如果是非资料描述器，优先使用字典中的属性。(译者注：这就是为何实例 ``a`` 的方法和属性重名时，比如都叫 ``foo`` Python会在访问 ``a.foo`` 的时候优先访问实例字典中的属性，因为实例函数的实现是个非资料描述器)

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

这个协定非常简单，并且提供了令人激动的可能。一些用例用途太多了以致于它们被打包成独立的方法。像属性(property), 方法(bound和unbound method), 静态方法和类方法都是基于描述器协定的

属性(properties)
----------------

调用 :func:`property` 是建立访问一个属性的描述器的简洁的方式。这个函数的原型::

    property(fget=None, fset=None, fdel=None, doc=None) -> property attribute

下面展示了一个典型的定义一个良好管理的属性 ``x`` 的情形::

    class C(object):
        def getx(self): return self.__x
        def setx(self, value): self.__x = value
        def delx(self): del self.__x
        x = property(getx, setx, delx, "I'm the 'x' property.")

想要看看 :func:`property` 是怎么用描述器实现的？ 这里有一个纯Python的等价实现::

    class Property(object):
        "Emulate PyProperty_Type() in Objects/descrobject.c"

        def __init__(self, fget=None, fset=None, fdel=None, doc=None):
            self.fget = fget
            self.fset = fset
            self.fdel = fdel
            self.__doc__ = doc

        def __get__(self, obj, objtype=None):
            if obj is None:
                return self
            if self.fget is None:
                raise AttributeError, "unreadable attribute"
            return self.fget(obj)

        def __set__(self, obj, value):
            if self.fset is None:
                raise AttributeError, "can't set attribute"
            self.fset(obj, value)

        def __delete__(self, obj):
            if self.fdel is None:
                raise AttributeError, "can't delete attribute"
            self.fdel(obj)

内建函数 :func:`property` 提供了属性访问的接口，之后的改变需要我们去介入一个函数。

对于一个实例，一个电子表格类可能提供了访问单元格的值的方式: ``Cell('b10').value``. 对这个程序随后的改善需要重新计算每个访问的控制。然而，程序员并不想影响已经写的那些直接访问这个属性的代码。那么来包装这个访问控制的方法就是用property资料描述器::

    class Cell(object):
        . . .
        def getvalue(self, obj):
            "Recalculate cell before returning value"
            self.recalc()
            return obj._value
        value = property(getvalue)

函数和方法
----------

Python的面向对象特征建立于函数环境, 非资料描述器把两者无缝地连接起来。

类的字典把方法当做函数存储。在定义类的时候，方法通常用关键字 :keyword:`def` 和 :keyword:`lambda` 来声明。唯一和一般的函数不同之处是第一个参数为对象实例保留。Python约定，这个参数通常是*self*, 但也可能叫 *this* ，或者其它什么变量名字吧。

为了支持方法调用，函数包含一个 :meth:`__get__` 方法来控制属性访问。这就是说所有的方法都是非资料描述器，它们返回有界还是无界的方法取决于他们是被类调用的还是被实例调用的。用Python来说就是::

    class Function(object):
        . . .
        def __get__(self, obj, objtype=None):
            "Simulate func_descr_get() in Objects/funcobject.c"
            return types.MethodType(self, obj, objtype)

在Python解释器里面看看描述器是怎么回事:

::
    >>> class D(object):
         def f(self, x):
              return x

    >>> d = D()
    >>> D.__dict__['f'] # 存储成一个function
    <function f at 0x00C45070>
    >>> D.f             # 从类来方法，返回unbound method
    <unbound method D.f>
    >>> d.f             # 从实例来访问，返回bound method
    <bound method D.f of <__main__.D object at 0x00B18C90>>

从输出来看，bound method 和unbound method是两个不同的类型.然而它们是这么实现的：在文件  
Objects/classobject.c(http://svn.python.org/view/python/trunk/Objects/classobject.c?view=markup)  
中用C实现的 :c:type:`PyMethod_Type`  是一个对象，但是根据 :attr:`im_self` 是否是 *NULL* (在C中等价于 *None* ) 分成两个不同的陈述。

同样，调用方法的结果依赖于 :attr:`im_self` 是否设置。如果设置了(意味着bound), 原来的函数(保存在 :attr:`im_func` 中)被调用，并且第一个参数设置成实例。如果unbound, 所有参数不变地传给那个函数。真实函数 :func:`instancemethod_call()` 的C的实现比这个稍微复杂些而已(有一些类型检查)。

静态方法和类方法
----------------

非资料描述器提供了一个简单的把函数绑定成一个实例的方法的通常模式。

简而言之，函数有个方法 :meth:`__get__` 的时候就会变成一个实例方法。非资料描述器把 ``obj.f(*args)`` 的调用 变成 ``f(obj, *args)``. 调用 ``klass.f(*args)`` 就相当于调用 ``f(*args)``.

下面的表格总结了这个绑定和它的两个最有用的变种:

      +-----------------+----------------------+------------------+
      | Transformation  | Called from an       | Called from a    |
      |                 | Object               | Class            |
      +=================+======================+==================+
      | function        | f(obj, \*args)       | f(\*args)        |
      +-----------------+----------------------+------------------+
      | staticmethod    | f(\*args)            | f(\*args)        |
      +-----------------+----------------------+------------------+
      | classmethod     | f(type(obj), \*args) | f(klass, \*args) |
      +-----------------+----------------------+------------------+

静态方法原样返回那个函数，调用``c.f``或者``C.f``都是等价的，都是在调用``object.__getattribute__(c, "f")`` 或者 ``object.__getattribute__(C, "f")`` 。就是说，这个函数可以同时用类和实例去访问。

那些不需要 ``self`` 变量做参数的函数适合用做静态方法。


