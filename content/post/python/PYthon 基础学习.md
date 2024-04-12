---
author: zty
title: 初识Python1
date: 2022-09-19
description: Python
series:
  - Python
tags : [
    python基础
]
categories : [zty
    编程基础
]
series : [论文用]
aliases : [python基础]
---
使用编译器将自身转换成机器语言的高级语言，通常称为编译型语言；而使用解释器将自身转换为机器语言的高级语言，通常称为解释型语言，python就属于解释型编程语言。

<!--more-->

|类型	|原理	|优点	|缺点|
|-|-|-|-|
|编译型语言	|通过专门的编译器，将所有源代码一次性转换成特定平台（Windows、Linux、macOS等）的机器码（以可执行文件的形式存在）。|	编译一次后，脱离了编译器也能运行，并且运行效率高。	|可移植性差，不够灵活。|
|解释型语言	|通过专门的解释器，根据需要可以将部分或全部源代码转换成特定平台（Windos、Linux、macOS等）的机器码。|	跨平台性好，通过不同的解释器，将相同的源代码解释成不同平台下的机器码。	|一边执行一边转换，效率较低。|

在python中也是用`#`来表示单行注释，`''`和`""`可以表示多行注释。


|||内置函数|||
|-|-|-|-|-|
|abs()|	delattr()|	hash()|	memoryview()|	set()|
|all()|	dict()|	help()|	min()|	setattr()|
|any()|	dir()|	hex()|	next()|	slicea()|
|ascii()|	divmod()|	id()|	object()|	sorted()|
|bin()	|enumerate()	|input()|	oct()|	staticmethod()|
|bool()	|eval()|	int()|	open()|	str()|
|breakpoint()|	exec()|	isinstance()|	ord()|	sum()|
|bytearray()	|filter()	|issubclass()	|pow()|	super()|
|bytes()	|float()	|iter()|	print()|	tuple()|
|callable()	|format()	|len()	|property()	|type()|
|chr()|	frozenset()	|list()|	range()|	vars()|
|classmethod()|	getattr()	|locals()	|repr()|	zip()|
|compile()	|globals()|	map()|	reversed()|	__import__()|
|complex()	|hasattr()	|max()	|round()	 |


|||Python |保留字一览表|||
|-|-|-|-|-|-|
|and	|as	|assert	|break	|class	|continue|
|def|	del	|elif|	else	|except|	finally|
|for	|from|	False|	global|	if	|import|
|in	|is	|lambda	|nonlocal	|not	|None|
|or	|pass|	raise|	return|	try|	True|
|while	|with|	yield|

`python的标识符命名规则和C语言的大体相似。`

- 标识符的命名，除了要遵守以上这几条规则外，不同场景中的标识符，其名称也有一定的规范可循，例如：
- 当标识符用作模块名时，应尽量短小，并且全部使用小写字母，可以使用下划线分割多个字母，例如 game_mian、game_register 等。
- 当标识符用作包的名称时，应尽量短小，也全部使用小写字母，不推荐使用下划线，例如 com.mr、com.mr.book 等。
- 当标识符用作类名时，应采用单词首字母大写的形式。例如，定义一个图书类，可以命名为 Book。
- 模块内部的类名，可以采用 "下划线+首字母大写" 的形式，如 _Book;
函数名、类中的属性名和方法名，应全部使用小写字母，多个单词之间可以用下划线分割；
- 常量命名应全部使用大写字母，单词之间可以用下划线分割；



