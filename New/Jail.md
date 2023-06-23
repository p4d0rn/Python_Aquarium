# 0x01 Python沙箱逃逸

和SSTI有点重合，又高于SSTI

题目中一般是发送python代码给服务器，服务器通过eval或exec执行

当然服务器会对输入的代码进行过滤，或禁用一些敏感模块如builtins

刚开始我觉得这些东西绕来绕去没啥意义，后面才理解到这对实际中绕WAF和编写免杀马有一定帮助

下面参考/搬运：https://zhuanlan.zhihu.com/p/578966149

题目来自HNCTF

# 0x02 Python特性

* Python中所有类均继承自**object基类**

* 类本身具有一些静态方法，如`bytes.fromhex`、`int.from_bytes` ；
  类的实例也能调用这些静态方法，如`b'1'.fromhex('1234')`返回`b'\x124'`
* Python中的一些魔术方法
  * `__init__`：构造器，实例化类的时候会被调用
  * `__len__`：返回对象的长度，调用`len(a)`实质上就是调用`a.__len__()`
  * `__str__`：类似Java的`toString`或PHP的`__toString`，把对象当成字符串处理时就会被调用
    如`print(a)`，会调用`a.__str__()`
  * `__getitem__`：根据索引返回对象的某个元素，可以替换中括号
  * `__setitem__`：为索引下的元素赋值
  * `__add__`、`__sub__`、`__mul__`、`__div__`、`__mod__` ：算数运算
  * `__and__`，`__or__`、`__xor__`：逻辑运算
  * `__eq__`，`__ne__`、`__lt__`、`__gt__`、`__le__`、`__ge__`：比较运算
  * `__getattr__`：获取对象的属性，同`a.b`
  * `__getattr__`：设置对象的属性
  * `__subclasses__`：返回当前类的所有子类，由于python中所有类均继承于object基类，因此一般该方法用在object类中来获取其所有子类，在里面找os模块中的类，进而RCE
  * `__dict__`：查看类中的所有属性名和属性值（注：`dict()`是将对象转为字典）
  * `__doc__`：类的帮助文档
  * `__class__`：返回当前对象所属的类
    如`''.__class__`返回`<class 'str'>`
  * `__base__`：返回当前类的基类
    如`str.__base__`会返回`<class 'object'>`
* python中的一些内置函数和变量
  * `dir`：查看对象的所有属性和方法
    在题目禁用引号以及小数点时，也可以先用拿到类所有可用方法
    再索引到方法名，并且通过`getattr`来拿到目标方法。
  * `chr`、`ord`：字符与ASCII码转换，用于绕WAF
  * `globals`：返回所有全局变量的函数
  * `locals`：返回所有局部变量的函数；
  * `__import__`：载入模块的函数。
    `import os`等价于`os = __import__('os')`
  * `__name__`：该变量指示当前运行环境位于哪个模块中
    `if __name__ == '__main__':`，就是来判断是否是直接运行该脚本
    如果是从另外的地方import的该脚本的话，那`__name__`就不为`__main__`，就不会执行之后的代码
  * `__builtins__`：包含当前运行环境中默认的所有函数与类
    题目中往往`__builtins__`被置为`None`，一般从基类object的子类中获取函数
  * `__file__`：该变量指示当前运行代码所在路径
    类似于PHP中的`highlight(__FILE__)`
    `open(__file__).read()`读取当前运行的python文件代码
  * `_`：该变量返回上一次运行的python语句结果。
    **该变量仅在运行交互式终端时会产生，在运行代码文件时不会有此变量**。

# 0x03 思路

* 优先考虑RCE
  `os.system('sh')`进交互式终端（`subprocess.popen()`也行）
  对于SSTI，没有交互式终端，`os.popen('ls').read()`（若可以出网直接弹shell）
* 如何RCE？
  1. `object.__subclasses__()`中找到`os`模块中的类（一般是`<class 'os._wrap_close'>`）
  2. 先拿到`__builtins__`，再`__import__('os').system('sh')`
* 类型转换：此处主要是`bytes`和`str`的类型转换。首先`bytes`可以通过可迭代的对象，如`tuple`和`list`来初始化，如`bytes([99, 108, 101, 97, 114, 108, 111, 118, 101, 55])`为`b'clearlove7'`；然后再通过`decode`方法转为`str`；
* 通过`chr`、`getattr`、`dir`来绕WAF