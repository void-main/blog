---
layout: post
title: "在Swift代码中整合C++类库"
date: 2014-07-01 20:39:34 +0800
comments: true
categories: 
  - swift
  - cpp
  - cocoa
  - app
---

最近想用Swift开发一些小玩具，其中一个应用需要用到Box2d这个物理引擎，所以就遇到了如何将C++代码与Swift代码整合的问题。

在项目中整合Box2d并不困难，可以直接在Podfile里面添加`pod 'box2d'`，比较麻烦的是怎么在代码中使用。

在WWDC的Session 406: `Integrating Swift with Objective-C`中，Apple只是介绍了怎么将Swift代码跟Objective-C代码做整合，但是没有提C++，后来在官方文档中看到了这样一段话：

> You cannot import C++ code directly into Swift. Instead, create an Objective-C or C wrapper for C++ code.

这就很简单了，首先我们需要创建一个ObjC的类，用类创建向导很容易就能完成这个工作：

![创建ObjCWrapper](/assets/Create-ObjC-Wrapper.png)

在创建过程中Xcode会提示是否需要创建bridge，选择创建就好了。

接下来就可以编辑`XXXX-Bridging-Header.h`这个文件了，根据我的需要，这里应该`#import <Box2d/Box2d.h>`，所以我就直接把这句话放到bridging header里面了，编译，BOOM!

> \<unknown\>:0: error: /path/to/project/Pods/Headers/Box2D/Common/b2Settings.h:22: 'cassert' file not found

如果google这个问题的话，可以看到各种答案都是说应该把`.m`文件替换成`.mm`文件，但是我现在压根没用上我刚刚创建的`VMBox2dWrapper.m`，这就是问题所在。这里需要做2个修改，一个是把`VMBox2dWrapper.m`的后缀替换成`.mm`，另外一个是把`#import <Box2d/Box2d.h>`移动到这个`.mm`文件里面，而`XXXX-Bridging-Header.h`这个文件里面`#import "VMBox2dWrapper.h"`。经过这两个改动以后，就可以顺利编译了。

还有一个问题是这个Wrapper类里面写什么，基本就是看项目需要用到什么再添加什么方法了，因为我的项目才刚刚开始，如果后续有什么需要注意的地方再来添加。