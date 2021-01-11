[toc]

# 元组

一个元组由几个被逗号隔开的值组成，例如

```python
>>> t = 12345, 54321, 'hello!'
>>> t[0]
12345
>>> t
(12345, 54321, 'hello!')
>>> # Tuples may be nested:
... u = t, (1, 2, 3, 4, 5)
>>> u
((12345, 54321, 'hello!'), (1, 2, 3, 4, 5))
>>> # Tuples are immutable:
... t[0] = 88888
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> # but they can contain mutable objects:
... v = ([1, 2, 3], [3, 2, 1])
>>> v
([1, 2, 3], [3, 2, 1])
```

一个特殊的问题是构造包含0个或1个元素的元组：为了适应这种情况，语法有一些额外的改变。空元组可以直接被一对空圆括号创建，含有一个元素的元组可以通过在这个元素后添加一个逗号来构建（圆括号里只有一个值的话不够明确）。丑陋，但是有效。例如

```python
>>> empty = ()
>>> singleton = 'hello',    # <-- note trailing comma
>>> len(empty)
0
>>> len(singleton)
1
>>> singleton
('hello',)
```

语句 `t = 12345, 54321, 'hello!'` 是 *元组打包* 的一个例子：值 `12345`, `54321` 和 `'hello!'` 被打包进元组。其逆操作也是允许的

```python
>>> x, y, z = t
```

# 集合

注意：要创建一个空集合你只能用 `set()` 而不能用 `{}`

类似于 列表推导式，集合也支持推导式形式

```
>>> a = {x for x in 'abracadabra' if x not in 'abc'}
>>> a
{'r', 'd'}
```

# 字典

```python
>>> tel = {'jack': 4098, 'sape': 4139}
>>> tel['guido'] = 4127
>>> tel
{'jack': 4098, 'sape': 4139, 'guido': 4127}
>>> tel['jack']
4098
>>> del tel['sape']
>>> tel['irv'] = 4127
>>> tel
{'jack': 4098, 'guido': 4127, 'irv': 4127}
>>> list(tel)
['jack', 'guido', 'irv']
>>> sorted(tel)
['guido', 'irv', 'jack']
>>> 'guido' in tel
True
>>> 'jack' not in tel
False
```

dict()构造函数可以直接从键值对序列里创建字典。

```python
>>> dict([('sape', 4139), ('guido', 4127), ('jack', 4098)])
{'sape': 4139, 'guido': 4127, 'jack': 4098}
```

此外，字典推导式可以从任意的键值表达式中创建字典

```python
>>> {x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}
```

当关键字是简单字符串时，有时直接通过关键字参数来指定键值对更方便

```python
>>> dict(sape=4139, guido=4127, jack=4098)
{'sape': 4139, 'guido': 4127, 'jack': 4098}
```

## 循环

当在字典中循环时，用 `items()` 方法可将关键字和对应的值同时取出

```python
>>> knights = {'gallahad': 'the pure', 'robin': 'the brave'}
>>> for k, v in knights.items():
...     print(k, v)
...
gallahad the pure
robin the brave
```

当在序列中循环时，用enumerate()函数可以将索引位置和其对应的值同时取出

```python
>>> for i, v in enumerate(['tic', 'tac', 'toe']):
...     print(i, v)
...
0 tic
1 tac
2 toe
```

当同时在两个或更多序列中循环时，可以用zip()函数将其内元素一一匹配。

```python
>>> questions = ['name', 'quest', 'favorite color']
>>> answers = ['lancelot', 'the holy grail', 'blue']
>>> for q, a in zip(questions, answers):
...     print('What is your {0}?  It is {1}.'.format(q, a))
...
What is your name?  It is lancelot.
What is your quest?  It is the holy grail.
What is your favorite color?  It is blue.
```

如果要逆向循环一个序列，可以先正向定位序列，然后调用reversed()函数

```python
>>> for i in reversed(range(1, 10, 2)):
...     print(i)
...
9
7
5
3
1
```

如果要按某个指定顺序循环一个序列，可以用sorted() 函数，它可以在不改动原序列的基础上返回一个新的排好序的序列

```python
>>> basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
>>> for f in sorted(set(basket)):
...     print(f)
...
apple
banana
orange
pear
```

