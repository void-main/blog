---
layout: post
title: "NSImage在读取高DPI图像时的bug及解决方案"
date: 2014-07-13 20:11:40 +0800
comments: true
categories: cocoa, NSImage, image
---
这可能是所有用NSImage的开发者都会遇到的一个坑：为什么我的图像用NSImage打开之后变小了？

比如下面这个图：

![Image Size with High DPI](/assets/high-dpi-image-size.png)

它的分辨率是2848x4288，但是，如果我用下面的代码打印出来的话，大小却只有854.4x1286.4。

``` objc
NSImage *srcImage = [[NSImage alloc] initWithContentsOfURL:url];
NSLog(@"before: %@", NSStringFromSize(srcImage.size));
```

这段代码看起来已经简洁的不能再简洁了吧，应该没有问题才对啊。但是实际问题出现在DPI这里。NSImage的size计算是按照DPI为72的值计算的，做个简单的实验，如果用Photoshop打开刚刚这幅图，然后用Image Size工具将DPI调成72（保持各种比例关系不变），就能看到这个854.4怎么来的了：

![Change DPI](/assets/try-to-change-dpi-with-photoshop.png)

其实要解决这个问题并不是特别困难。尽管`NSImage`在计算尺寸的时候是按72dpi来计算的，但是`NSImage`的内部表示`NSBitmapImageRep`还是有字段保留着图像的实际尺寸，分别是`pixelsWide`跟`pixelsHigh`。因此只要利用这两个实际大小来计算，或者干脆中心绘制一下这个NSImage就可以了。核心代码如下：

``` objc
- (NSBitmapImageRep *)bitmapImageRepresentation {
    // NSImage可能包含很多representation，需要迭代一下
    NSArray * imageReps = [self representations];
    float width = 0;
    float height = 0;

    for (NSImageRep * imageRep in imageReps) {
        // 利用pixelsWide跟pixelsHigh来获得实际图像分辨率
        if ([imageRep pixelsWide] > width) width = [imageRep pixelsWide];
        if ([imageRep pixelsHigh] > height) height = [imageRep pixelsHigh];
    }
    
    if(width < 1 || height < 1)
        return nil;
    
    // 重新绘制
    NSBitmapImageRep *rep = [[NSBitmapImageRep alloc]
                             initWithBitmapDataPlanes: NULL
                             pixelsWide: width
                             pixelsHigh: height
                             bitsPerSample: 8
                             samplesPerPixel: 4
                             hasAlpha: YES
                             isPlanar: NO
                             colorSpaceName: NSDeviceRGBColorSpace
                             bytesPerRow: 0
                             bitsPerPixel: 0];
    
    NSGraphicsContext *ctx = [NSGraphicsContext graphicsContextWithBitmapImageRep: rep];
    [NSGraphicsContext saveGraphicsState];
    [NSGraphicsContext setCurrentContext: ctx];
    // 实际绘制代码，把全部图像(fromRect: NSZeroRect)画到全尺寸的矩形中(drawInRect:NSMakeRect(0, 0, width, height))
    [self drawInRect:NSMakeRect(0, 0, width, height) fromRect:NSZeroRect operation:NSCompositeCopy fraction:1.0];
    [ctx flushGraphics];
    [NSGraphicsContext restoreGraphicsState];
    
    return rep;
}
```

这个核心文件我已经托管到[github](https://github.com/void-main/NSImage-BitmapRep)了。