---
title: 使用hugo搭建个人blog
date: 2019-07-01T09:21:57+08:00
topics:
  - blog
  - hugo
  - travis
tags:
  - hugo
  - blog
  - travis
summary: 本文介绍如何基于hugo搭建自己的个人blog，包括blog的意义、blog服务选择、hugo介绍以及使用travis进行一定程度自动化等内容。
---
## TL;DR
- 程序员一定要写技术博客！
- 如果你只是想写点东西，那么推荐使用[掘金](https://juejin.im/)、[知乎专栏](https://zhuanlan.zhihu.com/)、[简书](https://www.jianshu.com/)、[medium](https://medium.com/)等服务，几乎可以认为完全免费，甚至通过激励计划可能赚钱；如果你选这条路，可以关掉这篇文章了
- 如果你想要有自己的一块空间，那么推荐使用[Github Pages](https://pages.github.com/)这类静态网页托管服务（过程没有想象的那么难），而且写完的内容可以分享到上面的网站

如果你决定了托管静态页面，那大概需要以下几步：

- 安装hugo
- hugo new site创建新blog站点
- 挑选主题并进行配置
- hugo new posts/new-blog
- 使用markdown写blog
- 使用Github Pages托管静态网站
- （可选）成为高贵的RMB玩家，选一个属于自己的域名
- **使用travis-ci完善自动发布流程**，这一步一般人不会告诉你

## Why Blogging?
作为一个老码农，我一直有一个不好的习惯，就是不写blog。作为一个佛系blogger（17年到现在已经停更了快2年），原因有很多，因为懒、因为不知道该写什么、因为不知道写了有什么用。

不过随着年纪增大，慢慢感觉记忆力有些衰退了，有的文档看过了以为自己记住了，但过了几天又忘记了，每次都要重新阅读，导致效率很低。而且阅读的内容总感觉没有深入理解，一些corner情况会考虑的不周。

就这个问题我进行了一些反思，主要原因是阅读时只是被动的接受，而没有主动思考，所以看过以后对内容的印象偏低，所以很快就忘了。

解决方法其实也很简单，只需要坚持一个原则，所有学过的东西都要有输出。输出可以有多种，可以是写demo项目，可以是用到实际工作中，可以是重新造个轮子，当然也可以是**写blog**。

写blog与其说是记录，更像是一个信息、知识的梳理过程。在这个过程中通过重新组织、理解来加深对信息的理解。

因此我决定重新开始写blog，确保做到**学到的新知识都记录相关blog**。

## 写Blog的平台
在决定写blog以后，第一个要做的选择是使用成熟的blogging服务，还是自己建站。

如果你只是想找个地方撰写内容，那么我建议还是使用现有的blogging服务吧，有很多的好处，包括内嵌的用户系统、feed流、自带图床、自带分享系统以及相应的激励体系。

我个人更倾向能有一块自己的空间，域名、风格可以更加个性化，所以我还是选择了自己建站。但是这两者并不冲突，例如我可以把自己blog的URL直接分享到掘金，这样二者的优势就可以部分兼得。

## 静态网页托管
我搭建个人blog的一个强需求是使用静态网页托管服务，比如Github Pages。

**千万不要**尝试通过租VPS来托管页面。这里面涉及到成本、资源安全等一系列问题，要维护起来非常麻烦。如果你不了解我可以举个简单的例子，你的blog里面有一个图片，如果你把图片放到服务器上，那么每次读者阅读你的blog，就会去你的服务器上下载这个图片，这些都需要耗费服务器的带宽资源，如果你的文章不小心成了爆款，那么恭喜，你的信用卡估计也快要爆了。你可能会说可以用CDN啊，没错，那你就需要在类似cloudflare的网站上注册，还得进行相关配置，这样就引入了另外一套系统，更加复杂了，不要忘记我们的初心只是想写个blog而已。所以还是建议直接使用Github Pages吧，让github的大佬们帮我们处理这些问题！

## Hugo简介
决定好使用Github Pages以后，我们就需要寻找**可以生成静态blog的框架**。目前比较火的有[hugo](https://gohugo.io/)和[hexo](https://hexo.io/)。Hugo使用Go语言开发，Hexo使用JS开发。这两者功能和使用上大同小异，我没有进行仔细的对比，随缘选择了hugo。

这里我不想对比hugo与hexo，在我看来这二者本质是相似的，我想重点介绍下hugo框架的大致原理，因为这有助于理解后续的网站托管过程。

### 静态页面生成器
hugo的主要工作是根据你撰写的内容生成静态页面。

静态页面的本质可以分为3部分：内容、样式和交互。样式和交互都是相对比较固定的要素，一但确定很少会变更；而内容是每篇blog都不同的地方，所以一个优秀的blog框架要做到的就是让blogger可以专注于内容撰写。

hugo解决问题的方法也基本是目前的标准方法，就是**模板填充**，具体来说，样式和交互抽象成**模板**，一组模板形成一个**主题**；模板里面会有一些**占位符**，等着用实际的blog内容填充。

在写blog时通常blog的布局都比较统一，我们只希望专注内容的撰写，**markdown**就是一种很好的语言。通过markdown我们可以在写内容的时候描述我们希望内容如何布局，但又不用关心具体的样式，这可以极大提高生产效率。hugo使用markdown撰写blog内容，但又增加了一部分blog特有的信息，这个稍后再说。

当blogger写完了一篇blog，就可以委托hugo帮他生成新的静态内容，hugo会使用blogger设置的模板加上blog的内容，通过**模板引擎**把占位符替换成实际blog的内容，并生成静态页面，这样一个流程就完成了，具体流程可以参考下图。

![Hugo流程]()

作为使用者，我们需要关心这几点：

1. 选择并配置一个自己喜欢的主题
2. 使用markdown撰写blog的内容
3. 生成的静态内容会在一个叫public的文件夹内
4. 有了完整的静态内容，就可以进行发布流程了

### 安装hugo
如果你的系统安装了homebrew（或者linuxbrew），那么hugo的安装非常简单，一个命令：`brew install hugo`，你也可以通过下载二进制的方式进行安装，具体可以参考[官网](https://gohugo.io/getting-started/installing)。

### 使用cli作为操作接口
根据刚刚的描述hugo作为blog框架帮我们做了很多事情，那我们怎么跟hugo交互呢，主要就是通过hugo cli。安装完hugo后，我们就可以使用 `hugo` 命令于hugo交互啦。我把常见命令与功能列成了一个表格，参考：

| 命令名称 | 参数 | 功能描述 | 样例 |
| ------ | ------ | ------ | ------ |
| hugo version | 无 | 检查版本号 | hugo version |
| hugo new site ${SITENAME} | SITENAME，站点名称 | 创建一个新的hugo站点，并填充初始化内容 | hugo new site my-blog |
| hugo new ${BLOGNAME} | BLOGNAME，blog的完整路径 | 在指定路径创建一个blog内容 | blog new content/posts/my-new-blog.md |
| hugo server | -D，编译draft内容，新blog默认是draft模式，不会生成京台内容，加上-D强制生成这些blog内容，主要用于本地预览 | 启动一个本地托管server，可以预览blog内容 | hugo server -D |
| hugo | 无 | 编译静态内容 | hugo |

下面会具体介绍几个常用的命令

## 创建站点
使用 `hugo new site XXX` 命令可以很便捷的创建一个hugo站点。

![hugo站点内容](/assets/blog-with-hugo/hugo_directory_structure.png)

可以看到hugo给新的站点创建了一个目录，并且填充了几个文件夹。各个文件夹的含义hugo有一个[官方文档](https://gohugo.io/getting-started/directory-structure/)介绍。

通常情况下你只需要记住：

- archetypes是使用hugo new命令创建新blog的模板，如果你想调整hugo new生成的文件默认内容就修改这个目录下的文件
- content下面放具体的动态内容，比如blog内容就会放到content下面
- static存放静态内容，blog中难免出现一些图片内容，放到这里就对了
- themes存放你选好的主题

## 基础配置
基础框架生成后，第一个要做的事情是挑选一下我们心仪的**主题**（还记得么，主题是一组模板），hugo有一个官方的主题站，叫[themes.gohugo.io](https://themes.gohugo.io/)，可以按喜好挑选，也可以自己创建主题，这[部分内容](https://gohugo.io/themes/creating/)我在这里就不赘述了。

以最简单的官方主题为例：

![hugo theme](/assets/blog-with-hugo/hugo_theme.png)

通常每个主题都会提供自己的安装方法，通常大同小异，主要就是把theme当作git的submodue引入进来，然后修改对用的config.toml配置即可。如果不知道如何配置，可以参考hugo主题提供的demo站点。

## 撰写blog
之前有提到过，每个blog除了内容部分以外，还有一个很重要的是描述blog的一些属性。举例来说，blog的标题叫什么、什么时候写的blog、blog的标签是啥、主题又是啥，这些都是blog的元数据。

hugo使用markdown来描述blog的正文内容，那这些元数据如何表示呢？hugo选择在markdown文件头添加一部分信息，并管这些信息叫**front matter**。

### blog的元数据，front matter
为了区别front matter与正文内容，front matter使用一对 `---` 作为起止界限。这篇blog的front matter如下：
```
---
title: "使用hugo搭建个人blog"
date: 2019-07-01T09:21:57+08:00
draft: false
topics:
  - blog
  - hugo
  - personal
tags:
  - hugo
  - blog
summary: 本文介绍如何基于hugo搭建自己的个人blog，包括blog的意义、blog服务选择、hugo介绍以及使用travis进行一定程度自动化等内容。
---
这里开始就是blog的正文了。
```

可以看到front matter使用toml，支持字典与数组来描述各个属性。具体的写法也很好理解，没必要啰嗦了。

### 使用hugo new命令创建blog内容模板
如果每次创建都需要啰哩啰嗦写一些通用头比较麻烦，因此hugo给我们提供了一个`hugo new XXX`的命令，通过这个命令，可以生成一个包含front matter的md文件，至此我们就可以打开自己喜欢的markdown编辑器，开心的写blog了！

### 预览
编写过程中我们可以使用 `hugo server -D` 启动一个本地的hugo server，然后访问它提供的http服务就可以预览内容了，使用这个功能可以帮助我们更好的感知发布后正式环境的样式，非常方便。而且这个server自带热加载功能，文件修改**保存**后会（fsnotify）自动刷新页面，非常方便。

预览如下：

![hugo preview](/assets/blog-with-hugo/hugo_preview.png)

点击read more还可以看到具体内容

![hugo preview](/assets/blog-with-hugo/hugo_preview_2.png)


## 发布！
历尽千辛万苦，终于把blog写好啦，接下来就到了最重要的发布环节！

### Github Pages
这里就要重点介绍下Github Pages这个良心服务啦，这个服务让用户可以为自己（或组织）以及自己的项目生成静态展示页面，并且免费提供托管服务。

Github Pages的详细介绍可以参考[官网](https://pages.github.com/)，我这里简单介绍下他的两种模式以及使用方法。

#### 个人页面
个人模式常用于介绍个人或者组织，比如Github现在的东家 [microsoft.github.io](https://microsoft.github.io)，他们的opensource网站就是托管在github上的。

对于这类网站，需要在Github上创建一个独特名字的repo，如果你的名字是`username`，那么这个repo的命名是：`username.github.io`。Github Pages会以静态网站的方式托管这个项目`master`分支的内容。刚刚微软的例子就可以对应到[microsoft/microsoft.github.io](https://github.com/microsoft/microsoft.github.io)这个项目。

#### 项目页面
刚刚可以看到，由于repo的命名规则，一个用户只能有一个介绍自己的repo。那么如果我想介绍某个特定的项目该怎么办呢？

Github Pages的解决方案是活用特定git的分支。具体来说，如果我的用户名是`username`，我的一个项目叫`project`并且我创建了`gh-pages`分支，那么可以通过`http://username.github.io/project`方式访问。Github Pages会识别这类分支并进行托管。

#### 总结一下
如果你想为自己设计一个个人主页，或者为你的组织设计一个介绍门户，那么可以使用第一种个人页面模式。如果你的每个项目都希望有一个专门的介绍页面，那么可以使用第二种方式。

具体的区别可以汇总为：

| 使用方式 | 数量限制 | 托管分支 | 访问域名 | 样例 |
| ------ | ----- | ------ | ------ | ------ |
| 个人页面 | 每个用户/组织一个 | master | username.github.io | microsoft.github.io |
| 项目页面 | 每个repo一个 | gh-pages | username.github.io/project | void-main.github.io/blog |

我自己的门户页面`voidmain.guru`是使用第一种方式，而blog站点`blog.voidmain.guru`则是开了一个`void-main/blog`的项目并使用项目页面托管的。这两种方案除了前者有唯一性限制外没有本质的区别，所以可以放心选用。

你可能注意到了，我的域名并不是标准Github Pages规范，这是因为Github Pages支持自定义域名。

自己的域名只要花钱就可以从[GoDaddy](https://www.godaddy.com/)这样的域名服务商买到，Github Pages提供了配置方法，这里我就不展开了，如果有兴趣可以参考[文档](https://help.github.com/en/articles/quick-start-setting-up-a-custom-domain)

#### 插曲：注意到域名了么？
Github主站的域名是`github.com`，为啥所有托管网站的域名都是`github.io`呢？这个主要是基于安全考虑，他们的[这篇blog](https://github.blog/2013-04-05-new-github-pages-domain-github-io/)值得一读。

## 实战
说了这么多理论，该动手实践一下啦！由于我个人页面已经被占用了，所以我们来展示一下如何使用一个项目页面做个人blog。**如果你选择使用个人页面托管，只是分支和访问url不同，其他的流程是一致的**。

### Step 1 - 本地blog配置

![Step 1](/assets/blog-with-hugo/InAction_Step1.png)

使用`hugo new site`命令创建本地hugo项目，然后选择对应的主题并进行配置。

基本配置完成后，使用`hugo new posts/my-first-post.md`创建一个blog，注意hugo会自动把blog内容放到`content`目录下。

最后使用`hugo server -D`在本地进行预览

### Step 2 - 源文件提交

![Step 2](/assets/blog-with-hugo/InAction_Step2.png)

为了保证我们内容的安全，在源文件写完以后需要提交到Github上。这一步就是基本的git操作，不再赘述。

### Step 3 - 生成静态文件

![Step 3](/assets/blog-with-hugo/InAction_Step3.png)

blog修改完成后，可以把blog的draft模式置为false，这样我们的hugo命令就会帮我们生成啦，生成的结果都会放到`public`目录下。

### Step 4 - 创建gh-pages分支并发布
这一步是单纯的git命令操作，我就不再截图了，基本就是checkout一个gh-pages分支，然后把里面的内容都删掉，把之前public的内容放置到根目录，然后commit、push即可。

### Step 5 - 访问

![Step 5](/assets/blog-with-hugo/InAction_Step5.png)

此时如果访问Github sample项目，在项目的设置中，就能看到Github已经检测到这个项目有Github Pages需求，同时会告知访问方式，点击就可以访问啦！

至此整个过程就完成啦，你已经可以看到自己的主页上线啦！

## 唉，发布太麻烦了
所以我们现在来梳理一下流程。每次要发布新的blog我们需要走这么几步：

1. 在源码分支撰写blog，编写完成后将源内容commit、push到源码分支上
2. 并通过hugo命令编译成静态页面，输出的结果在`public`目录下
3. 将`public`目录的内容复制到托管分支（取决于你是个人主页还是项目主页，分别复制到`master`和`gh-pages`分支）
4. 将托管分支的内容commit、push到Github上。

这里面涉及大量分支切换与git操作，很繁琐且容易出错。

我们设想一下理想的方式是只操作源分支（不做分支切换），blog内容写完以后，只要把修改push到源码分支，然后页面的内容会自动更新。

我们可以使用travis帮我们实现这个工作。

我们可以配置travis监听某个项目特定分支的变更，如果发现变更，执行我们预设的脚本，这样我们就可以在脚本中写好命令实现之前的编译、复制、切换分支、push流程。这个流程有一些配置，网上有 [很多](https://docs.travis-ci.com/user/deployment/pages/) [很好](https://ssk7833.github.io/blog/2016/01/21/using-TravisCI-to-deploy-on-GitHub-pages/) 的 [教程](https://neveryu.github.io/2019/02/05/travis-ci/)。

放心，这个脚本不用你来写！你可以直接复制我的[个人页面](https://github.com/void-main/void-main.github.io/blob/source/.travis.yml)模板或[项目页面](https://github.com/void-main/blog/blob/master/.travis.yml)模板。

有了travis，我们目前的blog流程就变成这样了：
![blog流程]()

1. `git checkout -b blog-XXX` 创建一个blog分支
2. 在新分支上使用 `hugo new content/posts/XXX.md` 撰写blog
3. 任意次git commit、git push到 blog-XXX 分支
4. 在确认文章可以发布后，将 `blog-XXX` 合并到 `source` 分支
5. `git push`
6. 等几分钟刷新页面即可

是不是相当方便 :)

### 总结
- blog是个好习惯，可以帮助我们梳理知识，加深理解与记忆，花点时间记blog不会有错
- 可以使用商业blog服务，也可以自己搭建静态站点
- hugo使用主题（一组模板）来定义样式与交互，使用markdown来表示内容，通过模板引擎生成静态页面
- 使用Github Pages托管静态页面，还可以自定义域名
- 使用Git管理blog内容，可以快速实现版本控制、发布修改等一系列功能
- 使用Travis帮我们发布静态页面，省去不必要的麻烦，让我们注意力专注在写作上