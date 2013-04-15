About
-----

Python技术文章的收集和翻译,资源可以来自外文python技术博客或者Stackoverflow或者自己编写.

约定和规范:
----------

- 文件名应是文章的英文名字。如果需要多个rst文件完成这个文章，在文件名后加上part1,part2.  e.g. : ``Python-Guide-Part1.rst``
- 翻译文章的文章标题应表明翻译。翻译文章应标明原文链接。
- 文章需要用到外部的代码文件，代码全部放在 ``docs/code/你的文章名字/`` 目录下
- TODO/authorname.rst来建立你的TODO文件，文件怎么写自己随意安排~

How-To
------

1. 在docs/下新建一个rst文件，然后在index.rst中追加上这个文件名(不含扩展名rst)

2. 开始写rst文章,  可以在docs/_build/html目录下开个服务器来预览

::

    python -m SimpleHTTPServer

3. ``make html`` 来生成html

4. 编辑好后，push到Github,  然后到 https://readthedocs.org/projects/pyzh/
   build使readthedocs上的文章同步到最新。

Requirements
------------

1. reStructureText语法. 一个 cheatSheet_ 
   
.. _cheatsheet: https://github.com/ralsina/rst-cheatsheet/blob/master/rst-cheatsheet.rst

2. Sphinx_ 

.. _Sphinx: http://sphinx-doc.org/
