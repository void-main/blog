---
layout: post
title: 使用Swift(1)
date: 2014-06-04 08:04:36 +0800
comments: true
categories: 
  - swiftapple
  - osx
  - cocoa
---

## 序言
Apple新推出的Swift编程语言无疑会成为最近码农研究的热点，现在官方有一本[官方Guide](https://itunes.apple.com/us/book/swift-programming-language/id881256329?mt=11)，这本书已经有国内的开发者开始翻译了（#该来的总会来的#）。

当然只看完Swift的语法还不足以进行Cocoa应用开发，用郭总的话说，语言并不会带来新的起跑线，要想开发优秀的Cocoa应用还是需要对Cocoa这个框架的深入理解。这就跟用RubyMotion仍然要学习Cocoa是一个道理，万变不离其宗。这方面就可以参考[Using Swift with Cocoa and Objective-C](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/buildingcocoaapps/index.html#//apple_ref/doc/uid/TP40014216-CH2-XID_0)这篇文档。

另外，Swift刚刚推出一天就已经有开发者用它开发了[FlappyBird](https://github.com/fullstackio/FlappySwift)，可见大家对这门语言的热情。这个系列的博客主要是记录我在学习Swift语言过程中感觉有趣的语法、用法，并不会对所有语法都逐一进行分析，而是举几个简单的例子帮我自己来理解Swift这门语言。如果这些例子能帮助你对Swift这门语言建立一些基本的认识或者感觉，那就再好不过了！

PS：博客内容的记录顺序大部分是根据官方Guide来的，因为这是我的阅读顺序，在阅读过程中可能会根据我的理解跳跃穿插一些内容。<b>另外因为我也是在逐渐阅读文档，所以可能写过的内容会有多次修订。</b>

## 环境要求
要使用Swift必须要安装Xcode 6 Beta版，安装Xcode6不需要10.10，在10.9上就可以使用。

## 代码
本文中使用道德测试代码都可以在[SwiftWithCocoa](https://github.com/void-main/SwiftWithCocoa)这个repo中找到，大部分代码都在`MyPlayground`中。

## “变量”不变
第一次接触这个概念是在了解Scala的时候，`Scala`中有两个关键字，分别是`val`和`var`，用`var`声明的变量跟其他语言中的一样，可以改变值，但是`val`声明的变量，一旦第一次赋值之后就无法改变了，也就是所谓的“变量”不变。这种类型主要应用在多线程的场景中，可以有效的避免资源抢占，死锁等情况的发生，从语言级保证了代码的稳定性和执行效率。

Swift也提供了类似的声明方法，分别是`let`和`var`，用`let`声明的是常量，用`var`声明的是变量。

## 基本数据类型
### Tuple
Tuple应该是我从python、ruby转到OC之后感觉最需要的类型。Tuple最大的贡献在于能轻便的创建一些临时对象，并在不同的领域使用。比如函数返回的时候可以利用tuple便捷的返回多个值，这是现在很多流行语言都支持的。

``` objc
// Returning from func
func response() -> (Int, String) {
    return (404, "Not Found")
}
var code: Int, description: String
(code, description) = response()
// code == 404
// description == "Not Found"
```

我记得我最开始学python，交换变量的方法真是让我震惊了，Swift里面（因为支持了tuple，所以）也有类似的方法了：

``` objc
// Swapping vars
var first = 1
var second = 2 // Change 2 to "ASDF" and see the error
(first, second) = (second, first)
// first == 2
// second == 1
```

需要注意的是，因为Swift强调的是类型安全，所以上面例子中的`first`和`second`必须要是同样的类型才能交换，如果类型不同需要进行显式的类型转换，下面会进行讨论。

总而言之有了tuple之后，用Swift传递和处理数据将变得非常灵活。

### 类型推断(Inference)与Optionals
Swift自夸的一个特性就是类型安全，变量声明（或第一次赋值）之后，编译器会根据开发者显式指定（`let num: Int`）或者隐式赋值推断（`let num = 1`）得到变量类型，一旦确定，之后就不能改变类型了，例如：

``` objc
var num = 1
num = "123" // Error!
```

第二行就会报错，因为num已经被编译器推测为Int类型了，所以再赋值为String就会出错，所以上面才提到的，使用tuple赋值的时候两种类型必须一致。为了解决这个问题，我们可以用强制类型转型：

``` objc
// Explict type conversion and unwarp
first = 1
var third = "123"
(first, third) = (third.toInt()!, String(first))
first
second
```

上面的例子中，就是把`third:String`利用`toInt()`强制转换成了`Int`，这样`first:Int`就能接受了。下面我们重点说一下`toInt()!`最后的这个`!`。

#### Optionals
Optionals实际是一种类型，这种类型就是在原有类型的后面加上一个大大的`?`。对于Cocoa开发者来说，这种变化应该并不陌生，因为Objective-C里面的`SEL`类型，`:`也是类型的一部分，比如`gotNotification`跟`gotNotification:`就是两个`SEL`对象。

如果说一个变量的类型是`XX?`，那么代表的意思是，要么这个变量有值，且类型为`XX`，要么这个变量没有值。这个类型主要用在可能会出错的时候，例如将字符串转换为数字，可能字符串根本代表的不是数字，对于这种情况有的语言选择抛出运行时异常，Swift则选择了返回一个值为空的optionals对象。在Swift的代码框架中，大量系统级的API返回的都是optionals对象。

但是如何使用`XX?`的值呢，或者说如何将`XX?`转换为`XX`呢？这就要用到`!`了。所以上一节中说到的`third.toInt()!`实际过程是`"123"`这个`String`类型的变量通过`toInt()`转换为了`Int?`，但是咱们的`first`对象类型为`Int`，所以需要将`Int?`通过`!`转换为`Int`。这下整个过程就清楚了。

另外optionals还常用在if判断中，可以通过`if let`来避免使用`!`：

``` objc
var text:String? = "123"
if let v = text {
    println(v)
}
```

最后值得一提的是隐式optionals对象，就是在定义的时候使用`!`而不是`?`，这样的好处是系统可以自动将`XX?`转换为`XX`对象了。

基于optionals还有更多神奇的语法糖，比如Craig在Keynote里面展示的那段代码，我们以后再分析。

### 语法糖 - Trailing closure syntax
本来准备再下篇博客中再分析closure，但是有一个语法感觉可以提前说明一下，就是`Trailing closure syntax`。在Objective-C中，大家都非常熟悉block的使用了，基本上很多常用的API都提供了block方式，比如`NSBlockOperation`：

``` objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:{
    NSLog(@"Got block!");
}];
[queue addOperation:blockOperation];
```

如果直接对应过来的话，应该写成：

``` objc
let operationQ = NSOperationQueue()
let operation = NSBlockOperation(block: {
    println("Got operation!")
})
operationQ.addOperation(operation)
```

对于这种最后一个参数接受closure的情况，可以将最后一个参数移到调用括号的外面，看起来就像定义了一个block一样：

``` objc
let operation = NSBlockOperation() {
    println("Got operation!")
}
operationQ.addOperation(operation)
```

这样写起来不仅省了代码，而且看上去也比较舒服了。
