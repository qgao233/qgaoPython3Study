# 面向对象

## 1 Python 命名空间和作用域

### 1.1 命名空间

namespace （命名空间）是映射到对象的名称。

>现在，大多数命名空间都使用 Python 字典实现，

命名空间的几个常见示例： 

* abs() 函数、内置异常等的内置函数集合；
* 模块中的全局名称；
* 函数调用中的局部名称；
* 对象的属性集合。

不同命名空间中的名称之间**绝对没有关系**；

例如，两个不同的模块都可以定义 maximize 函数，且不会造成混淆。用户使用函数时必须要在函数名前面附加上模块名。

表达式 `modname.funcname` 中，`modname` 是模块对象，`funcname` 是模块属性。

**模块属性**和模块中定义的**全局名称**之间存在直接的**映射**：它们共享相同的命名空间！

属性可以是只读或者可写的。

如果可写，则可对属性赋值。如使用 `modname.the_answer = 42` 。而`del` 语句可以删除可写属性。

>例如， del modname.the_answer 会删除 modname 对象中的 the_answer 属性。

命名空间是在不同时刻创建的，且拥有不同的生命周期。

* 内置名称的命名空间是在 Python 解释器启动时创建的，永远不会被删除。

* 模块的**全局命名空间**在读取模块定义时创建；通常，模块的命名空间也会持续到解释器退出。从脚本文件读取或交互式读取的，由解释器顶层调用执行的语句是 `__main__` 模块调用的一部分，也拥有自己的全局命名空间。内置名称实际上也在模块里，即 builtins 。

* 函数的**本地命名空间**在调用该函数时创建，并在函数返回或抛出不在函数内部处理的错误时被删除。 （实际上，用“遗忘”来描述实际发生的情况会更好一些。） 当然，每次递归调用都会有自己的本地命名空间。

### 1.2 作用域

作用域 是在命名空间中可直接访问的 Python 程序的文本区域。 “可直接访问” 的意思是，对名称的非限定引用会在命名空间中查找名称。

作用域虽然是静态确定的，但会被动态使用。执行期间的任何时刻，都会有 3 或 4 个命名空间可被直接访问的嵌套作用域：

* 最内层作用域，包含局部名称，并首先在其中进行搜索

* 封闭函数的作用域，包含非局部名称和非全局名称，从最近的封闭作用域开始搜索

* 倒数第二个作用域，包含当前模块的全局名称

* 最外层的作用域，包含内置名称的命名空间，最后搜索

**绑定变量到不同作用域：**

* 使用 `global` 语句把该变量声明为**全局变量**，则所有引用和赋值将直接指向包含该模块的全局名称的中间作用域。

* 使用 `nonlocal` 语句把该变量声明为**非局部变量**，重新绑定在最内层作用域以外找到的变量。

>未声明为非局部变量的变量是只读的，（写入只读变量会在最内层作用域中创建一个 新的 局部变量，而同名的外部变量保持不变。）

通常，

* 当前局部作用域将（按字面文本）引用当前函数的局部名称作为**局部命名空间**。
* 在函数之外，局部作用域引用与全局作用域一致的命名空间：**模块的命名空间**。 

>类定义会在局部命名空间内再放置另一个命名空间。

划重点，作用域是按字面文本确定的：无论该函数从什么地方或以什么别名被调用，模块内定义的函数的全局作用域就是该模块的命名空间。另一方面，实际的名称搜索是在运行时动态完成的。但是，Python 正在朝着“编译时静态名称解析”的方向发展，因此不要过于依赖动态名称解析！（局部变量已经是被静态确定了。）

Python 有一个特殊规定。如果不存在生效的 global 或 nonlocal 语句，则对名称的赋值总是会进入最内层作用域。赋值不会复制数据，只是将名称绑定到对象。删除也是如此：语句 `del x` 从局部作用域引用的命名空间中移除对 x 的绑定。所有引入新名称的操作都是使用局部作用域：尤其是 import 语句和函数定义会在局部作用域中绑定模块或函数名称。

* global 语句用于表明特定变量在全局作用域里，并应在全局作用域中重新绑定；
* nonlocal 语句表明特定变量在外层作用域中，并应在外层作用域中重新绑定。

### 1.3 作用域和命名空间示例

