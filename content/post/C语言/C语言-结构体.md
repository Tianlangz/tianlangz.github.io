---
author: zty
title: C语言-结构体
date: 2022-05-15
description:  C语言
series:
  - C语言
tags : [
    C语言基础,二级考试
]
categories : [
    编程基础
]
series : [二级考试]
aliases : [C语言基础]
---
```
基本结构体
结构体数组
结构体指针

```
<!--more-->
# 结构体详解
我们之前讲过了数组，数组中是存储一组相同类型数据的集合，但在实际应用中，有时候我们需要存储的并不是相同的类型的数据，那存储不同类型的数据就不能用一个是数组来存放了。

在C语言中，可以使用结构体(Struct)来存放一组不同类型的数据：
```
struct 结构体名{
    结构体包含的变量和数组;
};
```
结构体大括号后面的分号一定不能省略。

结构体是一个包含多个类型数据或数组，它们可以是同类型的，也可以是不同类型的。而存在结构体中的每个变量都是这个结构体的成员(Member):
```c
struct stu{
    char *name;
    int num;
    int age;
    char group;
    float score;
};
```
此结构体总共包含了五个成员，结构体成员的定义方式和变量与数组的定义方式并无不同，只是不允许初始化。

当你定义完一个结构体变量后它就成为了程序员自己定义的一种数据类型，和其他的数据类型并无不同，而且她还可以包含多个不同数据类型的数据。

像 int、float、char 等是由C语言本身提供的数据类型，不能再进行分拆，我们称之为`基本数据类型`；而结构体可以包含多个基本类型的数据，也可以包含其他的结构体，我们将它称为`复杂数据类型`或`构造数据类型`。

## 结构体变量
既然结构体是一种数据类型，那我们就可以用它来定义变量。
```c
struct stu stu1, stu2;
```
用结构体定义两个变量，他们都是我们所定义的结构体类型，都由五个成员组成，记住struct关键字不能少。

stu就跟我们常定义的普通数据类型类似，因为stu也只是代表了是程序员自己定义的一种数据类型，只是其中包含的是多种类型的数据。

当然我们也可以在定义结构体时，定义结构体变量：
```c
struct stu{
    char *name;
    int num;
    int age;
    char group;
    float score;
}stu1, stu2;
```

在理论上讲结构体的各个成员在内存中是连续存储的，和数组非常类似，也就是说在理论上讲结构体在内存中共占用4+4+4+1+4=17个字节
![](/images/结构体1.jpg)

但是在编译器具体实现中，各成员之间可能会存在缝隙，会变成同一长度进行计算，例如上述情况，在系统中计算出占用内存大小会显示20个字节
![](/images/结构体2.jpg)

而这种情况就不得不提内存对齐机制了：
    - 默认情况：由编译器和操作系统决定，一般来说32位系统对齐系数为4（字节）；64位系统对齐系数为8（字节）
    - 自定义情况：通过#pragma pack(n) 定义
在这里我就不过多赘述了。

# 成员的获取和赋值
成员赋值的格式为：
```
结构体变量名.成员名;
```
以下两种方式都可以给结构体变量赋值。
```c
#include <stdio.h>
int main(){
    struct{
        char *name;  //姓名
        int num;  //学号
        int age;  //年龄
        char group;  //所在小组
        float score;  //成绩
    } stu1;
    //给结构体成员赋值
    stu1.name = "Tom";
    stu1.num = 12;
    stu1.age = 18;
    stu1.group = 'A';
    stu1.score = 136.5;
    //读取结构体成员的值
    printf("%s的学号是%d，年龄是%d，在%c组，今年的成绩是%.1f！\n", stu1.name, stu1.num, stu1.age, stu1.group, stu1.score);
    return 0;
}
```
```c
#include <stdio.h>
int main(){
    struct{
        char *name;  //姓名
        int num;  //学号
        int age;  //年龄
        char group;  //所在小组
        float score;  //成绩
    } stu1, stu2 = {"Tom", 12, 18, 'A', 136.5};
    printf("%s的学号是%d，年龄是%d，在%c组，今年的成绩是%.1f！\n", stu1.name, stu1.num, stu1.age, stu1.group, stu1.score);
    return 0;
}
```

需要注意的是，结构体是一种自定义的数据类型，是创建变量的模板，不占用内存空间；结构体变量才包含了实实在在的数据，需要内存空间来存储。

