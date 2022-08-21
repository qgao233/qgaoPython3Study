# 函数

## 1 定义函数

```
def fib(n):
    """Print a Fibonacci series up to n."""
```

函数内的第一条语句是字符串时，该字符串就是文档字符串。

没有 return 语句的函数查看返回值，可使用 `print()` ，打印值为 `None` （是一个内置名称）。

## 2 定义详解

函数定义支持**可变数量**的参数。

### 2.1 默认值参数

定义函数，一个必选形参，两个可选形参：

```
def ask_ok(prompt, retries=4, reminder='Please try again!'):
    while True:
        ok = input(prompt)
        if ok in ('y', 'ye', 'yes'):
            return True
        if ok in ('n', 'no', 'nop', 'nope'):
            return False
        retries = retries - 1
        if retries < 0:
            raise ValueError('invalid user response')
        print(reminder)
```

调用函数：

* 只给出必选实参：ask_ok('Do you really want to quit?')

* 给出一个可选实参：ask_ok('OK to overwrite the file?', 2)

* 给出所有实参：ask_ok('OK to overwrite the file?', 2, 'Come on, only yes or no!')

**注意：**

默认值只计算一次。

```
i = 5

def f(arg=i):
    print(arg)

i = 6
f()     # 5
```

默认值为列表、字典或类实例等可变对象时，会产生与该规则不同的结果。例如，下面的函数会累积后续调用时传递的参数：

```
def f(a, L=[]):
    L.append(a)
    return L

print(f(1))     # [1]
print(f(2))     # [1, 2]
print(f(3))     # [1, 2, 3]
```

不想在后续调用之间共享默认值时，应以如下方式编写函数：

```
def f(a, L=None):
    if L is None:
        L = []
    L.append(a)
    return L
```

### 2.2 关键字参数

调用函数的另一种方法称为**关键字参数**：`f(key=value)`

>关键字形参也叫作**命名形参**。

```
def parrot(voltage, state='a stiff', action='voom', type='Norwegian Blue'):
    print("-- This parrot wouldn't", action, end=' ')
    print("if you put", voltage, "volts through it.")
    print("-- Lovely plumage, the", type)
    print("-- It's", state, "!")

# 如此调用：

parrot(1000)                                          # 1 positional argument
parrot(voltage=1000)                                  # 1 keyword argument
parrot(voltage=1000000, action='VOOOOOM')             # 2 keyword arguments
parrot(action='VOOOOOM', voltage=1000000)             # 2 keyword arguments
parrot('a million', 'bereft of life', 'jump')         # 3 positional arguments
parrot('a thousand', state='pushing up the daisies')  # 1 positional, 1 keyword
```

下面这些调用方式**无效**：

```
parrot()                     # required argument missing
parrot(voltage=5.0, 'dead')  # non-keyword argument after a keyword argument
parrot(110, voltage=220)     # duplicate value for the same argument
parrot(actor='John Cleese')  # unknown keyword argument
```

**总结**

* 只要必选实参**用了**关键字参数，后面的可选参数**必须**也用关键字参数。
* 但如果必选实参**没用**关键字参数，后面可选参数可以**选择性**使用关键字参数，
    * 但如果后面可选参数**不使用**关键字参数，则是按照**位置顺序**传参。

#### 2.2.1 特殊形参

最后一个形参为 `**name` 形式时，接收一个**字典**（详见 映射类型 --- dict），该字典包含与函数中**已定义形参**对应**之外**的**所有关键字参数**。

`**name` 形参可以与 `*name` 形参（下下小节介绍）组合使用（*name 必须在 **name 前面）， 

`*name` 形参接收一个 **元组**，该元组包含**形参列表之外**的**位置参数**。

例如，可以定义下面这样的函数：

```
def cheeseshop(kind, *arguments, **keywords):
    print("-- Do you have any", kind, "?")
    print("-- I'm sorry, we're all out of", kind)
    for arg in arguments:
        print(arg)
    print("-" * 40)
    for kw in keywords:
        print(kw, ":", keywords[kw])
```

调用该函数：

