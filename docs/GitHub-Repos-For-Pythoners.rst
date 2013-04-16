:Date: 2013-03-01 14:01:01

==================================
Github上Python开发者应该关心的Repo
==================================

:作者: hit9
:日期: 2013-01-30
:注: 欢迎fork后pull request来丰富这个文章.

.. Contents::

carbaugh/lice
-------------

lice_ : Generate license files for your projects

.. _lice: https://github.com/jcarbaugh/lice

一个用来为你的项目生成许可证的工具。这下可方便了，不用手工的去修改了！

coleifer/peewee
---------------

peewee_: a small, expressive orm -- supports postgresql, mysql and sqlite 

你在用SQLAlchemy ? 我强烈推荐你看下peewee

来看一个sample::

    User.select().where(User.active == True).order_by(User.username)

.. _peewee: https://github.com/coleifer/peewee

一个单文件的Python ORM.相当轻巧，支持三个数据库。而且，它最讨人喜欢的是它的轻量级的语法。

docopt/docopt
-------------

docopt_ : Pythonic command line arguments parser, that will make you smile 

.. _docopt: https://github.com/docopt/docopt

用过doctest? 那来看看docopt。有时候你用py写一个命令行程序,需要接收命令行参数，看看这个例子::

    """
    Usage: test.py <file> [--verbose]
    """
    
    from docopt import docopt
    
    print docopt(__doc__)

如果你这么执行程序::

    python test.py somefile --verbose

你会得到这样的输出::

    {'--verbose': True, '<file>': 'somefile'}

hhatto/autopep8
---------------

autopep8_ : A tool that automatically formats Python code to conform to the PEP 8 style guide. 

.. _autopep8: https://github.com/hhatto/autopep8

每个Python程序员都应该checkout的repo.自动的把你的Python代码转成符合PEP8风格的代码.

使用 ``-i`` 参数来直接修改你的 Python文件::

    autopep8 -i mycode.py

kachayev/fn.py
--------------

fn.py_ : Functional programming in Python: implementation of missing features to enjoy FP

.. _fn.py: https://github.com/kachayev/fn.py

这是个很有趣的项目，来弥补Python在函数式编程方面没有的一些特性。来看个sample::

    from fn import _
    assert list(map(_ * 2, range(5))) == [0,2,4,6,8]

nose-devs/nose
--------------

nose_ : nose is nicer testing for python

.. _nose: https://github.com/nose-devs/nose

或许nose已经不是新鲜的测试框架了，现在还有很多新的测试框架诞生，不过大家都在用它，而且似乎没要离开nose的意思。

amoffat/sh
----------

sh_ : Python subprocess interface

.. _sh: https://github.com/amoffat/sh

这个库已经被津津乐道很久了。看代码::

    from sh import git

    git.clone("https://github.com/amoffat/sh")

是不是比 ``os.system`` 更简洁明了。

Lokaltog/powerline
------------------

如果你是个linux(or mac)下的开发者，又喜欢在终端下工作的话，你一定喜欢用powerline来美化自己的工作空间。

之前github上兴起了vim-powerline,tmux-powerline,还有powerline-bash,现在Lokaltog提供了一个统一的解决方案，只要安装这个python包，再追加些东西到配置文件就可以使用漂亮的powerline了

具体的效果请见repo : https://github.com/Lokaltog/powerline

benoitc/gunicorn
----------------

gunicorn_ : gunicorn 'Green Unicorn' is a WSGI HTTP Server for UNIX, fast clients and sleepy applications

.. _gunicorn: https://github.com/benoitc/gunicorn

一个Python WSGI UNIX的HTTP服务器,从Ruby的独角兽（Unicorn)项目移植。Gunicorn大致与各种Web框架兼容.

一个例子，运行你的flask app::

    gunicorn myproject:app

使用起来超级简单！

faif/python-patterns
--------------------

python-patterns_ : A collection of design patterns implemented (by other people) in python

.. _python-patterns : https://github.com/faif/python-patterns

这个repo收集了很多设计模式的python写法
