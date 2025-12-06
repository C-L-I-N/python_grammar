# 🔑 Python 关键字语法规则汇总

Python 关键字是**保留字**，不能用作变量名、函数名或任何其他标识符。

---

## 📋 全部关键字一览（Python 3.11+）

```python
import keyword
print(keyword.kwlist)
```

共 **35** 个关键字：

| 类别 | 关键字 |
|------|--------|
| 布尔/空值 | `True`, `False`, `None` |
| 控制流 | `if`, `elif`, `else`, `for`, `while`, `break`, `continue`, `pass` |
| 逻辑运算 | `and`, `or`, `not` |
| 函数相关 | `def`, `return`, `lambda`, `yield` |
| 类相关 | `class` |
| 异常处理 | `try`, `except`, `finally`, `raise`, `assert` |
| 导入模块 | `import`, `from`, `as` |
| 作用域 | `global`, `nonlocal` |
| 上下文管理 | `with` |
| 成员检查 | `in`, `is` |
| 删除 | `del` |
| 异步 | `async`, `await` |
| 匹配 | `match`, `case` (3.10+) |

---

## 📖 一、布尔值与空值

### `True` / `False` — 布尔值

```python
is_active = True
is_deleted = False

# 布尔运算
print(True and False)   # False
print(True or False)    # True
print(not True)         # False
```

### `None` — 空值

```python
result = None  # 表示"什么都没有"

def greet():
    print("Hello")
    # 没有 return，默认返回 None

x = greet()
print(x)  # None

# 判断是否为 None（推荐用 is）
if result is None:
    print("没有值")
```

---

## 📖 二、条件判断

### `if` / `elif` / `else`

```python
score = 85

if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

> 💡 **注意**：Python 使用缩进来表示代码块，不用 `{}` 花括号！

### 三元表达式

```python
age = 20
status = "成年" if age >= 18 else "未成年"
```

---

## 📖 三、循环控制

### `for` — 遍历循环

```python
# 遍历列表
fruits = ['apple', 'banana', 'cherry']
for fruit in fruits:
    print(fruit)

# 遍历范围
for i in range(5):      # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):   # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2):  # 0, 2, 4, 6, 8 (步长为2)
    print(i)

# 遍历字典
person = {'name': '张三', 'age': 25}
for key, value in person.items():
    print(f"{key}: {value}")

# 带索引遍历
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
```

### `while` — 条件循环

```python
count = 0
while count < 5:
    print(count)
    count += 1

# 无限循环（需配合 break）
while True:
    user_input = input("输入 'quit' 退出: ")
    if user_input == 'quit':
        break
```

### `break` — 跳出循环

```python
for i in range(10):
    if i == 5:
        break  # 立即退出整个循环
    print(i)
# 输出: 0, 1, 2, 3, 4
```

### `continue` — 跳过当前迭代

```python
for i in range(10):
    if i % 2 == 0:
        continue  # 跳过偶数，继续下一次循环
    print(i)
# 输出: 1, 3, 5, 7, 9
```

### `pass` — 占位符（什么都不做）

```python
# 定义空函数或空类时使用
def my_function():
    pass  # TODO: 稍后实现

class MyClass:
    pass

# 空循环
for i in range(10):
    pass
```

---

## 📖 四、逻辑运算符

### `and` — 逻辑与

```python
age = 25
has_id = True

if age >= 18 and has_id:
    print("可以购买")

# 短路求值：第一个为 False 就不会计算第二个
result = False and print("不会执行")
```

### `or` — 逻辑或

```python
is_vip = False
has_coupon = True

if is_vip or has_coupon:
    print("享受折扣")

# 常用于设置默认值
name = "" or "匿名用户"  # "匿名用户"
value = None or 100       # 100
```

### `not` — 逻辑非

```python
is_logged_in = False

if not is_logged_in:
    print("请先登录")

# 等价于
if is_logged_in == False:
    print("请先登录")
```

---

## 📖 五、函数相关

### `def` — 定义函数

```python
# 基本函数
def greet(name):
    return f"Hello, {name}!"

# 默认参数
def greet(name="World"):
    return f"Hello, {name}!"

# 可变参数 *args
def sum_all(*numbers):
    return sum(numbers)

print(sum_all(1, 2, 3, 4))  # 10

# 关键字参数 **kwargs
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="张三", age=25)
```

### `return` — 返回值

```python
def add(a, b):
    return a + b

