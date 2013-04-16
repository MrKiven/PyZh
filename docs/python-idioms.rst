:Date: 2013-02-23

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

以前Python中的异常仅仅是简单的字符串。现在不同了，因为类有了很多新的进步。特别是，你可以为异常定义子类，可以选择性的捕捉一些异常或者捕捉它们的超类。异常类一般不复杂.一个典型的异常类看起来是这样的::

    class MyException:
        def __init__(self, value):
            self.value = value
        def __str__(self):
            return `self.value`

这样使用::

    try:
        do_stuff()
        if something_bad_has_happened():
            raise MyException, "something bad happened"
    except MyException, e:
        print "My exception occurred, value: ", e.value

列表生成式
----------

这是Python中全新的一个特征，来源于函数式编程语言Haskell(很酷的编程语言，顺便告诉你，你应该看看haskell)

其思想是:有时你想要为具有某些特征的对象做一个链表，比如你想要为0到20的偶数做一个链表::

    results = []
    for i in range(20):
        if i % 2 == 0:
            results.append(i)

``results`` 里面就是结果：``[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]`` (没有20,因为range(20)是从0到19).但是同样的事情你可以用列表生成式来做地更简洁些::

   results = [x for x in range(20) if x % 2 == 0]

列表生成式是循环的语法糖.你可以做些更复杂的::

    results = [(x, y)
               for x in range(10)
               for y in range(10)
               if x + y == 5
               if x > y]

结果 ``results`` 是 ``[(3, 2), (4, 1), (5, 0)]`` . 所以你可以在方括号中写任意多个for和if语句(可能更多，详细参见文档), 你可以用列表生成式来实现快速排序算法::

    def quicksort(lst):
        if len(lst) == 0:
            return []
        else:
            return quicksort([x for x in lst[1:] if x < lst[0]]) + [lst[0]] + \
                   quicksort([x for x in lst[1:] if x >= lst[0]])

优美吗? :-)

函数式编程
----------

Python实现了很多平常只出现在函数式编程语言(像lisp和ML)中的函数和特性。

1. :meth:`map` :meth:`reduce` :meth:`filter` 函数

   :meth:`map` 需要一个函数和几个序列做参数(通常一个)，然后对于序列的每个元素作为函数的参数,所有的返回值产生一个新的序列作为map的返回值。比如你想要把一个字符串链表转换成数字链表::

        lst = ["1", "2", "3", "4", "5"]
        nums = map(string.atoi, lst)  # [1, 2, 3, 4, 5]
        
   (译者注:Py2.7中使用 ``map(int, lst)`` )

   你可以对两个参数的函数使用map::

    def add(x, y):
        return x + y

    lst1 = [1, 2, 3, 4, 5]
    lst2 = [6, 7, 8, 9, 10]
    lst_sum = map(add, lst1, lst2)

    # lst_sum == [7, 9, 11, 13, 15]

   (译者注:这个函数可以有任意多参数，map的后面的参数要跟相应多的序列即可)

   你可以使用 :meth:`reduce` 来把一个序列减少成一个值。第一个参数是函数，这个函数首先作用于序列的第一个和第二个元素，然后用返回的值继续和序列的第三个元素执行这个函数。。。直到剩下一个值，作为reduce的返回值.比如，你可以这么来求0到9的和::

    reduce(lambda x, y: x+y, range(10))

   (译者注:这里为了讲解，一般推荐直接用函数 :meth:`sum` )

   你可以使用 :meth:`filter` 来生成一个序列的子集。比如，获取0到100的所有奇数::

        nums = range(0,101)  # [0, 1, ... 100]
        
        def is_odd(x):
            return x % 2 == 1
        
        odd_nums = filter(is_odd, nums)  # [1, 3, 5, ... 99]

2. ``lambda`` 关键字

   lambda 语句声明了一个匿名的函数,很多时候我们在reduce，map等函数中使用的函数只使用了一次。这些函数可以被简洁地声明为匿名函数::

       lst1 = [1, 2, 3, 4, 5]
       lst2 = [6, 7, 8, 9, 10]
       lst_elementwise_sum = map(lambda x, y: x + y, lst1, lst2)
       lst1_sum = reduce(lambda x, y: x + y, lst1)
       nums = range(101)
       odd_nums = filter(lambda x: x % 2 == 1, nums)

   注意，你仍然可以在lambda函数中使用这个匿名函数外定义的变量。这叫做“词法范围”(lexical scoping),在Python 2.2时官方性地给出了介绍.就像这样::

       a = 1
       add_a = lambda x: x + a
       b = add_a(10)  # b == 11
   
   lambda中的 ``a`` 是在上一行定义的。如果你认为这是显然的，很好！事实证明很多Python开发者花了很多时间来搞明白它。

   关于lambda你可以参见下 lisp 或 scheme的文档。

3. ``apply`` 函数

   在python中函数是对象，你可以像操作数和字符串一样操作它们(把它们存在变量中等等)，有时你有一个函数对象，你想把程序中生成的一个序列作为参数传给这个函数::

       # Sorry about the long variable names ;-)

       args = function_returning_list_of_numbers()
       f    = function_returning_a_function_which_operates_on_a_list_of_numbers()

       # You want to do f(arg[0], arg[1], ...) but you don't know how many
       # arguments are in 'args'.  For this you have to use 'apply':

       result = apply(f, args)

       # A trivial example:
       args = [1, 1]
       two = apply(lambda x, y: x + y, args)  # == 2

   译者注:之所以我们不经常用 :meth:`apply` ,一般可以这么做::
       
       args = [1, 1]
       foo = lambda x, y: x+y
       foo(*args)

生成器和迭代器
--------------

这是一个高级的(并且很cool)的话题，我们这里没有地方展开说了。如果你关心这个，去看Python文档吧.-_-

PEPS
----

Python社区非常活跃，"comp.lang.python"小组有很多关于Python应该添加什么新特征的讨论。

所有PEP,见这里:http://www.python.org/dev/peps/
