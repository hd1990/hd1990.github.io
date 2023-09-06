---
layout: post
title: "python进阶（一）: *args 和 **kwargs, pdb, Map, Reduce, Filter"
date: 2023-01-17
description: "主要包括python一些不太常见的知识点以及python高阶编程需要用到的知识点等等"
tag: Python
---   

主要包括python一些不太常见的知识点以及python高阶编程需要用到的知识点等等。
比如说*args 和 **kwargs在函数传参以及函数调用中的用法，以及python中的装饰器等等。
还比如说pdb在程序运行时查看变量的值，查看程序的执行流程，以及查看函数的调用栈。
还例如，python中的Map、Reduce、Filter、Lambda等在函数式编程中的用法。

## *args 和 **kwargs

其实并不是必须写成\*args 和\**kwargs。 只有变量前面的 \*(星号)才是必须的. 你也可以写成\*var 和\**vars. 而写成\*args 和\**kwargs只是一个通俗的命名约定。

\*args 和 \**kwargs 主要用于函数定义。 你可以将不定数量的参数传递给一个函数。

预先并不知道, 函数使用者会传递多少个参数给你, 所以在这个场景下使用这两个关键字。

**\*args** 是用来发送一个非键值对的可变数量的参数列表给一个函数.

```python
def test_var_args(f_arg, *argv):
    print("first normal arg:", f_arg)
    for arg in argv:
        print("another arg through *argv:", arg)

test_var_args('yasoob', 'python', 'eggs', 'test')
```
这会产生如下输出:
```console
first normal arg: yasoob
another arg through *argv: python
another arg through *argv: eggs
another arg through *argv: test
```

**\*\*kwargs** 允许你将不定长度的键值对, 作为参数传递给一个函数. 如果你想要在一个函数里处理带名字的参数, 你应该使用**\*\*kwargs**.

举个例子：

```python
def greet_me(**kwargs):
    for key, value in kwargs.items():
        print("{0} == {1}".format(key, value)) # .format()是python3的新特性, 用于格式化字符串,{0}表示第一个参数,{1}表示第二个参数
        # 也可以用%来格式化字符串
        print("%s == %s" % (key, value)) %s表示字符串,%d表示整数,%f表示浮点数
gree_me(name="yasoob")
```
这会产生如下输出:
```console
name == yasoob
name == yasoob
```

### 使用 *args 和 **kwargs 来调用函数

那现在我们将看到怎样使用*args和**kwargs 来调用一个函数。 假设，你有这样一个小函数：
```python
def test_args_kwargs(arg1, arg2, arg3):
    print("arg1:", arg1)
    print("arg2:", arg2)
    print("arg3:", arg3)
```
我们可以使用下面的方式来调用这个函数：
```python
# first with *args
args = ("two", 3, 5)
test_args_kwargs(*args)
# now with **kwargs:
kwargs = {"arg3": 3, "arg2": "two", "arg1": 5}
test_args_kwargs(**kwargs)
```
这会产生如下输出:
```console
arg1: two
arg2: 3
arg3: 5
arg1: 5
arg2: two
arg3: 3
```
### 使用顺序

那么如果你想在函数里同时使用所有这三种参数， 顺序是这样的：
```python
some_func(fargs, *args, **kwargs)
```

```python
def test_args_kwargs(arg1, arg2, arg3, *args, **kwargs):
    print("arg1:", arg1)
    print("arg2:", arg2)
    print("arg3:", arg3)
    for arg in args:
        print("another arg:", arg)
    for kwarg in kwargs:
        print("another keyword arg: {0} = {1}".format(kwarg, kwargs[kwarg]))
```

### 什么时候使用它们？
要看你的需求而定，最常见的用例是在写函数装饰器的时候，此外它也可以用来做猴子补丁(monkey patching)。猴子补丁的意思是在程序运行时(runtime)修改某些代码。

打个比方，你有一个类，里面有个叫get_info的函数会调用一个API并返回相应的数据。如果我们想测试它，可以把API调用替换成一些测试数据。例如：

```python
import someclass # someclass是你定义的类

def get_info(self, *args): # 这里的self是类的实例，*args是可变参数，可以传入任意多个测试数据
    return "Test data"

someclass.get_info = get_info
```

## 调试（Debugging）

利用好调试，能大大提高你捕捉代码Bug的。
Python debugger(pdb)可以帮助我们在程序运行时查看变量的值，查看程序的执行流程，以及查看函数的调用栈。

pdb 模块定义了一个交互式源代码调试器，用于 Python 程序。它支持在源码行间设置（有条件的）断点和单步执行，检视堆栈帧，列出源码列表，以及在任何堆栈帧的上下文中运行任意 Python 代码。它还支持事后调试，可以在程序控制下调用。

