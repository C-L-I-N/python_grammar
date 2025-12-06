# Python 核心概念全解析

## 1. 模块 (Module)

**模块就是一个 `.py` 文件**，包含 Python 代码（函数、类、变量等）。

```python
# module.py 就是一个模块
def add(a, b):
    return a + b

variable = 10
```

**导入方式：**

```python
import module                    # 导入整个模块
from module import add           # 导入特定函数
from module import *             # 导入所有公开内容（不推荐）
import module as m               # 别名导入
```

---

## 2. 包 (Package)

**包是一个包含 `__init__.py` 的目录**，用于组织多个模块。

```
package/
├── __init__.py      # 包的初始化文件（必须存在）
├── module.py
└── subpackage/
    ├── __init__.py
    └── another.py
```

**`__init__.py` 的作用：**

```python
# package/__init__.py
from . import module        # 导入子模块，使 package.module 可用
from .module import add     # 直接暴露函数，使 package.add 可用

__all__ = ['module', 'add'] # 定义 from package import * 时导出的内容
```

---

## 3. 命名空间 (Namespace)

**命名空间是名称到对象的映射**，Python 有四种命名空间（LEGB 规则）：

```python
x = "全局"                    # Global - 全局命名空间

def outer():
    x = "闭包"                # Enclosing - 闭包命名空间
    
    def inner():
        x = "局部"            # Local - 局部命名空间
        print(x)              # 输出: 局部
        print(len)            # Built-in - 内置命名空间
    
    inner()
```

**查找顺序：Local → Enclosing → Global → Built-in**

---

## 4. 作用域 (Scope)

**作用域决定变量在哪里可见**：

```python
global_var = "全局变量"

def func():
    local_var = "局部变量"
    
    global global_var         # 声明使用全局变量
    global_var = "修改全局"
    
def outer():
    outer_var = "外层变量"
    
    def inner():
        nonlocal outer_var    # 声明使用外层变量
        outer_var = "修改外层"
```

---

## 5. 对象 (Object)

**Python 中一切皆对象**。每个对象都有：

- **身份 (Identity)**：`id(obj)` 返回内存地址
- **类型 (Type)**：`type(obj)` 返回类型
- **值 (Value)**：对象存储的数据

```python
x = [1, 2, 3]
print(id(x))      # 140234567890
print(type(x))    # <class 'list'>
print(x)          # [1, 2, 3]

# 可变对象 vs 不可变对象
mutable = [1, 2]      # 列表可变
immutable = (1, 2)    # 元组不可变
```

---

## 6. 类 (Class)

**类是创建对象的蓝图**：

```python
class Person:
    species = "人类"                    # 类属性
    
    def __init__(self, name, age):      # 构造方法
        self.name = name                # 实例属性
        self.age = age
    
    def greet(self):                    # 实例方法
        return f"你好，我是{self.name}"
    
    @classmethod
    def get_species(cls):               # 类方法
        return cls.species
    
    @staticmethod
    def is_adult(age):                  # 静态方法
        return age >= 18
    
    @property
    def info(self):                     # 属性装饰器
        return f"{self.name}, {self.age}岁"

# 继承
class Student(Person):
    def __init__(self, name, age, grade):
        super().__init__(name, age)     # 调用父类构造
        self.grade = grade
```

---

## 7. 函数 (Function)

**函数是可重用的代码块**：

```python
# 基本函数
def add(a, b):
    """文档字符串"""
    return a + b

# 参数类型
def func(pos, /, pos_or_kw, *, kw_only, **kwargs):
    pass
    # pos: 仅位置参数
    # pos_or_kw: 位置或关键字参数
    # kw_only: 仅关键字参数
    # kwargs: 关键字参数字典

# 默认参数（注意：不要用可变对象作默认值！）
def greet(name, greeting="你好"):
    return f"{greeting}, {name}"

# *args 和 **kwargs
def flexible(*args, **kwargs):
    print(args)     # 元组
    print(kwargs)   # 字典

# Lambda 表达式（匿名函数）
square = lambda x: x ** 2
```

---

## 8. 迭代器 (Iterator)

**迭代器是实现了 `__iter__()` 和 `__next__()` 的对象**：

