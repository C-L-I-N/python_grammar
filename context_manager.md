# 🔧 上下文管理（Context Manager）详解

上下文管理是 Python 中一个非常优雅的资源管理机制，通过 `with` 关键字实现。

---

## 📖 一、为什么需要上下文管理？

想象一个场景：你打开了一个文件，读取内容后需要关闭它。

### ❌ 传统写法（容易出错）

```python
f = open('data.txt', 'r')
content = f.read()
f.close()  # 如果上面出错，这行永远不会执行！
```

**问题**：如果 `f.read()` 发生异常，`f.close()` 不会被执行，文件句柄泄漏！

### ⚠️ 改进写法（繁琐）

```python
f = open('data.txt', 'r')
try:
    content = f.read()
finally:
    f.close()  # 无论如何都会关闭
```

### ✅ 上下文管理器（优雅）

```python
with open('data.txt', 'r') as f:
    content = f.read()
# 自动关闭！即使发生异常也会关闭
```

---

## 📖 二、工作原理

`with` 语句背后的机制：

```
进入 with 块 → 调用 __enter__() → 执行代码块 → 调用 __exit__()
                    ↓                              ↓
              返回给 as 变量                  清理资源（即使异常）
```

### 执行流程

1. **进入**：调用对象的 `__enter__()` 方法
2. **绑定**：将 `__enter__()` 的返回值赋给 `as` 后的变量
3. **执行**：执行 `with` 代码块
4. **退出**：无论是否发生异常，都调用 `__exit__()` 方法

---

## 📖 三、常见使用场景

### 1️⃣ 文件操作

```python
# 读文件
with open('input.txt', 'r', encoding='utf-8') as f:
    content = f.read()

# 写文件
with open('output.txt', 'w', encoding='utf-8') as f:
    f.write("Hello, World!")

# 同时操作多个文件
with open('a.txt') as f1, open('b.txt') as f2:
    data1 = f1.read()
    data2 = f2.read()

# Python 3.10+ 可以用括号换行
with (
    open('a.txt') as f1,
    open('b.txt') as f2,
    open('c.txt') as f3
):
    pass
```

### 2️⃣ 数据库连接

```python
import sqlite3

with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    rows = cursor.fetchall()
# 连接自动关闭
```

### 3️⃣ 线程锁

```python
import threading

lock = threading.Lock()

with lock:
    # 临界区 - 只有一个线程能执行
    shared_resource += 1
# 锁自动释放
```

### 4️⃣ 网络请求

```python
import requests

with requests.Session() as session:
    response = session.get('https://api.example.com/data')
    data = response.json()
# Session 自动关闭
```

### 5️⃣ 临时目录

```python
import tempfile

with tempfile.TemporaryDirectory() as tmpdir:
    # 在临时目录中操作
    filepath = f"{tmpdir}/temp.txt"
    with open(filepath, 'w') as f:
        f.write("临时数据")
# 临时目录自动删除
```

---

## 📖 四、自定义上下文管理器

### 方法一：类实现（实现 `__enter__` 和 `__exit__`）

```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        print("打开文件...")
        self.file = open(self.filename, self.mode)
        return self.file  # 返回给 as 后的变量
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("关闭文件...")
        if self.file:
            self.file.close()
        # 返回 True 表示异常已处理，不再抛出
        # 返回 False 或 None 表示异常继续抛出
        return False

# 使用
with FileManager('test.txt', 'w') as f:
    f.write("Hello!")
# 输出:
# 打开文件...
# 关闭文件...
```

### 方法二：使用 `@contextmanager` 装饰器（更简洁）

```python
from contextlib import contextmanager
import time

@contextmanager
def timer():
    start = time.time()
    yield  # yield 之前是 __enter__，之后是 __exit__
    elapsed = time.time() - start
    print(f"耗时: {elapsed:.4f} 秒")

# 使用
with timer():
    time.sleep(1)
# 输出: 耗时: 1.00xx 秒
```

### 带返回值的 contextmanager

```python
from contextlib import contextmanager

@contextmanager
def open_file(name, mode):
    f = open(name, mode)
    try:
        yield f  # yield 的值会传给 as
    finally:
        f.close()

with open_file('test.txt', 'r') as f:
    content = f.read()
```

---

## 📖 五、`__exit__` 的参数详解

