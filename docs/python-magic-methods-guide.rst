:Date: 2013-07-05 22:34:00

======================
(译)Python魔法方法指南
======================

:原文: http://www.rafekettler.com/magicmethods.html
:原作者: Rafe Kettler
:翻译: hit9
:原版(英文版) Repo: https://github.com/RafeKettler/magicmethods

.. Contents::

简介
----

本指南归纳于我的几个月的博客，主题是 **魔法方法** 。

什么是魔法方法呢？它们在面向对象的Python的处处皆是。它们是一些可以让你对类添加“魔法”的特殊方法。
它们经常是两个下划线包围来命名的（比如 `__init__` ， `__lt__` ）。但是现在没有很好的文档来解释它们。
所有的魔法方法都会在Python的官方文档中找到，但是它们组织松散。而且很少会有示例（有的是无聊的语法描述，
语言参考）。

所以，为了修复我感知的Python文档的缺陷，我开始提供更为通俗的，有示例支持的Python魔法方法指南。我一开始
写了一些博文，现在我把这些博文总起来成为一篇指南。

希望你喜欢这篇指南，一篇友好，通俗易懂的Python魔法方法指南！

构造方法
--------

我们最为熟知的基本的魔法方法就是 `__init__` ，我们可以用它来指明一个对象初始化的行为。然而，当我们调用
`x = SomeClass()` 的时候， `__init__` 并不是第一个被调用的方法。事实上，第一个被调用的是 `__new__` ，这个
方法才真正地创建了实例。当这个对象的生命周期结束的时候， `__del__` 会被调用。让我们近一步理解这三个方法：

- `__new__(cls,[...)` 

  `__new__` 是对象实例化时第一个调用的方法，它只取下 `cls` 参数，并把其他参数传给 `__init__` 。 `__new__` 
  很少使用，但是也有它适合的场景，尤其是当类继承自一个像元组或者字符串这样不经常改变的类型的时候。我不打算深入讨论
  `__new__` ，因为它并不是很有用， `Python文档 <http://www.python.org/download/releases/2.2/descrintro/#__new__>`_ 中
  有详细的说明。

- `__init__(self,[...])`

  类的初始化方法。它获取任何传给构造器的参数（比如我们调用 `x = SomeClass(10, 'foo')` ， `__init__` 就会接到参数
  `10` 和 `'foo'` 。 `__init__` 在Python的类定义中用的最多。

- `__del__(self)` 

  `__new__` 和 `__init__` 是对象的构造器， `__del__` 是对象的销毁器。它并非实现了语句 `del x` (因此该语句不等同于 `x.__del__()`)。而是定义了当对象被垃圾回收时的行为。
  当对象需要在销毁时做一些处理的时候这个方法很有用，比如 `socket` 对象、文件对象。但是需要注意的是，当Python解释器退出但对象仍然存活的时候， `__del__` 并不会
  执行。 所以养成一个手工清理的好习惯是很重要的，比如及时关闭连接。

这里有个 `__init__` 和 `__del__` 的例子::

    from os.path import join
    
    class FileObject:
        '''Wrapper for file objects to make sure the file gets closed on deletion.'''
    
        def __init__(self, filepath='~', filename='sample.txt'):
            # open a file filename in filepath in read and write mode
            self.file = open(join(filepath, filename), 'r+')
    
        def __del__(self):
            self.file.close()
            del self.file


操作符
------

使用Python魔法方法的一个巨大优势就是可以构建一个拥有Python内置类型行为的对象。这意味着你可以避免使用非标准的、丑陋的方式来表达简易的操作。
在一些语言中，这样做很常见::

    if instance.equals(other_instance):
        # do something

你当然可以在Python也这么做，但是这样做让代码变得冗长而混乱。不同的类库可能对同一种比较操作采用不同的方法名称，这让使用者需要做很多没有必要的工作。运用魔法方法的魔力，我们可以定义方法 `__eq__` ::

    if instance == other_instance:
        #do something

这是魔法力量的一部分，这样我们就可以创建一个像内建类型那样的对象了！

