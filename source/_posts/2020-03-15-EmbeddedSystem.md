---
title:  "Embedded_System"
date:   2020-03-15 12:16:18 +0800
categories:
- C/C++ Embedded
tags:
- 嵌入式 


---

这篇文章主要记述了嵌入式系统的一些笔记。

<!--more-->

![1583895066834](https://i.loli.net/2020/03/29/Rjme4ygfMbuYkcD.png)

![1583135575292](https://i.loli.net/2020/03/30/OAK3RvSBZ5pMqEF.png)

# NandFlash与NorFlash的区别

![1584345668683](https://i.loli.net/2020/03/30/JnTlka8t6hqpEvr.png)

![1584345681200](https://i.loli.net/2020/03/30/n1cMJYev75dIQmg.png)

# STM32

```c
OSTaskCreate(start_task,(void *)0,(OS_STK *)&START_TASK_STK[START_STK_SIZE-1],START_TASK_PRIO );

//这个函数用来管理任务的执行，任务可以在多任务开始或运行之前被创建。

INT8U  OSTaskCreate (void   (*task)(void *p_arg),

​                     void    *p_arg,

​                     OS_STK  *ptos,

​                     INT8U    prio)
```

第一个参数一个指针，也就是用户代码的首地址; 在平时使用中我们把自己创建的任务的名字作为这个参数就可以了； 第三个参数是指向任务堆栈栈顶的指针，通常我们把创建的任务的堆栈数组的首地址给第三个参数就可以了；第四个参数是任务的优先级；

void   (*task)(void *p_arg)-------> 定义了一个指向函数的指针task，所指向的函数无返回值，参数为可指向任意类型的指针；

普通的禁止和使能中断，在禁止中断时有触发中断的事件发生，当使能中断时该中断不会在响应，说明普通的禁止中断是阻止了中断的发生。



而在进入临界段时，有中断发生，在退出临界段时中断会得到响应，说明临街段只是延迟了中断的响应时间，并没有真正的阻止中断。

# VCZH大佬

< >引用的是编译器的类库路径里面的头文件

" "引用的是你程序目录的[相对路径](https://www.baidu.com/s?wd=相对路径&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YvmWw-rycYrHIBPj9WPARd0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPjc4rjndP1f4)中的头文件

![1576645965036](https://i.loli.net/2020/03/30/o5bcml1y2hHi63a.png)

![](https://i.loli.net/2020/03/30/F85vsyDShmIuOLe.jpg)

C语言定义变量后面加冒号，数字什么意思?

该种形式出现于结构体或共用体的定义中，是位域定义的标准形式。

其使用方式为

```c
struct name
{
type var_name : n;
};
```

含义为，在结构体name汇总，成员变量var_name占用空间为n位。

n为正整数，其值必须小于type类型占用的位数。比如type如果是int，占4字节32位，那么n必须是1~31之间的整数。

对于位域类型的成员，在赋值时如果实际值超过n位所能表达的范围，那么超出部分将会被截掉，只保存低位值。如int var:4,本身只有4位的空间，如果赋值var = 20, 由于20的二进制值为10100，实际为五位，这时var实际被赋值的就是低四位，0100，即4。



由于C语言中的地址是针对字节计算的，所以位域类型的成员变量不支持取地址操作，即对于变量v, 如果存在位域成员变量var，那么&a.var是非法的，编译会出错。

![1576646119827](https://i.loli.net/2020/03/30/l1gSMzWcPZXrGYa.png)

![image-20200330195249657](https://i.loli.net/2020/03/30/rf7C3MPA2eTIUg1.png)