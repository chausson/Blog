---
title: 如何正确使用UIImage加载图片
categories: 图片 #分类
date: 2016-07-04 11:37:52
tags:
---

# 关于UIImage
首先我想先介绍一下UIImage这种数据类型,它是一种管理图片Data数据的对象,对象是不可变的,当你想要使用动画的时候一个图片的对象可能包含一个或一组照片，它的继承关系如下:
``` bash
├── UIImage
|   ├── NSObject<NSSecureCoding>

```
由此可见UIImage是直接继承NSObject,没有UIResponer等响应事件，可以把它理解成一种保存图片数据的对象。

## 官方文档的介绍是这样的:
* A UIImage object manages image data in your app. You use image objects to represent image data of all kinds, and the UIImage class is capable of managing data for all image formats supported by the underlying platform. Image objects are immutable, so you always create them from existing image data, such as an image file on disk or programmatically created image data. An image object may contain a single image or a sequence of images you intend to use in an animation.

在项目中经常会用到UIImage这种数据,并且包括它的一些相关api，比如
``` obj-c
	[UIImage imageNamed:]
 [UIImage imageWithData:]
	[UIImage imageWithData: scale:]
	[UIImage imageWithContentsOfFile:]
	[UIImage imageWithCGImage:]
	[UIImage imageWithCIImage:];
	[UIImage animatedImageNamed: duration:]
```
# API的介绍
``` obj-c
  [UIImage imageNamed:]
```
在项目中使用最多或者最常见的API应该就是这个imageNamed了，可是你是否真的熟悉它的用法呢？下面着重的说一下的这个API。
* 如果系统的缓存中有该图片对象，将会从缓存中获取，如果没有该对象，那它将会从沙盒中寻找该名称PNG格式的图片，保存至对象中，并加载到缓存中。
* 它的优点是会把所有该方法创建的UIImage加载时会进行缓存,当频繁需要加载该图片资源的时候考虑使用。
* iOS的内存非常重要并且在内存消耗过大时，首先会强制释放缓存中的图片，即会遇到memory warnings。
* 因为从iOS9之后它的线程才变的安全,之前的线程不安全性导致出了很多内存泄露的问题,例如当一个UIView对象的animationImages是一个装有UIImage对象动态数组NSMutableArray，并进行逐帧动画。当使用imageNamed的方式加载图像到一个动态数组NSMutableArray，这将会很有可能造成内存泄露。
* 不过当某些资源文件过大时，并且只会使用一次的情况下,会特别占用资源，所以当所需要加载的图片不频繁使用的情况下不推荐使用该方法来初始化UIImage,应使用其他方法来创建UIImage(比如imageWithContentsOfFile: or imageWithData)，这样就不会加载到内存中。

``` obj-c
  [UIImage imageWithData:]
  [UIImage imageWithContentsOfFile:]
```
* Data是将传入的NSData文件数据编码成UIImage,如果不能将该数据转换成UImage的数据格式则会返回空,也可以通过如下方法控制数据的倍数:

``` obj-c
  + (UIImage *)imageWithData:(NSData *)data
                       scale:(CGFloat)scale
```

* File是根据文件的路径去创建Image对象，和imageNamed:不同的是它不会加到系统的缓存中，用完就会被释放.



``` obj-c
  [UIImage imageWithCGImage:]
```
首先先介绍下CGImageRef是什么?它是定义在QuartzCore框架中的一个结构体指针，用C语言编写。在CGImage.h文件中，我们可以看到下面的定义：
``` bash
typedef struct CGImage *CGImageRef;
CGImageRef 和 struct CGImage * 
```
它们两者是完全等价的。这个结构用来创建像素位图，可以通过操作存储的像素位来编辑图片。
通过如下方法重新绘制图片的绘制方向和倍数:
``` obj-c
+ (UIImage *)imageWithCGImage:(CGImageRef)imageRef
                        scale:(CGFloat)scale
                  orientation:(UIImageOrientation)orientation
```

``` obj-c
  [UIImage imageWithCIImage:]
```
CIImage是CoreImage框架最常用的类(这里就不对CoreImag多做介绍了,需要了解可以看这里[CoreImage](http://www.csdn.net/article/2015-02-13/2823961-core-image)),这里是将创建CIImage对象转换成UIImage对象,并且它有和CGImage相同的扩展方法,通过如下方法重新绘制图片的绘制方向和倍数:
``` obj-c
+ (UIImage *)imageWithCGImage:(CGImageRef)imageRef
                        scale:(CGFloat)scale
                  orientation:(UIImageOrientation)orientation
```

``` obj-c
  [UIImage animatedImageNamed: duration:]
```
官方文档中的解释是这样的:根据给出的名称在资源包中去加载一系列图片,按照图片的名称序号从0开始一直到1024比如图片名称叫icon,那资源包中必须要有icon0,icon1,icon3等。
为了尝试效果我进行了以下尝试，将三张图片分别命名icon0,icon1,icon3,代码如下:
``` obj-c
    UIImageView *image = [[UIImageView alloc]initWithImage:[UIImage animatedImageNamed:@"icon" duration:3]];
    image.backgroundColor = [UIColor grayColor];
    image.frame  = CGRectMake(130, 250, 100, 50);
    [self.view addSubview:image];
```
展示效果:
<img src="/gif/animation.gif"  title="animation"><a href=""></a>

## 内存问题
当系统内存紧张时，UIImage会将图片数据从UIImage对象中清理出去来以节省系统内存，这里的清理行为只是清理UIImage内部存储的图片数据，并不清理UIImage对象本身。当程序使用一个图片数据被清理过的UIImage对象时，该UIImage将会自动从原始的图片文件中加载图片数据。

## 总结
尽量避免使用UIImage加载过大（如大于1024像素×1024像素）的图片，如果程序实在需要加载这种大图片，可以考虑将该图片分解成多张小图片进行加载。
根据不同的情况需要使用不同的图片处理方式,由此衍生出各种不同的Loader。我的想法是封装一个ImageLoader类在工程中，可以增加各种不同的接口应对各种情况，如果创建的UIImage对象需要缓存我们可以加一个参数isCache。比如:
``` obj-c
+ (UIImage *)ch_imageName:(NSString *)name isCache:(BOOL)cache;
```
所有初始化UIImage或者转换UIImage的格式以及网络加载图片的方法都可以通过Loader去调用。