# Class

## 继承

```python
class DerivedClassName(BaseClassName):
    <statement-1>
    .
    .
    .
    <statement-N>
```

Python有两个内置函数可被用于继承机制：

-   使用 [`isinstance()`](https://docs.python.org/zh-cn/3.8/library/functions.html#isinstance) 来检查一个实例的类型: `isinstance(obj, int)` 仅会在 `obj.__class__` 为 [`int`](https://docs.python.org/zh-cn/3.8/library/functions.html#int) 或某个派生自 [`int`](https://docs.python.org/zh-cn/3.8/library/functions.html#int) 的类时为 `True`。
-   使用 [`issubclass()`](https://docs.python.org/zh-cn/3.8/library/functions.html#issubclass) 来检查类的继承关系: `issubclass(bool, int)` 为 `True`，因为 [`bool`](https://docs.python.org/zh-cn/3.8/library/functions.html#bool) 是 [`int`](https://docs.python.org/zh-cn/3.8/library/functions.html#int) 的子类。 但是，`issubclass(float, int)` 为 `False`，因为 [`float`](https://docs.python.org/zh-cn/3.8/library/functions.html#float) 不是 [`int`](https://docs.python.org/zh-cn/3.8/library/functions.html#int) 的子类。

Python 也支持一种多重继承。 带有多个基类的类定义语句如下所示:

```python
class DerivedClassName(Base1, Base2, Base3):
    <statement-1>
    .
    .
    .
    <statement-N>
```

对于多数应用来说，在最简单的情况下，你可以认为搜索从父类所继承属性的操作是深度优先、从左至右的，当层次结构中存在重叠时不会在同一个类中搜索两次。 因此，如果某一属性在 `DerivedClassName` 中未找到，则会到 `Base1` 中搜索它，然后（递归地）到 `Base1` 的基类中搜索，如果在那里未找到，再到 `Base2` 中搜索，依此类推。

由于存在对于类私有成员的有效使用场景（例如避免名称与子类所定义的名称相冲突），因此存在对此种机制的有限支持，称为 *名称改写*。 任何形式为 `__spam` 的标识符（至少带有两个前缀下划线，至多一个后缀下划线）的文本将被替换为 `_classname__spam`，其中 `classname` 为去除了前缀下划线的当前类名称。 这种改写不考虑标识符的句法位置，只要它出现在类定义内部就会进行。



# 生成器

[生成器](https://docs.python.org/zh-cn/3.8/glossary.html#term-generator) 是一个用于创建迭代器的简单而强大的工具。 它们的写法类似于标准的函数，但当它们要返回数据时会使用 [`yield`](https://docs.python.org/zh-cn/3.8/reference/simple_stmts.html#yield) 语句。 每次在生成器上调用 [`next()`](https://docs.python.org/zh-cn/3.8/library/functions.html#next) 时，它会从上次离开的位置恢复执行（它会记住上次执行语句时的所有数据值）。 一个显示如何非常容易地创建生成器的示例如下:

```python
def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index]
```

```python
>>> for char in reverse('golf'):
...     print(char)
...
f
l
o
g
```

可以用生成器来完成的操作同样可以用前一节所描述的基于类的迭代器来完成。 但生成器的写法更为紧凑，因为它会自动创建 [`__iter__()`](https://docs.python.org/zh-cn/3.8/reference/datamodel.html#object.__iter__) 和 [`__next__()`](https://docs.python.org/zh-cn/3.8/reference/expressions.html#generator.__next__) 方法。

另一个关键特性在于局部变量和执行状态会在每次调用之间自动保存。 这使得该函数相比使用 `self.index` 和 `self.data` 这种实例变量的方式更易编写且更为清晰。

除了会自动创建方法和保存程序状态，当生成器终结时，它们还会自动引发 [`StopIteration`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#StopIteration)。 这些特性结合在一起，使得创建迭代器能与编写常规函数一样容易。



## 生成器表达式

某些简单的生成器可以写成简洁的表达式代码，所用语法类似列表推导式，但外层为圆括号而非方括号。 这种表达式被设计用于生成器将立即被外层函数所使用的情况。 生成器表达式相比完整的生成器更紧凑但较不灵活，相比等效的列表推导式则更为节省内存。

示例:

```python
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