def get_user():
    return "张三", 25  # 返回元组

name, age = get_user()  # 解包

# 提前返回
def validate(value):
    if not value:
        return False
    # 继续处理...
    return True
```

### `lambda` — 匿名函数

```python
# 语法：lambda 参数: 表达式
square = lambda x: x ** 2
print(square(5))  # 25

add = lambda a, b: a + b
print(add(3, 5))  # 8

# 常与 sorted, map, filter 配合
students = [('张三', 85), ('李四', 92), ('王五', 78)]
sorted_students = sorted(students, key=lambda x: x[1], reverse=True)

numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

### `yield` — 生成器

```python
def count_up(n):
    i = 1
    while i <= n:
        yield i  # 暂停并返回值
        i += 1

# 使用生成器
for num in count_up(5):
    print(num)  # 1, 2, 3, 4, 5

# 生成器表达式
squares = (x**2 for x in range(10))
```

---

## 📖 六、类相关

### `class` — 定义类

```python
class Person:
    # 类变量
    species = "人类"
    
    # 构造方法
    def __init__(self, name, age):
        self.name = name  # 实例变量
        self.age = age
    
    # 实例方法
    def greet(self):
        return f"我是{self.name}，{self.age}岁"
    
    # 类方法
    @classmethod
    def get_species(cls):
        return cls.species
    
    # 静态方法
    @staticmethod
    def is_adult(age):
        return age >= 18

# 使用
p = Person("张三", 25)
print(p.greet())

# 继承
class Student(Person):
    def __init__(self, name, age, grade):
        super().__init__(name, age)
        self.grade = grade
```

---

## 📖 七、异常处理

### `try` / `except` / `finally` / `raise`

```python
# 基本异常处理
try:
    result = 10 / 0
except ZeroDivisionError:
    print("不能除以零！")

# 捕获多种异常
try:
    x = int("abc")
except (ValueError, TypeError) as e:
    print(f"发生错误: {e}")

# 捕获所有异常
try:
    risky_operation()
except Exception as e:
    print(f"发生错误: {e}")

# else 和 finally
try:
    result = 10 / 2
except ZeroDivisionError:
    print("除零错误")
else:
    print("计算成功")  # 没有异常时执行
finally:
    print("清理工作")  # 无论如何都执行

# 主动抛出异常
def validate_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄不合理")
    return age
```

### `assert` — 断言

```python
def divide(a, b):
    assert b != 0, "除数不能为零"
    return a / b

# 用于调试和测试
age = 25
assert age >= 0, "年龄必须是正数"
assert isinstance(age, int), "年龄必须是整数"
```

---

## 📖 八、模块导入

### `import` / `from` / `as`

```python
# 导入整个模块
import math
print(math.sqrt(16))  # 4.0

# 导入并重命名
import numpy as np
import pandas as pd

# 导入特定内容
from math import sqrt, pi
print(sqrt(16))  # 4.0
print(pi)        # 3.14159...

# 导入并重命名
from datetime import datetime as dt
now = dt.now()

# 导入所有（不推荐）
from math import *
```

---

## 📖 九、作用域

### `global` — 全局变量

```python
count = 0  # 全局变量

def increment():
    global count  # 声明使用全局变量
    count += 1

increment()
print(count)  # 1
```

### `nonlocal` — 外层函数变量

```python
def outer():
    x = 10
    
    def inner():
        nonlocal x  # 声明使用外层函数的变量
        x += 1
        return x
    
    return inner()

print(outer())  # 11
```

---

## 📖 十、上下文管理

### `with` — 上下文管理器

```python
# 文件操作（自动关闭）
with open('file.txt', 'r') as f:
    content = f.read()
# 文件自动关闭，即使发生异常

# 等价于
f = open('file.txt', 'r')
try:
    content = f.read()
finally:
    f.close()

# 多个上下文管理器
with open('a.txt') as f1, open('b.txt') as f2:
    data1 = f1.read()
    data2 = f2.read()

# 数据库连接、锁等也常用 with
import threading
lock = threading.Lock()
with lock:
    # 临界区代码
    pass
```

---

## 📖 十一、成员检查

### `in` — 成员存在检查

