[toc]

# 当作计算器

## 数字

在交互模式下，上一次打印出来的表达式被赋值给变量 `_`。这意味着当你把Python用作桌面计算器时，继续计算会相对简单，比如:

```python
>>> tax = 12.5 / 100
>>> price = 100.50
>>> price * tax
12.5625
>>> price + _
113.0625
>>> round(_, 2)
113.06
```

##  字符串

### 转义

Python 也内置对复数的支持，使用后缀 `j` 或者 `J` 就可以表示虚数部分（例如 `3+5j` ）。

如果你不希望前置了 `\` 的字符转义成特殊字符，可以使用 *原始字符串* 方式，在引号前添加 `r` 即可:

```
>>> print('C:\some\name')  # here \n means newline!
C:\some
ame
>>> print(r'C:\some\name')  # note the r before the quote
C:\some\name
```

### 跨行

字符串字面值可以跨行连续输入。一种方式是用三重引号：`"""..."""` 或 `'''...'''`。字符串中的回车换行会自动包含到字符串中，如果不想包含，在行尾添加一个 `\` 即可。如下例:

```python
print("""\
Usage: thingy [OPTIONS]
     -h                        Display this usage message
     -H hostname               Hostname to connect to
""")
```

将产生如下输出（注意最开始的换行没有包括进来）:

```
Usage: thingy [OPTIONS]
     -h                        Display this usage message
     -H hostname               Hostname to connect to
```

相邻的两个或多个 *字符串字面值* （引号引起来的字符）将会自动连接到一起. 把很长的字符串拆开分别输入的时候尤其有用:

```python
>>> text = ('Put several strings within parentheses '
...         'to have them joined together.')
>>> text
'Put several strings within parentheses to have them joined together.'
```

只能对两个字面值这样操作，变量或表达式不行

# 流程控制

## range

给定的终止数值并不在要生成的序列里；`range(10)` 会生成10个值，并且是以合法的索引生成一个长度为10的序列。range也可以以另一个数字开头，或者以指定的幅度增加（甚至是负数；有时这也被叫做 '步进'）

```python
range(5, 10)
   5, 6, 7, 8, 9

range(0, 10, 3)
   0, 3, 6, 9

range(-10, -100, -30)
  -10, -40, -70
```

如果你只打印 range，会出现奇怪的结果:

```python
>>> print(range(10))
range(0, 10)
```

[`range()`](https://docs.python.org/zh-cn/3.8/library/stdtypes.html#range) 所返回的对象在许多方面表现得像一个列表，但实际上却并不是。此对象会在你迭代它时基于所希望的序列返回连续的项，但它没有真正生成列表，这样就能节省空间。

我们称这样对象为 [iterable](https://docs.python.org/zh-cn/3.8/glossary.html#term-iterable)，也就是说，适合作为这样的目标对象：函数和结构期望从中获取连续的项直到所提供的项全部耗尽。 我们已经看到 [`for`](https://docs.python.org/zh-cn/3.8/reference/compound_stmts.html#for) 语句就是这样一种结构，而接受可迭代对象的函数的一个例子是 [`sum()`](https://docs.python.org/zh-cn/3.8/library/functions.html#sum):

```python
>>> sum(range(4))  # 0 + 1 + 2 + 3
6
```

稍后我们将看到更多返回可迭代对象以及将可迭代对象作为参数的函数。 最后，也许你会很好奇如何从一个指定范围内获取一个列表。 以下是解决方案：

```python
>>> list(range(4))
[0, 1, 2, 3]
```

# 函数

## 参数默认值

最有用的形式是对一个或多个参数指定一个默认值。这样创建的函数，可以用比定义时允许的更少的参数调用，比如:

```python
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

这个函数可以通过几种方式调用:

-   只给出必需的参数：`ask_ok('Do you really want to quit?')`
-   给出一个可选的参数：`ask_ok('OK to overwrite the file?', 2)`
-   或者给出所有的参数：`ask_ok('OK to overwrite the file?', 2, 'Come on, only yes or no!')`

