---
title: Elementary Python for R user, translation.
date: 2023-10-22
tags: [python]
categories: 计算机
description: 翻译
---

[原文](https://cran.r-project.org/web/packages/reticulate/vignettes/python_primer.html)，是R::reticulate提供的小教程。

## R用户指南：Python入门

你可能会发现自己想阅读和理解一些Python代码，甚至将一些Python代码移植到R中。这份指南旨在帮助你尽快完成这些任务。正如你将看到的，R和Python足够相似，以至于可以在不学习全部Python的情况下做到这一点。我们从容器类型的基础知识开始，逐步介绍到类、特殊方法、迭代器协议、上下文协议等！

### 空格

在Python中，空格很重要。在R中，表达式是通过`{}`组合成代码块的。在Python中，这是通过使表达式共享一个缩进级别来完成的。例如，一个带有R代码块的表达式可能是：

```r
if (TRUE) {
  cat("This is one expression. \n")
  cat("This is another expression. \n")
}
#> This is one expression.
#> This is another expression.
```

在Python中的等效代码是：

```python
if True:
  print("This is one expression.")
  print("This is another expression.")
#> This is one expression.
#> This is another expression.
```

Python接受制表符或空格作为缩进空格，但当它们混合使用时，规则会变得复杂。大多数风格指南建议（并且IDE默认使用）仅使用空格。

### 容器类型

在R中，`list()`是一个可以用来组织R对象的容器。R的`list()`功能丰富，没有单一的直接等价物在Python中支持所有相同的功能。相反，有（至少）4种不同的Python容器类型需要你注意：列表、字典、元组和集合。

#### 列表

Python列表通常使用裸括号`[]`创建。Python内置的`list()`函数更像是一个强制函数，更接近于R的`as.list()`。关于Python列表最重要的是它们是就地修改的。注意下面的例子中，`y`反映了对`x`所做的更改，因为两个符号指向的底层列表对象就地被修改了。

```python
x = [1, 2, 3]
y = x    # `y`和`x`现在引用同一个列表！
x.append(4)
print("x is", x)
#> x is [1, 2, 3, 4]
print("y is", y)
#> y is [1, 2, 3, 4]
```

一个可能让R用户担忧的Python习惯是通过对`append()`方法来扩展列表。在R中扩展列表通常是缓慢的，最好避免。但由于Python的列表是就地修改的（并且在追加项目时避免了完整列表的复制），因此就地扩展Python列表是高效的。

你可能会遇到的围绕Python列表的一些语法糖是使用`+`和`*`与列表。这些是连接和复制操作符，类似于R的`c()`和`rep()`。

```python
x = [1]
x
#> [1]
x + x
#> [1, 1]
x * 3
#> [1, 1, 1]
```

你可以使用尾随`[]`的整数来索引列表，但请注意索引是基于0的。

```python
x = [1, 2, 3]
x[0]
#> 1
x[1]
#> 2
x[2]
#> 3

try:
  x[3]
except Exception as e:
  print(e)
#> list index out of range
```

在索引时，负数从容器的末尾开始计数。

```python
x = [1, 2, 3]
x[-1]
#> 3
x[-2]
#> 2
x[-3]
#> 1
```

你可以使用`:`在括号中切片列表。请注意切片语法不包括切片范围的末尾。你还可以可选地指定步长。

```python
x = [1, 2, 3, 4, 5, 6]
x[0:2] # 获取索引位置0,1的项
#> [1, 2]
x[1:]  # 从索引位置1开始获取所有项
#> [2, 3, 4, 5, 6]
x[:-2] # 从开始获取直到倒数第二个项
#> [1, 2, 3, 4]
x[:]   # 获取所有项（用于复制列表，以便不就地修改的习语）
#> [1, 2, 3, 4, 5, 6]
x[::2] # 获取所有项，步长为2
#> [1, 3, 5]
x[1::2] # 从索引1开始获取所有项，步长为2
#> [2, 4, 6]
```

#### 元组

元组的行为类似于列表，除了它们是不可变的，并且没有像`append()`这样的就地修改方法。它们通常使用裸`()`构建，但括号不是严格要求的，你可能会看到一个隐式的元组仅由逗号分隔的表达式定义。因为括号也可以用来指定像`(x + 3) * 4`这样的表达式的运算顺序，所以需要特殊的语法来定义长度为1的元组：尾随逗号。元组最常见的是在函数中遇到，这些函数接受可变数量的参数。

```python
x = (1, 2) # 长度为2的元组
type(x)
#> <class 'tuple'>
len(x)
#> 2
x
#> (1, 2)

x = (1,) # 长度为1的元组
type(x)
#> <class 'tuple'>
len(x)
#> 1
x
#> (1,)

x = () # 长度为0的元组
print(f"{type(x) = }; {len(x) = }; {x = }")
#> type(x) = <class 'tuple'>; len(x) = 0; x = ()
# 示例插值字符串字面量

x = 1, 2 # 也是一个元组
type(x)
#> <class 'tuple'>
len(x)
#> 2

x = 1, # 小心一个尾随逗号！这是一个元组！
type(x)
#> <class 'tuple'>
len(x)
#> 1
```

##### 打包和解包

元组是驱动Python中《打包》和《解包》语义的容器。Python提供了方便，允许你在单个表达式中分配多个符号。这称为《解包》。

例如：

```python
x = (1, 2, 3)
a, b, c = x
a
#> 1
b
#> 2
c
#> 3
```

（你可以使用``zeallot::`%<-%``从R访问类似的解包行为）。

元组解包可以在多种上下文中发生，例如迭代：

```python
xx = (("a", 1),
      ("b", 2))
for x1, x2 in xx:
  print("x1 = ", x1)
  print("x2 = ", x2)
#> x1 =  a
#> x2 =  1
#> x1 =  b
#> x2 = 2
```

如果你尝试将容器解包到错误数量的符号，Python会抛出错误：

```python
x = (1, 2, 3)
a, b, c = x # 成功
a, b = x    # 错误，x有太多值要解包
#> ValueError: too many values to unpack (expected 2)
a, b, c, d = x # 错误，x没有足够的值要解包
#> ValueError: not enough values to unpack (expected 4, got 3)
```

可以使用`*`作为前缀来解包可变数量的参数。（当我们讨论函数时，你将再次看到`*`前缀）

```python
x = (1, 2, 3)
a, *the_rest = x
a
#> 1
the_rest
#> [2, 3]
```

你也可以解包嵌套结构：

```python
x = ((1, 2), (3, 4))
(a, b), (c, d) = x
```

#### 字典

字典与R环境最为相似。它们是一个容器，你可以通过名称来检索项目，尽管在Python中名称（在Python的术语中称为_key_）不需要像在R中那样是字符串。它可以是任何具有`hash()`方法的Python对象（这意味着，它可以是几乎所有的Python对象）。它们可以使用`{key: value}`这样的语法创建。像Python列表一样，它们是就地修改的。请注意，`r_to_py()`将R命名列表转换为字典。

```python
d = {"key1": 1,
     "key2": 2}
d2 = d
d
#> {'key1': 1, 'key2': 2}
d["key1"]
#> 1
d["key3"] = 3
d2 # 就地修改！
#> {'key1': 1, 'key2': 2, 'key3': 3}
```

像R环境（与R的命名列表不同），你不能使用整数来索引字典以获取特定索引位置的项目。字典是_无序_容器。
（然而—从Python 3.7开始，字典确实保留了项目插入顺序）。

```python
d = {"key1": 1, "key2": 2}
d[1] # 错误
#> KeyError: 1
```

与R的命名列表语义最接近的容器是`OrderedDict`，
但在Python代码中相对不常见，因此我们不再进一步介绍。

#### 集合

集合是一个容器，可以用来高效地跟踪唯一项目或去除列表中的重复项。它们使用`{val1, val2}`（像字典，但没有`:`）构建。将它们视为只使用键的字典。集合有许多用于成员操作的高效方法，如`intersection()`, `issubset()`, `union()`等。

```python
s = {1, 2, 3}
type(s)
#> <class 'set'>
s
#> {1, 2, 3}

s.add(1)
s
#> {1, 2, 3}
```

### 使用`for`进行迭代

Python中的`for`语句可以用来迭代任何类型的容器。

```python
for x in [1, 2, 3]:
  print(x)
#> 1
#> 2
#> 3
```

R有一个相对有限的对象集合可以传递给`for`。相比之下，Python提供了一个迭代器协议接口，这意味着作者可以定义具有自定义行为的对象，这些对象被`for`调用。（我们将在讨论类时提供一个如何定义自定义可迭代对象的例子）。你可能想从R使用reticulate来使用Python可迭代对象，因此有助于稍微剥离语法糖，展示Python中`for`语句正在做什么，以及如何手动逐步进行。

发生两件事：首先，从提供的对象构建一个迭代器。然后，新迭代器对象被反复调用`next()`直到耗尽。

```python
l = [1, 2, 3]
it = iter(l) # 创建一个迭代器对象
it
#> <list_iterator object at 0x1402267a0>

# 调用迭代器的 `next` 直到耗尽：
next(it)
#> 1
next(it)
#> 2
next(it)
#> 3
next(it)
#> StopIteration
```

在R中，你可以使用reticulate以相同的方式遍历迭代器。

```r
library(reticulate)
l <- r_to_py(list(1, 2, 3))
it <- as_iterator(l)

iter_next(it)
#> 1.0
iter_next(it)
#> 2.0
iter_next(it)
#> 3.0
iter_next(it, completed = "StopIteration")
#> [1] "StopIteration"
```

迭代字典首先需要了解你是在迭代键、值还是两者。字典有方法允许你指定。

```python
d = {"key1": 1, "key2": 2}
for key in d:
  print(key)
#> key1
#> key2

for value in d.values():
  print(value)
#> 1
#> 2

for key, value in d.items():
  print(key, ":", value)
#> key1 : 1
#> key2 : 2
```

#### 推导式

推导式是特殊语法，允许你构建一个像列表或字典这样的容器，同时对每个元素执行一个小操作或单个表达式。你可以将其视为R的`lapply`的特殊语法。

例如：

```python
x = [1, 2, 3]

# 从x构建一个列表推导式，其中你为每个元素添加100
l = [element + 100 for element in x]
l
#> [101, 102, 103]

# 从x构建一个字典推导式，其中键是一个字符串。
# Python的str()类似于R的as.character()
d = {str(element) : element + 100
     for element in x}
d
#> {'1': 101, '2': 102, '3': 103}
```

### 使用`def`定义函数

Python函数使用`def`语句定义。指定函数参数和默认值的语法与R非常相似。

```python
def my_function(name = "World"):
  print("Hello", name)

my_function()
#> Hello World
my_function("Friend")
#> Hello Friend
```

等效的R代码片段将是

```r
my_function <- function(name = "World") {
  cat("Hello", name, "\n")
}

my_function()
#> Hello World
my_function("Friend")
#> Hello Friend
```

与R函数不同，函数中的最后一个值不会自动返回。Python需要一个明确的返回语句。

```python
def fn():
  1
print(fn())
#> None

def fn():
  return 1
print(fn())
#> 1
```

（对于高级R用户的注释：Python没有与R的参数“承诺”等效的东西。函数参数默认值在函数构造时评估一次。如果你定义了一个Python函数，并且默认参数值为可变对象，如Python列表，这可能会让你感到惊讶！）

```python
def my_func(x = []):
  x.append("was called")
  print(x)

my_func()
#> ['was called']
my_func()
#> ['was called', 'was called']
my_func()
#> ['was called', 'was called', 'was called']
```

你还可以使用与R中的`...`类似的语法来定义接受可变数量参数的Python函数。一个值得注意的区别是R的`...`对命名和未命名参数没有区分，但Python确实有区分。在Python中，在单个`*`前缀捕获未命名参数，两个`**`表示捕获关键字参数。

```python
def my_func(*args, **kwargs):
  print("args = ", args) # args是一个元组
  print("kwargs = ", kwargs) # kwargs是一个字典

my_func(1, 2, 3, a = 4, b = 5, c = 6)
#> args =  (1, 2, 3)
#> kwargs =  {'a': 4, 'b': 5, 'c': 6}
```

在函数定义签名中的`*`和`**`将参数打包，在函数调用中它们解包参数。在函数调用中解包参数等同于在R中使用`do.call()`。

```python
def my_func(a, b, c):
  print(a, b, c)

args = (1, 2, 3)
my_func(*args)
#> 1 2 3

kwargs = {"a": 1, "b": 2, "c": 3}
my_func(**kwargs)
#> 1 2 3
```

### 使用`class`定义类

可以说，在R中，代码组合的首要单位是`function`，而在Python中，它是`class`。你可以是一个非常有生产力的R用户，而从未使用过R6、引用类或类似的R等价物，这些等价物是Python`class`的对象导向风格的。

在Python中，理解`class`对象的基础知识是必要的，因为`class`是你在Python中组织和查找方法的方式。
（与R的方法不同，R的方法是通过从通用方法派发的）。幸运的是，`class`的基础知识是容易理解的。

如果你第一次接触面向对象编程，不要被吓倒。我们将首先构建一个简单的Python类进行演示。

```python
class MyClass:
  pass # `pass`意味着什么也不做。

MyClass
#> <class '__main__.MyClass'>
type(MyClass)
#> <class 'type'>
```

实例化类后，你可以与之交互，你会看到它已经带有很多属性（Python中的`dir()`等同于R中的`names()`）：

```python
dir(MyClass)
#> ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
```

#### 为什么会有这么多下划线？

Python通常通过将名称包裹在双下划线中来表示某些东西是特殊的。一个被双下划线包裹的特殊标记通常被称为“dunder”。“特殊”不是一个技术术语，它只是意味着该标记调用了Python语言功能。一些dunder标记只是代码作者可以插入特定语法糖的方式，其他是由解释器提供，否则很难获得的值，还有的是用于扩展语言接口的（例如，迭代器协议），最后，一小部分dunders是真正难以理解的。幸运的是，作为R用户，你只需要了解一些容易理解的dunders。

当你阅读Python代码时，最常遇到的dunder方法是`__init__()`。这是一个在类构造函数被调用时调用的函数，也就是说，当一个类被**实例化**时。它的目的是初始化新类实例。（在非常复杂的代码库中，你可能还会遇到定义了`__new__`的类，这在`__init__`之前被调用）。

```python
class MyClass:

  print("MyClass's definition body is being evaluated")

  def __init__(self):
    print(self, "is initializing")
#> MyClass's definition body is being evaluated

print("MyClass is finished being created")
#> MyClass is finished being created

instance = MyClass()
#> <__main__.MyClass object at 0x140266330> is initializing
print(instance)
#> <__main__.MyClass object at 0x140266330>

instance2 = MyClass()
#> <__main__.MyClass object at 0x11e3ad490> is initializing
print(instance2)
#> <__main__.MyClass object at 0x11e3ad490>
```

请注意几点：

- `class`语句接受一个代码块，该代码块定义的公共缩进级别。代码块具有与其他任何接受代码块的表达式（如`if`和`def`）完全相同的语义。类的主体只在**第一次**被评估，即在类构造函数第一次被创建时。注意，这里定义的任何对象都由类的所有实例共享！

- `__init__`只是一个普通函数，像任何其他函数一样用`def`定义。除了它在类体内部定义。

- `__init__`接受一个参数：`self`。
`self`是正在初始化的类实例（注意`self`和`instance`之间的相同内存地址）。还要注意，我们调用`MyClass()`创建类实例时没有提供`self`，`self`被解释器插入到函数调用中。

- 每次创建新实例时都会调用`__init__`。

在`class`代码块中定义的函数称为方法，重要的是要知道每次从类实例调用方法时，实例都会被插入到函数调用中作为第一个参数。这适用于类中定义的所有函数，包括dunders。（唯一的例外是如果函数被装饰有类似`@classmethod`或`@staticmethod`的装饰器）。

```python
class MyClass:
  def a_method(self):
    print("MyClass.a_method() was called with", self)

instance = MyClass()
instance.a_method()
#> MyClass.a_method() was called with <__main__.MyClass object at 0x11e3c7f20>

MyClass.a_method()     # 错误，缺少必需的参数`self`
#> TypeError: MyClass.a_method() missing 1 required positional argument: 'self'
MyClass.a_method(instance) # 与instance.a_method()相同
#> MyClass.a_method() was called with <__main__.MyClass object at 0x11e3c7f20>
```

其他一些值得了解的dunder包括：

- `__getitem__`：当使用`[`进行子集操作时调用的函数（等同于在R中定义`[` S3方法）。

- `__getattr__`：当使用`.`进行子集操作时调用的函数（等同于在R中定义`$` S3方法）。

- `__iter__`和`__next__`：由`for`调用的函数。

- `__call__`：当类实例像函数一样被调用时调用（例如，`instance()`）。

- `__bool__`：由`if`和`while`调用（等同于R中的`as.logical()`，但只返回标量，而不是向量）。

- `__repr__`, `__str__`, 用于格式化和漂亮的打印函数（类似于R中的`format()`, `dput()`, 和`print()`方法）。

- `__enter__`和`__exit__`：由`with`调用的函数。

- 许多内置的Python函数只是调用dunder的糖，例如：调用`repr(x)`与`x.__repr__()`相同。
其他内置函数只是调用dunder的糖，包括`next()`, `iter()`, `str()`, `list()`, `dict()`, `bool()`, `dir()`, `hash()`等等！

#### 迭代器，再次访问

现在我们已经掌握了`class`的基础知识，是时候重新访问迭代器了。首先，一些术语：

**可迭代**：可以被迭代的东西。
具体来说，是一个定义了`__iter__`方法的类，其工作是返回一个**迭代器**。

**迭代器**：迭代的东西。
具体来说，是一个定义了`__next__`方法的类，其工作是每次被调用时返回下一个元素，然后在耗尽时抛出一个`StopIteration`异常。

常见的是看到既是可迭代又是迭代器的类，其中`__iter__`方法只是一个返回`self`的存根。

这是一个Python`range`的自定义可迭代/迭代器实现（类似于R中的`seq`）

```python
class MyRange:
  def __init__(self, start, end):
    self.start = start
    self.end = end

  def __iter__(self):
    # 重置我们的计数器。
    self._index = self.start - 1
    return self

  def __next__(self):
    if self._index < self.end:
      self._index += 1 # 增加
      return self._index
    else:
      raise StopIteration

for x in MyRange(1, 3):
  print(x)
#> 1
#> 2
#> 3

# 手动做`for`做的事情
r = MyRange(1, 3)
it = iter(r)
next(it)
#> 1
next(it)
#> 2
next(it)
#> 3
next(it)
#> StopIteration
```

### 使用`yield`定义生成器

生成器是包含一个或多个`yield`语句的特殊Python函数。一旦在传递给`def`的代码块中包含`yield`，语义就发生了显著变化。你不再仅仅定义一个函数，而是一个生成器构造函数！反过来，调用生成器构造函数会创建一个生成器对象，这只是另一种类型的迭代器。

示例：

```python
def my_generator_constructor():
  yield 1
  yield 2
  yield 3

# 乍一看，它看起来像一个常规函数
my_generator_constructor
#> <function my_generator_constructor at 0x1402579c0>
type(my_generator_constructor)
#> <class 'function'>

# 但是调用它返回一些特别的东西，一个'generator object'>
my_generator = my_generator_constructor()
my_generator
#> <generator object my_generator_constructor at 0x11e3ff530>
type(my_generator)
#> <class 'generator'>

# 生成器对象既是可迭代的也是迭代器
# 它的`__iter__`方法是只是一个返回`self`的存根
iter(my_generator) == my_generator == my_generator.__iter__()
#> True

# 像任何其他迭代器一样逐步进行
next(my_generator)
#> 1
my_generator.__next__() # next()只是调用dunder的糖
#> 2
next(my_generator)
#> 3
next(my_generator)
#> StopIteration
```

遇到`yield`就像按下函数执行的暂停按钮，它保留了函数体中所有内容的状态，并将控制权返回给迭代生成器对象的任何东西。调用生成器对象上的`next()`会恢复函数体的执行，直到遇到下一个`yield`，或者函数结束。

### `import`和模块

在R中，作者可以将他们的代码捆绑到可共享的扩展中，称为R包，R用户可以通过`library()`或`::`从R包中访问对象。在Python中，作者将代码捆绑到**模块**中，用户使用`import`访问模块。考虑这行代码：

这行语句让Python去文件系统，找到一个名为‘numpy’的已安装Python模块，加载它（通常意味着：评估它的`__init__.py`文件并构建一个`module`类型），并将其绑定到符号`numpy`。

在R中，这最接近于：

```r
dplyr <- loadNamespace("dplyr")
```

#### 模块在哪里找到？

在Python中，模块被搜索的文件系统位置可以从`sys.path`列表中访问（和修改）。这是Python的`.libPaths()`的等价物。
`sys.path`通常包含当前工作目录、包含内置标准库的Python安装、管理员安装的模块、用户安装的模块、来自环境变量如`PYTHONPATH`的值，以及其他代码在当前Python会话中直接对`sys.path`所做的任何修改（尽管这在实践中相对不常见）。

```python
import sys
sys.path
#> ['', '/Users/tomasz/.pyenv/versions/3.12.4/bin', '/Users/tomasz/.pyenv/versions/3.12.4/lib/python312.zip', '/Users/tomasz/.pyenv/versions/3.12.4/lib/python3.12', '/Users/tomasz/.pyenv/versions/3.12.4/lib/python3.12/lib-dynload', '/Users/tomasz/.virtualenvs/r-reticulate/lib/python3.12/site-packages', '/Users/tomasz/github/rstudio/reticulate/inst/python', '/Users/tomasz/.virtualenvs/r-reticulate/lib/python312.zip', '/Users/tomasz/.virtualenvs/r-reticulate/lib/python3.12', '/Users/tomasz/.virtualenvs/r-reticulate/lib/python3.12/lib-dynload']
```

你可以通过访问dunder `__path__`或`__file__`来检查模块被加载的位置（在解决安装问题时特别有用）：

```python
import os
os.__file__
#> '/Users/tomasz/.virtualenvs/r-reticulate/lib/python3.12/os.py'
numpy.__path__
#> ['/Users/tomasz/.virtualenvs/r-reticulate/lib/python3.12/site-packages/numpy']
```

一旦加载了模块，你就可以使用`.`来访问模块中的符号（等同于`::`，或者可能是R中的`$.environment`）。

还有特殊的语法来指定导入时模块绑定到的符号，以及导入特定符号。

```python
import numpy        # 导入
import numpy as np  # 导入并绑定到自定义符号`np`
np is numpy         # 测试相同性，类似于identical(np, numpy)
#> True

from numpy import abs # 只导入`numpy.abs`，绑定到`abs`
abs is numpy.abs
#> True

from numpy import abs as abs2 # 只导入`numpy.abs`，绑定到`abs2`
abs2 is numpy.abs
#> True
```

如果你在寻找R的`library()`的Python等价物，它使包的所有导出符号可用，可能是使用`import`和`*`

通配符，尽管这样做相对不常见。`*`通配符将扩展以包括模块中的所有符号，或者如果定义了`__all__`，则包括`__all__`中列出的所有符号。

Python不像R那样区分包导出和内部符号。在Python中，所有模块符号都是平等的，尽管有命名约定，即打算为内部使用的符号都带有单个前导下划线。（两个前导下划线调用高级语言功能称为“名称修饰”，这超出了本介绍的范围）。

### 整数和浮点数

R用户通常不需要知道整数和浮点数之间的区别，但Python不是这种情况。
如果这是你第一次接触数值数据类型，以下是基础知识：

- 整数类型只能表示像`1`或`2`这样的整数，不能表示像`1.2`这样的浮点数。

- 浮点类型可以表示任何数字，但有一定的精度损失。

在R中，像`12`这样的裸字面量数字会产生浮点类型，而在Python中，它会产生整数。你可以在R中通过添加`L`后缀来产生整数字面量，如`12L`。许多Python函数期望整数，并在提供浮点数时会出错。

例如，假设我们有一个期望整数的Python函数：

```python
def a_strict_Python_function(x):
  assert isinstance(x, int), "x is not an int"
  print("Yay! x was an int")
```

从R调用它时，你必须确保用整数调用它：

```r
library(reticulate)
py$a_strict_Python_function(3)             # 错误
#> x is not an int
py$a_strict_Python_function(3L)            # 成功
#> Yay! x was an int
py$a_strict_Python_function(as.integer(3)) # 成功
#> Yay! x was an int
```

### R向量怎么样？

R是一种首先为数值计算而设计的语言。数值向量数据类型深深植根于R语言中，以至于语言甚至不区分标量和向量。相比之下，Python中的数值计算能力通常由第三方包（Python术语中的**模块**）提供。

在Python中，`numpy`模块最常用于处理数据的连续数组。最接近R数值向量的等价物是numpy数组，有时是标量数字列表（一些Python用户可能会在这里支持`array.array()`，但在实际Python代码中很少遇到，我们不再进一步提及）。

教授NumPy接口超出了本初级指南的范围，但值得指出一些对习惯于R数组的用户可能遇到的陷阱：

- 当索引到多维numpy数组时，可以省略尾随维度，并隐式地视为缺失。结果是，迭代数组意味着迭代第一个维度。例如，这会迭代矩阵的行。

```python
import numpy as np
m = np.arange(12).reshape((3,4))
m
#> array([[ 0,  1,  2,  3],
#>        [ 4,  5,  6,  7],
#>        [ 8,  9, 10, 11]])
m[0, :] # 第一行
#> array([0, 1, 2, 3])
m[0]    # 也是第一行
#> array([0, 1, 2, 3])

for row in m:
  print(row)
#> [0 1 2 3]
#> [4 5 6 7]
#> [ 8  9 10 11]
```

- 许多numpy操作会就地修改数组！这可能会让习惯于R的复制修改语义的R用户感到惊讶。不幸的是，没有简单的方案或命名约定可以依靠，以快速确定特定方法是否就地修改或创建新数组副本。唯一可靠的方法是查阅文档，并在`reticulate::repl_python()`中进行小实验。

### 装饰器

装饰器只是接受函数作为参数的函数，然后通常返回另一个函数。任何函数都可以被调用为装饰器，使用`@`语法，这只是糖，用于这个简单的动作：

```python
def my_decorator(func):
  func.x = "a decorator modified this function by adding an attribute `x`"
  return func

def my_function(): pass
my_function = my_decorator(my_function)

# @只是上述两行的花哨语法
@my_decorator
def my_function(): pass
```

你可能会经常遇到的一个装饰器是：

- `@property`，当访问属性时自动调用类方法（类似于R中的`makeActiveBinding()`）。

### `with`和上下文管理

任何定义了`__enter__`和`__exit__`方法的对象都实现了“上下文”协议，并且可以传递给`with`。例如，这是一个自定义的上下文管理器实现，暂时更改当前工作目录（类似于R的`withr::with_dir()`）

```python
from os import getcwd, chdir

class wd_context:
  def __init__(self, wd):
    self.new_wd = wd

  def __enter__(self):
    self.original_wd = getcwd()
    chdir(self.new_wd)

  def __exit__(self, *args):
    # __exit__接受一些通常被忽略的额外参数
    chdir(self.original_wd)

getcwd()
#> '/Users/tomasz/github/rstudio/reticulate/vignettes'
with wd_context(".."):
  print("in the context, wd is:", getcwd())
#> in the context, wd is: /Users/tomasz/github/rstudio/reticulate
getcwd()
#> '/Users/tomasz/github/rstudio/reticulate/vignettes'
```

### 了解更多

希望这个简短的Python入门指南为你提供了一个良好的基础，让你有信心阅读Python文档和代码，并通过reticulate从R中使用Python模块。当然，还有更多关于Python的知识需要学习。在Google上搜索Python问题通常会得到大量的结果，但并不总是按照最有用排序。针对初学者的博客文章和教程可能很有价值，但请记住，Python的官方文档通常非常好，当你有问题时，它应该是你的第一个目的地。

https://docs.python.org/3/

https://docs.python.org/3/library/index.html

要更全面地学习Python，内置的官方教程也非常好和全面（但这需要时间承诺才能从中获得价值）https://docs.python.org/3/tutorial/index.html

最后，不要忘记通过在`reticulate::repl_python()`中进行小实验来巩固你的理解。

感谢阅读！

