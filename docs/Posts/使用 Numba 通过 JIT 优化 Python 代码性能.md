---
share: true
tags:
- Notes
---
# 什么是 JIT，什么是 Numba
> When a call is made to a Numba decorated function it is compiled to machine code “just-in-time” for execution and all or part of your code can subsequently run at native machine code speed!


JIT（just-in-time）指即时编译，也就是在运行时将代码编译为机器码，JIT的目的是通过编译优化，提高运行速度。特别是Python这种解释型语言，通过JIT可以极大地加速。

Numba是一个用于 Python 的 JIT库，和别的Python JIT方案相比，Numba的好处是可以即插即用地用于现有的Python项目。在已有的Python项目中，只要在某个函数上加上 `@jit`装饰器，就可以在第一次执行该函数时，将函数编译为原生机器码，之后的执行都会直接执行机器码，从而大大提高运行速度。

```python
from numba import jit
import pandas as pd

x = {'a': [1, 2, 3], 'b': [20, 30, 40]}

@jit
def use_pandas(a): # Function will not benefit from Numba jit
    df = pd.DataFrame.from_dict(a) # Numba doesn't know about pd.DataFrame
    df += 1                        # Numba doesn't understand what this is
    return df.cov()                # or this!

print(use_pandas(x))
```
# 什么样的代码可以使用JIT加速
Numba所做的 JIT 本质上是Numba帮助用户实现了一个支持Python子集、一部分Python内置标准库和 Numpy API的编译器。所以，用于JIT的Python代码不能超过Numba所支持的这一部分范围。
此处所说的 JIT 只包括 Numba 的 nopython mode。Numba 还有一种 JIT 模式，叫做 object mode。Object mode 支持在 JIT 函数中调用 Python 解释器，因此能支持编译更广范围的 Python 代码（各种没有 JIT 的自定义类型，例如`pandas.DataFrame`）。但是 object mode 往往不能获得加速，需要尽量避免使用。在默认模式下，`@jit`会首先尝试使用 nopython mode编译，如果失败，会降级到 object mode。为了避免自动降级到 object mode，影响性能，可以使用 `@jit(nopython=True)`或者 `@njit` 。
[官方文档中关于 nopython mode和object mode的详细对比](https://numba.pydata.org/numba-doc/dev/user/performance-tips.html#no-python-mode-vs-object-mode)
> For best performance avoid using this mode!

## Python语言子集
[Supported Python features](https://numba.pydata.org/numba-doc/dev/reference/pysupported.html#)
基本的Python控制流（for，if，while，yield等）都是支持的，也支持list comprehension。对于用于创建 Numpy array 的 list comprehension，Numba还有特殊的优化，可以跳过中间变量，直接创建 array。
```python
from numba import jit
import numpy as np

@jit(nopython=True)
def f(n):
  return np.array([ [ x * y for x in range(n) ] for y in range(n) ])
```
## 容器
Python中的容器（tuple，list，set，dict等等）都可以放不同类型的对象，但是 Numba中的容器除了 tuple 都只能放相同类型的元素。在 Python 中，int 是 float 的子类型，但是在 Numba中，就和 C 一样，int 和 float是不同的类型。
Tuple 的使用和Python几乎没有区别，只有不支持`tuple()`构造函数而已。
List 可以作为JIT函数的输入和输出，Numba会自动推断 list 元素的类型，这被称为 list reflection。但是，list reflection 可能开销较大，特别是对于大的 list。另外，reflected list的元素不能是 reflected type，因为 list-of-list 不能在 Numba 中使用。
Dict 在 Numba 中不能使用，但是 set 可以使用。
作为 list 和 dict 的替代，Numba有 TypedList 和 TypedDict 对象。这两个对象高效地实现了 list和dict 的功能。在 JIT 函数中，调用 `dict()`或者`{}`就会自动创建 TypedDict（在支持 TypedDict 之前的版本，尝试创建字典会编译失败）。
TypedList和TypedDict都可以手动指定类型，因为很多时候自动推断类型都会失败。手动指定类型分别是通过`TypedList.empty_list`和`TypedDict.empty`方法，如下所示：
```python
In [5]: nb.typed.List.empty_list(nb.int64)
Out[5]: ListType[int64]([, ...])

In [6]: nb.typed.Dict.empty(nb.int64, nb.float64)
Out[6]: DictType[int64,float64]<iv=None>({})
```
## Numpy函数
因为 Numba的目标主要是加速计算密集型代码，因此特别支持了科学计算的事实标准，Numpy。支持的 Numpy AP I接口可以参考[官方文档](https://numba.pydata.org/numba-doc/dev/reference/numpysupported.html#)。
但是值得注意的是，几乎所有 Numpy API，在 Numba中都只支持一部分参数 ，一般都不支持 `axis`参数。因此，如果需要在特定维度或者多个维度计算，一般都需要自己写循环。
例如，如下的代码，Numba是不支持的：
```python
np.ones((10, 10)).sum(axis=-1)
```
## 如何指定各种 Python 类型对应的 Numba类型
### 字符串
# 什么样的代码不适合使用JIT加速
## JIT函数调用的开销很大
## 转换数据格式的开销很大
## 避免使用 list 或者 dict 和 JIT函数交互
# 使用 Portable Cache，减少编译耗时
# Takeaways：如何面向 JIT 设计你的代码