```
def scope_test():
    def do_local():
        spam = "local spam"

    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"

    def do_global():
        global spam
        spam = "global spam"

    spam = "test spam"
    do_local()                                  # 不会改变 scope_test 对 spam 的绑定
    print("After local assignment:", spam)      # test spam
    do_nonlocal()                               # 改变 scope_test 对 spam 的绑定
    print("After nonlocal assignment:", spam)   # nonlocal spam
    do_global()                                 # 改变模块层级的绑定，而且，global 赋值前没有 spam 的绑定。
    print("After global assignment:", spam)     # nonlocal spam

scope_test()
print("In global scope:", spam)                 # global spam
```

## 2 类

### 2.1 类定义

```
class ClassName:
    ...
```

当进入类定义时，将创建一个新的命名空间，并将其用作局部作用域 --- 因此，所有对局部变量的赋值都是在这个新命名空间之内。 特别的，函数定义会绑定到这里的新函数名称。

当（从结尾处）正常离开类定义时，将创建一个 类对象。 这基本上是一个包围在类定义所创建命名空间内容周围的包装器；离开范围后，原始的（在进入类定义之前起作用的）局部作用域将重新生效，类对象将在这里被绑定到类定义头所给出的类名称 (在这个示例中为 ClassName)。

### 2.2 类对象

注意：类对象不是实例！

类对象支持两种操作：

* 属性引用：类的属性引用参考java的静态属性的引用
* 实例化

#### 2.2.1 属性引用

有效的属性名称是类对象被创建时存在于类命名空间中的所有名称。 因此，如果类定义是这样的:

```
class MyClass:
    """A simple example class"""
    i = 12345

    def f(self):
        return 'hello world'
```

那么 `MyClass.i` 和 `MyClass.f` 就是有效的属性引用，将分别返回一个整数和一个函数对象。 

类属性也可以被赋值，因此可以通过赋值来更改 `MyClass.i` 的值。

`__doc__` 也是一个有效的属性，`MyClass.__doc__` 将返回所属类的文档字符串: "A simple example class"。

#### 2.2.2 实例化

```
x = MyClass()
```

>实例化时会去调用构造方法。

许多类喜欢创建带有特定初始状态的自定义实例。 为此类定义可能包含一个名为 `__init__()` 的特殊方法（构造方法），就像这样:

```
def __init__(self):
    self.data = []
```

带参数的构造方法：

```
>>> class Complex:
...     def __init__(self, realpart, imagpart):
...         self.r = realpart
...         self.i = imagpart
...
>>> x = Complex(3.0, -4.5)
>>> x.r, x.i
(3.0, -4.5)
```

### 2.3 实例对象

对象的属性引用有两种有效的属性名称：

* 数据属性：实例变量
* 方法

**数据属性**不需要声明；像局部变量一样，它们将在**第一次被赋值时产生**。 例如，如果 x 是上面创建的 MyClass 的实例，则以下代码段将打印数值 16，且不保留任何追踪信息:

```
x.counter = 1
while x.counter < 10:
    x.counter = x.counter * 2
print(x.counter)
del x.counter
```

**方法：**

实例对象的有效方法名称依赖于其所属的类。 根据定义，一个类中所有是函数对象的属性都是定义了其实例的相应方法。

>意思就是，在从类对象的角度看是函数的定义，而从实例对象的角度看是可以被实例调用的方法。

举例：

因为 MyClass.f 是一个函数，x.f 是有效的方法引用。
因为 MyClass.i 不是函数，而 x.i 不是方法。

但是 x.f 与 MyClass.f 并不是一回事 --- x.f是一个 方法对象，不是函数对象。

>我服了，贼简单的定义，为什么官网能说得那么绕。看到这里的小朋友，如果不信，可以去官网瞅瞅所有上面的这一段，简直坑爹...
>而你们现在看到的都是我中文转中文又翻译了一遍的东西。

### 2.4 方法对象

通常，方法在绑定后可以如此被调用:

```
x.f()
```

在 MyClass 示例中，这将返回字符串 'hello world'。 但是，x.f 是一个方法对象，它可以**被保存**起来以后再调用。 例如:

```
xf = x.f
while True:
    print(xf())
```

上面调用 `x.f()` 时并没有带参数，而原`f`的定义为`def f(self)`，其实，调用 `x.f()` 其实就相当于 `MyClass.f(x)`。

方法的特殊之处就在于**实例对象**会作为函数的**第一个参数**被传入。

