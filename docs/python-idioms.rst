================
(译)Python的惯例
================

:原文: http://courses.cms.caltech.edu/cs11/material/python/misc/python_idioms.html

每一种计算机语言都有其使用惯例，就是那些完成给定任务所采用的典型方法。Python也不例外。一些使用惯例并不为人所知，所以我们在这里收集一下。我们同样添加了一些可能你在阅读教程的时候没有注意到的一些特性。这些惯例在本文中将按照它们的使用难度和使用的频率的顺序展开。

**提醒!** 本文一些东西可能过时了，请参照最新的Python官方文档.

.. Contents::

请看这些文档...
---------------

虽然它们并不是真正的使用惯例，但是你应该知道它们

1. 长整数

2. 函数的可选参数

3. 函数的关键字参数

4. 类的 :meth:`getattr` 方法和 :meth:`__getattr__` 方法

5. 操作符重载

6. 多继承

7. 文档字符串(docstring)

8. 正则表达式

迭代一个数组
------------

Python 的for语句并不和C语言的一样，更像其他语言的foreach. 如果你需要在迭代中使用索引，标准的做法是::

    array = [1, 2, 3, 4, 5]  # or whatever

    for i in range(len(array)):
        # Do something with 'i'.

这是相当笨拙的，更干净简洁的做法是::

    array = [1, 2, 3, 4, 5]  # or whatever

    for i, e in enumerate(array):
        # Do something with index 'i' and its corresponding element 'e'.

打断无限循环
------------

Python并不像C语言有"do/while"语句，它只有一个while循环语句和for循环语句，有时你并不会提前知道什么时候循环会结束，或者你需要打破循环。一个不能再普通的例子就是正在按行迭代一个文件内容::

    file = open("some_filename", "r")

    while 1:   # infinite loop
        line = file.readline()
        if not line:  # 'readline()' returns None at end of file.
            break

        # Process the line.

这样做并不聪明，但是足够规范。对于文件来说还有一个更漂亮的做法::

    file = open("some_filename", "r")

    for line in file:
        # Process the line.

注意Python中也有一个continue语句(就像C中的)来跳过本次循环进入下一次循环。

注意内置函数 :meth:`file` 跟 :meth:`open` 一样，而且现在更流行了(因为对象的构造器应该和这个对象一个名字)

序列乘法
--------

在Python中，链表和字符串都是序列，它们有很多一样的操作(比如 :meth:`len`).一个不太明显的惯用手法是序列乘法.想要得到一个包含100个0的链表，你可以这么做::

    zeroes = [0] * 100

类似地，可以这样做来获取一个包含100个空格的字符串::

   spaces = 100 * " "

这很方便。

xrange
------

有的时候你想要生成一个长链表但是并不想把它立刻存在内存中。比如，你想要迭代1到1,000,000,000，但是你并不想把这些数都存在内存中。这样你就不会想用 :meth:`range` .取而代之你应该用 :meth:`xrange` ,它是 :meth:`range` 的一个延迟加载的版本(lazy version), 也就是说它只会在需要的时候生成那个数。所以你可以这么写::

    for i in xrange(1000000000):
        # do something with i...

而且，内存使用会很平稳

"Print to"语法
--------------

最近,">>" 操作符被重载了, 这样你就可以像下面那样在print语句中使用它了::

    print >> sys.stderr, "this is an error message"

>>右边应该是一个文件对象。

译者注例子(Python2.7) ::

    print >>  file("myfile", "w"), "hello world"

异常类
------