这个示例还介绍了 [`in`](https://docs.python.org/zh-cn/3.8/reference/expressions.html#in) 关键字。它可以测试一个序列是否包含某个值。

默认值是在 *定义过程* 中在函数定义处计算的，所以

```python
i = 5

def f(arg=i):
    print(arg)

i = 6
f()
```

会打印 `5`。

**重要警告：** 默认值只会执行一次。这条规则在默认值为可变对象（列表、字典以及大多数类实例）时很重要。比如，下面的函数会存储在后续调用中传递给它的参数:

```python
def f(a, L=[]):
    L.append(a)
    return L

print(f(1))
print(f(2))
print(f(3))
```

这将打印出

```python
[1]
[1, 2]
[1, 2, 3]
```

如果你不想要在后续调用之间共享默认值，你可以这样写这个函数:

```python
def f(a, L=None):
    if L is None:
        L = []
    L.append(a)
    return L
```

## 关键字参数

也可以使用形如 `kwarg=value` 的关键字参数来调用函数。例如下面的函数:

```python
def parrot(voltage, state='a stiff', action='voom', type='Norwegian Blue'):
    print("-- This parrot wouldn't", action, end=' ')
    print("if you put", voltage, "volts through it.")
    print("-- Lovely plumage, the", type)
    print("-- It's", state, "!")
```

接受一个必需的参数（`voltage`）和三个可选的参数（`state`, `action`，和 `type`）。这个函数可以通过下面的任何一种方式调用:

```python
parrot(1000)                                          # 1 positional argument
parrot(voltage=1000)                                  # 1 keyword argument
parrot(voltage=1000000, action='VOOOOOM')             # 2 keyword arguments
parrot(action='VOOOOOM', voltage=1000000)             # 2 keyword arguments
parrot('a million', 'bereft of life', 'jump')         # 3 positional arguments
parrot('a thousand', state='pushing up the daisies')  # 1 positional, 1 keyword
```

但下面的函数调用都是无效的:

```python
parrot()                     # required argument missing
parrot(voltage=5.0, 'dead')  # non-keyword argument after a keyword argument
parrot(110, voltage=220)     # duplicate value for the same argument
parrot(actor='John Cleese')  # unknown keyword argument
```

```python
def parrot(state='a stiff', voltage, action='voom', type='Norwegian Blue'):
    
SyntaxError: non-default argument follows default argument
```

在函数调用中，关键字参数必须跟随在位置参数的后面。传递的所有关键字参数必须与函数接受的其中一个参数匹配（比如 `actor` 不是函数 `parrot` 的有效参数），它们的顺序并不重要。这也包括非可选参数，（比如 `parrot(voltage=1000)` 也是有效的）。不能对同一个参数多次赋值。下面是一个因为此限制而失败的例子:

```python
>>> def function(a):
...     pass
...
>>> function(0, a=0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: function() got multiple values for keyword argument 'a'
```

当存在一个形式为 `**name` 的最后一个形参时，它会接收一个字典，其中包含除了与已有形参相对应的关键字参数以外的所有关键字参数。 这可以与一个形式为 `*name`，接收一个包含除了已有形参列表以外的位置参数的元组的形参组合使用 (`*name` 必须出现在 `**name` 之前。) 例如，如果我们这样定义一个函数:

```python
def cheeseshop(kind, *arguments, **keywords):
    print("-- Do you have any", kind, "?")
    print("-- I'm sorry, we're all out of", kind)
    for arg in arguments:
        print(arg)
    print("-" * 40)
    for kw in keywords:
        print(kw, ":", keywords[kw])
```

它可以像这样调用:

```python
cheeseshop("Limburger", "It's very runny, sir.",
           "It's really very, VERY runny, sir.",
           shopkeeper="Michael Palin",
           client="John Cleese",
           sketch="Cheese Shop Sketch")
```

当然它会打印:

```python
-- Do you have any Limburger ?
-- I'm sorry, we're all out of Limburger
It's very runny, sir.
It's really very, VERY runny, sir.
----------------------------------------
shopkeeper : Michael Palin
client : John Cleese
sketch : Cheese Shop Sketch
```

注意打印时关键字参数的顺序保证与调用函数时提供它们的顺序是相匹配的。

## 仅位置/关键字参数

请考虑以下示例函数定义并特别注意 `/` 和 `*` 标记:

```python
>>> def standard_arg(arg):
...     print(arg)
...
>>> def pos_only_arg(arg, /):
...     print(arg)
...
>>> def kwd_only_arg(*, arg):
...     print(arg)
...
>>> def combined_example(pos_only, /, standard, *, kwd_only):
...     print(pos_only, standard, kwd_only)
```

第一个函数定义 `standard_arg` 是最常见的形式，对调用方式没有任何限制，参数可以按位置也可以按关键字传入:

```python
>>> standard_arg(2)
2

>>> standard_arg(arg=2)
2
```

第二个函数 `pos_only_arg` 在函数定义中带有 `/`，限制仅使用位置形参。:

```python
>>> pos_only_arg(1)
1

>>> pos_only_arg(arg=1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: pos_only_arg() got an unexpected keyword argument 'arg'
```

第三个函数 `kwd_only_args` 在函数定义中通过 `*` 指明仅允许关键字参数:

```python
>>> kwd_only_arg(3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: kwd_only_arg() takes 0 positional arguments but 1 was given

>>> kwd_only_arg(arg=3)
3
```

而最后一个则在同一函数定义中使用了全部三种调用方式:

```python
>>> combined_example(1, 2, 3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: combined_example() takes 2 positional arguments but 3 were given

>>> combined_example(1, 2, kwd_only=3)
1 2 3

>>> combined_example(1, standard=2, kwd_only=3)
1 2 3

>>> combined_example(pos_only=1, standard=2, kwd_only=3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: combined_example() got an unexpected keyword argument 'pos_only'
```

## Lambda

可以用 [`lambda`](https://docs.python.org/zh-cn/3.8/reference/expressions.html#lambda) 关键字来创建一个小的匿名函数。这个函数返回两个参数的和： `lambda a, b: a+b` 。Lambda函数可以在需要函数对象的任何地方使用。它们在语法上限于单个表达式。从语义上来说，它们只是正常函数定义的语法糖。与嵌套函数定义一样，lambda函数可以引用所包含域的变量:

```python
>>> def make_incrementor(n):
...     return lambda x: x + n
...
>>> f = make_incrementor(42)
>>> f(0)
42
>>> f(1)
43
```

上面的例子使用一个lambda表达式来返回一个函数。另一个用法是传递一个小函数作为参数:

```python
>>> pairs = [(1, 'one'), (2, 'two'), (3, 'three'), (4, 'four')]
>>> pairs.sort(key=lambda pair: pair[1])
>>> pairs
[(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]
```

// *key* 指定带有单个参数的函数，用于从 *iterable* 的每个元素中提取用于比较的键 (例如 `key=str.lower`)。 默认值为 `None` (直接比较元素)。

## 标注

函数标注以字典的形式存放在函数的 `__annotations__` 属性中，并且不会影响函数的任何其他部分。 形参标注的定义方式是在形参名称后加上冒号，后面跟一个表达式，该表达式会被求值为标注的值。 返回值标注的定义方式是加上一个组合符号 `->`，后面跟一个表达式，该标注位于形参列表和表示 [`def`](https://docs.python.org/zh-cn/3.8/reference/compound_stmts.html#def) 语句结束的冒号之间。 下面的示例有一个位置参数，一个关键字参数以及返回值带有相应标注:

```python
>>> def f(ham: str, eggs: str = 'eggs') -> str:
...     print("Annotations:", f.__annotations__)
...     print("Arguments:", ham, eggs)
...     return ham + ' and ' + eggs
...
>>> f('spam')
Annotations: {'ham': <class 'str'>, 'return': <class 'str'>, 'eggs': <class 'str'>}
Arguments: spam eggs
'spam and eggs'
```

# 数组

-   `list.append`(*x*)

    在列表的末尾添加一个元素。相当于 `a[len(a):] = [x]` 。

-   `list.extend`(*iterable*)

    使用可迭代对象中的所有元素来扩展列表。相当于 `a[len(a):] = iterable` 。

-   `list.insert`(*i*, *x*)

    在给定的位置插入一个元素。第一个参数是要插入的元素的索引，所以 `a.insert(0, x)` 插入列表头部， `a.insert(len(a), x)` 等同于 `a.append(x)` 。

-   `list.remove`(*x*)

    移除列表中第一个值为 *x* 的元素。如果没有这样的元素，则抛出 [`ValueError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#ValueError) 异常。

-   `list.pop`([*i*])

    删除列表中给定位置的元素并返回它。如果没有给定位置，`a.pop()` 将会删除并返回列表中的最后一个元素。（ 方法签名中 *i* 两边的方括号表示这个参数是可选的，而不是要你输入方括号。你会在 Python 参考库中经常看到这种表示方法)。

-   `list.clear`()

    移除列表中的所有元素。等价于``del a[:]``

-   `list.index`(*x*[, *start*[, *end*]])

    返回列表中第一个值为 *x* 的元素的从零开始的索引。如果没有这样的元素将会抛出 [`ValueError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#ValueError) 异常。可选参数 *start* 和 *end* 是切片符号，用于将搜索限制为列表的特定子序列。返回的索引是相对于整个序列的开始计算的，而不是 *start* 参数。

-   `list.count`(*x*)

    返回元素 *x* 在列表中出现的次数。

-   `list.sort`(***, *key=None*, *reverse=False*)

    对列表中的元素进行排序（参数可用于自定义排序，解释请参见 [`sorted()`](https://docs.python.org/zh-cn/3.8/library/functions.html#sorted)）。

-   `list.reverse`()

    翻转列表中的元素。

-   `list.copy`()

    返回列表的一个浅拷贝，等价于 `a[:]`。

## 栈

列表方法使得列表作为堆栈非常容易，最后一个插入，最先取出（“后进先出”）。要添加一个元素到堆栈的顶端，使用 `append()` 。要从堆栈顶部取出一个元素，使用 `pop()` ，不用指定索引。例如

```python
>>> stack = [3, 4, 5]
>>> stack.append(6)
>>> stack.append(7)
>>> stack
[3, 4, 5, 6, 7]
>>> stack.pop()
7
>>> stack
[3, 4, 5, 6]
>>> stack.pop()
6
>>> stack.pop()
5
>>> stack
[3, 4]
```

## 列表

列表也可以用作队列，其中先添加的元素被最先取出 (“先进先出”)；然而列表用作这个目的相当低效。因为在列表的末尾添加和弹出元素非常快，但是在列表的开头插入或弹出元素却很慢 (因为所有的其他元素都必须移动一位)。

若要实现一个队列，可使用 [`collections.deque`](https://docs.python.org/zh-cn/3.8/library/collections.html#collections.deque)，它被设计成可以快速地从两端添加或弹出元素。例如

```python
>>> from collections import deque
>>> queue = deque(["Eric", "John", "Michael"])
>>> queue.append("Terry")           # Terry arrives
>>> queue.append("Graham")          # Graham arrives
>>> queue.popleft()                 # The first to arrive now leaves
'Eric'
>>> queue.popleft()                 # The second to arrive now leaves
'John'
>>> queue                           # Remaining queue in order of arrival
deque(['Michael', 'Terry', 'Graham'])
```

## 列表推导式

假设我们想创建一个平方列表，像这样

```python
>>> squares = []
>>> for x in range(10):
...     squares.append(x**2)
...
>>> squares
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

等价于

```python
squares = list(map(lambda x: x**2, range(10)))
```

```python
squares = [x**2 for x in range(10)]
```

上面这种写法更加简洁易读。

列表推导式的结构是由一对方括号所包含的以下内容：一个表达式，后面跟一个 `for` 子句，然后是零个或多个 `for` 或 `if` 子句。 其结果将是一个新列表，由对表达式依据后面的 `for` 和 `if` 子句的内容进行求值计算而得出。 举例来说，以下列表推导式会将两个列表中不相等的元素组合起来:

```python
>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
```

而它等价于

```python
>>> combs = []
>>> for x in [1,2,3]:
...     for y in [3,1,4]:
...         if x != y:
...             combs.append((x, y))
...
>>> combs
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)
```

```python
>>> vec = [-4, -2, 0, 2, 4]
>>> # create a new list with the values doubled
>>> [x*2 for x in vec]
[-8, -4, 0, 4, 8]
>>> # filter the list to exclude negative numbers
>>> [x for x in vec if x >= 0]
[0, 2, 4]
>>> # apply a function to all the elements
>>> [abs(x) for x in vec]
[4, 2, 0, 2, 4]
>>> # call a method on each element
>>> freshfruit = ['  banana', '  loganberry ', 'passion fruit  ']
>>> [weapon.strip() for weapon in freshfruit]
['banana', 'loganberry', 'passion fruit']
>>> # create a list of 2-tuples like (number, square)
>>> [(x, x**2) for x in range(6)]
[(0, 0), (1, 1), (2, 4), (3, 9), (4, 16), (5, 25)]
>>> # the tuple must be parenthesized, otherwise an error is raised
>>> [x, x**2 for x in range(6)]
  File "<stdin>", line 1, in <module>
    [x, x**2 for x in range(6)]
               ^
SyntaxError: invalid syntax
>>> # flatten a list using a listcomp with two 'for'
>>> vec = [[1,2,3], [4,5,6], [7,8,9]]
>>> [num for elem in vec for num in elem]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 嵌套的列表推导式

列表推导式中的初始表达式可以是任何表达式，包括另一个列表推导式。

考虑下面这个 3x4的矩阵，它由3个长度为4的列表组成

```
>>> matrix = [
...     [1, 2, 3, 4],
...     [5, 6, 7, 8],
...     [9, 10, 11, 12],
... ]
```

下面的列表推导式将交换其行和列

```python
>>> [[row[i] for row in matrix] for i in range(4)]
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```

如上节所示，嵌套的列表推导式是基于跟随其后的 [`for`](https://docs.python.org/zh-cn/3.8/reference/compound_stmts.html#for) 进行求值的，所以这个例子等价于:

```
>>> transposed = []
>>> for i in range(4):
...     transposed.append([row[i] for row in matrix])
...
>>> transposed
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```

反过来说，也等价于

```python
>>> transposed = []
>>> for i in range(4):
...     # the following 3 lines implement the nested listcomp
...     transposed_row = []
...     for row in matrix:
...         transposed_row.append(row[i])
...     transposed.append(transposed_row)
...
>>> transposed
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```

实际应用中，你应该会更喜欢使用内置函数去组成复杂的流程语句。 [`zip()`](https://docs.python.org/zh-cn/3.8/library/functions.html#zip) 函数将会很好地处理这种情况

```python
>>> list(zip(*matrix))
[(1, 5, 9), (2, 6, 10), (3, 7, 11), (4, 8, 12)]
```

## del 语句

有一种方式可以从列表按照给定的索引而不是值来移除一个元素: 那就是 [`del`](https://docs.python.org/zh-cn/3.8/reference/simple_stmts.html#del) 语句。 它不同于会返回一个值的 `pop()` 方法。 `del` 语句也可以用来从列表中移除切片或者清空整个列表（我们之前用过的方式是将一个空列表赋值给指定的切片）。 例如:

```python
>>> a = [-1, 1, 66.25, 333, 333, 1234.5]
>>> del a[0]
>>> a
[1, 66.25, 333, 333, 1234.5]
>>> del a[2:4]
>>> a
[1, 66.25, 1234.5]
>>> del a[:]
>>> a
[]
```

[`del`](https://docs.python.org/zh-cn/3.8/reference/simple_stmts.html#del) 也可以删除整个变量

```python
>>> del a
```

