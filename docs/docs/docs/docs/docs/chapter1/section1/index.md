# 存储结构

## 1 字符串

```
# 次方
-3 ** 2   # 优先级：** > -    被解析成：-(3**2) = -9

# 索引
word = 'Python'
word[-1]  # 'n'
word[42]  # 报错

# 切片，省略开始索引时，默认值为 0，省略结束索引时，默认为到字符串的结尾：
word[:2]   # 'Py'
word[4:]   # 'on'
word[-2:]  # 'on'
word[4:42] # 'on'
word[42:]  # ''

# 字符串不能修改
word[0] = 'J'   # 报错
'J' + word[1:]  # 'Jython'

# len
s = 'supercalifragilisticexpialidocious'
len(s)     # 34
```

## 2 列表

列表：用`[]`标注，`,`分隔。

可以包含不同类型的元素，但一般情况下，各个元素的类型相同：

```
squares = [1, 4, 9, 16, 25]

# 索引
squares[-1]   # 25

# 切片，返回包含请求元素的新列表（浅拷贝）
squares[-3:]  # [9, 16, 25]
letters = ['a', 'b', 'c', 'd', 'e', 'f', 'g']
letters[2:5] = ['C', 'D', 'E']
letters       # ['a', 'b', 'C', 'D', 'E', 'f', 'g']
letters[2:5] = []
letters       # ['a', 'b', 'f', 'g']

# 合并
squares + [36, 49, 64, 81, 100]   # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

# append() 列表结尾添加新元素
squares.append(216)  # [1, 4, 9, 16, 25, 216]

# len
letters = ['a', 'b', 'c', 'd']
len(letters)   # 4

# 嵌套列表：创建包含其他列表的列表
a = ['a', 'b', 'c']
n = [1, 2, 3]
x = [a, n]
x                   # [['a', 'b', 'c'], [1, 2, 3]]
x[0]                # ['a', 'b', 'c']
x[0][1]             # 'b'
```

## 3 代码上手

斐波那契数列：

```
a, b = 0, 1
while a < 1000:
    print(a, end=',')
    a, b = b, a+b

# 0,1,1,2,3,5,8,13,21,34,55,89,144,233,377,610,987,
```

* 多重赋值：第1行和第4行，先计算`=`后面的结果，计算结束后再赋给前面的变量。
* 缩进区分各语句块。
* `print()`默认换行，可使用`end`来替换换行符，同时和`console.log()`一般用法。

