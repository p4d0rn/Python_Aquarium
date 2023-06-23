dunder：double underscore 双下划线

Special Method in Python😋

## 0x00 Get Prepared

3 types of method in Python：

- 实例方法（没有装饰器修饰）
- 静态方法（`@staticmethod`）
- 类方法（`@classmethod`）

```python
class Test(object):
    method = 'test'

    def normMethod(self):
        print("Normal Method Called")

    @staticmethod
    def sMethod():
        print("Static Method Called")

    @classmethod
    def clsMethod(cls):
        print(f"{cls.method} for classMethod")

# Test.normMethod()  missing 1 required positional argument: 'self'
Test.sMethod()
Test.clsMethod()
test = Test()
test.normMethod()
test.clsMethod()
```

静态方法和类方法都可以通过类直接调用，不用实例化类。

### 实例方法

第一个参数self，表示类的实例化对象，只有类的实例才能调用实例方法

### 静态方法

类似普通函数。无法访问类或对象，可以将其作为辅助功能使用，且其不会绑定到实例对象上，内存中只生成一个，节省开销。静态方法放类外边也不影响，主要是放类里面给其一个作用域，方便管理。

### 类方法

第一个参数cls，表示类本身。应用场景：对类的初始化参数进行预处理

重构类的时候不需要修改构造函数，只需添加使用`@classmethod`修饰的处理函数

```python
import json


class Person(object):

    def __init__(self, name, age, hobby):
        self.name = name
        self.age = age
        self.gender = hobby

    @classmethod
    def class_method_create(cls, id_info):
        id_info = json.loads(id_info)
        person = cls(**id_info)   # cls()等于__init__()
        return person

    def __repr__(self):
        return f'name:{self.name}, age:{self.name}, gender:{self.gender}'


info = '{"name": "taco", "age": "18", "hobby": "paly"}'
p = Person.class_method_create(info)
print(p)
```

## 0x01 Basic

- `__new__`
- `__init__`
- `__del__`
- `__str__`
- `__repr__`
- `__format__`

```python
class A:
    def __new__(cls):
        print("__new__")
        return super().__new__(cls)

    def __init__(self):
        print("__init__")

o = A()
```

`__new__`：class -> object 的过程，返回一个对象

`__init__`：初始化 object 的过程，无返回值

可以理解为：

obj = __new__(A)

__init__(obj)

`__new__`的使用场景：singleton pattern

```python
class A:
    def __del__(self):
        print("__del__")

o = A()
x = o

del o

print("finish")  # 不打印__del__
```

`__del__`：对象被释放的时候触发

python中的关键字`del`能让对象的引用次数计数器-1，当引用数为0时，对象就会被释放

```python
class A:
    def __repr__(self):
        return "<A repr>"

    def __str__(self):
        return "<A str>"


print(A())  # <A str>
```

`__repr__`：详细版

`__str__`：简洁版 

```python
class A:
    def __format__(self, format_spec):
        if format_spec == 'x':
            return "0xA"
        return "<A>"


print(f"{A()}")
print(f"{A():x}")
```

`__format__`：自定义格式化字符串

```python
class A:
    def __bytes__(self):
        print("__bytes__ called")
        return bytes([0, 1])


print(bytes(A()))
```

## 0x02 Rich Comparison

比较两个类是否相等时，或没有定义`__eq__`，默认调用`is`去比较

```python
class Date:
    def __init__(self, year, month, date):
        self.year = year
        self.month = month
        self.date = date

    def __eq__(self, other):
        print("__eq__")
        return (self.year == other.year and
                self.month == other.month and
                self.date == other.date)


x = Date(2023, 2, 23)
y = Date(2023, 2, 23)
print(x == y)  # x.__eq__(y)
print(x != y)  # 默认会对__eq__的结果取反，也可以定义__ne__
>`：`__gt__`   `<`：`__lt__`    `>=`：`__ge__`     `<=`：`__le__
```

注意：若改变了`__eq__`方法，类原本默认的`__hash__`方法也会失效，需要重新定义

```python
class Date:
    def __init__(self, year, month, date):
        self.year = year
        self.month = month
        self.date = date

    def __eq__(self, other):
        print("__eq__")
        return (self.year == other.year and
                self.month == other.month and
                self.date == other.date)

    def __hash__(self):
        # 关键属性组成tuple传入hash
        return hash((self.year, self.month, self.date))


print(hash(Date(2023, 2, 23)))
```

类实例用作条件判断 `__bool__`

```python
class Date:
    def __init__(self, year, month, date):
        self.year = year
        self.month = month
        self.date = date

    def __bool__(self):
        print("__bool__")
        return True


x = Date(2023, 2, 23)
print(bool(x))
if x:
    print("Hello!")
```

## 0x03 Attribute

访问不存在的属性，调用`__getattr__`

```python
class A:
    def __init__(self):
        self.exist = "yes"

    def __getattr__(self, item):
        print(f"getting {item}")
        return None


o = A()
print(o.exist)
print(o.test)  # test以字符串形式传入
```

尝试访问对象属性，调用`__getattribute__`

```python
class A:
    def __init__(self):
        self.data = "666"
        self.count = 0

    def __getattribute__(self, item):
        if item == 'data':
            self.count += 1
        return super().__getattribute__(item)


o = A()
print(o.data)
print(o.count)  # 1
```

注意：上面执行`self.count`也会去调用`__getattr__`（`__getattr__`若属性存在会去调用`__getattr__`），注意不要陷入无限递归中

修改对象属性，调用`__setattr__`

```python
class A:
    def __init__(self):
        self.data = "666"
        self.count = 0

    def __setattr__(self, key, value):
        print(f"setting {key}")
        super().__setattr__(key, value)


o = A()
print(o.data)
##########################
setting data
setting count
666
```