```
cheeseshop("Limburger", "It's very runny, sir.",
           "It's really very, VERY runny, sir.",
           shopkeeper="Michael Palin",
           client="John Cleese",
           sketch="Cheese Shop Sketch")

# 输出结果如下：

-- Do you have any Limburger ?
-- I'm sorry, we're all out of Limburger
It's very runny, sir.
It's really very, VERY runny, sir.
----------------------------------------
shopkeeper : Michael Palin
client : John Cleese
sketch : Cheese Shop Sketch
```

>注意，关键字参数在输出结果中的顺序与调用函数时的顺序一致。

### 2.3 特殊参数

为了让代码易读、高效，通过在函数定义添加 `/` 和 `*` 限制参数的传递方式。

>`/` 和 `*`是可选的。

```
def f(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2):
      -----------    ----------     ----------
        |                |               |
       位置         位置或关键字        关键字
```

例子：

```
def standard_arg(arg):     # 无限制
    print(arg)

def pos_only_arg(arg, /):  # 仅限位置参数
    print(arg)

def kwd_only_arg(*, arg):  # 仅限关键字参数
    print(arg)

def combined_example(arg, /, pos_or_kwd):   # / 后可以是 位置或关键字 或 仅限关键字 形参。
    print(pos_only, pos_or_kwd)

def combined_example(pos_or_kwd, *, kwd_only): # * 前可以是 位置或关键字 或 仅限位置 形参。
    print(pos_or_kwd, kwd_only)

def combined_example(pos_only, /, standard, *, kwd_only):
    print(pos_only, standard, kwd_only)
```

#### 2.3.1 特殊使用

```
# kwds 把 name 当作键，因此，可能与位置参数 name 产生潜在冲突：
def foo(name, **kwds):
    return 'name' in kwds

foo(1, **{'name': 2})  # 报错，不可能返回 True，因为关键字 'name' 总与第一个形参绑定，即'name' = 1, kwds中'name' = 2，1!=2
```

```
# 加上 / （仅限位置参数），此时，函数定义把 name 当作位置参数，'name' 也可以作为关键字参数的键：
def foo(name, /, **kwds):
    return 'name' in kwds

foo(1, **{'name': 2})  # true   判断为在kds是否存在有'name'这个键了。
```

换句话说，函数定义中的`name`是个形参，而return后面的`name`是个字符串，两者已经没有绑定在一起，因此字符串可以在 `**kwds` 中使用，而不产生歧义。

### 2.4 任意实参列表

`*args` 形参后的任何形式参数只能是仅限关键字参数，

```
def concat(*args, sep="/"):
    return sep.join(args)

concat("earth", "mars", "venus")           # 'earth/mars/venus'
concat("earth", "mars", "venus", sep=".")  # 'earth.mars.venus'
```

### 2.5 解包实参列表

*：解包列表

```
list(range(3, 6))    # [3, 4, 5]
args = [3, 6]
list(range(*args))   # [3, 4, 5]  用 * 操作符把实参从列表或元组解包成两个独立的 start 和 stop 实参：
```

**：解包字典

>即json或map格式的数据

```
def parrot(voltage, state='a stiff', action='voom'):
    print("-- This parrot wouldn't", action, end=' ')
    print("if you put", voltage, "volts through it.", end=' ')
    print("E's", state, "!")

d = {"voltage": "four million", "state": "bleedin' demised", "action": "VOOM"}
parrot(**d)
# This parrot wouldn't VOOM if you put four million volts through it. E's bleedin' demised !
```

### 2.6 Lambda 表达式

在语法上，匿名函数只能是单个表达式。

用 lambda 表达式返回函数：

```
def make_incrementor(n):
    return lambda x: x + n

f = make_incrementor(42)
f(0)   # 42
f(1)   # 43
```

匿名函数用作传递的实参：

```
pairs = [(1, 'one'), (2, 'two'), (3, 'three'), (4, 'four')]
pairs.sort(key=lambda pair: pair[1])
pairs  # [(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]
```

## 3 编码约定

* 缩进，用 4 个空格
* 用空行分隔函数和类，及函数内较大的代码块。
* 最好把注释放到单独一行。
* 使用文档字符串。
* 命名类用 UpperCamelCase，
* 命名函数与方法用 lowercase_with_underscores。
* 命名方法中第一个参数总是用 self