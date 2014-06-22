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

就像你可以使用比较操作符来比较类的实例，你也可以定义数值操作符的行为。固定好你的安全带，伙计，这样的操作符有很多。看在组织的份上，我把它们分成了五类：一元操作符，正常算数操作符，反射算数操作符（后面会涉及更多），增强赋值操作符，和类型转换操作符。


一元操作符
==========

未完待续..
