---
author: Hugo Authors
title: Python的输入输出
date: 2022-10-01
description: Python
series:
  - Python

---

输入输出函数的使用
<!--more-->
# Python input()函数：获取用户输入的字符串
input()是Python的内置函数，用于从控制台读取用户输入的内容。input()函数是以字符串的形式处理用户输入的内容，所以用户输入的内容可以包含任何内容。

```
str = input(tipmsg)
```
- str 表示一个字符串类型变量，input会将读到的字符放入str中。
- tipmsg表示提示信息，它会显示在控制台上，告诉用户应该输入怎样的内容；如果不写，就不会有任何提示信息。

```python
a = input("Enter a number: ")
b = input("Enter another number: ")
print("aType: ", type(a))
print("bType: ", type(b))
result = a + b
print("resultValue: ", result)
print("resultType: ", type(result))
```
运行结果示例：
```python
Enter a number: 100↙
Enter another number: 45↙
aType:  <class 'str'>
bType:  <class 'str'>
resultValue:  10045
resultType:  <class 'str'>
```
↙表示按下回车键，按下回车键input()读取就结束了。

本例中我们输入了两个整数，希望计算出它们的和，但是事与愿违，Python只是将它们当成了字符串，`+`起到了拼接字符串的作用，而不是求和的作用。

当然我们要是想将字符型转换成想要的类型，比如：
-
- int(string)将字符串转换成int型
- float(string)将字符串转换成浮点型
- bool(string)将字符串转换成布尔型

```python
a = input("Enter a number: ")
b = input("Enter another number: ")
a = float(a)
b = int(b)
print("aType: ", type(a))
print("bType: ", type(b))
result = a + b
print("resultValue: ", result)
print("resultType: ", type(result))
```
运行结果：
```python
Enter a number: 12.5↙
Enter another number: 64↙
aType:  <class 'float'>
bType:  <class 'int'>
resultValue:  76.5
resultType:  <class 'float'>
```


上面讲解的是 Python 3.x 中 input() 的用法，但是在较老的 Python 2.x 中情况就不一样了。Python 2.x 共提供了两个输入函数，分别是 input() 和 raw_input()：
-
- Python 2.x raw_input() 和 Python 3.x input() 效果是一样的，都只能以字符串的形式读取用户输入的内容。
- Python 2.x input() 看起来有点奇怪，它要求用户输入的内容必须符合 Python 的语法，稍有疏忽就会出错，通常来说只能是整数、小数、复数、字符串等。

比较强迫的是，Python 2.x input() 要求用户在输入字符串时必须使用引号包围，这有违 Python 简单易用的原则，所以 Python 3.x 取消了这种输入方式。

```python
a = input("Enter a number: ")
b = input("Enter another number: ")
print "aType: ", type(a)
print "bType: ", type(b)
result = a + b
print "resultValue: ", result
print "resultType: ", type(result)
```
在 Python 2.x 下运行该代码：
```python
Enter a number: 45↙
Enter another number: 100↙
aType:  <type 'int'>
bType:  <type 'int'>
resultValue:  145
resultType:  <type 'int'>
```

# Python print()函数的高级用法

之前我们使用print()函数时，都只是输出一个变量，但实际上print()函数完全可以输出多个变量，而且它具有更多丰富的功能。

print()函数的详细语法格式：
```python
print(value,……,sep = '',end = '\n',file = sys.stdout,flush = False)
```
```python
user_name ＝ 'Charlie'
user_age = 8
#同时输出多个变量和字符串
print("读者名：",user_name,"年龄：",user_age)
```

运行上面代码，可以看到如下输出结果：
```
读者名： Charlie 年龄： 8
```

## 指定分隔符sep
从输出结果来看，使用print()函数输出多个变量时，print()函数输出多个变量时，print()函数默认用空格隔开多个变量，如果读者希望改变默认的分隔符，可通过sep参数进行改变
```python
#同时输出多个变量，并且指定分隔符
print("读者名：",user_name "年龄：",user_age,sep='|')
```
运行结果
```
读者名：|Charlie|年龄：|8
```

## 更改输出结束后是否换行end
在默认的情况下，print()函数会在执行结束后自动换行，因为print()函数中的end参数的值默认为`\n`。如果希望print()函数结束后不换行，则更改end参数的默认值即可：
```python
print(40,'\t',end="")
print(50,'\t',end="")
print(60,'\t',end="")
```
运行结果：
```
40  50  60
```

## 输出目标的更换file
file参数指定了print()函数的输出目标，file参数的默认值式sys.stdout，该默认值表示系统标准输出，也就是表示输出到屏幕，因此print()函数的默认值也是输出到屏幕，实际上我们可以用这个参数，将输出内容输出到指定的文件中
```python
f = open("demo.txt","w")#打开文件以写入
print('飞流直下三千尺',file = f)
print('疑是银河落九天',file = f)
f.close()
```
## 控制输出缓存flush
print()函数的flush参数用来控制输出缓存，该参数一般保持false即可，这样可以保持较好的性能。