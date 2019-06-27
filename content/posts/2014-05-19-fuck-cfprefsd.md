---
layout: post
title:  NSUserDefaults无法保存？
date:   2014-05-18 18:27:30
categories: 
  - cocoa
---
FML。。花了一上午调了一个bug。。

事情是这样的，我正在写的这个mac应用用到了Core Data，所以就把Core Data的文件放到了`~/Library/Container/my.app.container`这个目录下。但是在开发的过程中entity的结构总会发生变化，在基本稳定之前我也不想写升级那些，所以就偷懒*把container目录给删了*。

上午在用`NSUserDefaults`保存用户的选项的时候，当前保存成功，调用`[[NSUserDefaults standardUserDefaults] synchronize]`也返回`YES`，但是就是重启应用之后保存的内容就消失了。去`~/Library/Preferences`目录下找也确实没有对应的文件。

调了一上午，尝试了各种解决方案，也没搞定，最后终于在这个[SO问题里面找到了答案](http://stackoverflow.com/questions/22242106/mac-sandbox-created-but-no-nsuserdefaults-plist)。

关键是answer下面的第一个comment：

>Also if you move the container while testing / debugging to the trash, the cfprefsd (see Activity Monitor) still keeps a link to the .plist. Empty the trash and force quit both cfprefsd (user and root).

用ps一看果然有2个cfprefsd，有一个应该就是之前删除container的时候留下的，把它kill了，然后重试就好了！

感谢 [@mahal](http://stackoverflow.com/users/317461/mahal-tertin)，真是救了我一命！