总之，调用一个具有 n 个参数的方法就相当于调用再多一个参数的对应函数，这个参数值为方法所属实例对象，位置在其他参数之前。

### 2.5 类和实例变量

```
class Dog:

    kind = 'canine'         # 被所有实例共享的类变量

    def __init__(self, name):
        self.name = name    # 实例变量

>>> d = Dog('Fido')
>>> e = Dog('Buddy')
>>> d.kind                  # shared by all dogs
'canine'
>>> e.kind                  # shared by all dogs
'canine'
>>> d.name                  # unique to d
'Fido'
>>> e.name                  # unique to e
'Buddy'
```

实际上，在 Python 中没有任何东西能强制隐藏数据（像java的private） --- 它是完全基于约定的。

每个值都是一个对象，因此具有 类 （也称为 类型），并存储为 `object.__class__` 。

## 3 继承

派生类定义的语法如下所示:

```
class DerivedClassName(BaseClassName):
    ...
```

名称 BaseClassName 必须定义于包含派生类定义的作用域中。 也允许用其他任意表达式代替基类名称所在的位置。 

例如，当基类定义在另一个模块中的时候:

```
class DerivedClassName(modname.BaseClassName):
```

如果调用同一基类中定义的另一方法，因为派生类重写了该方法，则会调用重写后的方法。

如果想直接调用基类中实现的方法，则可以这样调用 `BaseClassName.methodname(self, arguments)`。 有时这对客户端来说也是有用的。 （请注意仅当此基类可在全局作用域中以 `BaseClassName` 的名称被访问时方可使用此方式。）

Python有两个内置函数可被用于检查继承机制：

* `isinstance()` ：检查一个实例的类型: `isinstance(obj, int)` 仅会在 `obj.__class__` 为 int 或某个派生自 int 的类时为 True。

* `issubclass()` ：检查类的继承关系: `issubclass(bool, int)` 为 True，因为 bool 是 int 的子类。 但是，issubclass(float, int) 为 False，因为 float 不是 int 的子类。

### 3.1 多重继承

带有多个基类的类定义语句如下所示:

```
class DerivedClassName(Base1, Base2, Base3):
```

在最简单的情况下，你可以认为搜索从父类所继承属性的操作是**深度优先、从左至右**的，当层次结构中存在重叠时不会在同一个类中搜索两次。 因此，如果某一属性在 DerivedClassName 中未找到，则会到 Base1 中搜索它，然后（递归地）到 Base1 的基类中搜索，如果在那里未找到，再到 Base2 中搜索，依此类推。

实际情况，动态改变顺序是有必要的，因为所有多重继承的情况都会显示出一个或更多的菱形关联（即至少有一个父类可通过多条路径被最底层类所访问）。

## 4 私有变量

由于存在对于类私有成员的有效使用场景（例如避免名称与子类所定义的名称相冲突），python有个机制：

* 名称改写：任何形式为 `__spam` 的标识符（至少带有两个前缀下划线，至多一个后缀下划线）的文本将被替换为 `_classname__spam`，其中 `classname` 为去除了前缀下划线的当前类名称。

>这种改写不考虑标识符的句法位置，只要它出现在类定义内部就会进行。

```
class Mapping:
    def __init__(self, iterable):
        self.items_list = []
        self.__update(iterable)

    def update(self, iterable):
        for item in iterable:
            self.items_list.append(item)

    __update = update   # __update被改写为_Mapping__update

class MappingSubclass(Mapping):

    __update = 0        # __update被改写为_MappingSubclass__update

    def update(self, keys, values):
        # provides new signature for update()
        # but does not break __init__()
        for item in zip(keys, values):
            self.items_list.append(item)
```

请注意传递给 exec() 或 eval() 的代码不会将发起调用类的类名视作当前类；这类似于 global 语句的效果，因此这种效果仅限于同时经过字节码编译的代码。 同样的限制也适用于 getattr(), setattr() 和 delattr()，以及对于 `__dict__` 的直接引用。

## 5 看不懂的补充系列

有时会需要使用类似于 Pascal 的“record”或 C 的“struct”这样的数据类型，将一些命名数据项捆绑在一起。 这种情况适合定义一个空类:

```
class Employee:
    pass

john = Employee()  # Create an empty employee record

# Fill the fields of the record
john.name = 'John Doe'
john.dept = 'computer lab'
john.salary = 1000
```

