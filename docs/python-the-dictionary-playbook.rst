=======================================================
Python:字典的剧本(翻译自Python:The Dictionary Playbook)
=======================================================


:原文: http://blog.amir.rachum.com/post/39501813266/python-the-dictionary-playbook

:翻译: hit9

我经常遇到各种五花八门的关于Python中的字典的代码用法，我决定在这个文章中展示一些并分享完成同样操作的更为简洁的做法。

.. Contents::

1.键的存在性(The "Are You There?")
----------------------------------

不推荐的::

    dct.has_key(key)

Python的做法::

    key in dct

2.Yoda 测试(The "Yoda Test")
-----------------------------

对于那些精通 "Are Your There" 的程序员，还有一个简单的同样也很恼人的行为，不仅仅对于dict而言，它很普遍

如果你非要Not ::

    not key in dct

English,你会说么？::

    key not in dct

3.无论如何都要取到值(The "Get The Value Anyway")
------------------------------------------------

这个是相当流行。你有一个dict和key,并且你想修改key对应的值，比如给它自增1(假如你在数数)。

比较逊一点的做法：::

    if key not in dct:
        dct[key] = 0
    dct[key] = dct[key] + 1

漂亮的做法::

    dct[key] = dct.get(key, 0) + 1

关于 :code:`dct.get(key[, default])` ,如果 ``key`` 存在,返回 ``dct[key]``,否则返回 ``default``

更棒的做法是:

如果你在使用Python 2.7,你就可以用Counter来计数了::

    >>> from collections import Counter
    >>> d = [1, 1, 1, 2, 2, 3, 1, 1]
    >>> Counter(d)
    Counter({1: 5, 2: 2, 3: 1})

这里还有些更完整的例子::

    >>> counter = Counter()
    ... for _ in range(10):
    ...     num = int(raw_input("Enter a number: "))
    ...     counter.update([num]) 
    ...
    ... for key, value in counter.iteritems():
    ...     print "You have entered {}, {} times!".format(key, value) 
    Enter a number: 1
    Enter a number: 1
    Enter a number: 2
    Enter a number: 3
    Enter a number: 51
    Enter a number: 1
    Enter a number: 1
    Enter a number: 1
    Enter a number: 2
    Enter a number: 3
    You have entered 1, 5 times!
    You have entered 2, 2 times!
    You have entered 3, 2 times!
    You have entered 51, 1 times!

4. 让它发生!(The "Make it Happen")
----------------------------------

有时你的字典里都是经常修改的对象，你需要初始化一些数据到这个字典，也需要修改其中的一些值。比如说，你在维护一个这样的dict:它的值都是链表。

写一个实现就是::

    dct = {} 
    for (key, value) in data: 
        if key in dct: 
            dct[key].append(value) 
        else: 
            dct[key] = [value]

更Pythonic的做法是:

::

    dct = {} 
    for (key, value) in data:
        group = dct.setdefault(key, []) # key might exist already 
        group.append(value)

``setdefault(key, default)`` 所做的是：如果存在，返回 ``dct[key]`` , 不存在则把 ``dct[key]`` 设为 ``default`` 并返回它。当一个默认的值是一个你可以修改的对象的时候这是很有用的。

使用 ``defaultdict`` ::

    dct = defaultdict(list) 
    for (key, value) in data: 
        dct[key].append(value) # all keys have a default already

``defaultdict`` 非常棒, 它每生成一对新的 ``key-value`` ，就会给value一个默认值, 这个默认值就是
``defaultdict`` 的参数。(注:defaultdict在模块 ``collections`` 中)

一个很有意思的就是，defaultdict实现的一行的tree: https://gist.github.com/2012250
