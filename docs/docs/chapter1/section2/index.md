# 流程控制

## 1 if

```
if x < 0:
    x = 0
    print('Negative changed to zero')
elif x == 0:
    print('Zero')
elif x == 1:
    print('Single')
else:
    print('More')
```

## 2 for

```
# 创建集合
users = {'Hans': 'active', 'Éléonore': 'inactive', '景太郎': 'active'}

# 在新复制的集合上遍历
for user, status in users.copy().items():
    if status == 'inactive':
        del users[user]

# 创建一个新集合来存储
active_users = {}
for user, status in users.items():
    if status == 'active':
        active_users[user] = status
```

### 2.1 for、while的else

`for` 循环中，可迭代对象中的元素全部循环完毕，或 `while` 循环的条件为假时，执行同一级的`else`。

```
for n in range(2, 10):
    for x in range(2, n):
        if n % x == 0:
            print(n, 'equals', x, '*', n//x)
            break
    else:
        print(n, 'is a prime number')
```

## 3 range()

可以生成算术序列（整数），返回的是可迭代对象（`iterable`）。

* range(num)：return [0, num)
* range(begin, end)：return [begin, end)
* range(begin, end, step)：return [begin, end) 步长为step。

```
# list()传入可迭代对象，生成一个列表
list(range(-10, -100, -30))   # [-10, -40, -70]

# sum()传入可迭代对象，求和
sum(range(4))  # 0 + 1 + 2 + 3 = 6
```

## 4 pass

pass 语句不执行任何操作。语法上需要一个语句，但程序不实际执行任何动作时，可以使用该语句。

```
while True:
    pass  # Busy-wait for keyboard interrupt (Ctrl+C)

# 创建了一个最小的类：
class MyEmptyClass:
    pass

# 用作函数或条件子句的占位符，让开发者聚焦更抽象的层次。
def initlog(*args):
    pass   # Remember to implement this!
```

## 5 match

比switch更牛逼的匹配。但不需要break控制。

普通用法不必说，python多了其它种类的匹配：

### 5.1 case中做筛选

使用 | （或）在一个模式中可以组合多个字面值：

```
case 401 | 403 | 404:
    return "Not allowed"
```

### 5.2 解包赋值与绑定变量

```
# point is an (x, y) tuple 元组
match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y={y}")
    case (x, 0):
        print(f"X={x}")
    case (x, y):
        print(f"X={x}, Y={y}")
    case _:
        raise ValueError("Not a point")
```

“变量名” `_` 被作为 通配符 并必定会匹配成功。

* 第一个模式有两个字面值，可以看作是上面所示字面值模式的扩展。
* 但接下来的两个模式结合了一个字面值和一个变量，而变量 绑定 了一个来自目标的值（point）。
* 第四个模式捕获了两个值，这使得它在概念上类似于解包赋值 (x, y) = point。

### 5.3 解构对象

```
class Point:
    x: int
    y: int

def where_is(point):
    match point:
        case Point(x=0, y=0):       # 类名后加一个类似于构造器的参数列表，这样做可以把属性放到变量里
            print("Origin")
        case Point(x=0, y=y):
            print(f"Y={y}")
        case Point(x=x, y=0):
            print(f"X={x}")
        case Point():
            print("Somewhere else")
        case _:
            print("Not a point")
```

**总结：**

* 可在 `dataclass` 等支持属性排序的**内置类**中使用**位置参数**。
* 还可在**类**中设置 `__match_args__` 特殊属性为模式的属性定义指定位置。

如果它被设为 `("x", "y")`，则以下模式均为等价的，并且都把 `y` 属性**绑定**到 `var` 变量：

```
Point(1, var)
Point(1, y=var)
Point(x=1, y=var)
Point(y=var, x=1)
```

### 5.4 如何理解

读取模式的推荐方式是将它们看做是：你会在赋值操作**左侧**放置的内容的**扩展形式**，以便理解各个变量将会被设置的值。

* match 语句赋值：
    * 只有单独的名称：例如上面的 `var`。
* match 语句不会赋值：
    * 带点号的名称：例如 `foo.bar`；
    * 属性名称：例如上面的 `x=` 和 `y=`；
    * 类名称：通过其后的 "(...)" 来识别，例如上面的 Point。

因此模式可以任意地嵌套。

例如，如果有一个由点（类实例）组成的短**列表**，则可使用如下方式进行匹配：

```
match points:
    case []:
        print("No points")
    case [Point(0, 0)]:
        print("The origin")
    case [Point(x, y)]:
        print(f"Single point {x}, {y}")
    case [Point(0, y1), Point(0, y2)]:
        print(f"Two on the Y axis at {y1}, {y2}")
    case _:
        print("Something else")
```

### 5.5 其他特性

#### 5.5.1 case中的if

```
match point:
    case Point(x, y) if x == y:
        print(f"Y=X at {x}")
    case Point(x, y):
        print(f"Not on the diagonal")
```

#### 5.5.2 仅了解

* 与解包赋值类似，元组和列表模式具有完全相同的含义，并且实际上能匹配任意序列。 但它们不能匹配迭代器或字符串。

* 序列模式支持扩展解包操作：`[x, y, *rest]` 和 `(x, y, *rest)` 的作用类似于解包赋值。 在 `*` 之后的名称也可以为 `_`，因此，`(x, y, *_)` 可以匹配包含至少两个条目的序列，而不必绑定其余的条目。

* 映射模式：`{"bandwidth": b, "latency": l}` 从字典中捕获 `"bandwidth"` 和 `"latency"` 的值。与序列模式不同，额外的键会被忽略。`**rest` 等解包操作也支持。但 `**_` 是冗余的，不允许使用。

* 使用 `as` 关键字可以捕获子模式：

    ```
    case (Point(x1, y1), Point(x2, y2) as p2): ...
    ```

    将把输入的第二个元素捕获为 `p2` (只要输入是包含两个点的序列)

* 大多数字面值是按相等性比较的，但是单例对象 `True`, `False` 和 `None` 则是按标识号比较的。

* 模式可以使用命名常量。 这些命名常量必须为带点号的名称以防止它们被解读为捕获变量:

    ```
    from enum import Enum
    class Color(Enum):
        RED = 'red'
        GREEN = 'green'
        BLUE = 'blue'

    color = Color(input("Enter your choice of 'red', 'blue' or 'green': "))

    match color:
        case Color.RED:
            print("I see red!")
        case Color.GREEN:
            print("Grass is green")
        case Color.BLUE:
            print("I'm feeling the blues :(")
    ```