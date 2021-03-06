---
layout:     post
title:      "The QuickDialog class layout analysis"
subtitle:   "QuickDialog结构解析"
date:       2014-03-08
author:     "mayfly"
header-img: "img/header/post-bg-ios.jpg"
tags:
    - iOS
---

QuickDialog可以帮助开发者快速创建复杂的表单，实现包括登录界面在内的各种样式的TableView输入界面，此外，还可以创建带有多个文本域的表格及项目。

QuickDialog通过对数据元素的峰值封装，根据不同元素的使用来简化UITableView的使用。
为了使用QuickDialog你必须知道三个不同的类：
* QuickDialogController——是UITableViewController的子类，用于显示对话框在你的应用中你可能
会创建这个类的子类来显示。
* QRootElement——一个对话框的根元素，sections与cells的容器，用户显示用户数据。每一个QuickDialogController一次只可以显示一RootElement。可以包含其他的rootElement的。
* QElement——一个element映射一个UItableViewCell，尽管它包括更多的功能，像能够从cell中读取值和有多个类型。QuickDialog也提供了多种内建element type，像 ButtonElement 和 EntryElement，但你也能够创建自己的element。


## 一、关于Elements
* QuickDialog 提供了可以应用于app的很多不同elements。
* QLabelElement: 简单的内建键值对映射
* QBadgeElement: 想label cell，但值是作为一个徽标显示的
* QBooleanElement: 显示一个开关
* QButtonElement: 在中间显示一个标题想一个button一样
*QDateTimeElement:允许你编辑dates，time，或者date + time值。编辑发生在新朴实出来的viwController中。
* QEntryElement: 输入字段，允许你收集用户的信息。可以自动扩展。
* QDecimalElement: 非常想一个可输入的field，但是只允许输入数字。可以预定义输入的精度。
* QFloatElement: 显示一个滑动栏
* QMapElement: 当被选中时，显示一个有location的全屏地图，需要经纬度值。
* QRadioElement:允许用户从多个可利用的选项中选择一个。自动push一个新的有已被选中的item的table。
* QTextElement: 可提供字体渲染的自由文本
* QWebElement: push一个在element中定义URL的简单浏览页面。

下面看下QuickDialog中element的继承关系：

 <img src="/img/post_2014/20140308_quickdialog_1.png" alt="the element class"/>

## 二 关于Sections
* sections 是一个简单的elements分组，默认情况下有一下几种特性：
* title/footer：页眉/页脚部分显示为简单的字符串
* headerView/footerView：这种情况下，Views可以替换掉titles的显示，可以简单的显示图片或自定义的Views

关于sections的结构图如下： 

<img src="/img/post_2014/20140308_quickdialog_2.png" alt="the Sections class"/>

* QRadioSection：内联的显示多个选项，而不是到另一个UIViewController中。
* QSortingSection：自动在sections中对cells进行排序。

## 三 关于自定义的UITableViewCell
在QuickDialog中自定义了一部分的UITableViewCell，来对应于定义的elements。其结构如下：

<img src="/img/post_2014/20140308_quickdialog_3.png" alt="the custom UITableViewCell"/>

## 四 关于QuickDialog的使用

集成QucikDialog到项目中\n 最简单的方法是把quickDialog作为git子模块添加到你的现有工程中，然后作为一部分导入到project中。 Terminal：
```
cd your-project-location git submodule add git@github.com:escoz/QuickDialog.git
```
这种方法是从github中自动copy的，因此你在将来可以方便的更新。

在XCode中： 打开已存在的project(或者创建个新的) 把从github分支上下载QuickDialog.xcodeproj拖拽到你的工程中（root或framework下) 在工程配置下：

* 在Build Phases，添加QuickDialog(the lib, not the example app)作为目标依赖
* 在Build Phases->Link binary with libraries下，添加这两个库：MapKit.framework and CoreLocation.framework
* 在Link binary with libraries下添加这个QuickDialog.a静态库
* 在你的prefix.pch未见中，添加#import
* 在你的工程配置中的“Build Settings”下 本地的"User Header Search Paths"设置，并设置值为"${PROJECT_DIR}/QuickDialog"(包括引号！)并且勾选"Recursive"选项。
* Debug值应当已经被设置，如果没有，修改它。
* 本地的 “Always Search User Paths”设置为YES
* 最后找到 “Other Linker Flags”选择项，并且添加至-ObjC 

**OK，that should be all there is. Here’s how you can create and display your first dialog from inside another UIViewController:**

```objc
 QRootElement *root = [[QRootElement alloc] init]; 
 root.title = @"Hello World"; 
 root.grouped = YES; 
 QSection *section = [[QSection alloc] init]; 
 QLabelElement *label = [[QLabelElement alloc] initWithTitle:@"Hello" Value:@"world!"];
[root addSection:section]; 
[section addElement:label];

UINavigationController *navigation = [QuickDialogController controllerWithNavigationForRoot:root]; 
[self presentModalViewController:navigation animated:YES];
```
The code above will create the form below:
<img src="/img/post_2014/20140308_quickdialog_4.png" alt="the custom UITableViewCell"/>

## 五：QuickDialog的数据交互
由于QuickDialog是通过对数据的封装实现对UI的控制的，所以要获得数据的话有两种方式，一种是直接通过elements的属性获取，一种是通过bind的对应键值来使用fetchValueUsingBindingsIntoObject来拉去的。这样的话必须bind中的key与放入对象的attribute的名称要一直，否则会拉去不到crash。

对QuickDialogTableView的点击事件可以直接设置block来进行控制，这样较容易控制的。
