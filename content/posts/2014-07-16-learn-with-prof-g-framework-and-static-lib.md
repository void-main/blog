---
layout: post
title: "跟郭总学开发（1）——Mac应用使用OpenCV的问题与解决方案"
date: 2014-07-16 09:34:09 +0800
comments: true
categories: 
  - learn
  - cocoa
  - framework
  - static-lib
---
最近有机会跟[郭总](http://longtimenoc.com)合作开发一些项目，真是每天都有新姿势啊，不记录下来都觉得可惜。

## 起因
先来描述一下遇到的问题：我们的项目要用到OpenCV，如果是为iOS开发的话，直接[编译生成opencv2.framework](http://docs.opencv.org/doc/tutorials/introduction/ios_install/ios_install.html)就可以了，iOS默认将framework静态编译到最终的二进制中，但是在Mac上<s>没有现成的framework</s> framework不会静态编译，仍然要复制到最终的bundle中，这就可能带来问题。

我是用`brew install opencv`安装的opencv，没有其他特殊指令，最终是在`/usr/local/Cellar/opencv/2.4.9/lib`下生成了一堆dylib文件，然后把这些文件放到项目里面，同时选择复制到最终的bundle中。但是这样产生的`.app`在运行时会有一些路径相关的错误（一般是image not found什么的），直接就崩了。

这样我们就只能选择用静态链接库了。如果是我来解决这个问题的话，我一定会想办法安装一个生成`.a`版本的opencv，很有可能会自己编译源代码。

<b>如果你比较急着用的话，可以直接[下载opencv_osx.a.zip](http://pan.baidu.com/s/1mg0ZfAw)。</b>

## 郭总说
让我们看看郭总是怎么搞的：

> 最后是去这下了个 iOS 的 framework 把里面二进制拿出来 去掉了arm [http://sourceforge.net/projects/opencvlibrary/files/opencv-ios/](http://sourceforge.net/projects/opencvlibrary/files/opencv-ios/)
> 
> 反正要给 iOS 模拟器就有 x64 i386
> 
> framework里面有个核心的二进制 加个.a后缀 就是 static lib 直接拿过来就行了
> 
> 我嫌大 用lipo拆开 去掉了arm的重合了一个
>
> 一共只有3行操作

喂喂，略显高端了吧！

这里的要点一方面是有的iOS framework因为要支持iOS模拟器，所以会在核心二进制里面包含x64和i386可用的static lib，我们如果有需要可以利用这些生成好的二进制；

另外一方面就是`lipo`，如果你跟我一样从来没见过这个命令，赶快去`man lipo`一下吧！

## 实战
接下来我们来实战一下，首先从郭总提供的url里面下载好这个framework，找到这个核心的二进制：

![Core Binary](/assets/find-core-binary.png)

接下来先看一下这个二进制包含了哪些arch：

    ➜ voidmain@MBP  ~/Desktop  lipo -info opencv2
    Architectures in the fat file: opencv2 are: armv7 armv7s i386 x86_64 arm64
    
正如郭总所说，这里面除了给iOS准备的arm系列外，还包含着i386跟x86_64这两个mac上可用的framework，然后我们就需要剔除arm系列，给这个二进制瘦身一下：

    ➜ voidmain@MBP  ~/Desktop  lipo opencv2 -extract i386 -extract x86_64 -output opencv2_osx.a
    ➜ voidmain@MBP  ~/Desktop  lipo -info opencv2_osx.a
    Architectures in the fat file: opencv2_osx.a are: i386 x86_64
    
这样就可以用了！

## 总结
iOS跟OS X的差别其实真的没有那么大～