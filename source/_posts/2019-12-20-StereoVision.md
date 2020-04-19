---
title:  "Stereo_Vision"
date:   2019-12-18 10:16:18 +0800
categories:
- C++
tags: 
- CV 
---


这篇文章主要记述了双目立体视觉的一些笔记。
<!--more-->









立体匹配算法分类：从算法本身的角度来说，立体匹配从大的方面分为两种，全局匹配和局部匹配。全局匹配比较经典的有动态规划、图割和置信度传播等，全局匹配算法优点是精度高，缺点是计算速度慢，实时性差；而局部匹配又分为基于区域的匹配，基于特征的匹配等。局部匹配优点是速度快，可以满足实时的要求，但是精度不如前者，基于区域的匹配精度尚可以接受，基于特征的匹配则只能得到稀疏的视差，需要靠插值完成视差图重建，如果要考察整个图的视差精度，则基于特征法不可靠。

图割，置信传播等全局优化立体匹配算法，由于运算过程中需要迭代求精，运算时间长，无法达到实时计算立体匹配的需求，然而实时性需求却广泛存在立体匹配的应用场景中。很多基于局部匹配的算法虽然运算时间短，但由于仅考虑匹配窗内的代价聚合，效果很差，视差图只有很多稀疏的视差点，还要经过插值计算，显然无法用于汽车导航，目标拾取等需要精确结果且对运算速度有一定要求的场景。

### 1局部代价聚合

