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

你当然可以在Python也这么做，但是没必要这么做。运用魔法方法的魔力，我们可以定义方法 `__eq__` ::

    if instance == other_instance:
        #do something

这是魔法力量的一部分，这样我们就可以创建一个像内建类型那样的对象了！

比较操作符
''''''''''


