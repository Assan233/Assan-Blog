---
title: V8 内存管理及垃圾回收机制
date: 2020-03-30 19:47:55
summary: V8 将内存分为两类：新生代内存空间和老生代内存空间，新生代内存空间主要用来存放存活时间较短的对象，老生代内存空间主要用来存放存活时间较长的对象。对于垃圾回收，新生代和老生代有各自不同的策略
categories: 转载
tags: 
- Chrome V8
- 垃圾回收机制
top: true
cover: true
img:
keywords: 
- Chrome V8
- 垃圾回收机制
---


由于 V8 引擎的原因，Node 在操作大内存对象时受到了一些限制，在 64 位的机器上，默认最大操作的对象大小约为 1.4G，在 32 位的机器上，默认最大操作的对象大小约为 0.7G。  
如果我们的 Node 程序会经常操作一些大内存的对象，可以对这个默认值进行修改：

```swift
node --max-old-space-size=1700 index.js
node --max-new-space-size=1024 index.js
```

其中，`max-old-space-size` 表示设置老生代内存空间的最大容量，`max-new-space-size` 表示这只新生代内存空间的最大容量。但这两个值也是有上限的，不能无限设置，其中老生代内存空间最大的值约为 1.7G，新生代最大内存空间约为 1.0G。  
至于新生代和老生代，这是 V8 对内存的一个分类，后文再介绍。  
回到操作大内存对象的问题，如果 1.7G 的内存还是不够大怎么办呢？要知道 1.7G 是在 V8 引擎层面上做出的限制，要想避开这种限制，我们可以使用 `Buffer` 对象，`Buffer` 对象的内存分配在 C++ 层面进行，不受 V8 引擎限制。  
注：通过 `process.memoryUsage()` 方法可以用来查看 V8 引擎的内存使用量，通过 `os.totalmem()` 方法和 `os.freemem()` 方法分别可以查看操作系统的总内存和空闲内存。

```dart
// 查看 V8 的内存使用情况
process.memoryUsage()
{ 
  rss: 31469568,
  heapTotal: 7708672,
  heapUsed: 5152856,
  external: 8609 
}

// 查看操作系统总内存
os.totalmem()
8279511040
// 查看操作系统的空闲内存
os.freemem()
1610977280
```

通过上面几个方法获取到的内存都是以字节为单位。

## 新生代和老生代

V8 将内存分为两类：新生代内存空间和老生代内存空间，新生代内存空间主要用来存放存活时间较短的对象，老生代内存空间主要用来存放存活时间较长的对象。对于垃圾回收，新生代和老生代有各自不同的策略，下面依次进行介绍。

## 新生代垃圾回收

新生代内存中的垃圾回收主要通过 Scavenge 算法进行，具体实现时主要采用了 Cheney 算法。Cheney 将内存空间一分为二，每部分都叫做一个 Semispace，这两个 Semispace 一个处于使用，一个处于闲置。处于使用中的 Semispace 也叫作 From，处于闲置中的 Semispace 也叫作 To。  
在垃圾回收运行时时，会检查 From 中的对象，当某个对象需要被回收时，将其留在 From 空间，剩下的对象移动到 To 空间，然后进行反转，将 From 空间和 To 空间互换。进行垃圾回收时，会将 To 空间的内存进行释放，如下图所示：

  

![](//upload-images.jianshu.io/upload_images/3831834-e536b4847cb877c7.png?imageMogr2/auto-orient/strip|imageView2/2/w/578/format/webp)

新生代垃圾回收.png

  

简而言之，就是 From 空间中最终存放不需要被回收的对象，To 空间最终中存放需要被回收的对象，当垃圾回收运行时，将 To 空间中的对象全部进行回收。

## 新生代对象的晋升

前面说过，新生代内存空间用来存放存活时间较短的对象，老生代内存空间用来存放存活时间较长的对象。新生代中的对象可以晋升到老生代中，具体有两种方式：

  ### 1. 多次声明晋升
在垃圾回收的过程中，如果发现某个对象之前被清理过，那么会将其晋升到老生代内存空间中
  ### 2. 占用大内存晋升
在 From 空间和 To 空间进行反转的过程中，如果 To 空间中(保留的数据)的使用量已经超过了 25%，那么就将 From 中的对象直接晋升到老生代内存空间中

## 老生代垃圾回收

说完新生代的垃圾回收，再来看下老生代中的垃圾回收。首先，老生代内存空间和新生代内存空间的结构不一样，其实是一个连续的结构，而不像新生代内存空间那样分为 From 和 To 两个部分：

  

![](//upload-images.jianshu.io/upload_images/3831834-1d07d7f8a236d28d.png?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

老生代内存空间.png

  

老生代内存空间中的垃圾回收有标记清除（Mark Sweep）和标记合并（Mark Compact）两种方式。

## Mark Sweep

Mark Sweep 是将需要被回收的对象进行标记，在垃圾回收运行时直接释放相应的地址空间，如下图所示(红色的内存区域表示需要被回收的区域)：

  

![](//upload-images.jianshu.io/upload_images/3831834-0f7cb788d78a70f6.png?imageMogr2/auto-orient/strip|imageView2/2/w/595/format/webp)

标记清除.png

  

如上图所示，使用 Mark Sweep 进行垃圾回收会产生一个问题，就是垃圾回收后内存会出现不连续的情况，为了解决这个问题，出现了 Mark Compact 方案。

## Mark Compact

Mark Compact 的思想有点像新生代垃圾回收时采取的 Cheney 算法：将存活的对象移动到一边，将需要被回收的对象移动到另一边，然后对需要被回收的对象区域进行整体的垃圾回收。

  

![](//upload-images.jianshu.io/upload_images/3831834-1ce7943b7e5b33b6.png?imageMogr2/auto-orient/strip|imageView2/2/w/630/format/webp)

标记合并.png

  

上图展示了在老生代内存空间使用 Mark Compact 进行垃圾回收的过程。

总结
--

本文主要简单介绍了 Node 的内存管理和垃圾回收相关的知识：

*   V8 的大对象操作限制问题和解决方案
*   获取内存使用量的几个方法
*   新生代和老生代的概念
*   新生代和老生代的垃圾回收方案

