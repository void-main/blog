---
layout: post
title: "帮他人提交Mac应用"
date: 2014-12-09 09:42:32 +0800
comments: true
categories: AppStore,Cocoa,Mac,Developer
---
最近做的一个外包项目要提交到应用市场了，但是他的开发者账户是个人账户，无法添加其他成员；同时我又不能要求他修改自己的Apple ID密码，然后发给我，这太不安全了。因为无法在XCode里面添加开发者账户，所以就不能使用XCode内置的工具上传应用了。对于这种情况，只能使用Apple提供的ApplicationLoader来进行应用发布，但是如何创建一个正确签名、可发布的应用包就成了很大的问题。

根据开发者文档来看，ApplicationLoader只接受ipa（iOS）、pkg（OSX与IAP）还有zip文件，因为我做的是Mac应用，所以就要想办法创建一个可用的pkg包。

经过一些搜索与尝试，最终还是成功把应用包提交了，下面就把过程总结一下。

## 生成app文件
具体如何archive，如何校对设置我就不详细说明了，如果有需要的话请自行google "App Distribution Guide"，值得一提的是，因为我们在XCode中没有证书与签名，所以在导出app的时候，只能选择最后一项（"Export as a Mac Application"）。

![Export as Mac Application](/assets/export-as-mac-application.png)

## 所需证书
创建pkg需要两步签名，首先要对刚刚生成的".app"签名，这里需要用到"Mac App Distribution"这个证书；接下来还要为生成的安装包签名，这里要用的是"Mac Installer Distribution"这个证书。

## 生成签名请求
虽然不能直接访问开发者账户，但是要将应用提交到应用市场一定要有开发者签名，这就需要有账户的人配合了。首先你要做好准备工作，或者说写申请。这里说的申请就是在本地创建一个签名请求。打开Keychain Access工具，按照下图选择：

![Create Signing Request](/assets/create-signing-request-file-from-access-chain.png)

因为要生成两个证书，而且根据我个人的测试，证书跟签名请求是一一对应的，所以在这一步需要创建两个签名请求文件，建议用"AppCertificateSigningRequest.certSigningRequest"跟"InstallerCertificateSigningRequest.certSigningRequest"来命名，其他的能区分的命名方式都可以。

## 指导他人生成证书
接下来就要用这个签名去请求证书了，具体的过程是：

1. 访问[开发者网站](http://developer.apple.com)，登陆Member Center。
2. 在Mac应用页面中，选择"Certificates, Identifiers & Profiles"中的"Certificates"那一项。
3. 点击右上角的"+"按钮，创建新证书。
4. 在"Production"分类中选择"Mac App Store"。
5. 在下一步页面中，选择"Mac App Distribution"。
6. 在下一步页面中，选择"AppCertificateSigningRequest.certSigningRequest"文件。
7. 点击"Generate"来生成证书（证书文件名默认为"mac_app.cer"）。
8. 重复1-7步，在第5步选择"Mac Installer Distribution"，在第六步中上传"InstallerCertificateSigningRequest.certSigningRequest"文件。这里第7步生成的证书的默认文件名是"mac_installer.cer"。

## 导入证书
分别双击导入"mac_app.cer"与"mac_installer.cer"，导入的时候选择“login”来导入当前用户的钥匙链中。导入之后最好通过名字过滤搜索，确认导入成功。

## 应用签名
有了证书之后就可以对之前生成的".app"文件签名了。命令如下：

`codesign -f -s "3rd Party Mac Developer Application: XXX" --entitlements “YYY.entitlements” "ZZZ.app"`

其中"3rd Party Mac Developer Application: XXX"就是证书中的那个名字，"YYY.entitlements"就是应用对应的entitlements的路径，"ZZZ.app"就是之前生成的app文件。

## 安装包签名
接着使用`productbuild`工具来生成安装包，命令如下：

`productbuild --component “ZZZ.app" /Applications --sign "3rd Party Mac Developer Installer: XXX" UUU.pkg`

同样的，将"ZZZ.app"跟"3rd Party Mac Developer Installer: XXX"替换为具体的应用名称和开发者名称，注意这里用的是Installer这个证书；"UUU.pkg"就是最终的安装包的名字。

## 验证安装包
到这里这个应用包就生成完毕了，最后要验证一下，命令是

`sudo installer -store -pkg UUU.pkg -target /`

在输出的内容中，特别要关注是否有一条关于未正确签名的警告。如果一切顺利的话这个包就可以用ApplicationLoader提交到应用市场了。

## 后记
这种方法不仅适用于帮他人打包，也适用于一些公司的自动化过程；另外，可能还有别的方法（比如将证书导入XCode中），我就没有详细研究了。