---
title: 数据结构实验 4 - 迷宫
date: 2018-10-18T23:57:13
description: "数据结构的迷宫实验的一个解法，有点复杂，记录一下"
featuredImage: "https://alicdn.kmahyyg.xyz/asset_files/aether/cat_tech.webp"
categories: ["school","code"]
draft: false
displayInMenu: false
displayInList: true
dropCap: false
---

# 写在最前

请先阅读 [我要提问](https://github.com/octowhale/Stop-Ask-Questions-The-Stupid-Ways/blob/master/README.md) 之后再看下文。

# 数据结构实验 4 - 迷宫

实验代码和实验要求请参见 [我的 Github 项目 (Private)](https://github.com/kmahyyg/datastru-ynu)

本篇博文将更加注重着眼于：

- 具体迷宫生成实现
- 自动完成迷宫实现
- 实现过程中新学到的东西和踩的坑

# stdarg.h

## 举一反三： Python - Non-keyword Argument

Pass the **variable length argument list with single asterisk**.

Inside the function, we have a loop which adds the passed argument and prints the result. We passed 3 different tuples with variable length as an argument to the function.

```python
def adder(*num):
    sum = 0
    
    for n in num:
        sum = sum + n

    print("Sum:",sum)

adder(3,5)
adder(4,5,6,7)
adder(1,2,3,5,6)
```

```
Sum: 8
Sum: 22
Sum: 17
```

## 举一反三： Python - Keyword Argument

Use a **dictionary-like parameters with double asterisk** passed to the function.

```python
def intro(**data):
    print("\nData type of argument:",type(data))

    for key, value in data.items():                          # inside loop
        print("{} is {}".format(key,value))

intro(Firstname="Sita", Lastname="Sharma", Age=22, Phone=1234567890)
intro(Firstname="John", Lastname="Wood", Email="johnwood@nomail.com", Country="Wakanda", Age=25, Phone=9876543210)
```

```
Data type of argument: <class 'dict'>
Firstname is Sita
Lastname is Sharma
Age is 22
Phone is 1234567890

Data type of argument: <class 'dict'>
Firstname is John
Lastname is Wood
Email is johnwood@nomail.com
Country is Wakanda
Age is 25
Phone is 9876543210
```

## stdarg.h 在 C 中的应用

建议少用或者尽量不用。

### 库变量与库宏

`va_list` ： 适用于 `va_start() va_arg() va_end()` 三个宏存储信息的类型，可近似等价于可变链表。

`void va_start(va_list ap, last_arg);` 初始化 `ap` 变量，与 `va_arg` `va_end` 共同使用，`last_arg` 是最后一个传递给函数的已知的固定参数，即省略号前的参数。

`type va_arg(va_list ap, type);` 这个宏检索函数参数列表 `va_list ap` 中类型为 `type` 的 **下一个参数** 。 该函数应当理解为 **指针** ，函数返回为 **有序列表中的符合对应类型的最近一个数据**，函数参数以数据结构 **栈** 形式存储，从左向右入栈。

`void va_end(va_list ap);` 这个宏允许使用了带有 `va_start` 宏的带有可变参数的函数返回，若函数返回前未调用 `va_end` ，则结果为 `undefined`.

### 具体代码

Compile ： `gcc -std=c99 test.c -o test`

test.c :

```c
#include <stdio.h>
#include <string.h>
#include <stdarg.h>

int echoinfo(char *strNotice, ...);

int main(int argc, char *argv[]){
    char noti[50] = {0};
    strcpy(noti,"The printout is");
    echoinfo(noti, argv[1], argv[2], argv[3]);
    return 0;
}

int echoinfo(char *strNotice, ...){
    char *str0 = NULL;
    char *str1 = NULL;
    char *str2 = NULL;
    va_list stArgv;         // define a param list
    va_start(stArgv, strNotice);      // pass the fixed param to the function
    str0 = va_arg(stArgv, char*);
    str1 = va_arg(stArgv, char*);
    str2 = va_arg(stArgv, char*);
    printf("%s: %s %s %s", strNotice, str0, str1, str2);
    va_end(stArgv);
    return 0;
}
```

# 迷宫生成

其实是一个 树/图 的遍历问题，这样也就关联上了两种常用算法：BFS（广度优先搜索）、DFS(深度优先搜索)，实际上就是一个 Single Source Shortest Path 寻找问题。

![gen-maze.png](https://alicdn.kmahyyg.xyz/asset_files/2018-mazegen-01.webp)

[点击此处查看视频演示](https://alicdn.kmahyyg.xyz/asset_files/2018-mazegen.flv)

## 推荐的生成思路

Once we have a grid filled with walls, we transform it into a maze as follows:

1. We pick a random cell
2. We select a random neighbouring cell that has not been visited
3. We remove the wall between the two cells and add the neighbouring cell to the list of cells having been visited.
4. If there are no unvisited neighbouring cell, we backtrack to one that has at least one unvisited neighbour; this is done until we backtrack to the original cell.

## Python 

由于之前完全没有接触过这类实现，本次实现将采用 Python 进行一次预实现。

**[Python 多维数组的浅拷贝问题 - 关于列表](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations)** 

**[Python 多维数组的浅拷贝问题 - 官方 FAQ](https://docs.python.org/3/faq/programming.html#faq-multidimensional-list)**

**[Python 3 zip() 函数](http://www.runoob.com/python3/python3-func-zip.html)**

### 关于 zip 函数

定义：`zip([iterable, ...])`

`zip()` 是 Python 的一个内建函数，它接受一系列可迭代的对象作为参数，将对象中对应的元素打包成一个个 tuple (元组)，然后返回由这些 tuples 组成的 list (列表)。若传入参数的长度不等，则返回 list 的长度和参数中长度最短的对象相同。利用 * 号操作符，可以将 list unzip (解压)，看下面的例子就明白了：

``` python
>>> a = [1,2,3]
>>> b = [4,5,6]
>>> c = [4,5,6,7,8]
>>> zipped = zip(a,b)
[(1, 4), (2, 5), (3, 6)]
>>> zip(a,c)
[(1, 4), (2, 5), (3, 6)]
>>> zip(*zipped)
[(1, 2, 3), (4, 5, 6)]
```

### random.randrange()

```
random.randrange(start, stop[, step])
Return a randomly selected element from range(start, stop, step). 
```

### random.shuffle()

```
random.shuffle(x[, random])¶
Shuffle the sequence x in place.

The optional argument random is a 0-argument function returning a random float in [0.0, 1.0); by default, this is the function random().
```


### 多维数组

官方的建议：

> listA = [[None] \*2] \*3
这样的使用 * 创建的一个列表，并没有创建内部列表的三次拷贝，而是创建了三次同样的链接到内部的列表。所以你如果修改任何一个内部列表，三个外部链接的值也会跟随改变。

正确的做法是：

1. 使用 `numpy` 等提供矩阵类型的工具。
2. 使用下列代码：

```python
A = [None] * 3
for i in range(3):
    A[i] = [None] * 2
```

等价于：

```python
w, h = 2, 3
A = [[None] * w for i in range(h)]
```

# 迷宫自动完成

![autowalk-maze.png](https://alicdn.kmahyyg.xyz/asset_files/2018-mazegen-02.webp)

# 编写过程中的实现问题

## Dirty Workaround (ycomlib.h)

请参考 Github Repo 中的对应文件。

## 最大的问题

Python 预实现过后，进行面向 C 的迁移。很多方面，例如 二维数组的传递、数组 Index 超界、递归超过上限 问题，导致了许多的 SEGV，最后还是没有解决。时间太紧张了，没时间了。整体的思路还算清晰，请参考上面的附图。也欢迎您和我联系，交换您的想法。

# Acknowledgement

- [Python official document list](https://docs.python.org/3/)
- [Recursive backtracker mtod for maze generation](https://blog.csdn.net/crystal_tyan/article/details/42523861)
- [Three widely used mtod for maze generation](https://blog.csdn.net/juzihongle1/article/details/73135920)

