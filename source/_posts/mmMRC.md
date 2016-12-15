---
title: 通过一个工程学习MRC内存管理
date: 2016-10-22 23:10:15
categories: iOS
---

## 前言
很早就想记录`MRC`下的内存管理，特别是`set`方法。
与`ARC`不同的`MRC`, 非常有趣。很多开发者是在`ARC`已经普及时学习的iOS，没有接触过`MRC`。那么，如果你对`MRC`有兴趣，可以通过这个工程，对之形成比较完整的理解。
<!--more-->
## 配置MRC环境
> 将环境改为`MRC`：打开XCode->点击工程->Build Phases->Compile Sources->点击要配置MRC环境的文件->添上`-fno-objc-arc`。若要改回`ARC`, 则将`-fno-objc-arc`改成`-fobjc-arc`.

![image](https://drops.azureedge.net/drops/files/acc_521713/1lO5?rscd=inline%3B%20filename%3DIMG_4593.PNG&rsct=image%2Fpng&se=2016-12-12T08%3A46%3A14Z&sig=j4VSAZhTPbaDTEnRPIR%2BZ4xby%2FdlizC2esP%2BW12t1qM%3D&sp=r&sr=b&st=2016-12-12T07%3A46%3A14Z&sv=2013-08-15)

![image](https://drops.azureedge.net/drops/files/acc_521713/wAjZ?rscd=inline%3B%20filename%3DIMG_4594.PNG&rsct=image%2Fpng&se=2016-12-12T08%3A50%3A28Z&sig=5yFCTx2fs7ykmkkoF%2FgwLyP5ODd08MdEl1u7tbmOLPI%3D&sp=r&sr=b&st=2016-12-12T07%3A50%3A28Z&sv=2013-08-15)


