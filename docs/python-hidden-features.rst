:Date: 2013-04-20 16:14:10

===================================
(译)Python的隐藏特性(StackOverflow)
===================================

:译: hit9
:原文: http://stackoverflow.com/questions/101268/hidden-features-of-python
:注: 这里忽略了生成器，装饰器，交换变量等熟知技巧

.. Contents::

函数参数unpack
--------------

老生常谈的了::


    def foo(x, y):
        print x, y

    alist = [1, 2]
    adict = {'x': 1, 'y': 2}

    foo(*alist)  # 1, 2
    foo(**adict)  # 1, 2

链式比较操作符
---------------

::

    >>> x = 3
    >>> 1 < x < 5
    True
    >>> 4 > x >=3
    True

注意函数的默认参数
------------------

::

    >>> def foo(x=[]):
    ...     x.append(1)
    ...     print x
    ...
    >>> foo()
    [1]
    >>> foo()
    [1, 1]

更安全的做法::

     >>> def foo(x=None):
     ...     if x is None:
     ...         x = []
     ...     x.append(1)
     ...     print x
     ...
     >>> foo()
     [1]
     >>> foo()
     [1]
     >>>

字典有个get()方法
-----------------

``dct.get(key[, default_value])`` , 当字典 ``dct`` 中找不到 ``key`` 时， ``get`` 就会返回 ``default_value`` ::

    sum[value] = sum.get(value, 0) + 1

带关键字的格式化
-----------------

::

    >>> print "Hello %(name)s !" % {'name': 'James'}
    Hello James !
    >>> print "I am years %(age)i years old" % {'age': 18}
    I am years 18 years old

更新些的格式化::

    >>> print "Hello {name} !".format(name="James")
    Hello James !

快有些模板引擎的味道了:)

for...else 语法
-----------------

::

    >>> for i in (1, 3, 5):
    ...     if i % 2 == 0:
    ...         break
    ... else:
    ...     print "var i is always an odd"
    ...
    var i is always an odd
    >>>

``else`` 语句块会在循环结束后执行，除非在循环块中执行 ``break``

dict 的特殊方法__missing__
---------------------------

Python 2.5之后引入的。当查找不到 ``key`` 的时候，会执行这个方法。

::

    >>> class Dict(dict):
    ...   def __missing__(self, key):
    ...     self[key] = []
    ...     return self[key]
    ...
    >>> dct = Dict()
    >>> dct["foo"].append(1)
    >>> dct["foo"].append(2)
    >>> dct["foo"]
    [1, 2]

这很像 ``collections.defaultdict`` 不是吗? ::

    >>> from collections import defaultdict
    >>> dct = defaultdict(list)
    >>> dct["foo"]
    []
    >>> dct["bar"].append("Hello")
    >>> dct
    defaultdict(<type 'list'>, {'foo': [], 'bar': ['Hello']})


切片操作的步长参数
------------------

还能用步长 ``-1`` 来反转链表::

    >>> a = [1, 2, 3, 4, 5]
    >>> a[::2]
    [1, 3, 5]
    >>> a[::-1]
    [5, 4, 3, 2, 1]
    >>>

另一种字符串连接
----------------

::

    >>> Name = "Wang" "Hong"
    >>> Name
    'WangHong'

连接多行::

    >>> Name = "Wang" \
    ...  "Hong"
    >>> Name
    'WangHong'

Python解释器中的"_"
-------------------

::

    >>> range(4)
    [0, 1, 2, 3]
    >>> _
    [0, 1, 2, 3]

``_`` 即Python解释器上一次返回的值

Python 描述器
--------------

Python描述器是Python 中很魔幻的东西，方法等都是描述器。不再举例

Zen
----

::

    import this

嵌套列表推导式
---------------

::

    >>> [(i, j) for i in range(3) for j in range(i)]
    [(1, 0), (2, 0), (2, 1)]


try/except/else
----------------

::

    try:
      put_4000000000_volts_through_it(parrot)
    except Voom:
      print "'E's pining!"
    else:
      print "This parrot is no more!"
    finally:
      end_sketch()


print 重定向输出到文件
----------------------

::

    >>> print >> open("somefile", "w+"), "Hello World"

注意打开的模式: ``"w+"`` 而不能 ``"w"`` ,  当然 ``"a"`` 是可以的

