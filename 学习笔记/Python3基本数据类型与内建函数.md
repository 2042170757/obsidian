---
title: Python3 基本数据类型与内建函数
date: 2026-04-22
tags:
  - Python
  - 编程学习
  - 课后作业
aliases:
  - Python数据类型
  - Python内建函数
---

# Python3 基本数据类型与内建函数

> 主要来源：菜鸟教程
> - https://www.runoob.com/python3/python3-data-type.html
> - https://www.runoob.com/python3/python3-type-conversion.html
> - https://www.runoob.com/python3/python3-number.html
> - https://www.runoob.com/python3/python3-string.html
> - https://www.runoob.com/python3/python3-list.html

---

## 一、基本数据类型概述

Python3 中常见的数据类型有：

| 数据类型 | 说明 | 可变性 |
|---------|------|-------|
| **Number（数字）** | int, float, bool, complex | 不可变 |
| **String（字符串）** | 文本数据 | 不可变 |
| **List（列表）** | 有序可改的序列 | 可变 |
| **Tuple（元组）** | 有序不可变的序列 | 不可变 |
| **Set（集合）** | 无序不重复的元素集合 | 可变 |
| **Dictionary（字典）** | 键值对映射 | 可变 |

> **可变 vs 不可变**：
> - 可变数据（3个）：List、Dictionary、Set
> - 不可变数据（3个）：Number、String、Tuple

---

## 二、数字类型 (Number)

### 2.1 数字类型分类

```python
# 整型 (int) - 没有大小限制
a = 10
b = -5

# 浮点型 (float)
c = 3.14
d = -2.5

# 布尔型 (bool) - True / False
e = True
f = False

# 复数 (complex)
g = 4 + 3j
```

### 2.2 数字运算

```python
# 基本运算
print(2 + 2)      # 4  (加法)
print(50 - 5*6)   # 20 (减法和乘法)
print(8 / 5)      # 1.6 (除法，总是返回浮点数)
print(8 // 5)     # 1  (整除)
print(8 % 5)      # 3  (取余)
print(2 ** 3)     # 8  (幂运算)

# 使用 _ 变量（上一个计算结果）
price = 100.50
tax = price * 0.125
print(price + tax)  # 113.0625
print(round(_, 2))  # 113.06
```

### 2.3 数学函数 (math 模块)

```python
import math

# 常用数学函数
print(math.ceil(4.3))    # 5  向上取整
print(math.floor(4.7))   # 4  向下取整
print(math.sqrt(16))     # 4.0 平方根
print(abs(-10))          # 10 绝对值
print(pow(2, 3))         # 8.0 幂运算
print(math.factorial(5)) # 120 阶乘

# 三角函数
print(math.sin(math.pi/2))  # 1.0
print(math.cos(0))          # 1.0

# 常量
print(math.pi)    # 3.141592653589793
print(math.e)     # 2.718281828459045
```

### 2.4 随机数函数 (random 模块)

```python
import random

# 随机生成 0-1 之间的浮点数
print(random.random())      # 0.4784904215869241

# 随机生成指定范围内整数
print(random.randint(1, 10))    # 1-10 之间的随机整数

# 随机选择序列中的一个元素
fruits = ['apple', 'banana', 'cherry']
print(random.choice(fruits))

# 随机打乱序列
lst = [1, 2, 3, 4, 5]
random.shuffle(lst)
print(lst)

# 从序列中随机选择不重复的k个元素
print(random.sample([1,2,3,4,5], 3))
```

---

## 三、字符串 (String)

### 3.1 字符串创建与访问

```python
# 创建字符串
str1 = 'hello'
str2 = "world"
str3 = '''多行
字符串'''

# 字符串索引（从0开始）
s = "Python"
print(s[0])   # P
print(s[-1])  # n (最后一个字符)

# 字符串切片 [start:end:step]
print(s[0:3])   # Pyt (不包括end)
print(s[::2])   # Pto (每隔一个字符)
print(s[::-1])  # nohtyP (反转)
```

### 3.2 字符串格式化

#### 方式1：% 格式化

```python
name = "小明"
age = 10
print("我叫 %s 今年 %d 岁!" % (name, age))

# 格式化符号
print("%c" % 65)      # A (格式化字符)
print("%s" % "hello") # hello
print("%d" % 10)      # 10
print("%f" % 3.14)    # 3.140000
print("%e" % 1000)    # 1.000000e+03
```

#### 方式2：f-string (Python 3.6+)

