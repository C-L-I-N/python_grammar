# 🛡️ Python 异常处理详解

异常处理是让程序能够**优雅地处理错误**，而不是直接崩溃。

---

## 📌 核心概念

```
程序运行 → 遇到错误 → 抛出异常 → 如果没有处理 → 程序崩溃！
                         ↓
                   如果被 try/except 捕获 → 执行处理代码 → 程序继续运行
```

---

## 📖 基本语法结构

```python
try:
    # 可能出错的代码
    risky_code()
except SomeError:
    # 出错时执行的代码
    handle_error()
else:
    # 没有出错时执行（可选）
    success_code()
finally:
    # 无论是否出错都执行（可选）
    cleanup_code()
```

---

## 🔹 1. 最简单的异常处理

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("错误：不能除以零！")
    
# 程序不会崩溃，会打印错误信息然后继续运行
print("程序继续执行...")
```

---

## 🔹 2. 捕获异常信息

```python
try:
    num = int("abc")
except ValueError as e:
    print(f"发生错误：{e}")  # 发生错误：invalid literal for int() with base 10: 'abc'
```

---

## 🔹 3. 捕获多种异常

```python
# 方式1：分开处理
try:
    x = int(input("输入数字: "))
    result = 10 / x
except ValueError:
    print("输入的不是数字！")
except ZeroDivisionError:
    print("不能输入零！")

# 方式2：合并处理
try:
    x = int(input("输入数字: "))
    result = 10 / x
except (ValueError, ZeroDivisionError) as e:
    print(f"输入错误：{e}")
```

---

## 🔹 4. `else` 子句 — 没有异常时执行

```python
try:
    result = 10 / 2
except ZeroDivisionError:
    print("除零错误")
else:
    print(f"计算成功！结果是 {result}")  # ✅ 执行这里
```

> 💡 **为什么用 else？** 把"可能出错的代码"和"成功后的代码"分开，更清晰！

---

## 🔹 5. `finally` 子句 — 无论如何都执行

```python
try:
    f = open("data.txt", "r")
    data = f.read()
except FileNotFoundError:
    print("文件不存在！")
finally:
    print("清理工作...")  # 无论是否出错都会执行
    # f.close()  # 确保文件被关闭
```

**典型应用场景**：
- 关闭文件
- 断开数据库连接
- 释放锁
- 清理临时资源

---

## 🔹 6. `raise` — 主动抛出异常

```python
def validate_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄不合理")
    return age

# 使用
try:
    validate_age(-5)
except ValueError as e:
    print(f"验证失败：{e}")
```

**重新抛出异常**：

```python
try:
    some_operation()
except Exception as e:
    print(f"记录日志：{e}")
    raise  # 重新抛出，让上层处理
```

---

## 🔹 7. `assert` — 断言（调试用）

```python
def divide(a, b):
    assert b != 0, "除数不能为零"  # 如果条件为 False，抛出 AssertionError
    return a / b

# 用于开发时的自我检查
age = -5
assert age >= 0, "年龄必须是非负数"  # AssertionError: 年龄必须是非负数
```

> ⚠️ **注意**：`assert` 可以用 `-O` 参数禁用，不要用于正式的输入验证！

---

## 📋 常见异常类型

| 异常 | 触发场景 |
|------|----------|
| `ValueError` | 值不合法，如 `int("abc")` |
| `TypeError` | 类型错误，如 `"2" + 2` |
| `ZeroDivisionError` | 除以零 |
| `IndexError` | 索引越界 |
| `KeyError` | 字典键不存在 |
| `FileNotFoundError` | 文件不存在 |
| `AttributeError` | 属性不存在 |
| `NameError` | 变量未定义 |
| `ImportError` | 导入失败 |

---

## 🎯 实际应用示例

```python
def safe_divide(a, b):
    """安全的除法函数"""
    try:
        result = a / b
    except ZeroDivisionError:
        return None
    except TypeError:
        return None
    else:
        return result

# 读取配置文件
def load_config(filename):
    try:
        with open(filename, 'r') as f:
            return f.read()
    except FileNotFoundError:
        print(f"配置文件 {filename} 不存在，使用默认配置")
        return {}
    except PermissionError:
        print(f"没有权限读取 {filename}")
        return {}
```

---

## ✅ 最佳实践

| 建议 | 说明 |
|------|------|
| 🎯 捕获具体异常 | 不要用 `except:` 或 `except Exception`，太宽泛 |
| 📝 记录异常信息 | 用 `as e` 获取详情 |
| 🔄 适时重新抛出 | 不知道怎么处理就 `raise` |
| 🧹 用 `finally` 清理 | 或者用 `with` 语句更好 |
| ❌ 不要静默吞掉异常 | `except: pass` 是最糟糕的写法！ |

```python
# ❌ 糟糕的写法
try:
    do_something()
except:
    pass  # 吞掉所有异常，出了问题都不知道

# ✅ 好的写法
try:
    do_something()
except SpecificError as e:
    logger.error(f"发生错误: {e}")
    # 采取适当的恢复措施
```

---

## 🔸 自定义异常

```python
# 定义自定义异常类
class AgeError(Exception):
    """年龄相关的异常"""
    pass

class NegativeAgeError(AgeError):
    """年龄为负数的异常"""
    def __init__(self, age):
        self.age = age
        super().__init__(f"年龄不能为负数: {age}")

# 使用自定义异常
def set_age(age):
    if age < 0:
        raise NegativeAgeError(age)
    return age

try:
    set_age(-5)
except NegativeAgeError as e:
    print(f"捕获到自定义异常: {e}")
    print(f"错误的年龄值: {e.age}")
```

---

## 🔸 异常链

```python
# 在处理异常时抛出新异常，保留原始异常信息
try:
    result = 10 / 0
except ZeroDivisionError as e:
    raise ValueError("计算失败") from e

# 输出会显示：
# ZeroDivisionError: division by zero
# The above exception was the direct cause of the following exception:
# ValueError: 计算失败
```

---

## 🌟 总结

异常处理是编写健壮程序的关键：

1. **`try/except`** — 捕获并处理异常
2. **`else`** — 没有异常时执行
3. **`finally`** — 无论如何都执行（清理资源）
4. **`raise`** — 主动抛出异常
5. **`assert`** — 调试时的断言检查

> 💡 **记住**：好的异常处理让程序更加健壮，也让调试更加容易！

