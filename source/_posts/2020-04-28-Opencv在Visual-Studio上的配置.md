---
title: Opencv在Visual Studio上的配置
tags:
  - Opencv
  - Visual Studio
categories:
  - Visual Studio
date: 2020-04-28 07:40:50
---

这篇文章记述了关于Opencv在Visual Studio上的配置。

<!--more-->

由于每个人的`OpenCV`、`PCL`等第三方软件安装位置不尽相同，所以项目文件的关于头文件包含路径和`lib`包含路径的配置也不尽相同，在此特意给出配置的一般流程：

1.右键解决方案-->选择属性

![lgTm8K.jpg](https://s2.ax1x.com/2020/01/08/lgTm8K.jpg)

2.进入项目属性页，选择`VC++`目录，点击包含目录

![lgTngO.jpg](https://s2.ax1x.com/2020/01/08/lgTngO.jpg)

3.把`OpenCV`的头文件路径填入到包含目录下。注意下方的`C:\Program Files\opencv`需要根据你自己的`opencv`实际路径进行替换

```c
 C:\Program Files\opencv\build\include\opencv
 C:\Program Files\opencv\build\include\opencv2
 C:\Program Files\opencv\build\include
```

4.把`OpenCV`的`lib`库的路径写到库目录里面。我图上的是`C:\Program Files\opencv\build\x64\vc15\lib`，你需要根据自己的实际安装路径以及下图的`VS`版本和`OpenCV`版本对照表来选择实际的路径。

![lgTiDJ.jpg](https://s2.ax1x.com/2020/01/08/lgTiDJ.jpg)

5.点击`链接器`-->`输入`-->`附加依赖项`，将项目所需要的lib库填写进去。这一步是很容易出错的，程序编译中如果出现下面这种错误，很大可能就是你链接的库出了问题。

  ```c
  error LNK2019: 无法解析的外部符号 __imp__wglCreateContext@4，该符号在函数 "protected: int __thiscall CSingleView::CreateViewGLContext(struct HDC__ *)" 中被引用
  ```

  赶紧检查以下问题你ok了吗？

  > 第四步所指定的lib路径是否正确？
  >
  > 如果是在`Debug`模式的话，应该链接的是`debug`的`lib`；要是在`release`模式应该链接的是`release`的`lib`。怎么区分一个lib是`debug`还是`realese`的呢？--很简单，`debug`版本的`lib`会带有`d字母`结尾的后缀。例如`opencv_calib3d340d.lib`、`opencv_core340d.lib`。
  >
  > `OpenCV`从第3版本开始会携带一个`opencv_world340d.lib`的`集大成者`，其实它就是包含了一系列单一的`lib`组成的聚合体。好处就是省却了我们一个一个的写项目需要的`lib`；坏处就是有时包含了我们并不需要的`lib`进来增加编译时间。不过你可以在`CMake`的时候通过一个`Build_World`选项来选择是否生成这个集大成者。

  6.将`OpenCV`的`Bin`目录下的`opencv_calib3d340d.dll`等动态链接库文件复制进`C:\Windows\System32`文件夹下面，不然程序运行时会弹出找不到`opencv_calib3d340d.dll`的错误，导致异常退出。这是一个一劳永逸的办法，只需要复制一次接下来就不用操心了；坏处就是这样做不仅会污染`纯净`的`windows`运行动态链接库空间；还会带来一些`潜在`的安全隐患。其他的办法包括将dll的文件夹写入环境变量的`path`路径下以及将`dll`文件复制到和程序同级的文件夹下，所有的做法都是让程序在运行时找到所需要的`dll`文件加载就可以了。

上述只是给出了关于`OpenCV`的配置，抛砖引玉，其他关于`PCL`等的配置大同小异，操作一遍之后就顺手了，其实最好根据自己的实际情况写一个`properties`属性页，然后直接包含它就行了，方便省事。

更多关于静态链接库和动态链接库的知识推荐查阅 [**程序员的自我修养—链接、装载与库**](https://github.com/Jessicahust/books/blob/master/%E7%A8%8B%E5%BA%8F%E5%91%98%E7%9A%84%E8%87%AA%E6%88%91%E4%BF%AE%E5%85%BB%E2%80%94%E9%93%BE%E6%8E%A5%E3%80%81%E8%A3%85%E8%BD%BD%E4%B8%8E%E5%BA%93.pdf)

附链关于`OpenCV`调试时`F11`进入源代码调试的教程：[在VS2017中实现OpenCV源码级调试](https://acewzj.github.io/2019/08/18/SourceDebug/) 