```python
name = "Alice"
age = 30

# 基础用法
print(f"My name is {name}, I am {age} years old")

# 表达式
print(f"1 + 2 = {1 + 2}")
print(f"10 / 3 = {10/3:.2f}")  # 保留两位小数

# 变量名显示 (Python 3.8+)
x = 1
print(f"{x+1=}")  # x+1=2
```

### 3.3 字符串内建函数

```python
s = "Hello World Hello Python"

# 查找类
print(s.find("World"))     # 6  找不到返回-1
print(s.index("World"))    # 6  找不到会报错
print(s.count("Hello"))    # 2  统计出现次数

# 分割与连接
print(s.split(" "))        # ['Hello', 'World', 'Hello', 'Python']
print("-".join(s))         # H-e-l-l-o- -W-o-r-l-d-...

# 大小写
print(s.upper())           # HELLO WORLD HELLO PYTHON
print(s.lower())           # hello world hello python
print(s.capitalize())      # Hello world hello python
print(s.title())           # Hello World Hello Python

# 判断类
print(s.startswith("Hello"))  # True
print(s.endswith("Python"))   # True
print(s.isdigit())            # False
print(s.isalpha())            # False (有空格)

# 替换
print(s.replace("Hello", "Hi"))  # Hi World Hi Python
print(s.strip())               # 去除两端空白
```

---

## 四、列表 (List)

### 4.1 列表创建

```python
# 直接创建
lst1 = [1, 2, 3, 4, 5]
lst2 = ['apple', 'banana', 'cherry']
lst3 = [1, 'hello', True, 3.14]  # 异构列表

# 使用 list() 函数
lst4 = list("hello")    # ['h', 'e', 'l', 'l', 'o']
lst5 = list((1, 2, 3))  # [1, 2, 3]  元组转列表

# 使用 range()
lst6 = list(range(5))       # [0, 1, 2, 3, 4]
lst7 = list(range(1, 10, 2)) # [1, 3, 5, 7, 9]

# 列表推导式
squares = [x**2 for x in range(10)]  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
evens = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]
```

### 4.2 列表访问与切片

```python
fruits = ['apple', 'banana', 'cherry', 'date', 'elderberry']

# 索引访问
print(fruits[0])    # apple
print(fruits[-1])   # elderberry

# 切片操作
print(fruits[1:3])     # ['banana', 'cherry']
print(fruits[:3])      # ['apple', 'banana', 'cherry']
print(fruits[::2])     # ['apple', 'cherry', 'elderberry']
print(fruits[::-1])    # 反转列表
```

### 4.3 列表增删改查

```python
lst = [1, 2, 3]

# 添加元素
lst.append(4)         # 末尾添加 [1, 2, 3, 4]
lst.insert(1, 99)     # 指定位置插入 [1, 99, 2, 3, 4]
lst.extend([5, 6])    # 扩展多个 [1, 99, 2, 3, 4, 5, 6]

# 删除元素
lst.pop()             # 删最后一个，返回被删除值
lst.pop(0)            # 删除指定索引
lst.remove(99)        # 删除第一个匹配的值
lst.clear()           # 清空列表

# 修改元素
lst = [1, 2, 3, 4, 5]
lst[0] = 100          # [100, 2, 3, 4, 5]
```

### 4.4 列表常用方法

```python
lst = [1, 2, 3, 2, 4, 2]

# 统计与查找
print(len(lst))           # 6  长度
print(lst.count(2))       # 3  元素出现次数
print(lst.index(3))       # 2  元素首次出现的索引

# 排序与反转
lst = [3, 1, 4, 1, 5, 9, 2, 6]
lst.sort()                # 原地排序 [1, 1, 2, 3, 4, 5, 6, 9]
lst.sort(reverse=True)    # 降序 [9, 6, 5, 4, 3, 2, 1, 1]
lst.reverse()             # 原地反转

# 复制
lst1 = [1, 2, 3]
lst2 = lst1.copy()        # 浅拷贝
lst3 = lst1[:]            # 另一种浅拷贝方式

# 列表运算
list1 = [1, 2, 3]
list2 = [4, 5, 6]
print(list1 + list2)      # [1, 2, 3, 4, 5, 6]  合并
print(list1 * 2)         # [1, 2, 3, 1, 2, 3]  重复
```

### 4.5 嵌套列表

```python
# 创建嵌套列表
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12]
]

# 访问元素
print(matrix[0])        # [1, 2, 3, 4]
print(matrix[0][1])     # 2
print(matrix[1][2])     # 7

# 列表推导式转置矩阵
transposed = [[row[i] for row in matrix] for i in range(4)]
print(transposed)  # [[1,5,9], [2,6,10], [3,7,11], [4,8,12]]
```

---