一段需要特定抽象数据类型的 Python 代码往往可以被传入一个模拟了该数据类型的方法的类作为替代。 例如，如果你有一个基于文件对象来格式化某些数据的函数，你可以定义一个带有 read() 和 readline() 方法从字符串缓存获取数据的类，并将其作为参数传入。

实例方法对象也具有属性: `m.__self__` 就是带有 m() 方法的实例对象，而 `m.__func__` 则是该方法所对应的函数对象。

## 6 迭代器

到目前为止，您可能已经注意到大多数容器对象都可以使用 for 语句:

```
for element in [1, 2, 3]:
    print(element)
for element in (1, 2, 3):
    print(element)
for key in {'one':1, 'two':2}:
    print(key)
for char in "123":
    print(char)
for line in open("myfile.txt"):
    print(line, end='')
```

**迭代原理：**

在幕后，for 语句会在容器对象上调用 iter()。 该函数返回一个定义了 `__next__()` 方法的迭代器对象，此方法将逐一访问容器中的元素。 当元素用尽时，`__next__()` 将引发 `StopIteration` 异常来通知终止 for 循环。 你可以使用 `next()` 内置函数来调用 `__next__()` 方法；这个例子显示了它的运作方式:

```
>>> s = 'abc'
>>> it = iter(s)
>>> it
<str_iterator object at 0x10c90e650>
>>> next(it)
'a'
>>> next(it)
'b'
>>> next(it)
'c'
>>> next(it)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
    next(it)
StopIteration
```

### 6.1 自定义迭代器

看过迭代器协议的幕后机制，给你的类添加迭代器行为就很容易了。 定义一个 `__iter__()` 方法来返回一个带有 `__next__()` 方法的对象。 如果类已定义了 `__next__()`，则 `__iter__()` 可以简单地返回 `self`:

```
class Reverse:
    """Iterator for looping over a sequence backwards."""
    def __init__(self, data):
        self.data = data
        self.index = len(data)

    def __iter__(self):
        return self

    def __next__(self):
        if self.index == 0:
            raise StopIteration
        self.index = self.index - 1
        return self.data[self.index]
>>>
>>> rev = Reverse('spam')
>>> iter(rev)
<__main__.Reverse object at 0x00A1DB50>
>>> for char in rev:
...     print(char)
...
m
a
p
s
```

### 6.2 生成器

一个用于创建迭代器的简单而强大的工具。

它们的写法类似于**标准的函数**，但当它们要返回数据时会使用 `yield` 语句。 每次在生成器上调用 `next()` 时，它会从上次离开的位置恢复执行（它会记住上次执行语句时的所有数据值）。 

一个显示如何非常容易地创建生成器的示例如下:

```
def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index]
>>>
>>> for char in reverse('golf'):
...     print(char)
...
f
l
o
g
```

可以用生成器来完成的操作同样可以用前一节所描述的基于类的迭代器来完成。 但生成器的写法更为紧凑，因为它会自动创建 `__iter__()` 和 `__next__()` 方法。

另一个关键特性在于局部变量和执行状态会在每次调用之间自动保存。 这使得该函数相比使用 `self.index` 和 `self.data` 这种实例变量的方式更易编写且更为清晰。

除了会自动创建方法和保存程序状态，当生成器终结时，它们还会自动引发 `StopIteration`。 这些特性结合在一起，使得创建迭代器能与编写常规函数一样容易。

### 6.3 生成器表达式

某些简单的生成器可以写成简洁的表达式代码，所用语法类似列表推导式，但外层为圆括号而非方括号。 这种表达式被设计用于生成器将立即被外层函数所使用的情况。 生成器表达式相比完整的生成器更紧凑但较不灵活，相比等效的列表推导式则更为节省内存。

示例:

```
>>>
>>> sum(i*i for i in range(10))                 # sum of squares
285

>>> xvec = [10, 20, 30]
>>> yvec = [7, 5, 3]
>>> sum(x*y for x,y in zip(xvec, yvec))         # dot product
260

>>> unique_words = set(word for line in page  for word in line.split())

>>> valedictorian = max((student.gpa, student.name) for student in graduates)

>>> data = 'golf'
>>> list(data[i] for i in range(len(data)-1, -1, -1))
['f', 'l', 'o', 'g']
```