调试器是可扩展的——调试器实际被定义为 Pdb 类。该类目前没有文档，但通过阅读源码很容易理解它。扩展接口使用了 bdb 和 cmd 模块。

### pdb
#### 从命令行运行
将 pdb.py 作为脚本调用，来调试其他脚本。你可以在命令行使用Python debugger运行一个脚本， 举个例子：
```commandline
python -m pdb myscript.py
```
这会触发debugger在脚本第一行指令处停止执行。这在脚本很短时会很有帮助。你可以通过(Pdb)模式接着查看变量信息，并且逐行调试。

#### 从脚本内部运行


同时，你也可以在脚本内部设置断点，这样就可以在某些特定点查看变量信息和各种执行时信息了。这里将使用**pdb.set_trace()**方法来实现。举个例子：
```python
import pdb
def make_bread():
    pdb.set_trace() # 这里设置断点,程序会在这里停止,你可以查看运行到这里时的变量信息，然后按任意键继续执行，或者按c继续执行到下一个断点
    return "I don't have time"
```
试下保存上面的脚本后运行之。你会在运行时马上进入debugger模式。现在是时候了解下debugger模式下的一些命令了。

| 命令 | 简写 | 说明 |
| --- | --- | --- |
| help | h | 显示帮助信息 |
| list | l | 查看当前行的代码段|
| continue | c | 继续执行到下一个断点 |
| quit | q | 退出debugger |
| print | p | 打印变量的值 |
| args | a | 打印函数的参数 |
| break | b | 设置断点 |
| bt |  | 打印函数的调用栈 |
| return | r | 打印函数的返回值 |
| next | n | 执行下一行代码 |
| step | s | 进入函数内部 |

单步跳过（next）和单步进入（step）的区别在于， 单步进入会进入当前行调用的函数内部并停在里面， 而单步跳过会（几乎）全速执行完当前行调用的函数，并停在当前函数的下一行。

## Map，Filter 和 Reduce

Map，Filter 和 Reduce 三个函数能为函数式编程提供便利。我们会通过实例一个一个讨论并理解它们。

### Map
Map会将一个函数映射到一个输入列表的所有元素上。这是它的规范：

```python
map(function_to_apply, list_of_inputs)
```

大多数时候，我们要把列表中所有元素一个个地传递给一个函数，并收集输出。比方说：

```python
items = [1, 2, 3, 4, 5]
squared = []
for i in items:
    squared.append(i**2)
```

这里我们使用了一个循环来计算列表中每个元素的平方Map可以让我们用一种简单而漂亮得多的方式来实现。

```python
items = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, items)) # map()返回一个迭代器，所以我们需要将它转换为一个列表
```


不仅用于一列表的输入， 我们甚至可以用于一列表的函数！也即是说，list_of_inputs不仅可以是一个列表，还可以是一个函数列表。比方说：

```python
def multiply(x):
    return (x*x)
def add(x):
    return (x+x)
def exponent(x):
    return (x**x) # x的x次方

funcs = [multiply, add,exponent] # 这里我们有一个函数列表,我们可以将它传递给map,
for i in range(5):
    value = list(map(lambda x: x(i), funcs)) # x(i)将会调用funcs中的函数,并传递i作为参数
    print(list(value))
```

输出结果为：
```console
[0, 0, 1]
[1, 2, 1]
[4, 4, 4]
[9, 6, 27]
[16, 8, 256]
```

### Filter

顾名思义，filter过滤列表中的元素，并且返回一个由所有符合要求的元素所构成的列表，符合要求即函数映射到该元素时返回值为True. 这里是一个简短的例子：

```python
number_list = range(-5, 5)
less_than_zero = filter(lambda x: x < 0, number_list) # lambda x: x < 0是一个函数，它将会映射到number_list中的每个元素上,符合要求的元素将会被保留
print(list(less_than_zero))  
```
输出结果为：
```console
[-5, -4, -3, -2, -1]
```
这个filter类似于一个for循环，但它是一个内置函数，并且更快。
注意：如果map和filter对你来说看起来并不优雅的话，那么你可以看看列表/字典/元组推导式。

### Reduce

当需要对一个列表进行一些计算并返回结果时，Reduce 是个非常有用的函数。举个例子，当你需要计算一个整数列表的乘积时。

通常在 python 中你可能会使用基本的 for 循环来完成这个任务。

现在我们来试试 reduce：

```python
from functools import reduce # python3中reduce被移到了functools模块中
product = reduce((lambda x, y: x * y), [1, 2, 3, 4]) # reduce()函数将会对列表中的元素进行迭代计算，x是上一次计算的结果，y是下一个元素，这里我们计算的是1*2*3*4，第一次迭代时x=1,y=2,第二次迭代时x=2,y=3,第三次迭代时x=6,y=4，最后保留的结果为24，是x的最终值。
print(product)
```
输出结果为：
```console
24
```