## 五、数据类型转换

### 5.1 隐式类型转换

Python自动将较低数据类型转换为较高数据类型：

```python
num_int = 10
num_float = 5.5
result = num_int + num_float  # 结果为 15.5 (float)
print(type(result))  # <class 'float'>
```

### 5.2 显式类型转换

| 函数 | 说明 | 示例 |
|------|------|------|
| `int(x)` | 转换为整数 | `int("123")` → `123` |
| `float(x)` | 转换为浮点数 | `float("3.14")` → `3.14` |
| `str(x)` | 转换为字符串 | `str(123)` → `'123'` |
| `bool(x)` | 转换为布尔值 | `bool(1)` → `True` |
| `list(s)` | 转换为列表 | `list("abc")` → `['a','b','c']` |
| `tuple(s)` | 转换为元组 | `tuple([1,2,3])` → `(1,2,3)` |
| `set(s)` | 转换为集合 | `set([1,2,2])` → `{1,2}` |
| `dict(d)` | 转换为字典 | `dict([('a',1)])` → `{'a':1}` |

**实际操作：**

```python
# 字符串转数字
x = int("25")           # 25
y = float("3.14")       # 3.14

# 数字转字符串
a = str(42)             # '42'
b = str(3.14159)        # '3.14159'

# 列表元组互转
lst = list((1, 2, 3))   # [1, 2, 3]
tup = tuple([4, 5, 6])  # (4, 5, 6)

# 去重
unique = set([1, 2, 2, 3])  # {1, 2, 3}
```

---

## 六、字典 (Dictionary) 常用函数

### 6.1 字典基本操作

```python
# 创建字典
dict1 = {"name": "Tom", "age": 20}
dict2 = dict(name="Alice", age=25)

# 访问值
print(dict1["name"])           # Tom
print(dict1.get("name"))       # Tom
print(dict1.get("gender", "unknown"))  # unknown (默认值)

# 添加/修改
dict1["city"] = "Beijing"      # 添加新键值对
dict1["age"] = 21              # 修改已有键的值
```

### 6.2 字典常用函数

```python
student = {"name": "Tom", "age": 20, "city": "Beijing"}

# 获取长度
print(len(student))   # 3

# 转换为字符串
print(str(student))   # "{'name': 'Tom', 'age': 20, 'city': 'Beijing'}"

# 获取视图对象
print(student.keys())    # dict_keys(['name', 'age', 'city'])
print(student.values())  # dict_values(['Tom', 20, 'Beijing'])
print(student.items())   # dict_items([('name','Tom'), ('age',20), ('city','Beijing')])

# 删除
removed = student.pop("age")  # 删除并返回值
print(removed)    # 20
print(student)    # {'name': 'Tom', 'city': 'Beijing'}

# 更新
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}
dict1.update(dict2)  # {'a': 1, 'b': 3, 'c': 4}
```

---

## 七、总结表

### 字符串常用函数

| 函数 | 说明 |
|------|------|
| `count()` | 统计出现次数 |
| `endswith()` | 判断结尾 |
| `startswith()` | 判断开头 |
| `index()` / `find()` | 查找位置 |
| `split()` | 分割字符串 |
| `join()` | 连接字符串 |
| `upper()` / `lower()` | 大小写转换 |
| `replace()` | 替换 |

### 列表常用函数

| 函数 | 说明 |
|------|------|
| `len()` | 获取长度 |
| `list()` | 转换为列表 |
| `append()` | 末尾添加 |
| `count()` | 统计元素次数 |
| `index()` | 查找元素索引 |
| `pop()` | 删除并返回元素 |
| `remove()` | 按值删除 |
| `sort()` | 排序 |
| `reverse()` | 反转 |
| `copy()` | 浅拷贝 |

### 字典常用函数

| 函数 | 说明 |
|------|------|
| `len()` | 获取键值对数量 |
| `str()` | 转换为字符串 |
| `get()` | 安全获取值 |
| `items()` | 获取键值对 |
| `keys()` | 获取所有键 |
| `values()` | 获取所有值 |
| `pop()` | 删除指定键 |

> [!tip] 提示
> - `find()` 和 `index()` 的区别：找不到时 `find` 返回 -1，`index` 抛出异常
> - `pop()` 和 `remove()` 的区别：`pop` 按索引删除返回删除的值，`remove` 按值删除
> - 字典的 `get()` 方法比直接用 `[]` 访问更安全，不会抛出 KeyError
> - f-string 是 Python 3.6+ 最推荐的字符串格式化方式

---

*整理时间：2026-04-22*
*参考资料：菜鸟教程、Python官方文档*