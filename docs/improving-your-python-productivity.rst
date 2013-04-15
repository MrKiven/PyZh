==========================
(译)提高你的Python编码效率
==========================

:作者: Oz Katz

:联系: https://twitter.com/ozkatz100

:翻译: hit9

:译者注:  原文链接 - http://ozkatz.github.io/improving-your-python-productivity.html

.. Contents::

我用Python编程有几年了, 并且我仍然经常惊讶于Python代码可以如何的简洁，如何的 DRY_ 。 我学到了很多小贴士和技巧，大多数来自于阅读开源项目的源代码，像 Django_, Flask_, Requests_ 等。

.. _DRY: http://en.wikipedia.org/wiki/Don't_repeat_yourself
.. _Django: https://www.djangoproject.com/
.. _Flask: http://flask.pocoo.org/
.. _Requests: http://docs.python-requests.org/en/latest/

这里我挑出了几个有时被大家忽略的几条，但是它们在日常工作中会有很大帮助。

1. 字典和集合推导式
-------------------

大多数Python开发者知道使用列表推导式。你不熟悉这一点？ 一个列表推导式是一个创造列表的简短方式::

    >>> some_list = [1, 2, 3, 4, 5]
    >>> another_list = [ x + 1 for x in some_list ]
    >>> another_list
    [2, 3, 4, 5, 6]

从Python 3.1开始(也反向地移植到了Python 2.7),我们可以用同样的方式创建集合和字典::

    >>> # Set Comprehensions
    >>> some_list = [1, 2, 3, 4, 5, 2, 5, 1, 4, 8]
    >>> even_set = { x for x in some_list if x % 2 == 0 }
    >>> even_set
    set([8, 2, 4])

    >>> # Dict Comprehensions
    >>> d = { x: x % 2 == 0 for x in range(1, 11) }
    >>> d
    {1: False, 2: True, 3: False, 4: True, 5: False, 6: True, 7: False, 8: True, 9: False, 10: True}


第一个例子中，我们用 ``some_list`` 建立了一个元素不重复的集合，但只有偶数。第二个字典的例子中展示了一个字典的创建，这个字典的键是1到10（包括10），值是布尔值，指明该键是不是一个偶数。

另一个值得注意的地方是集合的文法，我们可以这么简单的创建一个集合::

    >>> my_set = {1, 2, 1, 2, 3, 4}
    >>> my_set
    set([1, 2, 3, 4])

而没有使用到内建的 ``set`` 方法

2.使用计数器对象计数
--------------------

很明显，但很容易遗忘。计数是一个寻常不过的编程任务，而且大多数情形下这不是个难事。不过计数可以更简单。

Python的 collections_ 库包含一个 ``dict`` 的子类，专门解决计数任务::

    >>> from collections import Counter
    >>> c = Counter('hello world')
    >>> c
    Counter({'l': 3, 'o': 2, ' ': 1, 'e': 1, 'd': 1, 'h': 1, 'r': 1, 'w': 1})
    >>> c.most_common(2)
    [('l', 3), ('o', 2)]

.. _collections: http://docs.python.org/2/library/collections.html

3. 漂亮地打印JSON
------------------

JSON是一个很棒的序列格式，如今广泛应用在API和web服务中，但是很难用裸眼来看大数据量的JSON,它们很长，还在一行里。

可以用参数 ``indent`` 来更好地打印JSON数据，这在跟 REPL或是日志打交道的时候很有用::

    >>> import json
    >>> print(json.dumps(data))  # No indention
    {"status": "OK", "count": 2, "results": [{"age": 27, "name": "Oz", "lactose_intolerant": true}, {"age": 29, "name": "Joe", "lactose_intolerant": false}]}
    >>> print(json.dumps(data, indent=2))  # With indention
    {
      "status": "OK",
      "count": 2,
      "results": [
        {
          "age": 27,
          "name": "Oz",
          "lactose_intolerant": true
        },
        {
          "age": 29,
          "name": "Joe",
          "lactose_intolerant": false
        }
      ]
    }


另外，去看看内建模块 ``pprint`` , 它可以帮助你漂亮地输出其它的东西。

4. 快速建立一个web服务
-----------------------

有时我们需要一个建立RPC服务简单而快速的方法。我们需要的只是让程序B去调用程序A(可能在另一个机器上)的方法。

我们不用了解关于这个的任何技术，但是我们只是需要这么个简单的东西，我们可以使用一个叫做 XML-RPC_ 的协议(对应的Python库实现 SimpleXMLRPCServer_ )来处理这种事。

.. _XML-RPC: http://en.wikipedia.org/wiki/XML-RPC
.. _SimpleXMLRPCServer: http://docs.python.org/2/library/simplexmlrpcserver.html

这里是一个简单粗糙的文件阅读服务器::

    from SimpleXMLRPCServer import SimpleXMLRPCServer

    def file_reader(file_name):
        with open(file_name, 'r') as f:
            return f.read()

    server = SimpleXMLRPCServer(('localhost', 8000))
    server.register_introspection_functions()

    server.register_function(file_reader)

    server.serve_forever()

响应它的客户端::

    import xmlrpclib
    proxy = xmlrpclib.ServerProxy('http://localhost:8000/')
    proxy.file_reader('/tmp/secret.txt')

现在我们就有了一个远程的文件阅读器，除了一点代码，没有外部依赖。(当然，不安全，所以只在"家"用这个吧)

5. Python的开源社区
--------------------

刚我一直在说Python的标准库了，这些库只要你安装Python就会包含在你的Python中。对于大多数的其他任务，这里有大量的社区维护的第三方库来满足我们的需求。

这是一个我挑选Python库的办法:

- 包含一个明确的协议，以便我们使用

- 积极活跃的开发和维护

- 可以用 ``pip`` 来安装，可以轻易地重复部署

- 拥有一个合适覆盖率的测试集

如果你发现了一个适合你需求的Python库，不要害羞，大多数开源项目欢迎我们贡献代码和协助，即使你不是一个Python老将。帮助之手随时受欢迎！

译者追加的技巧
--------------

原文评论里的一些技巧, 值得一看！

- 快速在一个目录建立HTTP服务器
  ::

    python -m SimpleHTTPServer

  在 Python 3 中::

    python -m http.server

- 命令行上漂亮地打印JSON::

     echo '{"json":"obj"}' | python -mjson.tool

  而且，如果你安装了 ``Pygments`` 模块，可以高亮地打印JSON::

    echo '{"json":"obj"}' | python -mjson.tool | pygmentize -l json

- 注意 ``{}`` 是一个空的字典，而不是空的集合