# 结构体数组
当然我们既然可以用结构体定义变量，那么我就可以用结构体定义数组：
```c
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组 
    float score;  //成绩
}class[5] = {
    {"Li ping", 5, 18, 'C', 145.0},
    {"Zhang ping", 4, 19, 'A', 130.5},
    {"He fang", 1, 18, 'A', 148.5},
    {"Cheng ling", 2, 17, 'F', 139.0},
    {"Wang ming", 3, 17, 'B', 144.5}
};
```
当我们是吧数组中的全部元素都赋值进去后，我们也可以不写数组长度。引用时也是一样，没有任何问题：
```c
#include <stdio.h>
struct{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
}class[] = {
    {"Li ping", 5, 18, 'C', 145.0},
    {"Zhang ping", 4, 19, 'A', 130.5},
    {"He fang", 1, 18, 'A', 148.5},
    {"Cheng ling", 2, 17, 'F', 139.0},
    {"Wang ming", 3, 17, 'B', 144.5}
};
int main(){
    int i, num_140 = 0;
    float sum = 0;
    for(i=0; i<5; i++){
        sum += class[i].score;
        if(class[i].score < 140) num_140++;
    }
    printf("sum=%.2f\naverage=%.2f\nnum_140=%d\n", sum, sum/5, num_140);
    return 0;
}
```
运行结果：
```
sum=707.50
average=141.50
num_140=2
```

当一个指针指向结构体时，我们就称它为结构体指针：
```
struct 结构体名 *变量名；
```
```c
//结构体
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
} stu1 = { "Tom", 12, 18, 'A', 136.5 };
//结构体指针
struct stu *pstu = &stu1;
```

也可以在定义结构体的同时定义结构体指针：
```c
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
} stu1 = { "Tom", 12, 18, 'A', 136.5 }, *pstu = &stu1;
```

在定义时一定记住结构体变量名和数组名是不一样的，数组名在表达式中会被转换为数组的指针，而结构体变量名不会，无论任何表达式中它表达的都是整个集合本身，想要去的地址必须加&。

`这里重点提一下结构体变量名和结构体名是不一样的，结构体是一种数据类型，是我们自己创建的一种数据类型，是一种创建一种变量的模板，编译器并不会给它们分配内存空间，就像int、char、float等关键字不占用内存一样；结构体变量才包含实实在在的数据，才需要内存来存储。`下面两种指针方式就是不可能的。
```c
struct stu *pstu = &stu;
struct stu *pstu = stu;
```

## 利用结构体指针获取结构体成员
利用结构体指针获取结构体成员，有两种方式：
```c
(*pointer).memberName
//或者
pointer -> memberName
```
第一种写法中，`.`的优先级高于`*`，第一种方式的括号一定不能少，不然他就会和后面的指向先结合，那样意义就完全不一样了。

第二种写法，`->`是一个新的运算符，习惯称它为箭头，有了它，可以通过结构体指针直接获取结构体成员；这也是`->`在C语言中的唯一用途。
```c
#include <stdio.h>
int main(){
    struct{
        char *name;  //姓名
        int num;  //学号
        int age;  //年龄
        char group;  //所在小组
        float score;  //成绩
    } stu1 = { "Tom", 12, 18, 'A', 136.5 }, *pstu = &stu1;
    //读取结构体成员的值
    printf("%s的学号是%d，年龄是%d，在%c组，今年的成绩是%.1f！\n", (*pstu).name, (*pstu).num, (*pstu).age, (*pstu).group, (*pstu).score);
    printf("%s的学号是%d，年龄是%d，在%c组，今年的成绩是%.1f！\n", pstu->name, pstu->num, pstu->age, pstu->group, pstu->score);
    return 0;
}
```

## 结构体指针作为函数参数
结构体变量名代表的是整个集合本身，作为函数参数时传递整个集合，也就是所有成员，而不是被编译器转换成一个指针，如果结构体成员较多且有成员为数组时，传送的时间和空间的开销就会很大，影响程序的运行效率，所以最好的办法就是使用结构体指针，这时由实参传向形参只是一个地址，非常快。
```c
#include <stdio.h>
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
}stus[] = {
    {"Li ping", 5, 18, 'C', 145.0},
    {"Zhang ping", 4, 19, 'A', 130.5},
    {"He fang", 1, 18, 'A', 148.5},
    {"Cheng ling", 2, 17, 'F', 139.0},
    {"Wang ming", 3, 17, 'B', 144.5}
};
void average(struct stu *ps, int len){
    int i, num_140 = 0;
    float average, sum = 0;
    for(i=0; i<len; i++){
        sum += (ps + i) -> score;
        if((ps + i)->score < 140) num_140++;
    }
    printf("sum=%.2f\naverage=%.2f\nnum_140=%d\n", sum, sum/5, num_140);
}
int main(){
    int len = sizeof(stus) / sizeof(struct stu);
    average(stus, len);
    return 0;
}

```
运行结果：
```
sum=707.50
average=141.50
num_140=2
```