基于窗结构局部立体匹配算法，按照匹配约束来搜索最佳的匹配点，在搜索求取左右两幅图像在视差d下一点的匹配代价时，实际是求得以该点为中心的匹配窗内所有点的代价的平均值（或者其他的度量方式）。如图（4-1）：
![image-20191228192046004](https://i.loli.net/2020/03/29/UM7wueaFz8NYHOh.png)
图 4-1 局部匹配算法的代价传递
我们把这一过程叫做代价聚类（Cost aggregation），这种基于区域的匹配方法利用局部窗口之间的相似性度量来匹配对应基元的空间坐标，对于连续性细节明显的区域效果较好。显然，此类方法对于匹配窗以外的点却无法影响该点的代价值，使得代价聚类的值不具有全局特性，也就丧失了匹配基元的全局结构特性，因此在纹理特征较低的区域非常容易产生误匹配。
如何在代价聚类中获取匹配基元的全局特征，进而使得局部代价聚合方法克服上述缺点，本章相对于基于区域的局部窗立体匹配方法，采用图论中的最小生成树方法，利用树结构进行全局代价聚合。

## 双边滤波

双边滤波（Bilateral filter）是一种可以保边去噪的滤波器。简单的说就是一种同时考虑了像素空间差异与强度差异的滤波器，因此具有保持图像边缘的特性。该滤波器可以由两个滤波参数进行控制。一个控制几何空间距离。另一个控制像素差。

![image-20191228192031108](https://i.loli.net/2020/03/29/e5UZLkOwYBDEnxR.png)

 

## 12-21-工作汇报

最近根据标准[双目数据集](http://vision.middlebury.edu/stereo/data/scenes2014/)利用一系列双目匹配算法进行了实验，本次实验共选取BM、SGBM、GC算法进行比较。其中各个算法的原理论文分别为:

- BM:[介绍双目匹配原理非常清晰的PDF](http://vision.deis.unibo.it/~smatt/Seminars/StereoVision.pdf)
- SGBM:[Stereo Processing by Semi-global Matching and Mutual Information](https://ieeexplore.ieee.org/abstract/document/4359315/)
- GC:[Realistic CG Stereo Image Dataset with Ground Truth Disparity ](http://www.cvlab.cs.tsukuba.ac.jp/~kfukui/english/epapers/trakmark2012.pdf)

以上原理的论文资料均已整理至/reference文件夹内.

> 另外关于近20年双目匹配的[论文整理](https://blog.csdn.net/xuyuhua1985/article/details/26283389)

### 论文实验结果如下：

我们采用的数据集标准图片为:

![image-20191219222637363](https://i.loli.net/2020/03/29/vbdLBYe2RDNMkQx.png)

**SADWindowSize**：匹配代价计算的窗口的大小,容许范围是`[5,255]`，一般应该在 `5x5` 至` 21x21 `之间，参数必须是奇数.`blockSize(SADWindowSize)` 越小，也就是匹配代价计算的窗口越小，视差图噪声越大；`blockSize`越大，视差图越平滑；太大的`size`容易导致过平滑，并且误匹配增多，体现在视差图中空洞增多；

**numberOfDisparities**:控制视差的数量.最大的深度值减去最小得到的值

### 使用BM算法:

![image-20200403110857415](https://i.loli.net/2020/04/03/q1JQaHMSUNgGduz.png)

### 使用SGBM算法:

![image-20191221161355163](https://i.loli.net/2020/03/29/buKD1eiTymXoNBQ.png)

### 使用GC算法:

![](https://i.loli.net/2020/03/30/NDTKwRxUpfnjHBY.png)

![image-20191219222703531](https://i.loli.net/2020/03/29/pxrBIXukhefKE36.png)

## 接下来工作进展:

进一步理解`SGB`算法代码实现原理,采用自己拍摄的牙齿图片用上述算法进行测试,进而整理出文档出来.

## 12-25-工作汇报

- 移植了`ADCensus`算法，算法相关原理参考论文 [On Building an Accurate Stereo Matching System on Graphics Hardware](http://www.nlpr.ia.ac.cn/2011papers/gjhy/gh75.pdf)

  > `ADCensus`双目立体匹配算法曾一度排在`MiddleBurry`官网上第一的位置，引用量很高。其主要优点是并行加速，精度又好，能有效兼顾这两点。
  >
  > [ADCensus详细介绍链接](https://www.cnblogs.com/sinbad360/p/7842009.html)
  >
  > 

- 对`ADCensus`进行标准`midleburry`数据集测试，测试结果如下图：

![image-20191225220436594](https://i.loli.net/2020/03/29/ukBfDqr6tngUMS8.png)

> 使用标准数据集进行匹配可以看出匹配效果还不错。

- 对`ADCensus`进行双目相机测试，测试结果如下图：

![image-20191225220552499](https://i.loli.net/2020/03/29/UKDT5LXv3ey8Cm2.png)

> 使用自己买的双目相机进行匹配可以看出匹配效果非常不好。

我开始猜想可能是因为我的双目相机左右成像色调差别太大导致的，根据`PS`调出的直方图可以看出标准数据集的左右照片直方图相似度很高；而我们自己买的双目相机拍出来的左右相机的直方图差别却很大。

![image-20191225210751049](https://i.loli.net/2020/03/29/qjcZ2mADX1WGdYU.png)

![image-20191225210727207](https://i.loli.net/2020/03/29/QrKXDnvIfiLy3ux.png)

![image-20191225210823719](https://i.loli.net/2020/03/30/BxdLDkMjN7eynX4.png)

![image-20191225210808554](https://i.loli.net/2020/03/29/5EHZ1xgz8nPMjGt.png)

为了验证自己的猜想，我又选取了另外一套用于自动驾驶的双目数据集`KITTI`，在经过`ADCensus`算法运行之后，匹配效果如图：

![image-20191225221540846](https://i.loli.net/2020/03/29/N4CMjAUvXw8GthK.png)

> 可以看出使用`KITTI`数据集匹配的效果也还不错。

综上：我打算接下来将自己买的双目相机拍出来的左右照片进行一下直方图匹配，看一下`ADCensus`算法在牙齿模型的匹配效果究竟如何，然后撰写最近一个星期的工作总结。

另外，我重新整理了`SVN`的代码与参考资料，对`OpenCV `3.40重新`CMake`，对于一些将来可能会用到的`CUDA`模块熟悉了一下。



## 12-28工作汇报

1.发现David 3D对于牙齿的建模效果不错，于是调研了一下：它们是采用双目相机+结构光来对牙齿模型进行建模的。

> **DAVID Laserscanner** is a software package for low-cost 3D laser scanning. It allows scanning and digitizing of three-dimensional objects using a camera (e.g. a web cam), a hand-held [line laser](https://en.wikipedia.org/wiki/Line_laser) (i.e. one that projects a line, not just a point), and two plain boards in the background. Its most distinctive feature is that the laser line can simply be swept over the object by hand (like a virtual brush) until the results are satisfactory. At the same time, DAVID Laserscanner generates 3D data in real time and shows them on the computer screen.
>
> The resulting 3D mesh can be exported into well-known file formats and can thus be imported and processed in most 3D applications. The software is also able to grab the texture and "stitch together" scans made from different viewing directions.

[关于David 3D的视频的详细介绍Intro to DAVID SLS-2 3D Scanner](https://www.bilibili.com/video/av80957886/)

![image-20191228122640270](https://i.loli.net/2020/03/29/CwR4hsUfi8edvNF.png)

2.联系了深圳威鑫视界公司对之前购买的双目摄像头重新进行烧录，这是烧录之前一对有着明显色调差异的双目照片：

![image-20191227204722135](https://i.loli.net/2020/03/29/UkrpEWsePXwxvS3.png)

经过对方重新烧录之后，可以看出两幅图片起码色调一致了：

![image-20191228213354929](https://i.loli.net/2020/03/29/3E2MwSqzFOnZe4R.png)

接着重新进行相机的标定与图片的矫正，此次共拍摄8组照片，双目匹配结果如下：

![image-20191228213816820](https://i.loli.net/2020/03/29/4x3DJ7bXTQYpq6A.png)

![image-20191228213855628](https://i.loli.net/2020/03/29/uqhUwGvLtXzBTcC.png)

![image-20191228214240868](https://i.loli.net/2020/03/29/Qqct3xFmIMSEobJ.png)

> 可以看出前两幅图建的不好，原因是左右图没有经过预处理，左右直方图区别过大导致匹配效果不好。从《基于双目立体视觉的三维重建方法研究_党乐》一文中可以根据他的做法进行左右图的预处理。

![image-20191227161700855](https://i.loli.net/2020/03/29/Og7HRikLVJTIjYs.png)

![image-20191227161716225](https://i.loli.net/2020/03/29/x7RoVkEubmIpc1h.png)

3.下一步计划看一下下面的这个方法：

> Tatsunori Taniai, Yasuyuki Matsushita, Yoichi Sato, and Takeshi Naemura. [Continuous 3D label stereo matching using local expansion moves](http://taniai.space/projects/stereo/). PAMI 40(11):2725-2739, 2018. [Code](https://github.com/t-taniai/LocalExpStereo).

以及经过预处理之后的牙齿匹配结果如何。

4.一些走的弯路;

> 双目经过棋盘格标定之后，在拍摄牙齿模型时，由于双目相机不具备自动变焦功能导致看不清楚，所以会手动拧镜头螺纹导致焦距发生变化，之前标定好的和焦距相关的数据有的就会发生变化，进而导致后续的工作出现差错。
>
> 和学姐讨论了一波，之所以我们双目匹配的效果不如人家的标准数据集好，是因为标准数据集经过了非常严格的光线调节，它们的左右图差距很小，这一点从直方图就可以看出来。我们能不能达到这样好的效果呢？一部分取决于预处理的效果，一部分取决于摄像头：如果摄像头拍的特别差，后期软件再怎么处理也是收效甚微。



## 1-3工作汇报

##### 全像素点云及重建结果

![image-20200104163734005](https://i.loli.net/2020/03/29/pOWLxoZuMC3wznc.png)

![image-20200104165024323](https://i.loli.net/2020/03/30/ZLKUYolNk7cRisA.png)

##### 0度像素点云及重建结果

![image-20200104163806059](https://i.loli.net/2020/03/30/nSOd2vy9PKWCwxE.png)

![image-20200104165929665](https://i.loli.net/2020/03/29/wFSm389bdTJezrM.png)

##### 45度点云及重建结果

![image-20200104171514994](https://i.loli.net/2020/03/29/gqirTzsxBR6E1YC.png)

![image-20200104171921350](https://i.loli.net/2020/03/29/zhZuJtaCAKXcEjT.png)

##### 90度点云及重建结果（失败）

![image-20200104172657147](https://i.loli.net/2020/03/29/VwMlQpBmTrznWAt.png)

![image-20200104173018825](https://i.loli.net/2020/03/29/3gVdlc8LJRo21qW.png)

##### 135度点云及重建结果（失败）

![image-20200104181416265](https://i.loli.net/2020/03/29/EFmLYzlXK4AUhuJ.png)

------

##### 预处理左右原图：

![image-20191231132610545](https://i.loli.net/2020/03/30/D2LWKHYrcu9UAPg.png)

![image-20191231132624181](https://i.loli.net/2020/03/30/ny5pbPQoXRIKFqE.png)

##### 经过中值滤波之后：

![image-20191231132210206](https://i.loli.net/2020/03/29/yFfm3vZC5eaE4nb.png)

![image-20191231132223593](https://i.loli.net/2020/03/29/ghKbdeqAX5DwLkG.png)

##### 经过clahe直方图均衡化之后

![image-20191231134546847](https://i.loli.net/2020/03/29/AfmkbQCPGzBViN7.png)

![image-20191231134602453](https://i.loli.net/2020/03/29/iawqlndyt3g2GcU.png)

##### 经过直方图匹配之后：

![image-20191231134624053](https://i.loli.net/2020/03/29/QWEwaCDkbfeLJ1v.png)

![image-20191231134647789](https://i.loli.net/2020/03/29/oXs5JEiKnPQcOA3.png)

##### unsharpMasking之后：

![image-20191231135358167](https://i.loli.net/2020/03/29/9eZdbnQas5SjIBY.png)

![image-20191231135407136](https://i.loli.net/2020/03/29/iRJLwet3TKaINzS.png)



![image-20191227161730595](https://i.loli.net/2020/03/29/TrBQMzuCO3E46LK.png)



![image-20191227161754508](https://i.loli.net/2020/03/29/fCDUd5GLIs8olg7.png)



	IplImage * img1 = cvLoadImage("../../DataSets/Tooth/Rectified/08left_picture.png", CV_LOAD_IMAGE_GRAYSCALE);
	IplImage * img2 = cvLoadImage("../../DataSets/Tooth/Rectified/08right_picture.png", CV_LOAD_IMAGE_GRAYSCALE);
此步骤计算初始匹配成本量。
由于可以在每个像素和每个视差级别同时执行计算，因此此步骤本质上是并行的。
我们主要关注的是开发一种具有高匹配质量的成本衡量方法。
通用成本度量包括绝对差（AD），Birchfield和Tomasi的不采样抽样度量（BT），基于梯度的度量以及诸如等级和人口普查之类的非参数转换[22]。
在Hirschmuller和Scharstein的最新评估中[3]，人口普查显示了局部和全局立体匹配方法的最佳总体结果。
尽管合并成本度量以提高准确性的想法似乎很简单，但是相对较少的工作探讨了该主题。
克劳斯等。
文献[6]提出将SAD和基于梯度的度量线性组合以进行成本计算。
他们的差异结果令人印象深刻，但是合并的好处却没有明确阐述。
人口普查使用像素强度的相对顺序（而不是强度值本身）对局部图像结构进行编码，因此可以容忍由于辐射度变化和图像噪声引起的异常值。
但是，此资产也可能在具有重复或类似局部结构的图像区域中引入匹配的歧义。
为了解决这个问题，应在该措施中纳入更详细的信息。
对于具有相似局部结构的图像区域，颜色（或强度）信息可能有助于减轻匹配的歧义。
而对于具有相似颜色分布的区域，通过窗口进行的普查变换比基于像素的强度差更鲁棒。
这种观察激发了一种综合措施。





![image-20191224213727896](https://i.loli.net/2020/03/29/OEkSHUCpNmuW3RX.png)















## 关于imwrite写入异常情况的处理

这是因为第一个参数path不给人家设置图片后缀格式导致人家找不到对应的解析器，所以抛出异常来了

![image-20191223230727334](https://i.loli.net/2020/03/29/ymxC36JlchMZ7Ba.png)






## 完成双目匹配到点云拼接的流程

[双目跑分](http://vision.middlebury.edu/stereo/eval3/)NOSS-ROB



- Superpixel alpha-expansion and normal adjustment for stereo matching



```c
	if (scale != 1.f)
	{
		Mat temp1, temp2;
		int method = scale < 1 ? INTER_AREA : INTER_CUBIC;
		resize(img1, temp1, Size(), scale, scale, method);
		img1 = temp1;
		resize(img2, temp2, Size(), scale, scale, method);
		img2 = temp2;
	}
```

## 内参intrinsics.yml

M1:3行3列的那个标定矩阵

D1:就是那个畸变参数`distCoeff`，注意是5个的还是9个的

```
 void compute( InputArray leftarr, InputArray rightarr, OutputArray disparr )
    {
        CV_INSTRUMENT_REGION()

        Mat left = leftarr.getMat(), right = rightarr.getMat();
        CV_Assert( left.size() == right.size() && left.type() == right.type() &&
                   left.depth() == CV_8U );

        disparr.create( left.size(), CV_16S );
        Mat disp = disparr.getMat();

        if(params.mode==MODE_SGBM_3WAY)
            computeDisparity3WaySGBM( left, right, disp, params, buffers, num_stripes );
        else if(params.mode==MODE_HH4)
            computeDisparitySGBM_HH4( left, right, disp, params, buffer );
        else
            computeDisparitySGBM( left, right, disp, params, buffer );

        medianBlur(disp, disp, 3);

        if( params.speckleWindowSize > 0 )
            filterSpeckles(disp, (params.minDisparity - 1)*StereoMatcher::DISP_SCALE, params.speckleWindowSize,
                           StereoMatcher::DISP_SCALE*params.speckleRange, buffer);
    }

```

## SAD方法就是以左目图像的源匹配点为中心，定义一个窗口D，其大小为（2m+1） (2n+1)，

```
    统计其窗口的灰度值的和，然后在右目图像中逐步计算其左右窗口的灰度和的差值，
    最后搜索到的差值最小的区域的中心像素即为匹配点。
    基本流程：
      1.构造一个小窗口，类似与卷积核。
      2.用窗口覆盖左边的图像，选择出窗口覆盖区域内的所有像素点。
      3.同样用窗口覆盖右边的图像并选择出覆盖区域的像素点。
      4.左边覆盖区域减去右边覆盖区域，并求出所有像素点差的绝对值的和。
      5.移动右边图像的窗口，重复3，4的动作。（这里有个搜索范围，超过这个范围跳出）
      6.找到这个范围内SAD值最小的窗口，即找到了左边图像的最佳匹配的像素块。
```



## img1 w.r.t. img2 and write it to disp1buf 就…而言with respect to

## OpenCV提供了以下四种立体匹配算法的函数：

1. Block Matching（BM） StereoBM
2. Semi-Global Block Matching（SGBM） StereoSGBM
3. Graph Cut（GC）cvStereoGCState()
4. Dynamic Programming（DP）cvFindStereoCorrespondence()

第一种就是简单的块匹配，第三，四种是基于全局的匹配，以下简单介绍一下第二种算法。我们知道局部匹配计算量小速度快，而全局匹配计算量大速度慢，所以作者提出了SGBM算法[5]: 第一步对每一个Pixel使用块匹配BM进行匹配，得到了全部Pixel的disparity map。

第二步对Disparity map建立图，用Graph Cut对其进行全局优化。利用Rectification将二维转化为一维：则对每一个像素的可能的Disparity值d，从以下4中里面选取一个最小值：

- 左相邻像素disparity取值为d时，其最小的cost值。
- 左相邻像素disparity取值为d-1时，其最小的cost值+惩罚1。
- 左相邻像素disparity取值为d+1时，其最小的cost值+惩罚1。
- 左相邻像素disparity取值为其他时，其最小的cost值+惩罚2。

它与GC的区别在于它的左相邻像素的Disparity是已知的（根据块匹配结果），而GC是未知的，所以GC要对每一种取值进行一次计算，计算量比SGBM要大得多。

[5]H. Hirschmuller, Stereo Processing by Semi-global Matching and Mutual Information,"IEEE Transactions on Pattern Analysis and Machine Intelligence, Vol. 30, Issue 2, pp. 328-341, Feb. 2008.

下图给出了各种不同算法的结果比较：

左右原图如下：



![image-20191219222650653](https://i.loli.net/2020/03/29/RlzvDBSPuYpndfg.png)





## 这个程序缺校正对齐那块以及自动调节参数那块内容

由于berry数据集不需要校正，所以和其他的程序有所不一样

如果把那些值设为0会怎么样？--可以绕过校正那一步直接到深度计算

添加一个可以在命令行动态修改参数的程序便于修改参数

![SGBM-9-32](https://i.loli.net/2020/03/30/uHJs1AY9hOvNpgB.jpg)









### CommandLineParser

比如说"{ c | camera | 0 | use camera or not}"都是用大括号和双引号引起来，然后中间的内容分成4断，用”|”分隔开，分别表示简称，文件来源，文件值和帮助语句。第二行和第三行表示打开摄像头和打开文件，文件的文件名等都在keys指针中了。

## 双目原理

## numberOfDisparities等参数

`left07.png right07.png --algorithm=bm --blocksize=9 --max-disparity=32 --scale=1 -i=intrinsics.yml -e=extrinsics.yml -o=diaparity.txt -p=pcl`

参数-algorithm共有**STEREO_BM**、**STEREO_SGBM**、**STEREO_HH**、**STEREO_VAR**、**STEREO_3WAY**

> OpenCV中SGBM算法的MODE_SGBM模式，实现了5个方向搜索，MODE_HH实现了8个方向搜索（可通过注释掉代码中部分，观察任一方向搜索得到的结果），这部分的代码，我已加了部分注释，可结合OpenCV源码对照观看，如下所示。
>
> MODE_SGBM_3WAY模式，顾名思义，从左侧、上侧和右侧三个方向进行搜索，代码中有并行处理部分，速度很快。
>
> MODE_HH4模式从上下左右四个方向搜索，代码中有并行处理部分，速度依旧很快。

\1. blockSize(SADWindowSize) 越小，也就是匹配代价计算的窗口越小，视差图噪声越大；blockSize越大，视差图越平滑；太大的size容易导致过平滑，并且误匹配增多，体现在视差图中空洞增多；
\2. 惩罚系数控制视差图的平滑度，P2>P1，P2越大则视差图越平滑；
\3. 八方向动态规划较五方向改善效果不明显，主要在图像边缘能够找到正确的匹配；

max-disparity**:必须为16的倍数

**blocksize**：匹配块的大小 Matched block size. It must be an odd number >=1 . Normally, it should be somewhere in the 3..11 range. SAD代价计算窗口大小，默认为5。窗口大小为奇数，一般在3*3 到21*21之间；1. blockSize(SADWindowSize) 越小，也就是匹配代价计算的窗口越小，视差图噪声越大；blockSize越大，视差图越平滑；太大的size容易导致过平滑，并且误匹配增多，体现在视差图中空洞增多；

**no-display**：管是否显示的

**scale**：调节图像大小的

**minDisparity**：最小的深度值，通常情况下为0，但是一些校正算法有时会使图片变形，所以你得根据实际情况进行设置

**numDisparities**：最大的深度值减去最小得到的值

**P1**：The first parameter controlling the disparity smoothness. See below.

**P2**：The second parameter controlling the disparity smoothness. The larger the values are, the smoother the disparity is.P1是相邻像素间差值变化正负1的惩罚。P2是相邻像素之间视差变化超过1的惩罚。The algorithm requires P2 > P1 .

**uniquenessRatio**：最佳（最小）计算成本函数值应“赢得”次优值以认为找到的匹配正确的百分比裕度。 通常，在5到15范围内的值就足够了。Margin in percentage by which the best (minimum) computed cost function value should "win" the second best value to consider the found match correct. Normally, a value within the 5-15 range is good enough.

**speckleWindowSize**：Maximum size of smooth disparity regions to consider their noise speckles and invalidate. Set it to 0 to disable speckle filtering. Otherwise, set it somewhere in the 50-200 range.

**mode**:将其设置为StereoSGBM :: MODE_HH即可运行满量程两遍动态编程算法。 它将消耗O（W * H * numDisparities）字节，对于640x480立体来说很大，而对于HD尺寸的图片来说很大。 默认情况下，它设置为false。

其中`minDisparity`是控制匹配搜尋的第一個引數，代表了匹配搜蘇從哪裡開始，numberOfDisparities表示最大搜尋視差數uniquenessRatio表示匹配功能函式，這三個引數比較重要，可以根據實驗給予引數值。

// 预处理滤波参数

`preFilterType`：预处理滤波器的类型，主要是用于降低亮度失真（photometric distortions）、消除噪声和增强纹理等, 有两种可选类型：CV_STEREO_BM_NORMALIZED_RESPONSE（归一化响应） 或者 CV_STEREO_BM_XSOBEL（水平方向Sobel算子，默认类型）, 该参数为 int 型；
**preFilterSize**：预处理滤波器窗口大小，容许范围是[5,255]，一般应该在 5x5…21x21 之间，参数必须为奇数值, int 型
**preFilterCap**：预处理滤波器的截断值，预处理的输出值仅保留[-preFilterCap, preFilterCap]范围内的值，参数范围：1 - 31（文档中是31，但代码中是 63）, int

**SADWindowSize：SAD**窗口大小，容许范围是[5,255]，一般应该在 5x5 至 21x21 之间，参数必须是奇数，int 型
minDisparity：最小视差，默认值为 0, 可以是负值，int 型
numberOfDisparities：视差窗口，即最大视差值与最小视差值之差, 窗口大小必须是 16 的整数倍，int 型
// 后处理参数

**textureThreshold**：低纹理区域的判断阈值。如果当前SAD窗口内所有邻居像素点的x导数绝对值之和小于指定阈值，则该窗口对应的像素点的视差值为 0（That is, if the sum of absolute values of x-derivatives computed over SADWindowSize by SADWindowSize pixel neighborhood is smaller than the parameter, no disparity is computed at the pixel），该参数不能为负值，int 型
**uniquenessRatio**：视差唯一性百分比， 视差窗口范围内最低代价是次低代价的(1 + uniquenessRatio/100)倍时，最低代价对应的视差值才是该像素点的视差，否则该像素点的视差为 0 （the minimum margin in percents between the best (minimum) cost function value and the second best value to accept the computed disparity, that is, accept the computed disparity d^ only if SAD(d) >= SAD(d^) x (1 + uniquenessRatio/100.) for any d != d*+/-1 within the search range ），该参数不能为负值，一般5-15左右的值比较合适，int 型
**speckleWindowSize**：检查视差连通区域变化度的窗口大小, 值为 0 时取消 speckle 检查，int 型
**speckleRange**：视差变化阈值，当窗口内视差变化大于阈值时，该窗口内的视差清零，int 型
// OpenCV2.1 新增的状态参数

**roi1, roi2**：左右视图的有效像素区域，一般由双目校正阶段的 cvStereoRectify 函数传递，也可以自行设定。一旦在状态参数中设定了 roi1 和 roi2，OpenCV 会通过cvGetValidDisparityROI 函数计算出视差图的有效区域，在有效区域外的视差值将被清零。
**disp12MaxDiff**：左视差图（直接计算得出）和右视差图（通过cvValidateDisparity计算得出）之间的最大容许差异。超过该阈值的视差值将被清零。该参数默认为 -1，即不执行左右视差检查。int 型。注意在程序调试阶段最好保持该值为 -1，以便查看不同视差窗口生成的视差效果。具体请参见《使用OpenGL动态显示双目视觉三维重构效果示例》一文中的讨论。
在上述参数中，对视差生成效果影响较大的主要参数是 SADWindowSize、numberOfDisparities 和 uniquenessRatio 三个，一般只需对这三个参数进行调整，其余参数按默认设置即可。

![image-20191221154914385](https://i.loli.net/2020/03/29/pePqy1cVSBmUvFK.png)

![image-20191221155003546](https://i.loli.net/2020/03/29/FmCKeQ5tTMzbWk2.png)



![image-20191220104349530](https://i.loli.net/2020/03/29/nkRLmKpv5Q2UwoB.png)



原来视差（disparity）指的是同一特征点在左右相机成像像素的距离差啊，我之前以为是求出来的灰度值是视差，天哪,这里越近的点越亮该如何理解？下面的图很好的给出了诠释。

> The disparity is the difference between the x coordinate of two corresponding points; it is typically encoded with greyscale image (closer points are brighter).

![image-20191220104922045](https://i.loli.net/2020/03/30/KS6mMe3yThlEuRH.png)

![image-20191220105323391](https://i.loli.net/2020/03/29/xdQYebCsry4mO3K.png)

还有一个问题，我之前在计算的时候有考虑过两个摄像头之间的那个间距b吗？

[双目照片](http://vision.deis.unibo.it/~smatt/stereo.htm)

![image-20191220110305832](https://i.loli.net/2020/03/29/avcsrYTZi8o6fFG.png)

通常会有一个预处理阶段主要是消除一些光学的扭曲：

典型的方法包括：

- Laplacian of Gaussian (LoG) filtering

- Subtraction of mean values computed in nearby pixels

- 双边滤波

- Census transform 

A、匹配代价计算

![image-20191220111053226](https://i.loli.net/2020/03/29/93peBV8jldiboHA.png)

![image-20191220111116408](https://i.loli.net/2020/03/29/yLuBJrV8DnoMlt5.png)

![image-20191220111153031](https://i.loli.net/2020/03/30/1l7JwnKTRIfXW4P.png)

怎样去提升土办法的表现呢？主要有两个不同的方法（非互相排斥）？

局部算法使用了WTA策略。但是减少了模棱两可的东西（通过在支持窗口上聚集匹配代价增加了信噪比的信号？）也就是内核或相关窗口。有时采用平滑项

> Local algorithms use the simple WTA disparity selection  strategy but reduce ambiguity (increasing the signal to  noise ratio (SNR)) by aggregating matching costs over a  support window (aka kernel or correlation window).  Sometime a smoothness term is adopted. Steps 1+2 (+ WTA)

全局或者半全局算法会通过使在整个双目匹配对的能量函数最小的的方法去寻找视差，

> algorithms search for disparity  assignments that minimize an energy function over the whole  stereo pair using a pixel-based matching cost (sometime the  matching cost is aggregated over a support). Steps 1+3

两个方法都假设屏幕是逐段的平滑。有时这个假设并不成立

局部算法隐式的假设，全局算法显式的进行假设

实际上，直接对一点来进行匹配，百分百会出现各种各样的问题，这个时候我们选择用一个固定窗口来替代一点，如下图所示。Warning,这样做就隐含了一个假设，认为窗口内部视差值相同，但是，显然的，这种假设太过想当然，也使得算法实际效果不好。

![image-20191220160220994](https://i.loli.net/2020/03/29/JO5B36srdezMhLE.png)

![image-20191220113042597](https://i.loli.net/2020/03/29/AI9FgxoERXMOGdj.png)

![image-20191220113011574](https://i.loli.net/2020/03/29/VKzC7keTPbHntOR.png)

常用的聚合方式包括，SSD的直接求和、DP搜索等等。SGBM用到的就是多方向DP搜索方法

![image-20191220113421395](https://i.loli.net/2020/03/29/LcZkbJvSgmuRCzr.png)

最小值{求当前像素与右边像素的差值的最小值，求当前像素与左边像素的最小值}

![image-20191220142057081](https://i.loli.net/2020/03/29/2dTaJ4kSLPsqYwO.png)

DSI是一个三维矩阵，每一个元素代表左边参考像素的光照强度与右边目标像素滑动距离d的光照强度之差

固定窗口为什么会表现的不尽如人意呢？

![image-20191220142653422](https://i.loli.net/2020/03/29/4JWyClMSIPQdDKo.png)

1.是因为固定窗口只会在前表面平行的进行移动，然而很多情况是物体表面是凹凸不平的，有时候物体是倾斜的，理想的情况是窗口也是倾斜的。

2.是因为固定窗口忽视了深度的不连续，在一个窗口内就有两种不同深度的背景



![image-20191220143111375](https://i.loli.net/2020/03/29/ZAserD1ohnIYtzx.png)

![image-20191220143129084](https://i.loli.net/2020/03/29/tcJA3rQspfKbYG8.png)

理想的cost aggregation strategy 应该尽可能的扩展具有相同深度的像素到它的区域里

![image-20191220143400586](https://i.loli.net/2020/03/29/JshpvSeCFaMHWyz.png)

![image-20191220145500641](https://i.loli.net/2020/03/29/TuWz7rU1gIAbtaF.png)

![image-20191220151510820](https://i.loli.net/2020/03/29/1GKs75YTejSfZqo.png)



匹配代价计算是整个立体匹配算法的基础，实际是对不同视差下进行灰度相似性测量。常见的方法有灰度差的平方SD（squared intensity differences），灰度差的绝对值AD（absolute intensity differences）等。另外，在求原始匹配代价时可以设定一个上限值，来减弱叠加过程中的误匹配的影响。以AD法求匹配代价为例，可用下式进行计算，其中T为设定的阈值。

![image-20191220110712285](https://i.loli.net/2020/03/30/XCAEOnS5UhcoK8H.png)



# QT  KITTI

其中：

- S_xx：1x2 矫正前的图像xx的大小
- K_xx：3x3 矫正前摄像机xx的校准矩阵
- D_xx：1x5 矫正前摄像头xx的失真向量
- R_xx：3x3 （外部）的旋转矩阵(从相机0到相机xx)
- T_xx：3x1 （外部）的平移矢量(从相机0到相机xx)
- S_rect_xx：1x2 矫正后的图像xx的大小
- R_rect_xx：3x3 纠正旋转矩阵(使图像平面共面)
- P_rect_xx：3x4 矫正后的投影矩阵

S_00: 640 480



DEFAULT_CAM_TO_CAM_TXT_PATH //
DEFAULT_IMU_TO_VELO_TXT_PATH //
DEFAULT_VELO_TO_CAM_TXT_PATH //
    读取了一个时间戳image_timestamp_files
    image_00 //left camera
    image_01 //right camera































