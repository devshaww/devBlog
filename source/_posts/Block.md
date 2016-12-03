---
title: Block进阶
date: 2016-10-22 20:50:19
categories: iOS
---

## 前言
在了解了`Block`的基础知识后，希望对它有更多的认识比如：
* `Block`里的`野指针`错误为什么会出现？怎样避免？
* `Block`里的`循环引用`的原理是什么？
* `Block`在`ARC`与`MRC`下有什么不同？

<!--more-->

在看[iOS与OS X多线程和内存管理]()时，看到`block`那章，我就觉得看不懂了。这一章包含太多的底层源码，非常枯燥而且不好理解，因此难以集中精神。
因此在我对之的理解稍微清晰了一些时，我就记录了下来。

> 将环境改为`MRC`：添上`-fno-objc-arc`。

## Block的种类
### NSStackBlock
* 引用了外部变量的`block`
* `MRC`下，`NSStackBlock`位于`栈`中，`block`函数体返回后，其位于的栈段会被清除，即`block`会无效，此时如果再次访问`block`，就会造成`野指针`错误。`解决方法`是：通过对block做`copy`操作，使之转换为位于`堆`中的`NSMallocBlock`。
* `ARC`下，与`MRC`不同的是，不需要再做`copy`操作，在`ARC`下，所有的block都会自动转到堆区，即NSMallocBlock。

#### e.g.

```
typedef void (^myBlock)();

myBlock block = ^ {
	NSLog("%d", a);
}


```

### NSMallocBlock
* 由`NSStackBlock copy`得来，位于内存的`堆`区。

#### e.g.

```



```


### NSGlobalStack
* 没有引用外部变量的`block`就是`NSGlobalStack`。
* 既不位于`堆`，也不位于`栈`。

#### e.g.
```
int (^sum)(int, int) = ^(int x, int y) {
	return x + y;
}

```

## 野指针错误
























