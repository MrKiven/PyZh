:Date: 2013-05-14 21:21:00

============================
使用Python解释器的一句话命令
============================

:作者: hit9

:联系: <nz2324 at 126 dot com>

全部是 ``python -m``  形式的

.. code:: bash


    python -m SimpleHTTPServer [port]  # 当前目录开启一个小的文件服务器, 默认端口8000
    # 另外，python 3,中是 python -m http.server
    python -m this               # python's Zen
    python -m calendar           # 显示一个日历
    echo '{"json":"obj"}' | python -mjson.tool  # 漂亮地格式化打印json数据
    echo '{"json":"obj"}' | python -mjson.tool | pygmentize -l json # 高亮地打印json格式化
    python -m antigravity       # 这个该自己试试
    python -m smtpd -n -c DebuggingServer localhost:20025  # mail server
