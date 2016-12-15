---
title: position和anchorPoint的区别
date: 2016-08-08 16:05:23
categories: iOS
---


> 整理自[苹果官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW3)

#### Layers使用两种坐标系
##### point-based
<!--more-->
* 当需要定义`layer`在屏幕中或是距另一个`layer`的位置时用到，比如`position`，`position`默认在屏幕正中
* 定义一个`layer`的具体位置，在此坐标系中用`size`和`position`可以确定下来
* `position`是定义`layer`相对父坐标系的位置的
* 虽然`layer`有`frame`属性，但它会随着`position`和`size`的改变而改变，且用得稍微少一些

##### unit coordinate
* 这种坐标系不是用来定义`layer`在屏幕中的位置的，比如`anchorPoint`。默认为`(0.5, 0.5)`, 这个属性也是和以往大家适应的位置属性不同的一点，建立了和`bounds`的关系，具体看下面的内容就懂了

![image](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_bounds_2x.png)

###### ** 可以看到，`position`是在中间的，`position`也是少数的会随着`anchorPoint`的改变而变化的属性。**

![image](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_unit_2x.png)

###### ** 在`unit coordinate`的世界里，要定义具体的数值时，用的都是比例，所以值为`0~1`。 **

## 什么是`anchorPoint`？`anchorPoint`跟`transform`和`position`密切相关。

![image](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_anchorpoint_position_2x.png)

###### ** 可以看出来，为什么说`anchorPoint`不是定义`layer`在屏幕中的位置的属性，上下两个图`anchorPoint`发生了变化，但是`layer`在屏幕中的位置是没有变的，但这是因为`position`的变化。 **

```bash
会发现"position"和"anchorPoint"的位置完全一样。
```

###### ** 任何时候都需要满足这样的公式 **
```bash
position.x = frame.origin.x + anchorPoint.x * bounds.size.width
```
```bash
position.y = frame.origin.y + anchorPoint.y * bounds.size.height
```

两者当中，一个是对于父视图而言，一个是对于layer本身而言，更直截了当一点说，`position`就是对`anchorPoint`在父视图中位置的数值化反应，而`anchorPoint`只是比例反应。

比如上图中第一张图，`layer`长`120`，宽`80`；`anchorPoint(0, 0)`；`position`就是`anchorPoint`在父视图中的坐标，即`(40, 60)`。

![image](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_anchorpoint_transform_2x.png)

```bash
看得出`anchorPoint`是`transform`的旋转围绕点。
```
