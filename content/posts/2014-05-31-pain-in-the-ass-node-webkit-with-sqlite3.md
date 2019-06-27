---
layout: post
title:  在node-webkit中使用sqlite3
date:   2014-05-31 15:38:27
categories: 
  - node-webkit
  - sqlite3
---
这两天在调研使用node-webkit开发Mac应用并且提交到Mac App Store的可能性。这类客户端应用有非常通用的一点就是需要一个本地数据存储功能。根据node-webkit的[官方wiki](https://github.com/rogerwang/node-webkit/wiki/Save-persistent-data-in-app)，我觉得最适合的就是所谓的Web SQL Database了，同时文档中也说是使用sqlite3来实现的。因此就需要实现sqlite3与node-webkit的整合。

为了测试，我直接clone了[一个windows版的demo](https://github.com/zycbob/node-webkit-sqlite3-windows-demo)。最开始我的尝试是使用node-webkit 0.9.2版（使用最新版的目的是为了保证node-webkit没有使用一些过时的或者private的api导致MAS审核悲剧）。首先我用`npm install sqlite3`安装了sqlite3 for node，结束之后运行`nw node-webkit-sqlite3-windows-demo`，结果提示error：


    Error: Cannot find module '/Users/voidmain/WorkSpace/NodeJS/node-webkit-sqlite3-windows-demo/node_modules/sqlite3/lib/binding/node-v11-darwin-ia32/node_sqlite3.node'
    

如果直接google这个错误会有人说明使用`nw-gyp rebuild --target=<node-webkit-version> --arch=ia32`来rebuild一下（如果没有安装nw-gyp的话需要先npm install一下），但是使用node-webkit 0.9.2的话，编译不会通过，会提示有error，于是继续google，终于在sqlite3项目的[issue 265](https://github.com/mapbox/node-sqlite3/issues/265)中看到了这个：

![node-webkit 0.9.2 with sqlite3](/assets/node-webkit-0_9_2-sqlite3-error.png)

看来是死胡同了，无奈只能使用更早版本的node-webkit，所以我换成了[node-webkit 0.8.6版](http://dl.node-webkit.org/v0.8.6/node-webkit-v0.8.6-osx-ia32.zip)，然后再次用nw-gyp编译，还是出错，错误是`Undefined variable module_name in binding.gyp while trying to load binding.gyp`，继续google，最后又在wiki中找到了解答：[Build native modules with nw gyp](https://github.com/rogerwang/node-webkit/wiki/Build-native-modules-with-nw-gyp)，关键的一段是：

>For some packages you may need to use node-pre-gyp (e.g. when you get the error "Undefined variable module_name in binding.gyp while trying to load binding.gyp"), which supports building for both node.js and node-webkit by using either node-gyp or nw-gyp.
>
``` bash
$ npm install node-pre-gyp
$ node-pre-gyp build --runtime=node-webkit --target=0.3.3
```

至此，再次编译就能通过，而且运行无压力了。但是仍然有两点顾虑，首先0.8.6版本的node-webkit是否对Mac App Store友好，其次，在sandboxing环境下数据库文件存储的位置问题。为了解决这两个疑惑可能只能通过一下小应用来测试一下了。有后续进展我会继续更新blog。
