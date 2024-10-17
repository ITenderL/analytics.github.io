## Python学习笔记-函数

## 1. 概述

- 函数的定义

是组织好的，可重复使用的，用来实现特定功能的代码段

- 函数的作用

为了得到一个针对特定需求、可供重复利用的代码段
提高程序的复用性，减少重复性代码，提高开发效率

## 2. 函数基础操作

### 2.1 函数的定义

如何定义个一个函数，需要用到关键字def

``` python
"""
演示函数的定义语法
"""


# 定义一个函数，输出相关信息
def say_hi():
    print("Hi 我是黑马程序员，学Python来黑马")


# 调用函数，让定义的函数开始工作
say_hi()
```

### 2.2 函数的参数

传入参数的功能是：在函数进行计算的时候，接受外部（调用时）提供的数据

``` python
"""
演示函数使用参数
"""

# 定义2数相加的函数，通过参数接收被计算的2个数字
def add(x, y, z):
    result = x + y + z
    print(f"{x} + {y} + {z}的计算结果是：{result}")

# 调用函数，传入被计算的2个数字
add(5, 6, 7)

```

**注意事项**

- 函数定义中的参数，称之为形式参数
- 函数调用中的参数，称之为实际参数
- 函数的参数数量不限，使用逗号分隔开
- 传入参数的时候，要和形式参数一一对应，逗号隔开

### 2.3 函数返回值

函数返回值通过return

``` python
"""
演示：定义函数返回值的语法格式
"""


# 定义一个函数，完成2数相加功能
def add(a, b):
    result = a + b
    # 通过返回值，将相加的结果返回给调用者
    return result
    # 返回结果后，还想输出一句话
    print("我完事了")


# 函数的返回值，可以通过变量去接收
r = add(5, 6)
print(r)

```

### 2.4 函数注释

``` python
# 定义函数，进行文档说明
def add(x, y):
    """
    add函数可以接收2个参数，进行2数相加的功能
    :param x: 形参x表示相加的其中一个数字
    :param y: 形参y表示相加的另一个数字
    :return: 返回值是2数相加的结果
    """
    result = x + y
    print(f"2数相加的结果是：{result}")
    return result

add(5, 6)
```

###  2.5 函数嵌套调用

``` python
"""
演示嵌套调用函数
"""


# 定义函数func_b
def func_b():
    print("---2---")


# 定义函数func_a，并在内部调用func_b
def func_a():
    print("---1---")

    # 嵌套调用func_b
    func_b()

    print("---3---")


# 调用函数func_a
func_a()

```

###  2.6 变量作用域

``` python
"""
演示在函数使用的时候，定义的变量作用域
"""

# 演示局部变量
# def test_a():
#     num = 100
#     print(num)
#
#
# test_a()
# 出了函数体，局部变量就无法使用了
# print(num)

# 演示全局变量
# num = 200
#
# def test_a():
#     print(f"test_a: {num}")
#
# def test_b():
#     print(f"test_b: {num}")
#
# test_a()
# test_b()
# print(num)

# 在函数内修改全局变量
# num = 200
#
# def test_a():
#     print(f"test_a: {num}")
#
# def test_b():
#     num = 500       # 局部变量
#     print(f"test_b: {num}")
#
# test_a()
# test_b()
# print(num)

# global关键字，在函数内声明变量为全局变量
num = 200


def test_a():
    print(f"test_a: {num}")


def test_b():
    global num  # 设置内部定义的变量为全局变量
    num = 500
    print(f"test_b: {num}")


test_a()
test_b()
print(num)

```

## 3. 函数进阶

###  3.1 函数的多返回值

``` python
"""
演示函数的多返回值示例
"""


# 演示使用多个变量，接收多个返回值
def test_return():
    return 1, "hello", True


x, y, z = test_return()
print(x)
print(y)
print(z)

```

### 3.2 函数的多种传参形式

``` python
"""
演示多种传参的形式
"""


def user_info(name, age, gender):
    print(f"姓名是:{name}, 年龄是:{age}, 性别是:{gender}")


# 位置参数 - 默认使用形式
user_info('小明', 20, '男')

# 关键字参数
user_info(name='小王', age=11, gender='女')
user_info(age=10, gender='女', name='潇潇')  # 可以不按照参数的定义顺序传参
user_info('甜甜', gender='女', age=9)


# 缺省参数（默认值）
def user_info(name, age, gender):
    print(f"姓名是:{name}, 年龄是:{age}, 性别是:{gender}")


user_info('小天', 13, '男')


# 不定长 - 位置不定长, *号
# 不定长定义的形式参数会作为元组存在，接收不定长数量的参数传入
def user_info(*args):
    print(f"args参数的类型是：{type(args)}，内容是:{args}")


user_info(1, 2, 3, '小明', '男孩')


# 不定长 - 关键字不定长, **号
def user_info(**kwargs):
    print(f"args参数的类型是：{type(kwargs)}，内容是:{kwargs}")


user_info(name='小王', age=11, gender='男孩')

```

### 3.3 函数作为参数传递

``` python
"""
演示函数作为参数传递
"""


# 定义一个函数，接收另一个函数作为传入参数
def test_func(compute):
    result = compute(1, 2)  # 确定compute是函数
    print(f"compute参数的类型是:{type(compute)}")
    print(f"计算结果：{result}")


# 定义一个函数，准备作为参数传入另一个函数
def compute(x, y):
    return x + y


# 调用，并传入函数
test_func(compute)

```

### 3.4 lambda匿名函数

``` python
"""
演示lambda匿名函数
"""


# 定义一个函数，接受其它函数输入
def test_func(compute):
    result = compute(1, 2)
    print(f"结果是:{result}")


# 通过lambda匿名函数的形式，将匿名函数作为参数传入
def add(x, y):
    return x + y


test_func(add)
test_func(lambda x, y: x + y)

```





