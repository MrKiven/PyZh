setup.py vs requirements.txt
============================

:原文: https://caremad.io/blog/setup-vs-requirement/
:作者: Donald
:译: hit9

.. Contents::

对于 ``setup.py`` 和 ``requirements.txt``
的角色有很多误解，很多人认为它们是两个重复的事情，甚至创造了 `工具 <https://pypi.python.org/pypi/pbr/#requirements>`_ 来处理
这种“重复”。

Python库
--------

这里所说的Python库是指那些被开发并且为了其他人来使用而发布的东西，你可以在
`PyPI <https://pypi.python.org/pypi>`_ 找到很多Python库。为了更好的推广和传播
自己，Python库会包含很多的信息，比如它的名字，版本号，依赖等等。而 ``setup.py``
就是用来提供这些信息的::

    from setuptools import setup
    
    setup(
        name="MyLibrary",
        version="1.0",
        install_requires=[
            "requests",
            "bcrypt",
        ],
        # ...
    )

这样很简单地声明了这个Python库的一些信息。但是，你并没有看到你可以从哪里获取这些依赖库。
这里并没有提供一个url或者一个文件系统来获取这些依赖，而只是告诉我们依赖是
``requests`` ， ``bcrypt``
，这很重要，我们可以称这种声明方式为“抽象的依赖”，它们只是以名字（或者加上了一个具体的版本号）
的方式来出现的。这就像我们说的“鸭子类型”，你不在乎它是什么样子的只要它看起来是 ``requests`` 。

Python应用
----------

这里所讲的Python应用是指你所要部署的一些东西，这是区别于我们之前所讲的Python库的。Python应用或许可以在
PyPI上找到，但是不像Python库，它们并不是一种可以被开发者使用多次的工具性的东西。PyPI上的Python应用经常会
在这个应用的旁边放置一个文件用来声明该应用部署的依赖。

一个应用经常会有很多依赖，或许会很复杂。这些依赖里很多没有一个名字，或者没有我们说所的那些信息。这便反映了 `pip <http://pip-installer.org/>`_
的requirements文件所做的事情了，一个典型的requirements文件看起来是这样的::

    # This is an implicit value, here for clarity
    --index https://pypi.python.org/simple/
    
    MyPackage==1.0
    requests==1.2.0
    bcrypt==1.0.2

这里每个依赖都标明了准确的版本号，一般一个Python库对依赖的版本比较宽松，而一个应用則会依赖比较具体的版本号。虽然也许跑其他
版本的 ``requests``
并不会出错，但是我们在本地测试顺利后，我们就会希望在线上也跑相同的版本。

文件的头部有一个 ``--index https://pypi.python.org/simple/``
，一般如果你不用声明这项，除非你使用的不是PyPI。然而它却是 ``requirements.txt`` 的一个重要部分，
这一行把一个抽象的依赖声明 ``requests==1.2.0`` 转变为一个具体的依赖声明 ``requests 1.2.0 from pypi.python.org/simple/``
，这不像“鸭子类型”，倒像一次 ``isinstance`` 检查。

那么，抽象和具体又有什么关系呢？
--------------------------------

读到这里你或许会说，OK, 我已经知道 ``setup.py``
是为可发行的Python库那些包准备的，而 ``requirements.txt``
是为那些不被经常作为工具利用的Python应用准备的，但是我已经把我的
``requirements.txt`` 读进来填充了我的 ``install_requires=[...]`` 啊（译者注：
比如你在 ``setup.py`` 中把 ``requirements.txt``
文件读取进来并切割成行列表，赋值给关键字
``install_requires``)，那我为何要在乎这个区别呢？

对于抽象依赖和具体依赖的区分是非常重要的，这点使我们的PyPI镜像源正常工作，这点允许我们可以在公司里搭建我们
私有的包索引服务，甚至这点允许了你去fork一个包并改造它。因为一个抽象的依赖只是一个名字和一个可选的版本号，你可以从
PyPi来安装它，或者从你自己的文件系统，你可以fork它并改造它，只要你指明了正确的名字和版本号你就可以一直这么使用下去。

一个极端点的情况是，你在该使用抽象依赖的地方使用了具体的依赖，这在Go语言中可以看到

.. code-block:: go

    import (
        "github.com/foo/bar"
    )

这里我们指明了一个具体的url。现在如果我以这种指明url的方式使用了这个库，而且现在我想要改造这个库（比如它缺失了我想要的某个功能，
或者有一个讨厌的bug)。我可能不仅仅需要fork ``bar``
这个库，还需要fork依赖这个库的其他库。(译者注：也就是说，想要替换一个底层依赖的话，需要改动依赖这个库的其他依赖对该库的依赖声明。)

Setuptools的dependency_links
----------------------------

Setuptools有一个功能叫做 `dependency_links <http://pythonhosted.org/setuptools/setuptools.html#dependencies-that-aren-t-in-pypi>`_ ::

    from setuptools import setup
    
    setup(
        # ...
        dependency_links = [
            "http://packages.example.com/snapshots/",
            "http://example2.com/p/bar-1.0.tar.gz",
        ],
    )

这一功能除去了依赖的抽象特性，直接把依赖的获取url标在了setup.py里。就像在Go语言中修改依赖包一样，我们只需要修改依赖链中每个包的 ``dependency_links`` 。

开发可复用的包与不重复自己
--------------------------

那么我们写依赖声明的时候需要在 ``setup.py`` 中写好抽象依赖，在 ``requirements.txt`` 中写好具体的依赖，但是我们并不想维护两份依赖文件，这样会让我们很难
做好同步。 ``requirements.txt`` 可以更好地处理这种情况，我们可以在有 ``setup.py`` 的目录里写下一个这样的 ``requirements.txt`` ::

    --index https://pypi.python.org/simple/
    
    -e .

这样 ``pip install -r requirements.txt`` 可以照常工作，它会先安装该文件路径下的包，然后继续开始解析抽象依赖，结合 ``--index`` 选项后转换为具体依赖然后再安装她们。

这个办法可以让我们解决一种类似这样的情形：比如你有两个或两个以上的包在一起开发但是是分开发行的，或者说你有一个尚未发布的包并把它分成了几个部分。如果你的顶层的包
依然仅仅按照“名字”来依赖的话，我们依然可以使用 ``requirements.txt`` 来安装开发版本的依赖包::

    --index https://pypi.python.org/simple/

    -e https://github.com/foo/bar.git#egg=bar
    -e .

这会首先从 https://github.com/foo/bar.git 来安装包 ``bar`` ， 然后进行到第二行 ``-e .`` ，开始安装 ``setup`` 中的抽象依赖，但是包 ``bar`` 已经安装过了，
所以 pip 会跳过安装，而是仍然使用github.com上安装了的开发版本的包 ``bar`` 。
