# 🔄 `yield` 关键字详解

`yield` 是用于创建**生成器（Generator）**的关键字。生成器是一种特殊的迭代器，可以**按需生成**值，而不是一次性把所有值都存储在内存中。

---

## 📖 基本概念

**普通函数** vs **生成器函数**：

```python
# 普通函数 - 一次性返回所有值
def get_numbers():
    return [1, 2, 3, 4, 5]  # 把5个数都装进列表，一次性返回

# 生成器函数 - 每次只生成一个值
def generate_numbers():
    yield 1  # 暂停，返回1
    yield 2  # 暂停，返回2
    yield 3  # 暂停，返回3
    yield 4  # 暂停，返回4
    yield 5  # 暂停，返回5
```

---

## 📖 `yield` 的工作原理

```python
def count_up(n):
    print("生成器启动")
    i = 1
    while i <= n:
        print(f"即将yield: {i}")
        yield i           # ← 暂停执行，返回 i 的值
        print(f"继续执行: {i}")
        i += 1
    print("生成器结束")

# 使用生成器
gen = count_up(3)

print(next(gen))  # 输出: 生成器启动 → 即将yield: 1 → 1
print(next(gen))  # 输出: 继续执行: 1 → 即将yield: 2 → 2
print(next(gen))  # 输出: 继续执行: 2 → 即将yield: 3 → 3
print(next(gen))  # 输出: 继续执行: 3 → 生成器结束 → StopIteration异常
```

**关键点**：
- `yield` 会**暂停**函数执行，保存当前状态
- 下次调用 `next()` 时，从暂停处**继续执行**
- 函数执行完毕时抛出 `StopIteration`

---

## 📖 `yield` vs `return` 对比

| 特性 | `return` | `yield` |
|------|----------|---------|
| 返回次数 | 只能返回一次 | 可以多次 yield |
| 函数状态 | 返回后函数结束 | 暂停，保留状态 |
| 返回类型 | 返回具体值 | 返回生成器对象 |
| 内存占用 | 一次性加载所有数据 | 按需生成，节省内存 |

---

## 📖 实际应用场景

### 1️⃣ 处理大数据（节省内存）

```python
# ❌ 不好：一次性读取所有行到内存
def read_all_lines(filename):
    with open(filename) as f:
        return f.readlines()  # 文件很大时会占用大量内存

# ✅ 好：逐行读取
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()

# 即使文件有10GB，内存也只保留一行
for line in read_lines('huge_file.txt'):
    process(line)
```

### 2️⃣ 无限序列

```python
def infinite_counter(start=0):
    n = start
    while True:  # 无限循环
        yield n
        n += 1

# 按需获取，不会内存溢出
counter = infinite_counter()
print(next(counter))  # 0
print(next(counter))  # 1
print(next(counter))  # 2
# ...可以一直取下去
```

### 3️⃣ 斐波那契数列

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 获取前10个斐波那契数
fib = fibonacci()
for _ in range(10):
    print(next(fib), end=' ')
# 输出: 0 1 1 2 3 5 8 13 21 34
```

### 4️⃣ 管道处理

```python
def read_data():
    for i in range(100):
        yield i

def filter_even(numbers):
    for n in numbers:
        if n % 2 == 0:
            yield n

def square(numbers):
    for n in numbers:
        yield n ** 2

# 链式处理：读取 → 过滤偶数 → 平方
pipeline = square(filter_even(read_data()))
for value in pipeline:
    print(value)  # 0, 4, 16, 36, 64...
```

---

## 📖 生成器表达式

类似列表推导式，但用 `()` 而不是 `[]`：

```python
# 列表推导式 - 立即创建所有元素
squares_list = [x**2 for x in range(1000000)]  # 占用大量内存

# 生成器表达式 - 按需生成
squares_gen = (x**2 for x in range(1000000))   # 几乎不占内存

# 使用
for sq in squares_gen:
    if sq > 100:
        break
    print(sq)
```

---

## 📖 `yield from` — 委托给子生成器

```python
def sub_generator():
    yield 1
    yield 2
    yield 3

def main_generator():
    yield 'start'
    yield from sub_generator()  # 委托给子生成器
    yield 'end'

for value in main_generator():
    print(value)
# 输出: start, 1, 2, 3, end
```

---

## 🌟 总结

| 要点 | 说明 |
|------|------|
| **本质** | 创建生成器，实现惰性求值 |
| **特点** | 暂停/恢复执行，保持函数状态 |
| **优势** | 节省内存，处理大数据/无限序列 |
| **使用** | `for` 循环、`next()` 函数、生成器表达式 |

> 💡 **记住**：当你需要处理大量数据但不想一次性加载到内存时，就用 `yield`！