比较操作符
''''''''''

Python包含了一系列的魔法方法，用于实现对象之间直接比较，而不需要采用方法调用。同样也可以重载Python默认的比较方法，改变它们的行为。下面是这些方法的列表：

- `__cmp__(self, other)`

  `__cmp__` 是所有比较魔法方法中最基础的一个，它实际上定义了所有比较操作符的行为（<,==,!=,等等），但是它可能不能按照你需要的方式工作（例如，判断一个实例和另一个实例是否相等采用一套标准，而与判断一个实例是否大于另一实例采用另一套）。 `__cmp__` 应该在 `self < other` 时返回一个负整数，在 `self == other` 时返回0，在 `self > other` 时返回正整数。最好只定义你所需要的比较形式，而不是一次定义全部。 如果你需要实现所有的比较形式，而且它们的判断标准类似，那么 `__cmp__` 是一个很好的方法，可以减少代码重复，让代码更简洁。


- `__eq__`(self, other)`

  定义等于操作符(==)的行为。

- `__ne__(self, other)`

  定义不等于操作符(!=)的行为。

- `__lt__(self, other)`

  定义小于操作符(<)的行为。

- `__gt__(self, other)`

  定义大于操作符(>)的行为。

- `__le__(self, other)`

  定义小于等于操作符(<)的行为。

- `__ge__(self, other)`

  定义大于等于操作符(>)的行为。

举个例子，假如我们想用一个类来存储单词。我们可能想按照字典序（字母顺序）来比较单词，字符串的默认比较行为就是这样。我们可能也想按照其他规则来比较字符串，像是长度，或者音节的数量。在这个例子中，我们使用长度作为比较标准，下面是一种实现::
    
    class Word(str):
        '''Class for words, defining comparison based on word length.'''

        def __new__(cls, word):
            # Note that we have to use __new__. This is because str is an immutable
            # type, so we have to initialize it early (at creation)
            if ' ' in word:
                print "Value contains spaces. Truncating to first space."
                word = word[:word.index(' ')] # Word is now all chars before first space
            return str.__new__(cls, word)

        def __gt__(self, other):
            return len(self) > len(other)
        def __lt__(self, other):
            return len(self) < len(other)
        def __ge__(self, other):
            return len(self) >= len(other)
        def __le__(self, other):
            return len(self) <= len(other)
    
 
现在我们可以创建两个 `Word` 对象（ `Word('foo')` 和 `Word('bar')`)然后根据长度来比较它们。注意我们没有定义 `__eq__` 和 `__ne__` ，这是因为有时候它们会导致奇怪的结果（很明显， `Word('foo') == Word('bar')` 得到的结果会是true）。根据长度测试是否相等毫无意义，所以我们使用 `str` 的实现来比较相等。

从上面可以看到，不需要实现所有的比较魔法方法，就可以使用丰富的比较操作。标准库还在 `functools` 模块中提供了一个类装饰器，只要我们定义 `__eq__` 和另外一个操作符（ `__gt__`, `__lt__` 等），它就可以帮我们实现比较方法。这个特性只在 Python 2.7 中可用。当它可用时，它能帮助我们节省大量的时间和精力。要使用它，只需要它 `@total_ordering` 放在类的定义之上就可以了

数值操作符
''''''''''

就像你可以使用比较操作符来比较类的实例，你也可以定义数值操作符的行为。固定好你的安全带，这样的操作符真的有很多。看在组织的份上，我把它们分成了五类：一元操作符，常见算数操作符，反射算数操作符（后面会涉及更多），增强赋值操作符，和类型转换操作符。


一元操作符
==========

一元操作符只有一个操作符。

- `__pos__(self)`

  实现取正操作，例如 `+some_object`。
  
- `__neg__(self)` 

  实现取负操作，例如 `-some_object`。
  
- `__abs__(self)`

  实现内建绝对值函数 `abs()` 操作。
  
- `__invert__(self)` 

  实现取反操作符 `~`。
  
- `__round__(self， n)` 

  实现内建函数 `round()` ，n 是近似小数点的数目。

- `__floor__(self)`

  实现 `math.floor()` 函数，即向下取整。

- `__ceil__(self)`

  实现 `math.ceil()` 函数，即向上取整。

- `__trunc__(self)`

  实现 `math.trunc()` 函数，即距离零最近的整数。


常见算数操作符
===============

现在，我们来看看常见的二元操作符（和一些函数），像+，-，*之类的，它们很容易从字面意思理解。

- `__add__(self, other)` 

  实现加法操作。
  
- `__sub__(self, other)`

  实现减法操作。

- `__mul__(self, other)` 

  实现乘法操作。

- `__floordiv__(self, other)`

  实现使用 `//` 操作符的整数除法。

- `__div__(self, other)`

  实现使用 `/` 操作符的除法。

- `__truediv__(self, other)`

  实现 `_true_` 除法，这个函数只有使用 `from __future__ import division` 时才有作用。

- `__mod__(self, other)`

  实现 `%` 取余操作。

- `__divmod__(self, other)`

  实现 `divmod` 内建函数。
  
- `__pow__` 

  实现 `**` 操作符。

- `__lshift__(self, other)`

  实现左移位运算符 `<<` 。
  
- `__rshift__(self, other)` 

  实现右移位运算符 `>>` 。
  
  
- `__and__(self, other)`
  
  实现按位与运算符 `&` 。
  
- `__or__(self, other)`

  实现按位或运算符 `|` 。
  
- `__xor__(self, other)`

  实现按位异或运算符 `^` 。
  

反射算数运算符
===============