```python
# 列表
fruits = ['apple', 'banana', 'cherry']
print('apple' in fruits)      # True
print('grape' not in fruits)  # True

# 字符串
text = "Hello, World!"
print('World' in text)        # True

# 字典（检查键）
person = {'name': '张三', 'age': 25}
print('name' in person)       # True
print('address' in person)    # False

# 配合循环
for fruit in fruits:
    print(fruit)
```

### `is` — 身份检查

```python
# 检查是否是同一个对象（内存地址）
a = [1, 2, 3]
b = a
c = [1, 2, 3]

print(a is b)      # True（同一对象）
print(a is c)      # False（内容相同但不同对象）
print(a == c)      # True（内容相等）

# 主要用于 None 检查
x = None
if x is None:
    print("x 是 None")

if x is not None:
    print("x 不是 None")
```

> ⚠️ **重要区别**：`==` 比较**值**，`is` 比较**身份（内存地址）**

---

## 📖 十二、删除

### `del` — 删除对象

```python
# 删除变量
x = 10
del x
# print(x)  # NameError

# 删除列表元素
fruits = ['apple', 'banana', 'cherry']
del fruits[1]
print(fruits)  # ['apple', 'cherry']

# 删除切片
del fruits[0:2]

# 删除字典键
person = {'name': '张三', 'age': 25}
del person['age']
print(person)  # {'name': '张三'}

# 删除整个对象
del fruits
```

---

## 📖 十三、异步编程（进阶）

### `async` / `await`

```python
import asyncio

# 定义异步函数
async def fetch_data():
    print("开始获取数据...")
    await asyncio.sleep(2)  # 模拟网络请求
    print("数据获取完成")
    return {"data": "result"}

# 多个异步任务
async def main():
    # 并发执行
    task1 = asyncio.create_task(fetch_data())
    task2 = asyncio.create_task(fetch_data())
    
    result1 = await task1
    result2 = await task2
    
    print(result1, result2)

# 运行
asyncio.run(main())
```

---

## 📖 十四、模式匹配（Python 3.10+）

### `match` / `case`

```python
def http_status(status):
    match status:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:  # 默认情况
            return "Unknown"

# 模式匹配更多用法
def describe_point(point):
    match point:
        case (0, 0):
            return "原点"
        case (0, y):
            return f"Y轴上，y={y}"
        case (x, 0):
            return f"X轴上，x={x}"
        case (x, y):
            return f"点 ({x}, {y})"
        case _:
            return "不是有效的点"
```

---

## 📝 关键字速查表

| 关键字 | 用途 | 示例 |
|--------|------|------|
| `True/False` | 布尔值 | `flag = True` |
| `None` | 空值 | `result = None` |
| `if/elif/else` | 条件判断 | `if x > 0: ...` |
| `for` | 遍历循环 | `for i in range(10): ...` |
| `while` | 条件循环 | `while x > 0: ...` |
| `break` | 跳出循环 | `break` |
| `continue` | 跳过迭代 | `continue` |
| `pass` | 占位符 | `pass` |
| `and/or/not` | 逻辑运算 | `a and b` |
| `def` | 定义函数 | `def func(): ...` |
| `return` | 返回值 | `return x` |
| `lambda` | 匿名函数 | `f = lambda x: x*2` |
| `yield` | 生成器 | `yield value` |
| `class` | 定义类 | `class MyClass: ...` |
| `try/except/finally` | 异常处理 | `try: ... except: ...` |
| `raise` | 抛出异常 | `raise ValueError()` |
| `assert` | 断言 | `assert x > 0` |
| `import/from/as` | 导入模块 | `import math` |
| `global` | 全局变量 | `global x` |
| `nonlocal` | 外层变量 | `nonlocal x` |
| `with` | 上下文管理 | `with open(): ...` |
| `in` | 成员检查 | `'a' in list` |
| `is` | 身份检查 | `x is None` |
| `del` | 删除 | `del x` |
| `async/await` | 异步 | `async def func(): ...` |
| `match/case` | 模式匹配 | `match x: case 1: ...` |

---

## 🌟 总结

掌握Python关键字是编程的基础：

1. **最常用**：`if/else`, `for`, `while`, `def`, `return`, `class`, `import`
2. **重要但容易忽视**：`in`, `is`, `with`, `lambda`
3. **异常处理**：`try/except/finally/raise` 是健壮代码的关键
4. **进阶**：`yield`, `async/await`, `match/case`

> 💡 **建议**：多写代码，在实践中熟悉这些关键字的用法！