```python
class MyContext:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """
        参数说明：
        - exc_type: 异常类型（如 ValueError, TypeError）
        - exc_val:  异常值/消息（如 "invalid value"）
        - exc_tb:   异常追踪信息（traceback 对象）
        
        如果没有异常，三个参数都是 None
        """
        if exc_type is not None:
            print(f"捕获异常: {exc_type.__name__}: {exc_val}")
            return True  # 返回 True 吞掉异常，不再向外抛出
        return False  # 返回 False 或 None，异常继续抛出
```

### 异常处理示例

```python
class SuppressError:
    """抑制特定类型的异常"""
    def __init__(self, *exceptions):
        self.exceptions = exceptions
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None and issubclass(exc_type, self.exceptions):
            print(f"已抑制异常: {exc_val}")
            return True  # 吞掉异常
        return False

# 使用
with SuppressError(ValueError, KeyError):
    raise ValueError("这个异常会被抑制")
print("程序继续执行")

# 也可以用标准库
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove('不存在的文件.txt')
```

---

## 📖 六、实用案例

### 计时器

```python
import time
from contextlib import contextmanager

@contextmanager
def timer(label="代码块"):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label} 耗时: {elapsed:.4f} 秒")

# 使用
with timer("数据处理"):
    data = [i ** 2 for i in range(1000000)]
```

### 临时修改工作目录

```python
import os
from contextlib import contextmanager

@contextmanager
def change_dir(path):
    old_dir = os.getcwd()
    os.chdir(path)
    try:
        yield
    finally:
        os.chdir(old_dir)

# 使用
with change_dir('/tmp'):
    print(os.getcwd())  # /tmp
print(os.getcwd())  # 恢复原目录
```

### 重定向输出

```python
import sys
from contextlib import contextmanager

@contextmanager
def redirect_stdout(file):
    old_stdout = sys.stdout
    sys.stdout = file
    try:
        yield
    finally:
        sys.stdout = old_stdout

# 使用
with open('output.log', 'w') as f:
    with redirect_stdout(f):
        print("这会写入文件而不是控制台")
```

### 数据库事务

```python
from contextlib import contextmanager

@contextmanager
def transaction(connection):
    try:
        yield connection
        connection.commit()
        print("事务提交成功")
    except Exception as e:
        connection.rollback()
        print(f"事务回滚: {e}")
        raise

# 使用
with transaction(db_connection) as conn:
    conn.execute("INSERT INTO users VALUES (...)")
    conn.execute("UPDATE accounts SET ...")
# 自动提交，出错自动回滚
```

---

## 📖 七、contextlib 模块常用工具

```python
from contextlib import (
    contextmanager,      # 装饰器，简化创建上下文管理器
    suppress,            # 抑制指定异常
    redirect_stdout,     # 重定向标准输出
    redirect_stderr,     # 重定向标准错误
    closing,             # 确保调用 close() 方法
    nullcontext,         # 空上下文（占位符）
    ExitStack,           # 动态管理多个上下文管理器
)

# suppress 示例
with suppress(FileNotFoundError):
    os.remove('不存在.txt')

# closing 示例（为没有实现上下文协议的对象添加支持）
from urllib.request import urlopen
with closing(urlopen('http://example.com')) as page:
    content = page.read()

# nullcontext 示例（条件性使用上下文管理器）
def process(file=None):
    cm = open(file) if file else nullcontext()
    with cm as f:
        data = f.read() if f else "默认数据"
```

---

## 📝 总结

| 特性 | 说明 |
|------|------|
| **语法** | `with 表达式 as 变量:` |
| **作用** | 自动管理资源的获取和释放 |
| **原理** | 调用 `__enter__` 和 `__exit__` 魔术方法 |
| **优点** | 代码简洁、安全（即使异常也能清理资源） |
| **创建方式** | 1. 类实现 `__enter__`/`__exit__` <br> 2. 使用 `@contextmanager` 装饰器 |
| **常用场景** | 文件、数据库连接、锁、网络连接、临时状态 |

> 💡 **黄金法则**：任何需要"获取-使用-释放"模式的资源，都应该使用上下文管理器！

---

## 🌟 记忆口诀

```
with 语句三步走：
进入调用 enter 头，
中间代码随便走，
退出 exit 扫尾收。
异常来了也不愁，
资源清理样样有！
```