还记得刚才我说会谈到反射运算符吗？可能你会觉得它是什么高端霸气上档次的概念，其实这东西挺简单的，下面举个例子::

    some_object + other

这是“常见”的加法，反射是一样的意思，只不过是运算符交换了一下位置::

    other + some_object
    
所有反射运算符魔法方法和它们的常见版本做的工作相同，只不过是处理交换连个操作数之后的情况。绝大多数情况下，反射运算和正常顺序产生的结果是相同的，所以很可能你定义 `__radd__` 时只是调用一下 `__add__`。注意一点，操作符左侧的对象（也就是上面的 `other` ）一定不要定义（或者产生 `NotImplemented` 异常） 操作符的非反射版本。例如，在上面的例子中，只有当 `other` 没有定义 `__add__` 时 `some_object.__radd__` 才会被调用。


- `__radd__(self, other)` 

  实现反射加法操作。
  
- `__rsub__(self, other)`

  实现反射减法操作。

- `__rmul__(self, other)` 

  实现反射乘法操作。

- `__rfloordiv__(self, other)`

  实现使用 `//` 操作符的整数反射除法。

- `__rdiv__(self, other)`

  实现使用 `/` 操作符的反射除法。

- `__rtruediv__(self, other)`

  实现 `_true_` 反射除法，这个函数只有使用 `from __future__ import division` 时才有作用。

- `__rmod__(self, other)`

  实现 `%` 反射取余操作符。

- `__rdivmod__(self, other)`

  实现调用 `divmod(other, self)` 时 `divmod` 内建函数的操作。
  
- `__rpow__` 

  实现 `**` 反射操作符。

- `__rlshift__(self, other)`

  实现反射左移位运算符 `<<` 的作用。
  
- `__rshift__(self, other)` 

  实现反射右移位运算符 `>>` 的作用。
  
- `__rand__(self, other)`
  
  实现反射按位与运算符 `&` 。
  
- `__ror__(self, other)`

  实现反射按位或运算符 `|` 。
  
- `__rxor__(self, other)`

  实现反射按位异或运算符 `^` 。
  

增强赋值运算符
===============

Python同样提供了大量的魔法方法，可以用来自定义增强赋值操作的行为。或许你已经了解增强赋值了，它融合了“常见”的操作符和赋值操作，如果你还是没听明白，看下面的例子::

    x = 5
    x += 1 # in other words x = x + 1
    
这些方法都应该返回左侧操作数应该被赋予的值（例如， `a += b` `__iadd__` 也许会返回 `a + b` ，这个结果会被赋给 a ）,下面是方法列表：

- `__iadd__(self, other)` 

  实现加法赋值操作。
  
- `__isub__(self, other)`

  实现减法赋值操作。

- `__imul__(self, other)` 

  实现乘法赋值操作。

- `__ifloordiv__(self, other)`

  实现使用 `//=` 操作符的整数除法赋值操作。

- `__idiv__(self, other)`

  实现使用 `/=` 操作符的除法赋值操作。

- `__itruediv__(self, other)`

  实现 `_true_` 除法赋值操作，这个函数只有使用 `from __future__ import division` 时才有作用。

- `__imod__(self, other)`

  实现 `%=` 取余赋值操作。
  
- `__ipow__` 

  实现 `**=` 操作。

- `__ilshift__(self, other)`

  实现左移位赋值运算符 `<<=` 。
  
- `__irshift__(self, other)` 

  实现右移位赋值运算符 `>>=` 。 
  
- `__iand__(self, other)`
  
  实现按位与运算符 `&=` 。
  
- `__ior__(self, other)`

  实现按位或赋值运算符 `|` 。
  
- `__ixor__(self, other)`

  实现按位异或赋值运算符 `^=` 。


类型转换操作符
===============

Python也有一系列的魔法方法用于实现类似 `float()` 的内建类型转换函数的操作。它们是这些：

- `__int__(self)`
  
  实现到int的类型转换。
  
- `__long__(self)`

  实现到long的类型转换。
  
- `__float__(self)`
  
  实现到float的类型转换。
  
- `__complex__(self)`

  实现到complex的类型转换。
  
- `__oct__(self)`

  实现到八进制数的类型转换。
  
- `__hex__(self)`

  实现到十六进制数的类型转换。
  
- `__index__(self)`

  实现当对象用于切片表达式时到一个整数的类型转换。如果你定义了一个可能会用于切片操作的数值类型，你应该定义 `__index__`。
  
- `__trunc__(self)`

  当调用 `math.trunc(self)` 时调用该方法， `__trunc__` 应该返回 `self` 截取到一个整数类型（通常是long类型）的值。
  
- `__coerce__(self)`
  
  该方法用于实现混合模式算数运算，如果不能进行类型转换， `__coerce__` 应该返回 `None` 。反之，它应该返回一个二元组 `self` 和 `other` ，这两者均已被转换成相同的类型。


未完待续..
