关于
----

利用readthedocs的Python技术文章的翻译和收集。

订阅
----

RSS: https://pyzh.readthedocs.org/en/latest/rss.xml

Online: https://pyzh.readthedocs.org/en/latest/

约定
----

- 文件名必须是英文。一个文章的多个文件如下命名::

    xxxx-Part1.rst, xxxx-Part2.rst ..

- 文第一行注明日期::

    :Date: 2013-04-15 22:00:00

- 翻译的文章，需要注明原文链接

如何开始
--------

1. clone 这个Repo

  ::

      git clone git@github.com:HIT-ON-Github/PyZh.git

2. 初始化环境

  ::

      cd PyZh
      virtualenv venv
      pip install -r requirements.pip

3. 编写文章

  ::

      vim docs/xxxxx.rst

4. 编译预览

  ::

      cd docs
      make html
      python -m SimpleHTTPServer

  打开 ``http://localhost:8000`` 预览
