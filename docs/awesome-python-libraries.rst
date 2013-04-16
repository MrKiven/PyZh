:Date: 2013-04-16 21:20:00

==================
简洁漂亮的Python库
==================

:作者: hit9

:联系: <hit9 at 126 dot com>

.. Contents::

摘要
----

工欲善其事，必先利其器。Python社区高产富饶，有大量的美丽，实用的库, 这里挑选出来一些接口设计简约的库。

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

path
----

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


Pocoo小组
---------

pocoo出的库，必属精品。 http://www.pocoo.org/

它的库很出名: flask, jinja2, pygments,sphinx

后言和链接
----------

- 你可能对 :ref:`github_awesome_python_repos` 感兴趣
- 看下这个gist https://gist.github.com/medecau/797129
- HN: https://news.ycombinator.com/item?id=4772261