```python
class Counter:
    def __init__(self, max_value):
        self.max = max_value
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.max:
            raise StopIteration
        self.current += 1
        return self.current

# 使用
for num in Counter(3):
    print(num)  # 1, 2, 3

# 内置迭代器函数
iter([1, 2, 3])     # 创建迭代器
next(iterator)       # 获取下一个元素
```

---

## 9. 生成器 (Generator)

**生成器是简化版的迭代器，使用 `yield` 关键字**：

```python
def countdown(n):
    while n > 0:
        yield n       # 暂停并返回值
        n -= 1

# 使用
for i in countdown(3):
    print(i)  # 3, 2, 1

# 生成器表达式
squares = (x**2 for x in range(10))

# 生成器的优势：惰性求值，节省内存
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()
```

---

## 10. 装饰器 (Decorator)

**装饰器用于修改或增强函数/类的行为**：

```python
import time
from functools import wraps

# 基本装饰器
def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"耗时: {time.time() - start:.2f}秒")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

# 带参数的装饰器
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

# 类装饰器
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
```

---

## 11. 上下文管理器 (Context Manager)

**用于管理资源的获取和释放，使用 `with` 语句**：

```python
# 使用类实现
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
        return False  # 不抑制异常

# 使用 contextlib
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.time()
    yield
    print(f"耗时: {time.time() - start:.2f}秒")

with timer():
    # 执行代码
    pass
```

---

## 12. 异常处理 (Exception Handling)

**用于处理程序运行时的错误**：

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"除零错误: {e}")
except (TypeError, ValueError) as e:
    print(f"类型或值错误: {e}")
except Exception as e:
    print(f"其他错误: {e}")
else:
    print("没有异常时执行")
finally:
    print("无论如何都执行")

# 抛出异常
raise ValueError("无效的值")

# 自定义异常
class CustomError(Exception):
    def __init__(self, message, code):
        super().__init__(message)
        self.code = code
```

---

## 13. 魔术方法 (Magic Methods / Dunder Methods)

**以双下划线开头和结尾的特殊方法**：

```python
class Vector:
    def __init__(self, x, y):      # 构造
        self.x, self.y = x, y
    
    def __str__(self):             # str() 和 print()
        return f"Vector({self.x}, {self.y})"
    
    def __repr__(self):            # 调试表示
        return f"Vector({self.x!r}, {self.y!r})"
    
    def __add__(self, other):      # + 运算符
        return Vector(self.x + other.x, self.y + other.y)
    
    def __eq__(self, other):       # == 运算符
        return self.x == other.x and self.y == other.y
    
    def __len__(self):             # len()
        return 2
    
    def __getitem__(self, index):  # 索引访问 obj[i]
        return (self.x, self.y)[index]
    
    def __call__(self, scale):     # 使对象可调用 obj()
        return Vector(self.x * scale, self.y * scale)
```

---

## 14. 描述符 (Descriptor)

**实现属性访问控制的协议**：

```python
class Positive:
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, type=None):
        return obj.__dict__.get(self.name, 0)
    
    def __set__(self, obj, value):
        if value < 0:
            raise ValueError("必须为正数")
        obj.__dict__[self.name] = value

class Account:
    balance = Positive()  # 描述符实例
```

---

## 15. 元类 (Metaclass)

**类的类，控制类的创建过程**：

```python
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    pass

# db1 和 db2 是同一个实例
db1 = Database()
db2 = Database()
```

---

## 概念关系图

```
┌─────────────────────────────────────────────────────────┐
│                    Python 程序                          │
├─────────────────────────────────────────────────────────┤
│  包 (Package)                                           │
│  ├── __init__.py                                        │
│  ├── 模块 (Module) ──────┬── 函数 (Function)            │
│  │                       ├── 类 (Class)                 │
│  │                       │   ├── 属性                   │
│  │                       │   ├── 方法                   │
│  │                       │   └── 魔术方法               │
│  │                       └── 变量                       │
│  └── 子包 (Subpackage)                                  │
├─────────────────────────────────────────────────────────┤
│  特殊机制                                               │
│  ├── 迭代器 / 生成器 ── 惰性求值                        │
│  ├── 装饰器 ── 函数增强                                 │
│  ├── 上下文管理器 ── 资源管理                           │
│  ├── 描述符 ── 属性访问控制                             │
│  └── 元类 ── 类创建控制                                 │
└─────────────────────────────────────────────────────────┘
```

