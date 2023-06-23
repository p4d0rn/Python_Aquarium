# Decorator

python中万物皆对象，包括函数也是一种对象，函数对象的特点是callable，后面加上小括号就能调用

```python
def double(x):
    return x * 2


def triple(x):
    return x * 3


def calc_num(func, x):
    print(func(x))


calc_num(double, 3)   # 6
calc_num(triple, 3)   # 9
def get_multiple_func(n):
    def multiple(x):
        return n * x

    return multiple


double = get_multiple_func(2)
triple = get_multiple_func(3)

print(double(3))  # 6
print(triple(3))  # 9
```

可以看到函数可以像对象一样作为实参被传递，有点类似C语言里的函数指针

装饰器（decorator）实际上是个形参为函数，返回值为函数的函数

就是个语法糖🍬

```python
def dec(f):
    pass


@dec
def double(x):
    return x * 2
@dec
def double(x):
    return x * 2

等价于 double = dec(double)
```

eg1：

```python
import time


def timeit(f):
    def wrapper(x):
        start = time.time()
        ret = f(x)
        print(time.time() - start)
        return ret

    return wrapper


@timeit
def my_func(x):
    time.sleep(x)  # my_func = timeit(my_func)


my_func(1)  # 1.009751319885254
```

有点类似AOP，这个装饰器的功能就是计算my_func这个函数的执行时间

eg2：带参数的装饰器

```python
import time


def timeit(iteration):

    def inner(f):
        def wrapper(*args, **kwargs):
            start = time.time()
            for _ in range(iteration):
                ret = f(*args, **kwargs)
            print(time.time() - start)
            return ret

        return wrapper

    return inner


@timeit(10)
def func(x):
    time.sleep(x)  # my_func = timeit(10)(my_func)


func(0.5) # 5.046420335769653
```

# @property

`@property` 是 Python 中的一个装饰器，它提供了一种方便的方式来访问类的实例属性，并允许在访问时执行额外的代码

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value <= 0:
            raise ValueError("Radius must be positive")
        self._radius = value

    @property
    def diameter(self):
        return 2 * self.radius

    @diameter.setter
    def diameter(self, value):
        self.radius = value / 2

    @property
    def area(self):
        return 3.14 * self.radius ** 2

    def __str__(self):
        return f"Circle({self.radius})"
```

我们定义了一个 `Circle` 类，它有一个 `radius` 属性表示圆的半径。我们通过定义 `radius` 的 `getter` 和 `setter` 方法来实现对属性的验证，确保只有正数的值可以被设置为半径。如果传入的值小于等于 0，将引发一个 `ValueError` 异常。这样，我们就可以确保 `Circle` 实例的半径始终是有效的。

```python
>>> circle = Circle(3)
>>> circle.radius
3
>>> circle.diameter
6
>>> circle.area
28.26

>>> circle.radius = -1
ValueError: Radius must be positive

>>> circle.diameter = 10
>>> circle.radius
5
>>> circle.area
78.5
```

在这个示例中，我们创建了一个 `Circle` 实例，并访问了它的 `radius`、`diameter` 和 `area` 属性。当我们尝试将半径设置为负数时，引发了一个 `ValueError` 异常。当我们将直径设置为 10 时，半径被正确设置为 5，并且我们可以使用 `area` 属性计算圆的面积。通过使用 `@property` 装饰器实现属性验证，我们可以确保 `Circle` 类的属性始终是有效的，并且减少了代码的重复性。

