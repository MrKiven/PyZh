:Date: 2018-05-07 21:47:01

======================= 
(译)谨慎使用format格式化
======================= 

:作者: mengxun

:联系: <mengxun1122@163.com>

:翻译: mengxun

:译者注:  原文链接- http://lucumr.pocoo.org/2016/12/29/careful-with-str-format/

.. Contents::

前言
----

这个问题我本来应该早点发现的，但是直到今天早些时候我才真正地意识到str.format这个语法一旦被恶意用户利用将会有多么严重的后果。这个语法会被当作绕过Jinja2的Sandbox检测的一种手段，通过这种手段你可以拿到你本来无权拿到的信息，所以我为这个语法写了一个更为安全的版本。
因为大家几乎没有意识到这个问题有多么容易被利用，所以我认为这个普遍问题还是相当严重的，并且我们需要探讨一下。

关键问题
--------

受.NET的启发，从Python2.6起新的字符串格式化语法被引入，在Rust和一些其他语言里也有相同的语法。这种语法被用于byte和unicode字符串（Python3只有unicode字符串）后面的.format方法中，它也可以拷贝到颗粒度更细的string.Formatter API里。
这种语法有个特点就是你可以在里面用位置参数和关键字参数，并且你可以对元素重新排序。但更厉害的特点是你可以访问对象的属性和其他组成部分，这就是造成不安全的原因。
你可以做下面的操作::

	>>> 'class of {0} is {0.__class__}'.format(42)
	"class of 42 is <class 'int'>"

从原则上来说：控制格式化字符串的人可以访问对象隐藏的内部属性

问题出现的场景
-------------

首先一个问题是为什么会有人控制字符串的格式化，这是几个场景：
	
	- 能接触到字符串文件的不可信翻译者。 这是一个很大的问题，因为许许多多的应用程序都会借助新式的格式化语法来翻译成多种语言。但是并非所有人都会检查传进来的字符。
	- 公开的配置项。一些系统用户可能被允许配置某些行为，它们有可能被配置成格式化字符串。特别是我见过用户可以在web应用中配置用于接受通知的邮箱，日志信息的格式和其他基本的模板。

危险程度
--------

如果只是基本的数据结构被传进来那倒还无妨，因为最糟的情况也就是，你可以发现一些比如上面的数字是个整数类型的这种内部信息。
然而一旦Python对象被传进来那就糟了，因为能从Python方法里暴露出来的东西实在是多的吓人。下面是假想的一个Web应用的例子，在这个例子中，密钥将会泄露::

CONFIG = {
    'SECRET_KEY': 'super secret key'
}

class Event(object):
    def __init__(self, id, level, message):
        self.id = id
        self.level = level
        self.message = message

def format_event(format_string, event):
    return format_string.format(event=event)

用户将拿到密钥，如果他们在这里传进来一个像这样的格式化字符串::

{event.__init__.__globals__[CONFIG][SECRET_KEY]}

译者注：读者可以这样做下测验，看看会返回什么::

event = Event(1,2,'test')
format_event('{event.__init__.__globals__[CONFIG][SECRET_KEY]}',event)


---------------

那么如果你确实需要一些人提供格式化字符串，那么你该怎么做呢？你可以使用一些内部方法去改变这种行为::

from string import Formatter
from collections import Mapping

class MagicFormatMapping(Mapping):
    """This class implements a dummy wrapper to fix a bug in the Python
    standard library for string formatting.

    See http://bugs.python.org/issue13598 for information about why
    this is necessary.
    """

    def __init__(self, args, kwargs):
        self._args = args
        self._kwargs = kwargs
        self._last_index = 0

    def __getitem__(self, key):
        if key == '':
            idx = self._last_index
            self._last_index += 1
            try:
                return self._args[idx]
            except LookupError:
                pass
            key = str(idx)
        return self._kwargs[key]

    def __iter__(self):
        return iter(self._kwargs)

    def __len__(self):
        return len(self._kwargs)

# This is a necessary API but it's undocumented and moved around
# between Python releases
try:
    from _string import formatter_field_name_split
except ImportError:
    formatter_field_name_split = lambda \
        x: x._formatter_field_name_split()

class SafeFormatter(Formatter):

    def get_field(self, field_name, args, kwargs):
        first, rest = formatter_field_name_split(field_name)
        obj = self.get_value(first, args, kwargs)
        for is_attr, i in rest:
            if is_attr:
                obj = safe_getattr(obj, i)
            else:
                obj = obj[i]
        return obj, first

def safe_getattr(obj, attr):
    # Expand the logic here.  For instance on 2.x you will also need
    # to disallow func_globals, on 3.x you will also need to hide
    # things like cr_frame and others.  So ideally have a list of
    # objects that are entirely unsafe to access.
    if attr[:1] == '_':
        raise AttributeError(attr)
    return getattr(obj, attr)

def safe_format(_string, *args, **kwargs):
    formatter = SafeFormatter()
    kwargs = MagicFormatMapping(args, kwargs)
    return formatter.vformat(_string, args, kwargs)

你现在可以使用safe_format去代替str.format了::

>>> '{0.__class__}'.format(42)
"<type 'int'>"
>>> safe_format('{0.__class__}', 42)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: __class__