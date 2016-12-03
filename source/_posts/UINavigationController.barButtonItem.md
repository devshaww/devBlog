---
title: UINavigationController 之 BarButtonItem
date: 2016-04-15 23:21:21
categories: iOS
toc: true
---
看了教学视频也还是被导航控制器弄晕了。

于是果断去看[相关官方文档](https://developer.apple.com/reference/uikit/uinavigationcontroller)。

自定义的问题

## The Left Item
<!--more-->
> navigation栈里除了根控制器的所有控制器左边都会有一个leftBarButtonItem来返回上一个控制器。

* 如果栈顶控制器的`leftBarButtonItem`被自定义了，它会被展示出来.
* 如果栈顶控制器的`leftBarButtonItem`没有被自定义，但是它的前一个控制器的backBarButtonItem是有对象的，那么当前控制器会展示那个item.
* 如果没有任何一个控制器有自定义，那么默认有个`back`按钮，并且`back`按钮的`title`是前一个控制器的`title`（中间的那个`title`）如果`navigation`栈中只有一个`viewcontroller`，不会有`back`按钮.

> 注意：有时back按钮的title可能会很长，无法适应提供的空间，navigation bar会用“Back”代替那个返回title，但是仅当当前back按钮由上一个控制器的backBarButtonItem提供时（也就是上面的第二条那样）；如果当前控制器是自定义的leftBarButton，则不会用“Back”代替。

## The Middle Item

* 如果有自定义则显示自定义，自定义用navigationItem的titleView属性.
* 如果没有自定义，navigation bar展示一个默认title，这个属性一般来自于控制器本身的title属性的值，如果不想用这个，可以设定控制器的navigationItem属性的title属性.

## The Right Item

* 通过控制器的navigationItem的rightBarButtonItem属性可以进行自定义.
* 如果没有任何自定义，默认没有任何东西展示.

* * * * *

###### 配置`navigation stack`中的控制器间的`segue`，标准的`show Detail`与`show`效果如下：
* show Segue：navigationController 对 viewcontroller进行压栈
* show Detail Segue：present it modally
* 别的segue的行为不变

<br/>


> 关键是对`backBarButtonItem`和`leftBarButtonItem`的理解。

> `backBarButtonItem` 和 `leftBarButtonItem`的区别主要是：

> 假设有控制器A和控制器B，控制器A在栈底，控制器B在栈顶.

> 那么控制器B的`leftBarButtonItem`，顾名思义，就是控制器B导航条左侧的那个item，是在控制器B的.m文件里设置的。

> 而控制器B的`backBarButtonItem`，只能在它之前的那个控制器A里去设置，它的意义是相对A来说的，意思是：返回A的那个`barButtonItem`．

> 拟人化一点

> A说：我要在我的控制器里设置一个返回我的item，其实这个item就会是B的`leftBarButtonItem`。
