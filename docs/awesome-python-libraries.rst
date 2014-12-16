:Date: 2013-04-16 21:20:00

======================
让人耳目一新的Python库
======================

:作者: hit9

:联系: <nz2324 at 126 dot com>

.. Contents::

摘要
----

工欲善其事，必先利其器。Python社区高产富饶，有大量的美丽，实用的库, 这里挑选出来一些接口设计简洁的库。

docopt
------

Github: https://github.com/docopt/docopt

Pythonic的命令行参数解析库::

    """Usage:
      quick_example.py tcp <host> <port> [--timeout=<seconds>]
      quick_example.py serial <port> [--baud=9600] [--timeout=<seconds>]
      quick_example.py -h | --help | --version
    
    """
    from docopt import docopt


    if __name__ == '__main__':
        arguments = docopt(__doc__, version='0.1.1rc')
        print(arguments)

requests
--------

Github: https://github.com/kennethreitz/requests

大神kennethreitz的作品，简易明了的HTTP请求操作库, 是urllib2的理想替代品

API简洁明了，这才是Python开发者喜欢的::

    >>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
    >>> r.status_code
    200
    >>> r.headers['content-type']
    'application/json; charset=utf8'
    >>> r.encoding
    'utf-8'
    >>> r.text
    u'{"type":"User"...'
    >>> r.json()
    {u'private_gists': 419, u'total_private_repos': 77, ...}

sh
---

http://amoffat.github.io/sh/

如其名，子进程接口。

::

    from sh import ifconfig
    print(ifconfig("wlan0"))

purl
----

github: https://github.com/codeinthehole/purl

拥有简洁接口的URL处理器::

    >>> from purl import URL
    >>> from_str = URL('https://www.google.com/search?q=testing')
    >>> u.query_param('q')
    u'testing'
    >>> u.host()
    u'www.google.com'

path.py
-------

github: https://github.com/jaraco/path.py

一个文件系统处理库，不过目前还在开发阶段

::

    from path import path
    d = path('/home/guido/bin')
    for f in d.files('*.py'):
    f.chmod(0755)

Peewee
------

https://github.com/coleifer/peewee

小型ORM, 接口很漂亮::

    # get tweets by editors ("<<" maps to IN)
    Tweet.select().where(Tweet.user << editors)

    # how many active users are there?
    User.select().where(User.active == True).count()

类似的我的 CURD.py (https://github.com/hit9/CURD.py) :) ::

    User.create(name="John", email="John@gmail.com")  # create

    User.at(2).update(email="John@github.com")  # update

    John = User.where(name="John").select().fetchone()  # read

    # who wrote posts?
    for post, user in (Post & User).select().fetchall():
        print "Author: %s, PostName: %s" % (user.name, post.name)


Pony ORM
--------

https://github.com/ponyorm/pony

一个十分独特的ORM，接口简单干净，最大的特点是支持使用generator的语法来进行查询，可以使查询语句变得简洁，灵活，而且漂亮。

例如可以使用如下的语句来进行一个查询::

    select(p for p in Product if p.name.startswith('A') and p.cost <= 1000)

同时，Pony ORM还提供了一个ER图编辑工具来进行数据库原型设计。

Spyne
-----

https://github.com/arskom/spyne

一个用于构建RPC服务的工具集，支持SOAP，JSON，XML等多种流行的协议。

现在有诸如 flask-restful 以及 django-rest-framework 等框架用于 REST 服务的开发，人们对于 REST 之外的框架似乎兴趣不大。Spyne 很好地填补了这一空白，它支持多种协议，而且本身也封装地相当好::

    class HelloWorldService(ServiceBase):
        @srpc(Unicode, Integer, _returns=Iterable(Unicode))
        def say_hello(name, times):
            for i in range(times):
                yield 'Hello, %s' % name
            
    application = Application([HelloWorldService],
        tns='spyne.examples.hello',
        in_protocol=Soap11(validator='lxml'),
        out_protocol=Soap11()
    )
    

短短几行代码便实现了一个支持SOAP 1.1 协议的服务器端application，接入任何一个WSGI兼容的服务器后端就可以运行了。

schema
------

https://github.com/halst/schema

同样是docopt的作者编写的，一个数据格式检查库，非常新颖::

    >>> from schema import Schema
    >>> Schema(int).validate(123)
    123
    >>> Schema(int).validate('123')
    Traceback (most recent call last):
    ...
    SchemaError: '123' should be instance of <type 'int'>

fn.py
-----

https://github.com/kachayev/fn.py

增强Python的函数式编程::

    from fn import _

    print (_ + 2) # "(x1) => (x1 + 2)"
    print (_ + _ * _) # "(x1, x2, x3) => (x1 + (x2 * x3))"

when.py
--------

https://github.com/dirn/When.py

友好的时间日期库::

    >>> import when
    >>> when.timezone()
    'Asia/Shanghai'
    >>> when.today()
    datetime.date(2013, 5, 14)
    >>> when.tomorrow()
    datetime.date(2013, 5, 15)
    >>> when.now()
    datetime.datetime(2013, 5, 14, 21, 2, 23, 78838)

clize
------

https://github.com/epsy/clize

用 docopt 写程序的使用doc是不是很爽，
clize是一个类似的库。可以用程序的函数名字来作为使用方法

::

    #!/usr/bin/env python
    
    from clize import clize
    
    @clize
    def echo(text, reverse=false):
        if reverse:
            text = text[::-1]
        print(text)
    if __name__ == '__main__':
        import sys
        echo(*sys.argv)


而这个小程序就可以这么使用::

    $ ./echo.py --help
    Usage: ./echo.py [OPTIONS] text
    
    Positional arguments:
      text
    
    Options:
      --reverse
      -h, --help   Show this help


Pocoo小组
---------

pocoo出的库，必属精品。 http://www.pocoo.org/

它的库很出名: flask, jinja2, pygments,sphinx

后言和链接
----------

- 你可能对 :ref:`github_awesome_python_repos` 感兴趣
- 看下这个gist https://gist.github.com/medecau/797129
- HN: https://news.ycombinator.com/item?id=4772261
