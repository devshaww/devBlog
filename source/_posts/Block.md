---
title: Block进阶
date: 2016-10-22 20:50:19
categories: iOS
---

#### 前言
在了解了`Block`的基础知识后，希望对它有更多的认识比如：
* `Block`里的`野指针`错误为什么会出现？怎样避免？
* `Block`里的`循环引用`的原理是什么？
* `Block`在`ARC`与`MRC`下有什么不同？

<!--more-->

在看[iOS与OS X多线程和内存管理]()时，看到`block`那章，我就觉得看不懂了。这一章包含太多的底层源码，非常枯燥而且不好理解，因此难以集中精神。
因此在我对之的理解稍微清晰了一些时，我就记录了下来。

> 将环境改为`MRC`：打开XCode->点击工程->Build Phases->Compile Sources->点击要配置MRC环境的文件->添上`-fno-objc-arc`。若要改回`ARC`, 则将`-fno-objc-arc`改成`-fobjc-arc`.

![image](https://drops.azureedge.net/drops/files/acc_521713/1lO5?rscd=inline%3B%20filename%3DIMG_4593.PNG&rsct=image%2Fpng&se=2016-12-12T08%3A46%3A14Z&sig=j4VSAZhTPbaDTEnRPIR%2BZ4xby%2FdlizC2esP%2BW12t1qM%3D&sp=r&sr=b&st=2016-12-12T07%3A46%3A14Z&sv=2013-08-15)

![image](https://drops.azureedge.net/drops/files/acc_521713/wAjZ?rscd=inline%3B%20filename%3DIMG_4594.PNG&rsct=image%2Fpng&se=2016-12-12T08%3A50%3A28Z&sig=5yFCTx2fs7ykmkkoF%2FgwLyP5ODd08MdEl1u7tbmOLPI%3D&sp=r&sr=b&st=2016-12-12T07%3A50%3A28Z&sv=2013-08-15)

#### Block的种类
##### NSStackBlock
* 引用了外部变量的`block`
* `MRC`下，`NSStackBlock`位于`栈`中，`block`函数体返回后，其位于的栈段会被清除，即`block`会无效，此时如果再次访问`block`，就会造成`野指针`错误。`解决方法`是：通过对block做`copy`操作，使之转换为位于`堆`中的`NSMallocBlock`。
* `ARC`下，与`MRC`不同的是，不需要再做`copy`操作，在`ARC`下，所有的block都会自动转到堆区，即NSMallocBlock。

##### e.g.

```
typedef void (^myBlock)();

char a = 'a';
myBlock block = ^ {
	NSLog("%d", a);
}

```

#### NSMallocBlock
* 由`NSStackBlock copy`得来，位于内存的`堆`区。


#### NSGlobalStack
* 没有引用外部变量的`block`就是`NSGlobalStack`。
* 既不位于`堆`，也不位于`栈`,是`程序段`。

##### e.g.
```
int (^sum)(int, int) = ^(int x, int y) {
	return x + y;
}

```

#### 野指针错误
```
typedef void (^block)();

block arc_getBlock() {
	char a = 'a';
    void (^myBlock) = ^{
    	printf("%c",a);
    };
    NSLog(@"myBlock:%@",myBlock);
    block block_copy = [[myBlock copy]autorelease];
    NSLog("block_copy:%@",block_copy);
    // return block_copy;
    return myBlock;
}

void test() {
	block myBlock = arc_getBlock();
    myBlock();
}
```
`arc_getBlock()`最后返回的是`block_copy`就不会出现野指针错误，而返回`myBlock`就会出现野指针错误，因为`myBlock`在栈上，返回后，就被清理了。
> 按理说是这样，但我做测试的时候，发现两者都可以。

* 全局和静态变量在内存中的位置是确定的，block copy时不会retain这些对象。
* 局部变量在block copy时，会被retain，增加其引用计数。
* 实例变量或属性在block copy时不是直接retain这个对象本身，而是retain self。
<br/>
下面来做一个稍大的测试以便更好地理解
#####e.g











> 最后，这里有一个从`OneVCat`(写对了么)的文中看到的关于block的[quiz](http://blog.parse.com/learn/engineering/objective-c-blocks-quiz/)，有兴趣可以做做。