省略号
------

在 ``Python 3`` 中你可以直接使用省略号这个文法::

    Python 3.2 (r32:88445, Oct 20 2012, 14:09:50)
    [GCC 4.5.2] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> ...
    Ellipsis

Python2 中呢? ::

    >>> class C(object):
    ...  def __getitem__(self, item):
    ...   return item
    ...
    >>> C()[1:2, ..., 3]
    (slice(1, 2, None), Ellipsis, 3)
    >>>

Python3中的元组unpack
----------------------

真的但愿Python2也这样::

    >>> a, b, *rest = range(10)
    >>> a
    0
    >>> b
    1
    >>> rest
    [2, 3, 4, 5, 6, 7, 8, 9]
    >>>

当然也可以取出最后一个::

    >>> first, second, *rest, last = range(10)
    >>> first
    0
    >>> second
    1
    >>> last
    9
    >>> rest
    [2, 3, 4, 5, 6, 7, 8]

pow()还有第三个参数
-------------------

我们都知道内置函数 ``pow``,  ``pow(x, y)`` 即 ``x ** y``

但是它还可以有第三个参数::

    >>> pow(4, 2, 2)
    0
    >>> pow(4, 2, 3)
    1

其实第三个参数是来求模的: ``pow(x, y, z) == (x ** y) % z``

注意，内置的 ``pow`` 和 ``math.pow`` 并不是一个函数，后者只接受2个参数

enumerate还有第二个参数
------------------------

``enumerate`` 很赞，可以给我们索引和序列值的对, 但是它还有第二个参数::

    >>> lst = ["a", "b", "c"]
    >>> list(enumerate(lst, 1))
    [(1, 'a'), (2, 'b'), (3, 'c')]

这个参数用来: 指明索引的起始值

显式的声明一个集合
-------------------

新建一个集合，我们会::

    >>> set([1,2,3])

在Python 2.7 之后可以这么写了::

    >>> {1,2,3}
    set([1, 2, 3])

用切片来删除序列的某一段
------------------------

::

    >>> a = [1, 2, 3, 4, 5, 6, 7]
    >>> a[1:4] = []
    >>> a
    [1, 5, 6, 7]

当然用 ``del a[1:4]`` 也是可以的

去除偶数项(偶数索引的)::

    >>> a = [0, 1, 2, 3, 4, 5, 6, 7]
    >>> del a[::2] 
    >>> a
    [1, 3, 5, 7]

isinstance可以接收一个元组
--------------------------

这个真的鲜为人知,  我们可以用 ``isinstance(x, (float, int))`` 来判断 ``x`` 是不是数::

    >>> isinstance(1, (float, int))                                                                                                         
    True
    >>> isinstance(1.3, (float, int))                                                                                                       
    True
    >>> isinstance("1.3", (float, int))                                                                                                     
    False

那么对于第三个测试，你把 ``str`` 加入元组就可以看到这是怎么回事了::

    >>> isinstance("1.3", (float, int, str))
    True

也就是那个元组里面是 **或** 的关系，只要是其中一个的实例就返回 ``True``

字典里的无限递归
-----------------

::

    >>> a, b = {}, {}
    >>> a['b'] = b
    >>> b['a'] = a
    >>> a
    {'b': {'a': {...}}}

当然你可以制作一个链表中的无限循环::

    >>> a, b =  [], []
    >>> a.append(b)                                                                                                                         
    >>> b.append(a)
    >>> a
    [[[...]]]

真心不知道有什么用，不过蛮好玩的不是吗

Python可以认识Unicode中的数字
-----------------------------

所以说，Python很赞::

    >>> int(u'1234')
    1234

不只是ASCII字符串的可以认出来，连Unicode的也可以。

不能访问到的属性
-----------------

回答这个答案的人太坏了:) ::

    >>> class O(object):pass
    ... 
    >>> o = O()
    >>> setattr(o, "can't touch this", 123)
    >>> o.can't touch this
      File "<stdin>", line 1
        o.can't touch this
                         ^
    SyntaxError: EOL while scanning string literal
    >>> 

不过，能用 ``setattr`` 设置属性，就可以用 ``getattr`` 取出

尾语
----

欢迎fork, 追加内容。
