## Python学习笔记-文件操作

## 1. 前言

1. 什么是编码？
  编码就是一种规则集合，记录了内容和二进制间进行相互转换的逻辑。
  编码有许多中，我们最常用的是UTF-8编码

2. 为什么需要使用编码？
  计算机只认识0和1，所以需要将内容翻译成0和1才能保存在计算机中。
  同时也需要编码， 将计算机保存的0和1，反向翻译回可以识别的内容

3. 什么是文件？

  内存中存放的数据在计算机关机后就会消失。要长久保存数据，就要使用硬盘、光盘、U 盘等设备。为了便于数据的管理和检索，引入了“文件”的概念。
  一篇文章、一段视频、一个可执行程序，都可以被保存为一个文件，并赋予一个文件名。操作系统以文件为单位管理磁盘中的数据。一般来说，文件可分为文本文件、视频文件、音频文件、图像文件、可执行文件等多种类别。

## 2. 文件操作

**在日常生活中，文件操作主要包括打开、关闭、读、写等操作。**

- 想想我们平常对文件的基本操作，大概可以分为三个步骤（简称文件操作三步走）：
  ① 打开文件
  ② 读写文件
  ③ 关闭文件

- 在Python，使用open函数，可以打开一个已经存在的文件，或者创建一个新文件，语法如下

``` python
open(name, mode, encoding)
```

name：是要打开的目标文件名的字符串(可以包含文件所在的具体路径)。
mode：设置打开文件的模式(访问模式)：只读(r)、写入(w)、追加(a)等。
encoding:编码格式（推荐使用UTF-8）

- mode常用的三种基础访问模式

| **模式** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| r        | 以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。 |
| w        | 打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，原有内容会被删除。如果该文件不存在，创建新文件。 |
| a        | 打开一个文件用于追加。如果该文件已存在，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |

### 2.1 文件读取

| 操作                                  | 功能                                    |
| ------------------------------------- | --------------------------------------- |
| 文件对象 = open(file, mode, encoding) | 打开文件获得文件对象                    |
| 文件对象.read(num)                    | 读取指定长度字节不指定num读取文件全部   |
| 文件对象.readline()                   | 读取一行                                |
| 文件对象.readlines()                  | 读取全部行，得到列表                    |
| for line in 文件对象                  | for循环文件行，一次循环得到一行数据     |
| 文件对象.close()                      | 关闭文件对象                            |
| with open() as f                      | 通过with open语法打开文件，可以自动关闭 |

``` python
# 打开文件
f = open("F:/test.txt", "r", encoding="utf-8")
print(type(f))
# read 读取文件
# print(f"读取10个字节的结果：{f.read(10)}")
# print(f"读取全部的结果：{f.read()}")

# readlines() 读取文件，读取全部行，到列表中
# lines = f.readlines()
# print(f"lines对象类型：{type(lines)}")
# print(f"lines内容：{lines}")

# readline() 读取文件 每次读取一行
# line1 = f.readline()
# line2 = f.readline()
# line3 = f.readline()
# line4 = f.readline()
# print(line1)
# print(line2)
# print(line3)
# print(line4)

# for循环读取文件
for line in f:
    print(line)

# 文件关闭，解除文件占用
f.close()

# with open语法操作文件,会自动关闭文件close
with open("F:/test.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line)

# 读取课后练习
f = open("F:/test.txt", "r", encoding="utf-8")
# content = f.read()
# count = content.count("帽衫")
# print(count)
count = 0
for line in f:
    # 去掉开头结尾的空格和换行符
    line = line.strip()
    words = line.split(" ")
    for word in words:
        if word == "帽衫":
            count += 1
print(count)
f.close()

```



### 2.2 文件写入

1.  写入文件使用open函数的”w”模式进行写入
2.  写入的方法有：
wirte()，写入内容
flush()，刷新内容到硬盘中

``` python
"""
演示文件的写入
"""

# 打开文件，不存在的文件, r, w, a
import time

# f = open("D:/test.txt", "w", encoding="UTF-8")
# # write写入
# f.write("Hello World!!!")       # 内容写入到内存中
# # flush刷新
# # f.flush()                       # 将内存中积攒的内容，写入到硬盘的文件中
# # close关闭
# f.close()                       # close方法，内置了flush的功能的
# 打开一个存在的文件
f = open("D:/test.txt", "w", encoding="UTF-8")
# write写入、flush刷新
f.write("黑马程序员")
# close关闭
f.close()

```

3. 注意：
   直接调用write，内容并未真正写入文件，而是会积攒在程序的内存中，称之为缓冲区
   当调用flush的时候，内容会真正写入文件
   这样做是避免频繁的操作硬盘，导致效率下降（攒一堆，一次性写磁盘）

   w模式，文件不存在，会创建新文件
   w模式，文件存在，会清空原有内容
   close()方法，带有flush()方法的功能

### 2.3 文件追加

1. 追加写入文件使用open函数的”a”模式进行写入
2. 追加写入的方法有（和w模式一致）：
wirte()，写入内容
flush()，刷新内容到硬盘中
3. 注意事项：
a模式，文件不存在，会创建新文件
a模式，文件存在，会在原有内容后面继续写入
可以使用”\n”来写出换行符

``` python
"""
演示文件的追加写入
"""

# 打开文件，不存在的文件
# f = open("D:/test.txt", "a", encoding="UTF-8")
# # write写入
# f.write("黑马程序员")
# # flush刷新
# f.flush()
# # close关闭
# f.close()
# 打开一个存在的文件
f = open("D:/test.txt", "a", encoding="UTF-8")
# write写入、flush刷新
f.write("\n月薪过万")
# close关闭
f.close()

```

